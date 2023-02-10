"  

#### Included Components

This section gives a brief overview of the components making up an installation of the the CI Fuzz backend.

### Backend API server

The main backend server offers a gRPC API which allows clients to set up fuzzing projects, start / monitor / stop fuzzers, inspect findings and much more. Additionally, the gRPC API is automatically converted to a REST API using a grpc gateway which allows for easier integration of the API in a JavaScript environment.

### Database

A Postgres database is used to persist fuzzing results and user data. The database configured in the included docker-compose setup works out of the box, however, it is also possible to connect to an already configured Postgres database.

### Gateway

The gateway service serves as a reverse proxy for all other components and is the single entry point for the CI Fuzz backend. It also hosts the Web App, does TLS termination and handles the OAuth flow.

### Reporting

The reporting service uses the API to generate reports in pdf, docx and xlsx format.

### Worker

The worker runs and monitors jobs are created by the API server. The most common job is the execution of a fuzzer and the evaluation of line coverage. All jobs are currently run inside Docker containers which are spawned alongside the docker-compose setup (Docker out of Docker).

The API Server is configured to start 4 worker instances by default.

### Web App

A single page web app which can be accessed to configure fuzzing projects, monitor fuzzer execution and get detailed information on findings that were detected by fuzzers. This is hosted by the Gateway container.

### Command Line Client

The installation also includes a command line client named cictl. It communicates with the API server via gRPC and can be used to start fuzzing jobs from CI/CD pipelines and other common tasks involving the CI Fuzz backend API.

### Starting The Service

You can enable the systemd service with

systemctl enable cifuzz.service

In this case, logs will be at /var/log/journal, the usual place for systemd services. Use the journalctl command to access them.

Alternatively, the cifuzz-server command was linked to the ~/bin folder of the user running the installer. This script is a wrapper around docker-compose to ensure the environment is set up correctly. It allows to manually start and stop the service as well as experimenting with new configuration.

~/bin/cifuzz-server --help

will provide further information.

If you start the service this way, output will be given to the terminal. You can affect this by rerouting with Shell capabilities or support provided by docker-compose.

#### Accessing the UI

Regardless of the way you started the service, the UI should be accessible at localhost:8080.

In order to do anything with the UI, you will need to setup an oauth app, see here.

Another way to interact with the service is to use the cictl utility. This has been installed at ~/bin/cictl and allows interaction with most of the CI Fuzz API via the command line.

In order to use it, you need to log in to the system. One way to do that is to create a user in the UI and create a service token. Then log in via cictl using that token:

echo $TOKEN | cictl -s localhost:6773 login

Just after installation, you can also use the demo admin account. The password to use was either given by you in form of the DEMO\_ORG\_ADMIN\_TOKEN environment variable from the .env files, or is still at the default value, demo\_org\_admin\_default\_token.

### Installation Alongside CI Fuzzing Binaries

Most of the ci-fuzz tooling installed by the first-generation CI Fuzz server is still used by the docker-compose based installation. However, the required binaries are bundled in the installer. You can still use the first-generation toolchain, but this is seen as a separate installation not interfering with the compose-based solution.

### Monitoring

Monitoring has been extracted into its own package. To add it, use cifuzz-server --file docker-compose.metrics.yaml.

If the Monitoring extra package is installed, monitoring is available via Prometheus and, more importantly, Grafana. Find the Grafana password in the .env file. Username is admin.

The backend makes use of postgresql queries for a subset of its metrics and exposes them in Prometheus format. This is part of the base docker-compose setup. You can reach it at the metrics endpoint.

#### Configuration

### Concepts

The installation consists of a a set of docker-compose files that are configured with multiple files containing environment variables. These reside in two places: the installation folder's on-prem directory contains the base layer for both the docker-compose and the .env files. A configuration overlay is applied by environment files from /etc/ci-fuzz if you decide to use the systemd service provided by the installer. If you roll your own startup mechanism, you can still overlay your configuration in the same way.

The service is managed by a script called cifuzz-server, which takes care of always applying the base layer and letting you or the systemd service select the overlay environment files.

Most of the configuration options can be set via environment variables provided to the Docker containers. The most relevant ones have been bundled under the namespace CIFUZZ\_ and exported to an environment file. Environment files can be provided to cifuzz-server executions via the --env-file switch.

Environment variables that are not explicitly listed can still be used. This holds true for changes to the JVM and configuration parameters for the backend as well as the various options for example Grafana allows. The usage of these environment variables hasn't changed, but you may need to edit the docker-compose files to pass them into the container. It's best to consult customer support for these changes.

To isolate deployments, the project name is set in the environment file. This will cause Docker to create a specific network to place the containers in.

Data will be kept in two different places: internal data, like the database, and monitoring data, will be stored in named docker volumes. The cifuzz data is stored at /var/ci-fuzz-data-dir by default. The directory will be created by docker if it doesn't exist.

If you use the monitoring add-on, changes to the Grafana dashboards will be persistent, and can be made persisting accross re-creations of the volume by exporting the Dashboards as json and placing them in the grafana provisioning folder. They will be automatically picked up.

### cifuzz-server Tool

The cifuzz-server tool is used to manage the service's state. In addition to providing a means to start and stop it, it also takes care of setting up the correct runtime environment. For that, it consumes docker-compose .yaml files as well as .env files.

Docker-compose by itself allows only to use a single environment file. The cifuzz-server differs from docker-compose in that it allows to layer multiple definition files. Variables defined in later files can override settings defined in earlier ones. An additional difference exists to the traditional way environment files are applied: inside each environment file, each line is evaluated by itself. This means that later lines can make use of definition in earlier lines.

To a limited degree, the same holds true for docker-compose .yaml files. Definitions made in files applied later can override earlier ones. The changes can be partial, but they cannot be destructive, i.e. you cannot remove a container defined in an earlier file.

To allow for flexibility and composability, the cifuzz-server tool provides a ground layer and allows to apply overlays through its arguments. A standard setup is managed by the systemd service installed by the installer.

To provide custom files, the options --file, --env-file, and --env-files-from are provided. The --file option is routed through to the docker-compose command and hence follows traditional semantics. The environment-specific options handle the .env files as described above.

cifuzz-server always loads on-prem/docker-compose.yaml and applies on-prem/cifuzz.env first. After that follow files ending in .env in a folder given by --env-files-from are applied in the order they are listed by the operating system. This allows you to order them by naming them appropriately. This option can only be appplied once. Finally, files given by --env-file options are applied in the order given. You can use any number of additional env files.

The systemd service set up by the installer looks for environment files in the folder /etc/cifuzz. Any files ending in .env will be applied as describes above.

There is no mechanism in place to automatically pick up add additional docker-compose files in the manner of the --env-files-from flag. If you need to add more .yaml files, you will have to adjust the cifuzz-server call set up in the systemd service.

### Provided docker-compose Files

The following docker-compose files are provided in the folder on-prem of the installation:

*   docker-compose.yaml defines the main configuration and is enough to bring up the service;
*   docker-compose.tls.yaml contains added definitions that make the system activate TLS;
*   docker-compose.monitoring.yaml contains definitions for a Prometheus and a Grafana instance that allow to analzyze and visualize metrics of the service.

These .yaml files define Docker containers. These are affected by environment variables from the .env files routed into the environment settings of the container.

#### Environment Variables

The environment variables governing the configuration of the docker containers are described in the file on-prem/cifuzz.env. You can use additional environment variable files to override them selectively as describe in the section about the cifuzz-server tool.

### Backend Configuration File

The backend server is configured using the file on-prem/ci-backend/config.edn. You will rarely have reason to visit this file, however, since the configuration items in there can be overridden by environment variables. The expected configuration points have already been wired into the relevant docker containers, in the form of environment variables.

Every setting in the .edn file can be overridden as described in the documentation.

> **Please Note:** The .edn file is not intended to be directly edited. It is listed here because it is a source for customization options. Please contact support if you need customization.

#### Maintenance

### start, stop, restart

If you use the systemd service:

systemctl stop|start|restart cifuzz.service

If you roll your own:

cifuzz-server down|up|<any valid docker-compose arg>

### System Service and Log Files

If using the systemd service, logs are in the journal:

journalctl -u cifuzz

If you don't use the service, capture the output of the cifuzz-server up command or collect container logs via cifuzz-server logs <container>.

#### Troubleshooting

### Inspecting The Effective Configuration

In order to see all values applied to the docker-compose setup, you can run the config subcommand using the arguments used in your call to cifuzz-server. If you're using the systemd service, look into the service defintion file. By default, it would look like this:

cifuzz-server --env-files-from /etc/cifuzz config

This will output the fully interpolated docker-compose setup.

### Re-Create Containers

When things get stuck, it may help to completely remove a service along with all the data stored on names docker volumes:

cifuzz-server down -v <service name>  
systemctl restart cifuzz.service

> **WARNING** This will remove your database. Make sure to have a backup to restore or not to down the database container.

Note that this does only remove docker-managed named volumes. Any data stored on local hard disks mounted into the container will persist until manually deleted.

The <service name> would be one of the services defined in docker-compose.yaml, e.g. ci-backend. If left empty, all containers will be removed.

You can also wipe out the whole installation by calling

docker-compose --env-file cifuzz.env down -v

"