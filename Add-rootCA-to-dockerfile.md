```Dockerfile
# Use an image that has apt-get available, like Ubuntu
FROM ubuntu:latest

# Avoid prompts from apt-get
ENV DEBIAN_FRONTEND=noninteractive

# Install ca-certificates and curl
RUN apt-get update && \
    apt-get install -y ca-certificates curl && \
    rm -rf /var/lib/apt/lists/*

# Replace this with your Nexus repository URL where local-ca.crt is located
ARG CERT_URL=<YOUR_NEXUS_REPO_URL>/local-ca.crt

# Download the certificate from Nexus
RUN curl -o /usr/local/share/ca-certificates/local-ca.crt $CERT_URL

# Update the CA certificates
RUN update-ca-certificates

# Rest of your Dockerfile...
```

In this version, `curl` is used to directly download the `local-ca.crt` file into the `/usr/local/share/ca-certificates/` directory, and then `update-ca-certificates` is run to update the CA certificates. Remember to replace `<YOUR_NEXUS_REPO_URL>` with the actual URL to your Nexus repository.
