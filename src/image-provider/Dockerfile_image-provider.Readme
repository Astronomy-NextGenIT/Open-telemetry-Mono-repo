# Dockerfile Explanation – Image Provider Service

This Dockerfile is used to create a Docker image for the **Image Provider Service**. It uses Nginx to serve static files and is configured to send telemetry data to OpenTelemetry.

---

## Complete Dockerfile

```dockerfile
# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0

FROM nginxinc/nginx-unprivileged:1.29.0-alpine3.22-otel

USER 101

COPY src/image-provider/static/ /static/
COPY src/image-provider/nginx.conf.template /nginx.conf.template

EXPOSE ${IMAGE_PROVIDER_PORT}

STOPSIGNAL SIGQUIT

CMD ["/bin/sh" , "-c" , "envsubst '$OTEL_COLLECTOR_HOST $IMAGE_PROVIDER_PORT $OTEL_COLLECTOR_PORT_GRPC $OTEL_SERVICE_NAME' < /nginx.conf.template > /etc/nginx/nginx.conf && cat /etc/nginx/nginx.conf && exec nginx -g 'daemon off;'"]
```

---

# Step-by-Step Explanation

## 1. Copyright Information

```dockerfile
# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0
```

### Purpose

These are comment lines.

* Show who owns the source code.
* Specify the software license being used.
* Do not affect Docker image creation.

---

## 2. Base Image

```dockerfile
FROM nginxinc/nginx-unprivileged:1.29.0-alpine3.22-otel
```

### Purpose

This line tells Docker which base image to use.

### Breakdown

| Part               | Meaning                            |
| ------------------ | ---------------------------------- |
| nginxinc           | Official Nginx image publisher     |
| nginx-unprivileged | Runs Nginx without root privileges |
| 1.29.0             | Nginx version                      |
| alpine3.22         | Lightweight Alpine Linux OS        |
| otel               | OpenTelemetry support included     |

### Why use this image?

* Small image size
* Better security
* Nginx already installed
* OpenTelemetry integration available

---

## 3. Switch User

```dockerfile
USER 101
```

### Purpose

Runs the container as user ID 101 instead of root.

### Why?

Running as root is a security risk.

Benefits:

* Reduces attack surface
* Follows container security best practices
* Prevents accidental system modifications

---

## 4. Copy Static Files

```dockerfile
COPY src/image-provider/static/ /static/
```

### Purpose

Copies static files from the project into the container.

### Example

If your project contains:

```text
src/image-provider/static/
├── image1.jpg
├── image2.png
└── logo.svg
```

After build:

```text
/static/
├── image1.jpg
├── image2.png
└── logo.svg
```

inside the container.

### Why?

Nginx serves these files to users.

---

## 5. Copy Nginx Configuration Template

```dockerfile
COPY src/image-provider/nginx.conf.template /nginx.conf.template
```

### Purpose

Copies the Nginx configuration template into the container.

### Why use a template?

The same image can be used in different environments:

* Development
* Testing
* Staging
* Production

Values are inserted dynamically when the container starts.

---

## 6. Expose Application Port

```dockerfile
EXPOSE ${IMAGE_PROVIDER_PORT}
```

### Purpose

Documents which port the container uses.

Example:

```bash
IMAGE_PROVIDER_PORT=8080
```

Then the container exposes:

```text
Port 8080
```

### Note

`EXPOSE` does not actually publish the port.

Publishing happens when running:

```bash
docker run -p 8080:8080 image-provider
```

---

## 7. Define Stop Signal

```dockerfile
STOPSIGNAL SIGQUIT
```

### Purpose

Specifies how Docker should stop Nginx.

### Default Behavior

Docker usually sends:

```text
SIGTERM
```

### Here

Docker sends:

```text
SIGQUIT
```

### Why?

Nginx handles SIGQUIT gracefully.

It:

* Finishes active requests
* Closes connections properly
* Shuts down cleanly

---

## 8. Container Startup Command

```dockerfile
CMD ["/bin/sh" , "-c" , "envsubst '$OTEL_COLLECTOR_HOST $IMAGE_PROVIDER_PORT $OTEL_COLLECTOR_PORT_GRPC $OTEL_SERVICE_NAME' < /nginx.conf.template > /etc/nginx/nginx.conf && cat /etc/nginx/nginx.conf && exec nginx -g 'daemon off;'"]
```

This is the most important part of the Dockerfile.

---

## Step 8.1 – Start Shell

```dockerfile
/bin/sh -c
```

Runs the following commands inside a shell.

---

## Step 8.2 – Replace Environment Variables

```bash
envsubst
```

Reads:

```text
/nginx.conf.template
```

and replaces variables like:

```text
$OTEL_COLLECTOR_HOST
$IMAGE_PROVIDER_PORT
$OTEL_COLLECTOR_PORT_GRPC
$OTEL_SERVICE_NAME
```

with actual runtime values.

### Example

Template:

```nginx
server {
    listen $IMAGE_PROVIDER_PORT;
}
```

Environment:

```bash
IMAGE_PROVIDER_PORT=8080
```

Generated configuration:

```nginx
server {
    listen 8080;
}
```

---

## Step 8.3 – Generate Final Nginx Configuration

```bash
> /etc/nginx/nginx.conf
```

Saves the processed configuration as:

```text
/etc/nginx/nginx.conf
```

This becomes the active Nginx configuration.

---

## Step 8.4 – Display Configuration

```bash
cat /etc/nginx/nginx.conf
```

Prints the generated configuration to container logs.

### Why?

Useful for troubleshooting.

You can verify:

* Port number
* OpenTelemetry settings
* Hostnames
* Nginx configuration values

---

## Step 8.5 – Start Nginx

```bash
exec nginx -g 'daemon off;'
```

### Meaning

Starts Nginx in the foreground.

Normally:

```bash
nginx
```

runs in background.

Containers require a foreground process.

Therefore:

```bash
daemon off;
```

keeps Nginx running as the main container process.

### Why use exec?

Replaces the shell process with Nginx.

Benefits:

* Proper signal handling
* Graceful shutdown
* Better container management

---

# Flow of Execution

```text
Container Starts
       │
       ▼
Load Environment Variables
       │
       ▼
Process nginx.conf.template
       │
       ▼
Generate nginx.conf
       │
       ▼
Print Configuration
       │
       ▼
Start Nginx
       │
       ▼
Serve Static Files from /static
```

# Summary

This Dockerfile:

1. Uses a lightweight Nginx image with OpenTelemetry support.
2. Runs as a non-root user for security.
3. Copies static image files into the container.
4. Copies an Nginx configuration template.
5. Exposes the configured application port.
6. Uses graceful shutdown with SIGQUIT.
7. Generates the final Nginx configuration at startup.
8. Starts Nginx in the foreground to serve static content.
