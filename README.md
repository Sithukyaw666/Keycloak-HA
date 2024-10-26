# Keycloak High Availability (HA) Setup

This project provides a configuration for deploying Keycloak in a high availability setup using Docker.

## Prerequisites

- Docker
- Docker Compose

## Introduction
A High Availability (HA) Keycloak cluster using Docker Compose consists of two Keycloak instances, a shared PostgreSQL database, and an NGINX frontend proxy. Here's an overview of this architecture:

### Components
1.  **Keycloak Instances**:
    -   **keycloak-one**: The first Keycloak server instance.
    -   **keycloak-two**: The second Keycloak server instance.
    
2.  **PostgreSQL Database**:
    -   A shared database for both Keycloak instances.
3.  **NGINX**:
    -   Acts as a load balancer, distributing incoming traffic between the two Keycloak instances.

### Clustering Mechanism

Keycloak utilizes Infinispan for caching, which supports distributed caching. By default, Infinispan in Keycloak uses UDP multicast as its transport protocol. However, since this approach broadcasts to all IP addresses on the same network, it can lead to conflicts if there are multiple independent Keycloak clusters. To avoid this issue, this architecture employs JDBC Ping as the transport protocol for clustering.
-   ****JDBC** Ping**: This protocol allows each Keycloak instance to store and retrieve cluster node information from the database. Upon startup, each instance queries the database for existing members, identifies the coordinator, and registers itself in the cluster. 
Visit [JGroups Documentation](http://jgroups.org/manual4/index.html#DiscoveryProtocols) for more information.

We use a custom NGINX configuration to load balance the two Keycloak instances with the default round-robin mechanism. By enabling SSL at the proxy level, we can use standard HTTP for internal communication between the NGINX proxy and the Keycloak instances. This means that SSL termination occurs at the proxy, where the encrypted traffic is decrypted before it reaches the Keycloak servers.
In this setup, the NGINX proxy handles all SSL-related processes, reducing the load on the Keycloak instances themselves. As a result, the backend servers can focus on their primary functions without the added overhead of managing SSL connections.


## Getting Started

### 1. Clone the Repository

Begin by cloning the GitHub repository to your local machine:

```bash
git clone https://github.com/Sithukyaw666/Keycloak-HA.git
cd Keycloak-HA
```
### 2. Build the Docker Image
Build the Docker image using the Dockerfile provided in the repository. Replace <image-name> and <tag> with your desired image name and tag:
```bash
docker build -t <image-name>:<tag> .
```
### 3. Update the docker-compose.yml
Open the docker-compose.yml file and update the placeholders:

Replace `<image-name>` with the name of your Docker image.
Replace `<database-vendor>`with your database vendor (e.g., postgres).
Replace `<database-host>` with the hostname of your database server (e.g., localhost or the service name if using Docker).
Replace `<database-name>`, `<database-username>`, and `<database-password>` with your database credentials.

### 4. Update the Nginx Configuration
Open the `nginx.conf` file and make the necessary updates:

Change the server name and any other configuration parameters to fit your environment.
Ensure SSL configuration is correct if you are using HTTPS.
### 5. Start the Services
Once you have updated the configurations, you can start the services using Docker Compose:
```bash
docker-compose up -d
```
This command will start all the defined services in the background.

### 6. Access Keycloak
After the services are up, you can access Keycloak by navigating to `https://<domain>:8443` in your web browser.

### 7. Admin Credentials
Use the following credentials to log in to the Keycloak admin console:
```
Username: admin
Password: admin
```
(You may want to change these credentials for production use.)

### Additional Notes
Ensure you have valid SSL certificates if running in a production environment.
Refer to the [Keycloak Documentation](https://www.keycloak.org/) for further information on configuration and usage.

