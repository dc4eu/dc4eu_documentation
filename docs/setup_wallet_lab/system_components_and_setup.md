# System Components and Setup

## Table of Contents

- [Introduction](#introduction)
- [Components Overview](#components-overview)
- [Service Descriptions](#service-descriptions)
  - [SATOSA (External Issuer)](#satosa-external-issuer)
  - [SimpleSAMLphp](#simplesamlphp)
  - [API Gateway](#api-gateway)
  - [UI Service](#ui-service)
  - [Issuer (Internal Issuer)](#issuer-internal-issuer)
  - [Verifier](#verifier)
  - [Registry](#registry)
  - [MockAS](#mockas)
  - [MongoDB](#mongodb)
  - [Jaeger](#jaeger)
- [Connectivity Levels Explained](#connectivity-levels-explained)
  - [Internal (Inside Docker Network)](#internal-inside-docker-network)
  - [Local (Mapped to Host)](#local-mapped-to-host)
  - [External (Internet-Accessible)](#external-internet-accessible)
  - [Port Summary Table](#port-summary-table)
- [SATOSA Authentication Flow](#satosa-authentication-flow)
  - [Metadata Retrieval](#metadata-retrieval)
  - [Authentication Flow](#authentication-flow)
  - [Importance of Consistent Hostname and Port Configuration](#importance-of-consistent-hostname-and-port-configuration)
    - [How to Prevent Issues](#how-to-prevent-issues)
- [Setting Up the Environment](#setting-up-the-environment)
  - [Cloning the Repository](#cloning-the-repository)
  - [Using a Reverse Proxy for the SATOSA Issuer with DPoP Support](#using-a-reverse-proxy-for-the-satosa-issuer-with-dpop-support)
    - [DPoP and Target URI Binding](#dpop-and-target-uri-binding)
    - [Proper Host Handling](#proper-host-handling)
    - [Required NGINX Configuration](#required-nginx-configuration)
      - [Explanation of Key Directives](#explanation-of-key-directives)
    - [Important Considerations](#important-considerations)
    - [Summary](#summary)
  - [TLS Certificate Requirements for SATOSA](#tls-certificate-requirements-for-satosa)
    - [Remediation Steps](#remediation-steps)
    - [Trust Considerations](#trust-considerations)
    - [Using self-signed certificates](#using-self-signed-certificates)
  - [TLS Certificate Requirements for SimpleSAMLphp](#tls-certificate-requirements-for-simplesamlphp)
  - [Managing Environment Variables](#managing-environment-variables)
    - [Explanation of `.env` Variables](#explanation-of-env-variables)
      - [General Configuration](#general-configuration)
      - [API Gateway](#api-gateway-1)
      - [Issuer (SATOSA)](#issuer-satosa)
      - [SimpleSAMLphp (SAML IdP)](#simplesamlphp-saml-idp)
      - [Jaeger Tracing](#jaeger-tracing)
      - [Credential Offer](#credential-offer)
    - [Configuration Guidelines](#configuration-guidelines)
- [Running the Setup Script](#running-the-setup-script)
  - [What This Script Does](#what-this-script-does)
- [Starting and Stopping Services](#starting-and-stopping-services)
  - [Start Services](#start-services)
  - [Stop Services](#stop-services)
- [Accessing the Services](#accessing-the-services)
  - [API Gateway (Swagger UI)](#api-gateway-swagger-ui)
  - [SimpleSAMLphp UI](#simplesamlphp-ui)
  - [SATOSA Metadata](#satosa-metadata)
  - [Jaeger Tracing UI](#jaeger-tracing-ui)

## Introduction

This guide provides step-by-step instructions for setting up the **Wallet**
**Lab services**, covering the configuration of hostnames, ports, and
certificates. It also explains the role of each service and how they interact
within the system.

## Components Overview

The services consist of multiple components, each serving a specific role in
credential issuance and verification. This section describes the components,
their roles, and functionality. The services are deployed using **Docker
Compose** and interact with each other over defined **ports and networks**.

## Service Descriptions

### SATOSA (External Issuer)

- **Role:** Issues verifiable credentials and authenticates users.
- **Functionality:**
  - Redirects unauthenticated users to SimpleSAMLphp for authentication.
  - Issues credentials based on validated user identities.

### SimpleSAMLphp

- **Role:** Acts as a SAML IdP backend for SATOSA.
- **Functionality:**
  - Authenticates users based on stored credentials.
  - Generates SAML metadata, which SATOSA uses for authentication.

### API Gateway

- **Role:** Serves as the entry point for all backend services.
- **Functionality:**
  - Stateless and scalable, can run across multiple servers or data centers.
  - Interfaces with SATOSA to fetch credentials during issuance.
  - Distributes requests to appropriate backend services.

### UI Service

- **Role:** Provides a user interface for backend operations.
- **Functionality:**
  - Allows document creation and management.
  - Assists in handling verifiable credentials.

### Issuer (Internal Issuer)

- **Role:** Handles internal credential issuance, but SATOSA serves as the
  actual credential issuer in this context.
- **Functionality:** Works within the backend for credential processing.

### Verifier

- **Role:** Verifies issued credentials (feature still in development).

### Registry

- **Role:** Manages credential registration (under development).

### MockAS

- **Role:** Generates mock documents (e.g., EHIC, PDA1) for use in credential issuance.

### MongoDB

- **Role:** Provides persistent data storage.

### Jaeger

- **Role:** Provides tracing and monitoring for service interactions.

## Connectivity Levels Explained

Understanding network layers is essential to configure the infrastructure
correctly:

### Internal (Inside Docker Network)

- Services that only communicate within the Docker network.
- Example: `MockAS`, `MongoDB`, `APIgw`
- Not accessible from outside the docker network.

### Local (Mapped to Host)

- Services are exposed to the local machine but do not need internet
  connectivity.
- Example: `Jaeger`, `UI`
- Accessible within the host system and the local network, unless prevented by,
  e.g., firewall rules.

### External (Internet-Accessible)

- Services that must be reachable from the public internet.
- Example: `SATOSA`, `SimpleSAMLphp`
- Required for authentication and credential issuance.

### Port Summary Table

Each service in the VC-service infrastructure operates on a specific port, which
determines how it communicates within the system. The port assignments define
whether a service is accessible externally (from the internet), locally (within
the host machine), or internally (within the Docker network only).

- **External:** The service must be accessible from the internet.
- **Local:** The service is available on the host system but does not require
  internet access.
- **Internal:** The service is only reachable inside the Docker network and does
  not need to be exposed externally.

The following table provides an overview of the ports and their respective
connectivity requirements:

| Service | Port | Connectivity Type | Notes |
| --- | --- | --- | --- |
| **SATOSA** | 8000 | External | Issues credentials & authenticates users |
| **SimpleSAMLphp** | 8443 | External | SAML authentication service |
| **ApiGW** | 8080 | Internal | No external access required |
| **UI** | TBD | Local | Used for backend operations |
| **Verifier** | TBD | Internal | Not yet fully implemented |
| **Registry** | TBD | Internal | Not yet fully implemented |
| **MockAS** | TBD | Internal | Testing authentication services |
| **MongoDB** | 27017 | Internal | Backend database |
| **Jaeger** | 16686 | Local | Used for monitoring and tracing |

## SATOSA Authentication Flow

The SimpleSAMLphp service acts as a SAML-IdP, handling authentication requests
from SATOSA. To function correctly, SATOSA must retrieve metadata from
SimpleSAMLphp to determine how authentication should be performed.

### Metadata Retrieval

When SATOSA starts, it must fetch metadata from SimpleSAMLphp. By default, the
metadata is retrieved from:

`https://simplesamlphp.example.com:8443/simplesaml/saml2/idp/metadata.php`

This metadata contains essential details, including authentication endpoints
where SATOSA must redirect users. **SimpleSAMLphp generates this metadata based
on how it is accessed**—specifically, the hostname and port that SATOSA uses to
connect.

SATOSA does not modify this metadata but relies on it as-is for authentication
redirections.

### Authentication Flow

After retrieving metadata, SATOSA follows this process to authenticate users:

1. User Initiates Authentication
   - A user attempts to access a SATOSA-protected endpoint.
2. SATOSA Redirects to SimpleSAMLphp
   - SATOSA reads the metadata to determine the correct Single Sign-On URL.
   - The user’s browser is redirected to SimpleSAMLphp for authentication.
3. User Authenticates with SimpleSAMLphp
   - The user enters their credentials.
   - If authentication is successful, SimpleSAMLphp generates a SAML assertion
     and redirects the user back to SATOSA.
4. SATOSA Processes the Authentication Response
   - SATOSA validates the SAML assertion.
   - If valid, SATOSA completes authentication and grants access to the user.

### Importance of Consistent Hostname and Port Configuration

Since SATOSA retrieves metadata from SimpleSAMLphp, the authentication redirect
URL is determined by this metadata.

If the hostname or port used during metadata retrieval differs from the one
required during redirection, authentication will fail due to incorrect
redirects.

#### How to Prevent Issues

- **The same hostname and port must be used for internal and external access.**
  - **Correct Example:**
    - **SATOSA fetches metadata from `https://simplesamlphp.example.com:8443`**
    - **SATOSA also redirects users to**
      **`https://simplesamlphp.example.com:8443` during authentication.**

This consistent setup avoids unnecessary authentication failures.

## Setting Up the Environment

To successfully set up the Docker Compose project, follow these steps:

### Cloning the Repository

Clone the vc_up_and_running repository from GitHub:

```bash
git clone git@github.com:dc4eu/vc_up_and_running.git
cd vc_up_and_running
```

### Using a Reverse Proxy for the SATOSA Issuer with DPoP Support

This section outlines how to correctly configure a reverse proxy (e.g., NGINX)
in front of the SATOSA Issuer component in order to support DPoP-bound access
tokens. It explains the background of the DPoP mechanism and why specific proxy
settings are critical for successful validation of DPoP proofs.

#### DPoP and Target URI Binding

Demonstration of Proof-of-Possession (DPoP) is a security mechanism used to bind
an access token to a public key held by the client. This prevents token misuse
in case of leakage. DPoP proofs are sent as JWTs in the `DPoP` header and must
include a claim called `htu`, representing the exact target URI that the request
is made to, excluding query and fragment parts.

The SATOSA Issuer must reconstruct the target URI from its WSGI environment to
verify that it matches the `htu` in the DPoP proof.

#### Proper Host Handling

In reverse proxy deployments, the SATOSA service does not directly receive
requests from the client. The original request's scheme, host, and port are
replaced by the proxy. If not properly forwarded, the SATOSA Issuer will
reconstruct a target URI that does not match the `htu` in the DPoP claim,
causing DPoP validation to fail.

Correct reconstruction of the full target URI requires the following information:

- Scheme (e.g., `https`)
- Host and optional port (e.g., `issuer.example.com:8000`)
- Path (e.g., `/token`, `/credential`, etc.)

The reverse proxy must preserve this information accurately in the request
headers forwarded to SATOSA.

#### Required NGINX Configuration

The following is an example NGINX configuration for setting up a reverse proxy
in front of SATOSA:

```nginx
http {
    upstream issuer {
        server issuer.example.com:8000;  # SATOSA backend container/service
    }

    server {
        listen 8000 ssl;
        listen [::]:8000 ssl;
        server_name my.external-host.com;

        ssl_certificate /etc/nginx/certs/cert.pem;
        ssl_certificate_key /etc/nginx/certs/privkey.pem;

        location / {
            proxy_pass https://issuer;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

##### Explanation of Key Directives

- `proxy_set_header Host $http_host;`

  - Ensures the original `Host` header, including port if present, is preserved.
    This is critical for `environ["HTTP_HOST"]` in the WSGI layer.
  - `$http_host` comes from the incoming `Host:` header, unlike `$host`, which
    is derived from the `Host:` header but defaults to the server name if it's
    missing.

- `proxy_set_header X-Forwarded-Proto $scheme;`

  - Optionally used by WSGI frameworks (via `ProxyFix` or Gunicorn's
    `forwarded_allow_ips`) to reconstruct `wsgi.url_scheme`.

- `proxy_pass https://issuer;`

  - Uses HTTPS for backend communication. This must match how SATOSA is
    configured (e.g., if it's serving HTTPS internally).

#### Important Considerations

- SATOSA should **never** be exposed directly to the public. It must only be
  reachable through the proxy.
- If the reverse proxy rewrites or strips path components, DPoP validation will
  fail due to mismatched `htu` claims.
- If you want the proxy to influence the `wsgi.url_scheme`, ensure that your
  WSGI server (e.g., Gunicorn) is configured with `forwarded_allow_ips` to trust
  headers like `X-Forwarded-Proto`.

#### Summary

DPoP requires exact URI matching between the request seen by the client and the
one reconstructed by the backend server. When deploying SATOSA behind a reverse
proxy:

- Ensure the original Host header, including the port if present, is preserved
  and forwarded to the backend service unchanged.
- Avoid rewriting or altering the request path

Proper configuration ensures DPoP-bound access tokens can be validated
correctly, maintaining the integrity and security of the SATOSA Issuer.

### TLS Certificate Requirements for SATOSA

Before initializing the SATOSA container, a valid TLS private key and
corresponding certificate must be present in the `./certs` directory. These
files must be named as follows:

- `https.key` — the private key file
- `https.crt` — the certificate file  

If these files are missing at startup, Docker will attempt to mount the named
volumes and instead create **empty directories** named `https.crt` and
`https.key`. These directories will be owned by `root`, which leads to HTTPS
binding failures in the container.

#### Remediation Steps

1. Stop all running containers:

   ```bash
   ./stop.sh
   ```

2. Remove any incorrectly created directories:

   ```bash
   sudo rm -r ./certs/https.crt ./certs/https.key ./satosa/https.crt ./satosa/https.key
   ```

3. Place the correct certificate and key files in the `./certs` directory:

   - `./certs/https.crt`
   - `./certs/https.key`

4. Restart the environment:

   ```bash
   ./start.sh
   ```

#### Trust Considerations

The certificate should be issued by a publicly trusted CA. If a self-signed
certificate is used, all dependent components, including wallets, federation
resolvers, and other relying parties, must be explicitly configured to trust the
certificate.

In setups where SATOSA is deployed behind a reverse proxy (e.g., F5, NGINX), it
is common to terminate TLS at the proxy using a public certificate, while using
a self-signed certificate between the proxy and SATOSA. This approach is valid
as long as the proxy handles trust and certificate validation externally.

For production environments and optimal interoperability, using a certificate
from a recognized CA at the external interface is strongly recommended.

#### Using self-signed certificates

If you want to setup the environment with self-signed certificates, the
following steps need to be configured in vc_up_and_running. Note that this does
not document the trust setup for other relying parties mentioned in
[Trust Considerations](#trust-considerations) above.

1. Create SATOSA certificate and key in certificates folder:
  
   ```bash
   openssl req -x509 \
   -newkey rsa:4096 \
   -keyout certs/https.key \
   -out certs/https.crt \
   -sha256 \
   -days 3650 \
   -nodes \
   --subj "/CN=<your-issuer-fqdn>" \
   -addext "subjectAltName=DNS:<your-issuer-fqdn>"
   ```

   Change `<your-issuer-fqdn>` to the correct FQDN.
  
   This SATOSA issuer certificate will need to be trusted by federation trust
   infrastructure, wallets and other relying parties.

2. Edit `docker-compose.yaml` to accept SimpleSAMLphp self-signed certificate
  in SATOSA:
  
   ```bash
   services:
    satosa:
      environment:
       - "REQUESTS_CA_BUNDLE=/etc/satosa/simplesaml_webcert.pem"
   ```
  
3. Create SimpleSAMLphp certificate and key and create environment with
   `./start.sh` script:
  
   If `./simplesaml/webcert/privkey.pem` does not exist the `./start.sh` will
   create a self-signed certificate for SimpleSAMLphp at startup and copy the
   certificate to `./satosa/simplesaml_webcert.pem`.

   ```bash
   ./start.sh
   ```

   Expected result:

   ```bash
   Create simplesamlphp web certificates.
   ```

### TLS Certificate Requirements for SimpleSAMLphp

To use an issued web certificate in SimpleSAMLphp, a valid TLS private key and
corresponding certificate must be present in the `./simplesaml/webcert`
directory. These
files must be named as follows:

- `cert.pem` - the certificate file
- `privkey.pem` - the private key file

If these are not present the `./start.sh` script will generate self signed
certificates.

1. Stop the containers:
  
   ```bash
   ./stop.sh
   ```

2. Copy your SimpleSAMLphp web certificates:

   ```bash
   cp <your-certificate-file> ./simplesaml/webcert/cert.pem &&
   cp <your-private-key-file> ./simplesaml/webcert/privkey.pem
   ```
  
3. Start the containers:

   ```bash
   ./start.sh
   ```

### Managing Environment Variables

The environment is defined in the `.env` file, which contains key configuration
variables required for setting up and running the system. In most cases,
modifying these variables is sufficient to configure the environment.

Changes to other files, such as the Docker Compose configuration, may lead to
unexpected behavior and should only be made when necessary.

#### Explanation of `.env` Variables

```bash
APIGW_HOST_PORT=8080
JAEGER_HOST_PORT=16686
TAG=0.5.17
DOCKERHUB_FQDN=docker.sunet.se
NETWORK_NAME=dc4eu_shared_network
ISSUER_HOST=issuer.example.com
ISSUER_URL=https://issuer.example.com:8000
SAML_IDP_HOST=simplesamlphp.example.com
SAML_MD_URL=https://simplesamlphp.example.com:8443/simplesaml/saml2/idp/metadata.php
SAML_DS_URL=https://ds.example.com
APIGW_HOST=apigw.example.com
APIGW_URL=http://apigw.example.com:8080
CREDENTIAL_OFFER_URL=https://dc4eu.wwwallet.org/cb
```

##### General Configuration

- TAG=0.5.17  
  Specifies the version tag of the deployed Docker images.
- DOCKERHUB_FQDN=docker.sunet.se  
  Docker registry URL for pulling images.
- NETWORK_NAME=dc4eu_shared_network  
  The name of the Docker network that all containers use for internal
  communication.

##### API Gateway

- APIGW_HOST_PORT=8080  
  The port mapped to the host for accessing the API Gateway.
- APIGW_HOST=apigw.example.com  
  The FQDN for the API Gateway.
- APIGW_URL=`http://apigw.example.com:8080`  
  The full URL where ApiGW is accessible.  
  **Important**: The API Gateway uses HTTP, not HTTPS.

##### Issuer (SATOSA)

- ISSUER_HOST=issuer.example.com  
  The FQDN of the SATOSA issuer service.
- ISSUER_URL=`https://issuer.example.com:8000`  
  The full URL where the issuer service is accessible externally.

##### SimpleSAMLphp (SAML IdP)

- SAML_IDP_HOST=simplesamlphp.example.com  
  The FQDN of the SimpleSAMLphp identity provider.  
  Important: This hostname must be the same internally in Docker and externally,
  otherwise, SATOSA will generate incorrect authentication redirection URLs.
- SAML_MD_URL=
 `https://simplesamlphp.example.com:8443/simplesaml/saml2/idp/metadata.php`  
  The URL where SATOSA retrieves SAML metadata from SimpleSAMLphp.
  - This URL is used both internally (inside Docker) and externally.
  - The hostname and port in this URL must match the actual external service.
  - If the internal hostname or port differs, authentication redirects will fail.
- SAML_DS_URL=`https://ds.example.com`  
  The Discovery Service (DS) URL.
  - This is only used if the internal SAML IdP is not used and
  - The metadata (SAML_MD_URL) contains multiple IdPs.
  - If set, users will be presented with a choice of IdPs for authentication.

##### Jaeger Tracing

- JAEGER_HOST_PORT=16686  
  The port mapped to the host for Jaeger, used for monitoring and tracing.

##### Credential Offer

- CREDENTIAL_OFFER_URL=`https://dc4eu.wwwallet.org/cb`  
  The callback URL where credentials are offered to wallets.

#### Configuration Guidelines

- Ensure hostnames are correctly set
  - If external access is needed, hostnames must be set to real FQDNs.
  - Internal hostnames must match the metadata configuration (e.g.,
    `SAML_MD_URL`).
- Ensure required ports are open and properly mapped
  - SATOSA (8000) and SimpleSAMLphp (8443) must be accessible externally.
  - API Gateway (8080) operates on HTTP and does not require external access.

By correctly configuring these variables, you ensure that services communicate
properly and that the system functions as expected.

## Running the Setup Script

The start.sh script initializes the environment, generates the required keys and
certificates, and starts the necessary services.

To run the script, execute:

```bash
./start.sh
```

### What This Script Does

The script performs the following setup tasks:

- Creates the required Docker network if it does not already exist.
- Generates private keys for credential signing.
- Creates SAML certificates for SATOSA and SimpleSAMLphp.
- Extracts SAML metadata from SATOSA, ensuring that SimpleSAMLphp is correctly
  configured.
- Starts all required services by launching Docker containers with
  docker-compose.

This ensures that all necessary configurations are in place before the system
starts running.

## Starting and Stopping Services

Once the environment is set up, you can manage the services as follows:

### Start Services

To start all services:

```bash
./start.sh
```

### Stop Services

To gracefully stop all running services:

```bash
./stop.sh
```

This will shut down all containers while preserving the configuration and stored
data.

## Accessing the Services

Once the environment is running, you can access various services via the
following URLs:

| **Service** | **URL** |
| --- | --- |
| **API Gateway (Swagger UI)** | `http://127.0.0.1:8080/swagger/index.html` |
| **SimpleSAMLphp UI** | `https://simplesamlphp.example.com:8443/simplesaml/` |
| **SATOSA Metadata** | `https://issuer.example.com:8000/.well-known/openid-configuration` |
| **Jaeger Tracing UI** | `http://127.0.0.1:16686` |

### API Gateway (Swagger UI)

The API Gateway provides an interface to interact with backend services. Swagger
UI allows you to test and explore available API endpoints.

- **URL:** `http://127.0.0.1:8080/swagger/index.html`
- **Functionality:**
  - View API documentation
  - Execute API requests directly from the browser

### SimpleSAMLphp UI

The **SimpleSAMLphp UI** provides a web interface for managing SAML
configurations and testing authentication.

- **URL:** `https://simplesamlphp.example.com:8443/simplesaml/`
- **Functionality:**
  - View SAML IdP configuration
  - Test authentication with pre-configured users

### SATOSA Metadata

The SATOSA Trust Infrastructure entity configuration can be accessed to verify
that the service is running.

- **URL:** `https://issuer.example.com:8000/.well-known/openid-configuration`
- **Functionality:**
  - Returns metadata about the SATOSA OpenID Connect frontend
  - Can be used to verify that SATOSA is operational

### Jaeger Tracing UI

Jaeger provides tracing and monitoring of requests across the **VC services**.

- **URL:** `http://127.0.0.1:16686`
- **Functionality:**
  - Monitor internal service communication in real-time
  - Debug request flows across the infrastructure
