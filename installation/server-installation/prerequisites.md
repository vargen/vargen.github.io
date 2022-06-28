# Prerequisites

### Hardware Requirements

In order to use the provided install mechanism and the resulting docker-compose setup, some prerequisites have to be met. We recommend using a server with at least:

* 32 GB RAM
* 16 or more CPU cores
* 100 GB storage
* A Linux operating system
* ssh access
* root privileges (if you want to use port 80/443)
* One port (443 recommended) open to access from CI/CD runners and users' computers

For a more detailed overview of the hardware and software technical prerequisites see [CI Fuzz Hardware and Software Technical Prerequisites.](https://help.code-intelligence.com/ci-fuzz-hardware-and-software-technical-prerequisites)

{% hint style="info" %}
The legacy server used separate ports for HTTP and GRPC. If you are migrating, please note the open port requirement (single port must be reachable from CI/CD runners and users' computers) and edit firewall rules if needed.
{% endhint %}

### Software Requirements

#### **Linux**

The only supported operating system is Linux. Most Linux distributions will be fine, while the most comfort can be achieved by using one that supports systemd. The installer provides a shortcut by setting up a systemd system service.

#### **Docker and Docker-Compose**

CI Fuzz uses Docker-Compose [docker-compose](https://docs.docker.com/compose/) to orchestrate its components. First, install docker for your distribution as depicted in the [docker engine documentation](https://docs.docker.com/engine/install/). To setup docker-compose, follow the [installation instructions here](https://docs.docker.com/engine/install/) for your distribution.&#x20;

The minimum supported docker-compose version is 1.17.0.

{% hint style="info" %}
Podman on RedHat has not been tested, only regular docker.
{% endhint %}

#### **Ports and Network Settings**

One port needs to be open to users, we recommend 443. For Web Fuzzing, a range of ports must be open to hosts that run the software under test. Default range is 6777-7777.

A DNS entry for your CI Fuzz server must exist, or you need a custom TLS certificate that is valid for an IP address.

#### **Azure Account**

If you don't provide the docker images yourself (explained below), you will have to log in to the Azure-Hosted [image registry](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-intro) in order for the installer to be able to download them. This works by Code Intelligence granting access to your Azure account.

#### **Azure CLI**

Once your Azure account has been granted access to the image registry, you will need to log in to be able to download docker images. To do so, you will need to install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/) tools.

You can also provide authentication to the Azure container registry in another form. Or circumvent the need by providing the images yourself. See Step 3 for more details.
