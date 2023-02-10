---
layout: default
#title: Configuration
parent: CI Fuzz
nav_order: 1
---
# **Configuring CI Server**
{: .no_toc }

This section provides details about the additional configuration options available in `CI Fuzz`.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

```
#
# Code Intelligence Fuzzing environment values.
#
# Note: Values in here can still be overridden by setting actual environment
# variables when calling docker-compose. In addition, some of the services allow
# setting arbitrary configuration options by following a naming scheme.
# Specifically, ci-backend employs a naming scheme explained [here](https://github.com/tolitius/cprop#speaking-env-variables)
# and Grafana describes its own [here](https://grafana.com/docs/grafana/latest/administration/configuration/#override-configuration-with-environment-variables).

# Name of the project. This serves as namespace for various docker resources.
COMPOSE_PROJECT_NAME=cifuzz

# The IPv6 subnet to use. If this is not set, no globally routable IPv6 network is assigned.
# See https://docs.docker.com/config/daemon/ipv6/ for global docker IPv6 configuration (might not be necessary).
# CIFUZZ_IPV6_SUBNET=2001:0db8:85a3::/64

#
# Image Versions
#

CIFUZZ_CONTAINER_REGISTRY=f5d6e55758e0d08d.azurecr.io
CIFUZZ_VERSION=2.42.0
CIFUZZ_DB_IMAGE=postgres:12.7
CIFUZZ_DB_METRICS_IMAGE=prometheuscommunity/postgres-exporter:v0.10.0
CIFUZZ_PROMETHEUS_IMAGE=prom/prometheus:v2.31.1
CIFUZZ_GRAFANA_IMAGE=grafana/grafana:9.1.5
CIFUZZ_BACKEND_IMAGE=${CIFUZZ_CONTAINER_REGISTRY}/cifuzz/ci-backend:${CIFUZZ_VERSION}
CIFUZZ_WORKER_IMAGE=${CIFUZZ_CONTAINER_REGISTRY}/cifuzz/ci-worker:${CIFUZZ_VERSION}
CIFUZZ_GATEWAY_IMAGE=${CIFUZZ_CONTAINER_REGISTRY}/cifuzz/gateway:${CIFUZZ_VERSION}
CIFUZZ_REPORTING_IMAGE=${CIFUZZ_CONTAINER_REGISTRY}/cifuzz/reporting:${CIFUZZ_VERSION}

#
# Server Setup
#

# Change this value to the domain / IP where the server should be reachable and adjust
# the protocol when TLS encryption is configured.
CIFUZZ_SERVER_ORIGIN=http://127.0.0.1:8080

# External HTTP/HTTPS port.
CIFUZZ_SERVER_PORT=8080

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

# Bitbucket OAuth 1.0
# Only use if you cannot use OAuth 2.
#
# The URL of the OAuth 1.0 provider.
CIFUZZ_BITBUCKET_URL=
# The location of private key.
# See https://help.code-intelligence.com/create-an-oauth-application about how to generate a keypair.
CIFUZZ_BITBUCKET_KEY=

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

# OIDC
#
# To configure your OIDC providers copy the OIDC config template
#
#  $ cp <install_dir>/oidc.yaml.template /etc/cifuzz/oidc.yaml
#
# and adjust the copied template as needed.

CIFUZZ_OIDC_YAML=/etc/cifuzz/oidc.yaml

# Password login
#
# This option adds a password login which allows logging in as a default user with the
# DEMO_ORG_ADMIN_TOKEN specified below.
# It is strongly recommended to use one of the OAuth options above and only use the
# password login for initial setup!

CIFUZZ_ENABLE_PASSWORD_LOGIN=0

# CHANGE ME!
#
# Password for a user that is created during startup.
DEMO_ORG_ADMIN_TOKEN=demo_org_admin_default_token

#
# Database
#

# In general, if you alter values here, you also have to alter values
# in db/create_database.sql and ci-backend/config-prod.edn, or pass
# the appropriate environment variables into the relevant docker
# containers.

# Name of database schema to use. If this is altered, the same value has to placed
# ci-backend/config.edn or set in env variable CI_BACKEND__DATABASE__NAME,
# so that ci-backend uses the same database.
POSTGRES_DB=ci_backend

# Postgres username. User to create.
POSTGRES_USER=ci

# Postgres password of user to create.
POSTGRES_PASSWORD=ci

#
# Backend
#

# Port for the backend and worker HTTP metrics endpoints.
CIFUZZ_BACKEND_METRICS_PORT=6777
CIFUZZ_WORKER_METRICS_PORT=6778

# Path for the configuration file for the server.
CIFUZZ_BACKEND_CONFIG=./ci-backend/config-prod.edn

# Path to backend data directory. Uploaded artifacts and fuzz target corpora are stored here.
CIFUZZ_DATA_DIR=${HOME}/.local/share/cifuzz

# Path for the worker to place temporary files.
CIFUZZ_WORKER_TEMP_DIR=/tmp/ci-worker-temp

# Workers communicating with the Server identify by giving this token, which has to be the same
# on Server and worker. Configure any value here for now, as long as it contains a colon.
CIFUZZ_WORKER_TOKEN=1:2345

# The URL the workers will send their final reports to. Needs to be where the gRPC API is hosted.
# The default is to use the Docker Host, where this is exposed.
CIFUZZ_GRPC_URL=ci-backend:6773

# The URL where workers will send raw coverage. Needs to be where the HTTP API (i.e., v2) is hosted.
# The default is to use the Docker Host, where this is exposed.
CIFUZZ_HTTP_URL=http://ci-backend:6777

# Set if the worker needs to be placed into a specific docker network in order to be able to
# communicate to service ci-backend. This is usually the case when the setup does not allow
# to reach it via the host network.
CIFUZZ_DOCKER_NETWORK=${COMPOSE_PROJECT_NAME}_default

# Set the number of job containers that should be kept around after exiting. Containers
# are removed in FIFO fashion if there's more of them than this number.
CIFUZZ_DOCKER_KEEP_JOB_CONTAINERS=5

## Which docker images to allow for fuzzing runs when importing artifacts
##
## Examples for valid values:
##     ':all'
##     '["ubuntu:rolling", "cifuzz/builder-zint"]'
##
CIFUZZ_ALLOWED_DOCKER_IMAGES_EDN=":all"

# The maximum size allowed for artifacts in the format "<number><unit>" where unit
# can be one of (k, M, G, T, P, E).
CIFUZZ_ARTIFACT_SIZE_LIMIT="300M"

# The maximum allowed storage space the artifacts are allowed to consume, before old
# artifacts get deleted. Size string in the format "<number><unit>" where unit can be
# one of (k, M, G, T, P, E).
CIFUZZ_ARTIFACT_STORAGE_QUOTA="100G"

## The maximum number of fuzzer container logs that will be kept per project.
CIFUZZ_MAX_FUZZER_LOG_FILES_PER_PROJECT=20
## The maximum number of unicode characters that will be stored in a single
## fuzzer log file.
CIFUZZ_MAX_FUZZER_LOG_UNICODE_CHARACTERS=2097152

## The number of worker threads to start. This governs how many jobs can be
## processed at any given time.
CIFUZZ_NUM_WORKER_THREADS=4

## The port range (inclusive) from which a random free port is selected for web app fuzzing.
## This option does not apply to in process fuzz tests.
CIFUZZ_WEB_FUZZER_PORT_MIN=6777
CIFUZZ_WEB_FUZZER_PORT_MAX=7777

## A set of keywords that enable or disable certain backend functionality.
##  :minijail - Runs the fuzzers inside a jail within the docker containers.
##  :web-app-fuzzing - Allows fuzzing of web applications.
CIFUZZ_BACKEND_FEATURE_FLAGS="#{:minijail :web-app-fuzzing}"

# Set the default limit of fuzzing minutes per month for newly created users and organizations. A value of 0
# indicates that no fuzzing at all is allowed. A value of -1 indicates unlimited fuzzing. Defaults to -1.

CIFUZZ_DEFAULT_FUZZING_MINUTES=-1


#
# Gateway
#

# CHANGE ME!
#
# Shared secret to authenticate the gateway and the backend server. It must be at least
# 32 bytes of hex encoded bytes. If openssl is available you can generate a valid random
# value with `openssl rand -hex 32`.
CIFUZZ_INTER_SERVICE_AUTH_KEY=dd3a074467aa8b4737ee956d33359beb0eaa1d3f5a142d74220c1f989cded765

# An organization to which all the users that log in will be added as members
# Must be provided via organization ID, which can be seen in the output of this command:
# docker exec -it cifuzz-db-1 psql -h localhost -U ci ci_backend -c 'select * from organization'
# limitation: the organization must only have one administrator, otherwise administrators will be
# demoted to members when they log in, until only one administrator remains
# CIFUZZ_DEFAULT_ORG="organizations/2"

#
# Grafana
#

# These variables will come into play when you add the docker-compose.monitoring.yaml
# docker-compose setting.

# Initial password to set up for the Grafana instance.
CIFUZZ_GRAFANA_PASSWORD=code-intelligence-grafana

#
# Debugging
#

# Activate debug logging for the gateway and ci-backend service. Additionally, the database
# port is exposed locally.
# Use with care!

# CIFUZZ_DEBUG=1

#
# Logging
#

# Path to a file containing log information. Currently supported is the setting
# of logging levels akin to the following sample:
# {:min-level :warn}
# The above sets a general log level. Individual namespaces can be set as well:
# {:min-level [["io.grpc.netty.*" :info]
#              ["io.netty.*" :info]
#              ["com.zaxxer.hikari.*" :info]
#              ["org.apache.http.*" :info]
#              ["ci-fuzz.backend.api.v1.grpc" :info]
#              ["ci-fuzz.backend.api.v2.http" :info]
#              ["*" :info]]}
# Each setting maps to namespace via regex and specifies a log level.
# Valid log levels are, in ascending severity:
# :trace :debug :info :warn :error :fatal :report.
# CIFUZZ_LOGGING_CONFIGURATION_FILE=/path/to/file
```
