# Installation

### **Step 1: Create an OAuth Application**

CI Fuzz uses OAuth for authenticating users. Learn how to set up an OAuth app [here](https://help.code-intelligence.com/create-an-oauth-application).

### **Step 2: Get a TLS certificate.**

For secure TLS connections to the fuzzing server, a TLS certificate is needed. Probably your company already uses a specific CA or maybe runs its own. If this is not the case, you can get free TLS certificates from [Let's Encrypt](https://letsencrypt.org/).

Just install the command line tool certbot and request a certificate with a single command:

```
sudo apt install certbot
sudo certbot certonly --standalone --preferred-challenges http -d
```

### **Step 3: Run the installer**

You should have received a link for the installer. Optionally you have also received a docker image bundle (CI Fuzz on-premise backend image bundle) from Code Intelligence.

#### **Azure Registry**

If you are using Azure registry images you don't need the docker image bundle:

```
az login
az acr login --name cifuzzpublictest
chmod +x ci-fuzz-on-prem-installer--linux
./ci-fuzz-on-prem-installer--linux
```

#### **Docker Image Bundle**

If you have the docker image bundle, you can skip the az commands and run the installer with the following option:

```
./ci-fuzz-on-prem-installer--linux --docker-images
```

If you were updating from previous version:

```
sudo systemctl daemon-reload
sudo systemctl restart cifuzz
```

### **Step 4: Configuration**

First create the directory that will contain configuration files:

```
sudo mkdir /etc/cifuzz/
```

Then create this file

```
/etc/cifuzz/config.env
```

With the following content:

```
#
# Code Intelligence Fuzzing environment values.
#
#
# Server Setup
#

# Change this value to the domain / IP where the server should be reachable and adjust
# the protocol when TLS encryption is configured.
CIFUZZ_SERVER_ORIGIN=https://cifuzz.company.org:443

# External HTTP/HTTPS port.
CIFUZZ_SERVER_PORT=443

# TLS Setup
#
# Both settings are required for using the TLS enabled docker-compose.tls.yaml file.
# The TLS settings will be applied automatically by the cifuzz-server tool when
# these variables are set.
CIFUZZ_CERT_FILE=
CIFUZZ_CERT_KEY=

#
# OAuth Apps
#

# GitHub
#
# Create a GitHub OAuth app by following the instructions here:
# https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app
#
# As callback URL use `https://<my-domain>/auth/github/callback`.
#
# Enter both the GitHub OAuth app ID and secret in the values below.
CIFUZZ_GITHUB_CLIENT_ID=
CIFUZZ_GITHUB_CLIENT_SECRET=
# Set for GitHub Enterprise
CIFUZZ_GITHUB_BASE_URL=

# GitLab
#
# Create a GitLab OAuth app by following the instructions here:
# https://docs.gitlab.com/ee/integration/oauth_provider.html
#
# As callback URL use `https://<my-domain>/auth/gitlab/callback`.
#
# Enter both the GitLab OAuth app ID and secret in the values below.
CIFUZZ_GITLAB_CLIENT_ID=
CIFUZZ_GITLAB_CLIENT_SECRET=

# Configuration for on-premise GitLab.
CIFUZZ_GITLAB_BASE_URL=
CIFUZZ_GITLAB_ROOT_CAS=

# Bitbucket
#
# Configure an OAuth consumer by following the instructions here:
# https://support.atlassian.com/bitbucket-cloud/docs/integrate-another-application-through-oauth/
#
# As callback URL use `https://<my-domain>:<port>/auth/bitbucket/callback`.
#
# Enter both the Bitbucket OAuth consumer ID and secret in the values below.
CIFUZZ_BITBUCKET_CLIENT_ID=
CIFUZZ_BITBUCKET_CLIENT_SECRET=

# Bitbucket on-premise oauth1
CI_AUTH_BITBUCKET_URL=

# Azure
#
# Create a OAuth / OpenID app by following the instructions here:
# https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/openidoauth-tutorial
#
# As callback URL use `https://<my-domain>/auth/azure/callback`.
#
# Enter both the Azure OAuth App ID and secret in the values below.
CIFUZZ_AZURE_CLIENT_ID=
CIFUZZ_AZURE_CLIENT_SECRET=

#
# Gateway
#

# CHANGE ME!
#
# Shared secret to authenticate the gateway and the backend server. It must be at least
# 32 bytes of hex encoded bytes. If openssl is available you can generate a valid random
# value with `openssl rand -hex 32`.
CIFUZZ_INTER_SERVICE_AUTH_KEY=0ec1302c35fcd8391ea57c2d6fcb63b1d9eca8af5845d79a4a51c1695d2462d3
```

Edit the variables:

* CIFUZZ\_SERVER\_ORIGIN - Change to the hosname of the host where you are running CI Fuzz server. Please set full origin here - protocol (https), hostname and port. This must match the origin set in your OAuth application.
* CIFUZZ\_SERVER\_PORT - 443 by default. You can change the port if needed
* CIFUZZ\_CERT\_FILE and CIFUZZ\_CERT\_KEY - Absolute paths to the certificate and key that CI Fuzz should use.
* OAuth Apps - Set according to the OAuth app that you have configured in step 1. One OAuth provider is usually enough, you can delete the sections that you will not use.
* CIFUZZ\_INTER\_SERVICE\_AUTH\_KEY - Please generate a random key with the command suggested in the comment, or its equivalent.

Optionally, for other available variables, see this [comprehensive list](https://help.code-intelligence.com/ci-fuzz-server-configuration-variables). Or, for the most up to date list, look in this file:

```
/opt/ci-fuzz-/cifuzz.env
```

### **Step 5: Run server**

```
sudo systemctl start cifuzz
sudo systemctl enable cifuzz #to make it autostart after reboot
```

Check if CI Fuzz containers have been started successfully:

```
sudo docker ps
```

Output should look similar to this:

```
CONTAINER ID   IMAGE                                                  COMMAND                  CREATED      STATUS      PORTS                                       NAMES
c328d2e9ad6b   prometheuscommunity/postgres-exporter:v0.10.0          "/bin/postgres_expor…"   2 days ago   Up 2 days   0.0.0.0:9187->9187/tcp, :::9187->9187/tcp   cifuzz_db-metrics_1
3ee314af29ce   cifuzzpublictest.azurecr.io/cifuzz/ci-backend:2.31.0   "/usr/bin/java -cp /…"   2 days ago   Up 2 days   0.0.0.0:6777->6777/tcp, :::6777->6777/tcp   cifuzz_ci-backend_1
2e4f9bb6c8e7   postgres:12.7                                          "docker-entrypoint.s…"   2 days ago   Up 2 days   5432/tcp                                    cifuzz_db_1
c24dd46702e8   cifuzzpublictest.azurecr.io/cifuzz/reporting:2.31.0    "/usr/bin/java -cp /…"   2 days ago   Up 2 days                                               cifuzz_reporting_1
60d8f8ca1940   cifuzzpublictest.azurecr.io/cifuzz/gateway:2.31.0      "/app/cmd/gateway/ba…"   2 days ago   Up 2 days   0.0.0.0:80->80/tcp, :::80->80/tcp           cifuzz_gateway_1
```

### **Advanced configuration options**

#### **Using your own reverse proxy/load balancer**

If you want to terminate TLS yourself, comment out CIFUZZ\_CERT\_FILE and CIFUZZ\_CERT\_KEY. Change CIFUZZ\_SERVER\_PORT to some other port, but keep port 443 (or other port where users will be accessing CI Fuzz) in CIFUZZ\_SERVER\_ORIGIN.

Set your reverse proxy or load balancer to direct HTTP and HTTP2/GRPC traffic to CIFUZZ\_SERVER\_PORT. Nginx example:

```
server {
 listen 443 ssl;
  ssl_certificate      /home/azureuser/tls/cifuzz.onprem.lab_cert.pem;
  ssl_certificate_key  /home/azureuser/tls/cifuzz.onprem.lab_key.pem;
 location / {
  proxy_pass http://localhost:80/;
 }
}
server {
 listen 6443 http2 ssl;
  ssl_certificate      /home/azureuser/tls/cifuzz.onprem.lab_cert.pem;
  ssl_certificate_key  /home/azureuser/tls/cifuzz.onprem.lab_key.pem;
 client_max_body_size 500M;
 location / {
   grpc_pass grpc://localhost:80;
 }
}
```

**Architecture, components troubleshooting**

Technical details about the CI Fuzz backend can be found [here .](https://help.code-intelligence.com/cifuzz-backend-architecture-and-advanced-tools)
