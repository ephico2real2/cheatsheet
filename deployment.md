The Deployment YAML to include liveness and readiness probes using TCP socket checks, as well as setting resource limits and requests for CPU and memory.

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
