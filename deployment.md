The Deployment YAML to include liveness and readiness probes using TCP socket checks, as well as setting resource limits and requests for CPU and memory.

### Updated Dockerfile

```Dockerfile
# syntax=docker/dockerfile:1

# Use the official Ubuntu image as the base image
FROM ubuntu:22.04

# Set environment variables
ENV FLASK_APP=app.py
ENV TZ=Europe/London
ENV TERM=xterm
ENV TEMPLATE_REPO_DIR=service_configurator_templates
ENV APP_DIR=/service_configurator_app
ENV SSH_KEY_PATH=/root/.ssh/id_rsa

# Install tzdata and configure it non-interactively
RUN ln -fs /usr/share/zoneinfo/$TZ /etc/localtime && \
    apt-get update && \
    apt-get install -y tzdata && \
    dpkg-reconfigure -f noninteractive tzdata && \
    apt-get clean

# Install system dependencies and Python
RUN apt-get update && apt-get install -y \
    python3.10 \
    python3.10-dev \
    python3.10-distutils \
    curl \
    software-properties-common \
    gcc \
    libblas-dev \
    libatlas-base-dev \
    libsasl2-dev \
    nano \
    python3-pip \
    python3-setuptools \
    python3-venv \
    python3-wheel \
    tzdata \
    zlib1g-dev \
    libssl-dev \
    mysql-client \
    default-libmysqlclient-dev \
    xmlsec1 \
    git \
    openssh-client \
    openssh-server \
    && rm -rf /var/lib/apt/lists/*

# Set up default Python version
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3.10 1 && \
    update-alternatives --set python /usr/bin/python3.10

# Create the .ssh and log directory
RUN mkdir -p /root/.ssh && \
    mkdir -p /opt/python/log && \
    chmod 600 "$SSH_KEY_PATH"

# Add GitHub to known hosts
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts

# Set the working directory inside the container
WORKDIR $APP_DIR

# Copy all files from the current directory to the working directory in the container
COPY . $APP_DIR

# Install Python dependencies from requirements.txt
RUN if [ -f requirements.txt ]; then \
       python3 -m pip install --no-cache-dir -r requirements.txt; \
    fi

# Copy the run.sh script to the container
COPY run.sh $APP_DIR/run.sh
RUN chmod +x $APP_DIR/run.sh

# Open port for access to the application outside the container
EXPOSE 5000

# Command to run the application
CMD ["$APP_DIR/run.sh"]
```

### Updated `run.sh` Script

```bash
#!/bin/bash

# Set environment variables
export TEMPLATE_REPO_DIR=service_configurator_templates
export FLASK_APP=app.py
export FLASK_RUN_HOST=0.0.0.0
export FLASK_RUN_PORT=5000
export SSH_KEY_PATH=/root/.ssh/id_rsa
export APP_DIR=/service_configurator_app

# Check if the SSH_KEY file exists
if [ ! -f "$SSH_KEY_PATH" ]; then
  echo "SSH_KEY file is not present at $SSH_KEY_PATH. Exiting."
  exit 1
fi

# Set permissions for the SSH key file
chmod 600 "$SSH_KEY_PATH"

# Check if GitHub's SSH key is already in known_hosts
if ! grep -q "github.com" /root/.ssh/known_hosts; then
  echo "GitHub SSH key not found in known_hosts. Scanning and adding it."
  ssh-keyscan github.com >> /root/.ssh/known_hosts
else
  echo "GitHub SSH key already in known_hosts."
fi

# Check if the REPO_URL environment variable is set
if [ -z "$REPO_URL" ]; then
  echo "REPO_URL environment variable is not set. Exiting."
  exit 1
fi

# Clone private repository or perform tasks requiring SSH access
if git clone $REPO_URL $APP_DIR/${TEMPLATE_REPO_DIR}; then
  echo "Repository cloned successfully."
else
  echo "Failed to clone repository. Exiting."
  exit 1
fi

# Change to the APP_DIR directory
cd $APP_DIR

# Start the Flask application
flask run
```

### Build and Run the Docker Image

1. **Build the Docker Image**:

   ```bash
   docker build -t python-flask-app .
   ```

2. **Run the Docker Container**: Mount the SSH key from the file system and pass the repository URL as an environment variable.

   ```bash
   docker run -p 5000:5000 -e REPO_URL=git@github.com:your-private-repo.git -v /path/to/your/id_rsa:/root/.ssh/id_rsa:ro python-flask-app
   ```

### Explanation

- **Dockerfile**:
  - Sets environment variables from the beginning.
  - Installs all necessary system and Python dependencies.
  - Creates required directories (`.ssh` and `/opt/python/log`) and sets permissions for the SSH key file.
  - Adds GitHub to known hosts.
  - Sets the working directory to `$APP_DIR`.
  - Copies application files and the `run.sh` script into the container.
  - Installs Python dependencies from `requirements.txt`.
  - Sets up the entry point to run `run.sh`.

- **run.sh**:
  - Sets the `TEMPLATE_REPO_DIR`, `FLASK_APP`, `FLASK_RUN_HOST`, `FLASK_RUN_PORT`, `SSH_KEY_PATH`, and `APP_DIR` environment variables.
  - Checks if the SSH key file exists at `$SSH_KEY_PATH`.
  - Sets the necessary permissions for the SSH key file.
  - Checks if GitHub's SSH key is already present in the `known_hosts` file. If not, it scans and adds it.
  - Checks if the `REPO_URL` environment variable is set.
  - Clones the private repository using the `REPO_URL` environment variable.
  - Changes the directory to `$APP_DIR` where `app.py` is located.
  - Starts the Flask application explicitly binding to `0.0.0.0` using the set environment variables.

- **Docker Run Command**:
  - Maps the host's SSH key file to the container's `/root/.ssh/id_rsa` path as a read-only volume.
  - Passes the `REPO_URL` as an environment variable.

This setup ensures that the SSH key is securely mounted from the host file system and used by the container, while the repository URL is provided via environment variables. The `APP_DIR` path is parameterized and used consistently across the Dockerfile and `run.sh`.

### Updated Deployment YAML (`deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-configurator
  labels:
    app: service-configurator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: service-configurator
  template:
    metadata:
      labels:
        app: service-configurator
    spec:
      containers:
      - name: service-configurator
        image: python-flask-app:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 5000
        env:
        - name: REPO_URL
          value: "git@github.com:your-private-repo.git"
        volumeMounts:
        - name: ssh-key-volume
          mountPath: /root/.ssh
          readOnly: true
          subPath: id_rsa
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:
          tcpSocket:
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          tcpSocket:
            port: 5000
          initialDelaySeconds: 5
          periodSeconds: 10
      volumes:
      - name: ssh-key-volume
        secret:
          secretName: ssh-key-secret
          items:
          - key: ssh-key
            path: id_rsa
```

### Service YAML (`service.yaml`)

No changes needed for the Service YAML.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-configurator
  labels:
    app: service-configurator
spec:
  type: ClusterIP
  ports:
  - port: 5000
    targetPort: 5000
  selector:
    app: service-configurator
```

### Secret YAML (`ssh-key-secret.yaml`)

No changes needed for the Secret YAML.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ssh-key-secret
  namespace: default
type: Opaque
data:
  ssh-key: $(cat ~/.ssh/id_rsa | base64 -w 0)
```

### Makefile

No changes needed for the Makefile.

```makefile
.PHONY: all deploy redeploy clean

NAMESPACE := default
DEPLOYMENT_NAME := service-configurator
SECRET_NAME := ssh-key-secret
DEPLOYMENT_YAML := deployment.yaml
SERVICE_YAML := service.yaml
SECRET_YAML := ssh-key-secret.yaml

all: deploy

deploy: create-secret create-deployment create-service

create-secret:
	@echo "Creating SSH key secret..."
	kubectl create secret generic $(SECRET_NAME) --from-file=ssh-key=$$HOME/.ssh/id_rsa --namespace=$(NAMESPACE) || kubectl apply -f $(SECRET_YAML)

create-deployment:
	@echo "Creating deployment..."
	kubectl apply -f $(DEPLOYMENT_YAML)

create-service:
	@echo "Creating service..."
	kubectl apply -f $(SERVICE_YAML)

redeploy: clean deploy

clean:
	@echo "Deleting deployment and service..."
	kubectl delete deployment $(DEPLOYMENT_NAME) --namespace=$(NAMESPACE) || true
	kubectl delete service $(DEPLOYMENT_NAME) --namespace=$(NAMESPACE) || true

# Optional: target to delete the secret if needed
clean-secret:
	@echo "Deleting secret..."
	kubectl delete secret $(SECRET_NAME) --namespace=$(NAMESPACE) || true
```

### README.md

No changes needed for the README.

```markdown
# Service Configurator Deployment

## Prerequisites

- Kubernetes cluster
- kubectl configured to interact with your cluster
- Docker image `python-flask-app:latest` built and available on your local Docker

## Step 1: Prepare SSH Key Secret

### Option 1: Using `kubectl` Command

```sh
kubectl create secret generic ssh-key-secret --from-file=ssh-key=$HOME/.ssh/id_rsa --namespace=default
```

### Option 2: Using YAML with Embedded `cat`

Create a `ssh-key-secret.yaml` file with the following contents:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ssh-key-secret
  namespace: default
type: Opaque
data:
  ssh-key: $(cat ~/.ssh/id_rsa | base64 -w 0)
```

Apply the Secret:

```sh
kubectl apply -f ssh-key-secret.yaml
```

## Step 2: Deploy the Application

### Using the Makefile

To deploy the application, run:

```sh
make deploy
```

To redeploy the application (clean and deploy), run:

```sh
make redeploy
```

To delete the deployment and service, run:

```sh
make clean
```

To delete the secret, run:

```sh
make clean-secret
```

## Deployment Details

- **Deployment Name:** service-configurator
- **Service Name:** service-configurator
- **Namespace:** default
- **Container Port:** 5000
- **Service Type:** ClusterIP

## Environment Variables

- **REPO_URL:** Set in the Deployment YAML to point to your private repository.

## Volumes

- **SSH Key Volume:** Mounted at `/root/.ssh` in the container with a subPath `id_rsa`.

## Resource Requests and Limits

- **Memory Requests:** 128Mi
- **CPU Requests:** 250m
- **Memory Limits:** 256Mi
- **CPU Limits:** 500m

## Health Checks

- **Liveness Probe:** TCP socket on port 5000, initial delay 30s, period 10s
- **Readiness Probe:** TCP socket on port 5000, initial delay 5s, period 10s

```

### Summary

This updated setup includes liveness and readiness probes using TCP socket checks and sets resource requests and limits for CPU and memory. The `Makefile` and other YAML files remain largely the same, with added instructions in the README to deploy and redeploy the application, ensuring the proper configuration of the Kubernetes resources.
