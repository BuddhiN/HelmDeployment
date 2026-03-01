# WSO2 API Manager - Kubernetes Helm Deployment (Pattern 2)

This repository contains Helm charts and Docker configurations for deploying WSO2 API Manager in a Kubernetes environment using **Pattern 2: Simple Scalable Setup with All-in-One and Universal Gateway**.

## Overview

This deployment pattern follows the **WSO2 API Manager Pattern 2** architecture, which is designed for scalable deployments separating the API control plane (All-in-One) from the gateway components (Universal Gateways).

**Deployment Architecture:**
- *1x API Manager All-in-One*: Control plane handling API management, governance, and Developer Portal
- *1x Universal Gateways*: Distributed gateway instances for API traffic handling
- *Separate Databases*: Dedicated databases for API Manager (`apim_db`) and shared data (`shared_db`)

For detailed information about this deployment pattern, refer to the [WSO2 API Manager Pattern 2 Documentation](https://apim.docs.wso2.com/en/latest/install-and-setup/setup/kubernetes-deployment/kubernetes/am-pattern-2-all-in-one-gw/).

## Repository Structure

```
.
├── 1 - api-manager/              # All-in-One API Manager component
│   ├── Dockerfile               # Custom Docker image for API Manager
│   └── all-in-one/             # Helm chart for All-in-One deployment
│       ├── Chart.yaml          
│       ├── values.yaml        
│       ├── confs/              
│       └── templates/        
│       └── tenant-test/          # helm resource templates 
│
├── 1 - universal-gateway/       # Universal Gateway component
│   ├── Dockerfile              # Custom Docker image for Gateway
│   └── gateway/               # Helm chart for Gateway deployment
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── confs/
│       └── templates/
│       └── tenant-test/          # helm resource templates 
│
├── 2 - identity-server/         # WSO2 Identity Server (optional)
│   ├── Dockerfile              # Docker image for Identity Server
│   └── identity-server/        # Helm chart for Identity Server
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── confs/
│       └── templates/
│       └── tenant-test/          # helm resource templates 
│
├── wso2carbon.jks              # WSO2 keystore
├── client-truststore.jks       # WSO2 Client truststore
├── tls.crt                     # TLS certificate for ingress
├── tls.key                     # TLS private key for ingress
```

## Prerequisites

### 1. Basic Configurations

Before starting the deployment, ensure you have:

- **Git**: Version control system
- **Helm**: Kubernetes package manager (v3.0+)
- **kubectl**: Kubernetes command-line tool
- **Docker**: Container runtime (for building custom images)
- **Kubernetes Cluster**: Version 1.19 or higher
- **NGINX Ingress Controller**: For managing ingress resources


### 2. Docker Image Prerequisites
If you have a valid WSO2 subscription you can can use Images including latest updated from WSO2 [private docker registry](https://registry.wso2.com)
Sample Dockkerfile contains WSO2 APIM image with update level 13

#### Building Custom Images (Production Recommended)

This repository includes Dockerfiles that add necessary PostgreSQL drivers.

```bash
# Build API Manager All-in-One image
cd 1\ -\ api-manager/
docker build -t apim-all-in-one:1.0 . 

# Build Universal Gateway image
cd ../1\ -\ universal-gateway/
docker build -t apim-universal-gw:1.0 .

# Push to your registry
docker push your-registry/your-docker-image-name:tag
```

**Note**: Custom images include PostgreSQL JDBC driver. Adjust the Dockerfile for other database drivers as needed.

### 3. Database Configuration

Please follow WSO2 official documentations to change databases. These sample helm charts are using PostgreSQL as a default database.'
[Working With Database](https://apim.docs.wso2.com/en/latest/install-and-setup/setup/setting-up-databases/overview/)

#### Create Databases

```bash
psql -h <POSTGRE_HOST_IP> -U <USER_NAME> -W

postgres# CREATE DATABASE apim_db;
postgres# CREATE DATABASE shared_db;

CREATE USER apimadmin WITH PASSWORD 'apimadmin_password';
postgres# grant all privileges on database apim_db to apimadmin;

CREATE USER sharedadmin WITH PASSWORD 'sharedadmin_password';
postgres# grant all privileges on database shared_db to sharedadmin;

```

#### Populate Database Schema

Download WSO2 API Manager 4.6.0 distribution and run:

```bash
# Initialize APIM database
psql -h <DB_HOST> -U apimadmin -d apim_db -f <APIM_HOME>/dbscripts/apimgt/postgresql.sql

# Initialize shared database
psql -h <DB_HOST> -U sharedadmin -d shared_db -f <APIM_HOME>/dbscripts/postgresql.sql
```

## Installation and Deployment

Please refer to WSO2 Official Document for [Pattern 2: API-M Deployment with Simple Scalable Setup](https://apim.docs.wso2.com/en/latest/install-and-setup/setup/kubernetes-deployment/kubernetes/am-pattern-2-all-in-one-gw/#11-add-ingress-controller)

### Step 1: Create Kubernetes Secrets

Create secret with default WSO2 keystores and truststores.

```bash
kubectl create secret generic apim-keystore-secret --from-file=wso2carbon.jks --from-file=client-truststore.jks

```

### Step 2: Configure Helm Values

Edit the `tenant-test/env-instance.yaml` files for both API Manager and Gateway components.

helm configuration can be found in following locations in the repository

##### All-in-one Configuration (`1 - api-manager/all-in-one/tenant-test/env-instance.yaml`)
##### Gateway Configuration (`1 - universal-gateway/gateway/tenant-test/env-instance.yaml`)


Update the following sections in helm charts:

```yaml
wso2:
  deployment:
    image:
      registry: "your-registry"
      repository: "your-repository"
      digest: "your-image-digest"

    startupProbe:
      initialDelaySeconds: your-prefered-value-in-second

    livenessProbe:
      initialDelaySeconds: your-prefered-value-in-second

    replicas: your-prefered-value
    minReplicas: your-prefered-value
    maxReplicas: your-prefered-value
      
  apim:
    configurations:
      adminUsername: "your-admin-username"
      adminPassword: "your-admin-user-password"
      
      databases:
        apim_db:
          url: "jdbc:postgresql://<your-postgres-hostname>/apim_db"
          username: "apimadmin"
          password: "apimadmin_password"
        shared_db:
          url: "jdbc:postgresql://<your-postgres-hostname>/shared_db"
          username: "sharedadmin"
          password: "sharedadmin_password"

```
### Step 3: Deploy Using Helm

API Manager instances deploy to Kubernertese default namespace. User your prefered namespace with -n flag. Ingress controller is deployed in **ingress-nginx** namespace

#### Deploy All-in-One
```bash
helm install apim wso2/wso2am-all-in-one --version 4.6.0-1 -f 1 - api-manager/all-in-one/tenant-test/env-instance.yaml
```

#### Deploy Universal Gateway
```bash
helm install apim-gw wso2/wso2am-universal-gw --version 4.6.0-1 -f 1 - universal-gateway/gateway/tenant-test/env-instance.yaml
```

### Step 4: Verify Deployment

```bash
# Check pod status
kubectl get pods

# Check services
kubectl get svc

# Check ingresses
kubectl get ingress -n ingress-nginx 

# Get external IP
kubectl get ingress -n ingress-nginx -o wide

# View logs
kubectl logs -f pod-name
```

### Step 5: DNS Configuration

#### Get External IP

```bash
kubectl get ingress -n ingress-nginx
```

#### Add DNS Records

Option A: **Using DNS Provider**

Add A records in your DNS provider:
```
am.example.com        -> <EXTERNAL-IP>
gw.example.com        -> <EXTERNAL-IP>
```

Option B: **Using /etc/hosts (Development)**

```bash
# Edit /etc/hosts
sudo nano /etc/hosts

# Add entries:
<EXTERNAL-IP> am.example.com gw.example.com
```

## Accessing Management Consoles

Once deployed and DNS is configured, access the following:

### API Manager Console

- **Publisher Portal**: `https://am.example.com/publisher`
  - Manage APIs, resources, and API lifecycle

- **Developer Portal**: `https://am.example.com/devportal`
  - Subscribe to APIs and manage applications

- **Admin Portal**: `https://am.example.com/admin`
  - Administrative functions and system configuration

### API Gateway

- **Gateway Endpoint**: `https://gw.example.com`
  - APIs are published to this endpoint

**Default Credentials:**
- Username: `admin`
- Password: `admin` (change in production)

## Configuration Options

### High Availability

Enable replica scaling for API Manager:

```yaml
wso2:
  deployment:
    replicas: your-prefered-value
    minReplicas: your-prefered-value
    maxReplicas: your-prefered-value
```

## Security Best Practices

### Production Deployment Checklist

- [ ] Change default admin credentials
- [ ] Encrypt keystore and truststore passwords
- [ ] Configure HTTPS/TLS for all endpoints
- [ ] Use private Docker registries for custom images
- [ ] Implement RBAC for Kubernetes access
- [ ] Configure network policies for pod communication
- [ ] Use secrets managers (AWS Secrets Manager, Azure Key Vault, GCP Secret Manager)
- [ ] Enable audit logging
- [ ] Configure resource limits and requests
- [ ] Set up monitoring and alerting

## Monitoring and Observability

### Configure Logging

Update log configuration in `confs/log4j2.properties`:

```properties
# Enable debug logging for specific modules
logger.org.apache.synapse.transport.http.headers=DEBUG
logger.org.apache.synapse.transport.nhttp.headers=DEBUG

# Configure log output
appender.file=org.apache.logging.log4j.core.appender.RollingFileAppender
appender.file.File=${carbon.home}/repository/logs/wso2carbon.log
```

## Related Resources

- [WSO2 API Manager Documentation](https://apim.docs.wso2.com/)
- [Pattern 2 Detailed Guide](https://apim.docs.wso2.com/en/latest/install-and-setup/setup/kubernetes-deployment/kubernetes/am-pattern-2-all-in-one-gw/)
- [Helm APIM Charts](https://github.com/wso2/helm-apim/tree/4.6.x)
- [Kubernetes Deployment Overview](https://apim.docs.wso2.com/en/latest/install-and-setup/setup/kubernetes-deployment/kubernetes/kubernetes-overview/)

## Contributing

When contributing to this repository:

1. Update Dockerfile when adding dependencies
2. Keep `env-instance.yaml` consistent across charts
3. Test changes in a local Kubernetes environment
4. Document configuration changes in this README

## Version Information

- **WSO2 API Manager**: 4.6.0
- **Kubernetes**: 1.35
- **Helm**: v3.15.4
- **Docker**: 27.4.0

---

**Last Updated**: 28 February 2026
