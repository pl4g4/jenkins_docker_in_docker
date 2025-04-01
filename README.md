# jenkins_docker_in_docker

This project sets up a Jenkins environment that includes a Jenkins container configured to build and deploy Docker images. The setup uses Docker Compose to orchestrate the containers and requires minimal setup to get started.

## Prerequisites

1. Install [Docker](https://docs.docker.com/get-docker/).
2. Install [Docker Compose](https://docs.docker.com/compose/install/).
3. Clone this repository to your local machine.

## Files Overview

### `docker-compose.yml`
This file defines two services:

- **`jenkins-docker`**
  - A Docker-in-Docker (DinD) service providing the Docker daemon for the Jenkins container.
  - Exposes port `2376` for Docker communication.
  - Uses the `jenkins` network with an alias `docker`.
  - Mounts the following volumes:
    - `jenkins-docker-certs`: Stores Docker TLS certificates.
    - `jenkins-data`: Shares Jenkins home directory.

- **`jenkins-latest`**
  - Jenkins container based on a custom Docker image built using the provided `Dockerfile`.
  - Configured to communicate with the `jenkins-docker` service.
  - Exposes ports `8080` (Jenkins UI) and `50000` (agent communication).
  - Mounts the following volumes:
    - `jenkins-data`: Shares Jenkins home directory.
    - `jenkins-docker-certs`: Mounts Docker TLS certificates in read-only mode.

### `Dockerfile`
This file builds a custom Jenkins image:

1. **Base Image**: `jenkins/jenkins:2.479.2-jdk17`.
2. **Docker CLI Installation**:
   - Installs Docker CLI tools to allow Jenkins to interact with Docker.
3. **Jenkins Plugins**:
   - Installs the `blueocean` and `docker-workflow` plugins.

## How to Run

1. Build and start the services using Docker Compose:
   ```bash
   docker-compose up -d
   ```

   This will:
   - Build the custom Jenkins image.
   - Start the `jenkins-docker` and `jenkins-latest` containers.

2. Access the Jenkins web interface:
   - Open your browser and navigate to `http://localhost:8080`.

3. Retrieve the initial Jenkins admin password:
   ```bash
   docker exec jenkins-latest cat /var/jenkins_home/secrets/initialAdminPassword
   ```
   Copy the output and use it to log in to Jenkins.

4. Configure Jenkins:
   - Install recommended plugins when prompted.
   - Create an admin user.

5. Verify Docker-in-Docker functionality:
   - In Jenkins, create a freestyle project.
   - Add a build step to execute the following shell command:
     ```bash
     docker --version
     ```
   - Save and run the project to ensure Docker is working within Jenkins.

## Cleanup

To stop and remove the containers and network:
```bash
docker-compose down
```

## Additional Notes

- **Volumes**:
  - `jenkins-data` persists Jenkins configuration and build data.
  - `jenkins-docker-certs` stores Docker TLS certificates for secure communication between services.

- **Network**:
  - A custom `jenkins` bridge network ensures isolated communication between services.

- **TLS Security**:
  - The `jenkins-docker` service uses `DOCKER_TLS_CERTDIR` for secure communication with the `jenkins-latest` container.

Enjoy your fully functional Jenkins with Docker integration!


