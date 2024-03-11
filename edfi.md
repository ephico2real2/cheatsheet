### Ed-Fi Docker Proof of Concept (POC) Summary and Detailed How-To Guide

This Confluence post provides a comprehensive overview and detailed instructions for setting up and deploying the Ed-Fi Operational Data Store (ODS) and API within a Dockerized environment. The guide emphasizes ease of setup, configuration flexibility, operational security, and technical details necessary for a successful proof of concept (POC).

## Summary of the POC

The POC demonstrates the deployment of Ed-Fi solutions using Docker, focusing on the Ed-Fi ODS and API. Docker Compose orchestrates containers running various components of the Ed-Fi technology suite, highlighting:

- **Ease of Setup**: Utilizing Docker Compose for quick deployment.
- **Configuration Flexibility**: Customizing environment variables and Docker Compose files.
- **Operational Security**: Enhancing security with PgBouncer and environment configurations.
- **Efficient Database Connection Pooling**: Optimizing database access to prevent server overload.
- **External Database Access**: Exposing PgBouncer ports (5401, 5402, 5403) via AWS Security Group to access ODS databases.
- **SSL Termination**: Utilizing an AWS Load Balancer to point to EC2 instances on port 443 and handle SSL termination.

## Detailed How-To Guide

### Preparing Your Docker Environment

1. **Install Docker and Docker Compose**

   Ensure Docker and Docker Compose are installed on your system by following the installation instructions on the [Docker website](https://docs.docker.com/get-docker/). Verify the installation with:
   ```bash
   docker --version
   docker-compose --version
   ```

### Setting Up the Ed-Fi-ODS-Docker Repository

2. **Clone the Repository and Checkout a Specific Tag**

   Clone the Ed-Fi-ODS-Docker repository and navigate into the directory. Then, checkout the specific tag `v2.1.5`:
   ```bash
   git clone https://github.com/Ed-Fi-Alliance/Ed-Fi-ODS-Docker.git
   cd Ed-Fi-ODS-Docker
   git checkout tags/v2.1.5
   ```

### Configuring Docker Compose and Environment Variables

3. **Launch Containers with Custom Environment and Compose File**

   Instead of using the `compose-sandbox-env.override.yml.example`, configure your deployment directly with Docker Compose using a specific environment file and Docker Compose file. For example:
   ```bash
   docker compose --env-file .env.dev -f compose-year-specific-env.example.yml up
   ```

### Managing Database Access and Security

4. **Expose Ports Via AWS Security Group**

   Configure the AWS Security Group to expose PgBouncer ports (5401, 5402, 5403) for accessing ODS databases via their respective PgBouncer.

### Deploying with Docker Compose


### Security Enhancements and SSL Termination

5. **SSL Termination with AWS Load Balancer**

   Create an AWS Load Balancer to direct traffic to the EC2 instances on port 443. Configure the Load Balancer for SSL termination to secure communications with the Ed-Fi APIs.

### Connection Pooling Configuration

6. **Server-Side Pooling with PgBouncer**

    Review and adjust PgBouncer settings in your Docker Compose and environment files to optimize connection pooling.

### Final Steps and Verification

7. **Accessing and Monitoring the Ed-Fi APIs**

    Once the deployment is successful, access the Ed-Fi ODS/API through the configured endpoints and monitor logs for any issues.


### Future Enhancements

The successful deployment of the Ed-Fi Docker Proof of Concept (POC) lays a foundation for a scalable and secure educational data management system. To further optimize and secure the deployment, the following future enhancements are proposed:

#### Enhanced Data Persistence with EBS

- **Objective**: Improve data persistence and scalability by utilizing Amazon Elastic Block Store (EBS) volumes.
- **Approach**:
  - **Add a Second EBS Volume**: Attach an additional EBS volume to the EC2 instance specifically for Docker data. This volume will store all Docker-related data, including images, containers, and volumes, ensuring data persistence beyond the lifecycle of any single container or instance.
  - **Change Docker Root Directory**: Configure Docker to use the newly attached EBS volume as its root directory. This change involves updating the Docker daemon's settings to point to the new directory on the EBS volume, ensuring that all Docker-related data is stored on the EBS volume.
  - **Set EBS Volume Persistence**: Configure the EBS volume to remain undeleted upon the EC2 instance's termination. This setting is crucial for data persistence, allowing the Docker data to be retained and reused even if the EC2 instance is terminated.

#### Automation with Ansible

- **Objective**: Streamline and automate the deployment process of the Ed-Fi Docker application to ensure consistency, repeatability, and efficiency.
- **Approach**:
  - **Develop Ansible Modules**: Create Ansible modules/scripts that automate the installation of Docker, Docker Compose, and any other necessary dependencies on the target EC2 instance. This automation will simplify the setup process, making it more accessible and less error-prone.
  - **Automate Ed-Fi Docker Application Deployment**: Extend the Ansible automation to include the deployment of the Ed-Fi Docker application. The automation should cover the cloning of the Ed-Fi-ODS-Docker repository, checking out the specific tag (e.g., v2.1.5), configuring environment variables, and launching Docker containers with the appropriate Docker Compose files.
  - **Configuration Management**: Ensure that the Ansible scripts manage and apply all necessary configurations, including AWS Security Group settings for exposing PgBouncer ports and configuring the AWS Load Balancer for SSL termination.

### Limitations with ECS and the Continued Use of Docker Compose

In the progression of deploying the Ed-Fi Docker application, a notable limitation has emerged with Amazon Elastic Container Service (ECS). The core of this challenge lies in the specific nature of the application's design, particularly concerning the PostgreSQL database setup and its intricate configurations required for the Ed-Fi ODS/API environment. These specifics have led to a decision to continue utilizing Docker Compose for deployment rather than transitioning to ECS. This section will elaborate on the limitations encountered and the rationale behind this decision.

#### Nature of the Application Design

The Ed-Fi Docker application is meticulously designed with a complex architecture that heavily relies on specific configurations for PostgreSQL databases, including connection pooling through PgBouncer and precise network setups between the database and application containers. The configuration and inter-dependencies within the Docker Compose setup are finely tuned to ensure seamless operation and communication across the various components.

#### Limitations with ECS

- **Custom Network Configurations**: ECS, while robust in handling container orchestration, presents challenges in replicating the precise network configurations and inter-container communications established in the Docker Compose files. The seamless network topology facilitated by Docker Compose is difficult to mimic in ECS without extensive custom configurations.
- **Persistent Storage Concerns**: The deployment intricacies with persistent storage for PostgreSQL databases in ECS complicate the migration. Docker Compose allows for straightforward management of persistent volumes attached to the PostgreSQL and PgBouncer containers, ensuring data persistence across container restarts and redeployments. Replicating this behavior in ECS requires intricate setup and management of persistent storage options like EBS, which ECS does not natively handle with the same level of simplicity as Docker Compose.
- **Application Compatibility**: The current design of the Ed-Fi Docker application, including its use of environment variables and .env files for configuration, is optimized for Docker Compose. Adapting the application to fit within the constraints and service paradigms of ECS would necessitate significant redesign efforts, impacting deployment efficiency and potentially introducing errors.

#### Rationale for Continuing with Docker Compose

Given these limitations, the decision to continue using Docker Compose is driven by several factors:

- **Ease of Deployment and Configuration**: Docker Compose provides a straightforward, declarative syntax for defining the multi-container application setup, making it easier to manage and deploy the Ed-Fi Docker application with its specific configurations.
- **Flexibility and Control**: Docker Compose allows for greater flexibility and control over the application environment, including network settings and persistent volumes, which are crucial for the PostgreSQL setup used by Ed-Fi.
- **Simplicity in Migration and Upgrades**: The use of Docker Compose facilitates easier migration and upgrade paths for the Ed-Fi Docker application, as changes can be implemented and tested with minimal impact on the existing deployment.

### Conclusion

While ECS offers scalability and management benefits for containerized applications, the specific requirements and configurations of the Ed-Fi Docker application favor the continued use of Docker Compose. This approach ensures that the deployment remains manageable, secure, and aligned with the application's design principles, without compromising on performance or data integrity. Future considerations for migrating to ECS or other orchestration platforms would require addressing these limitations and potentially redesigning aspects of the application to ensure compatibility and operational efficiency.
