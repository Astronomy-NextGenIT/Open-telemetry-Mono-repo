# Dockerfile Explanation – Quote Service

This Dockerfile is used to build and run the **Quote Service**, which is a PHP application. 
It follows a **multi-stage build approach** to keep the final image smaller and more secure.

The service also includes **OpenTelemetry support** so that traces and metrics can be collected automatically.

---

# What Does This Service Do?

The Quote Service:

* Runs a PHP application
* Serves quote-related data
* Uses Composer for dependency management
* Supports Protocol Buffers (protobuf)
* Sends telemetry data using OpenTelemetry

---

# Complete Dockerfile

```dockerfile
FROM ghcr.io/mlocati/php-extension-installer:2.9.11 AS installer

FROM docker.io/library/composer:2.8.12 AS vendor

WORKDIR /tmp/

COPY ./src/quote/composer.json composer.json

RUN composer install \
    --ignore-platform-reqs \
    --no-interaction \
    --no-plugins \
    --no-scripts \
    --no-dev \
    --prefer-dist

FROM docker.io/library/php:8.4-cli-alpine3.22

COPY --from=installer /usr/bin/install-php-extensions /usr/local/bin/

RUN install-php-extensions opcache pcntl protobuf opentelemetry

WORKDIR /var/www

USER www-data

COPY --from=vendor /tmp/vendor/ vendor/

COPY ./src/quote/app/ app/
COPY ./src/quote/public/ public/
COPY ./src/quote/src/ src/

EXPOSE ${QUOTE_PORT}

CMD ["php", "public/index.php"]
```

---

# Stage 1 – PHP Extension Installer Stage

This stage provides a tool that helps install PHP extensions easily.

---

## Step 1: Use PHP Extension Installer Image

```dockerfile
FROM ghcr.io/mlocati/php-extension-installer:2.9.11 AS installer
```

### Purpose

Uses a special image that contains:

```text
install-php-extensions
```

utility.

### Why?

Installing PHP extensions manually can be complicated.

This tool makes installation simple.

Example:

```bash
install-php-extensions opcache
```

instead of manually downloading and compiling extensions.

---

# Stage 2 – Composer Dependency Stage

This stage downloads PHP packages required by the application.

---

## Step 2: Use Composer Image

```dockerfile
FROM docker.io/library/composer:2.8.12 AS vendor
```

### Purpose

Uses the official Composer image.

### What is Composer?

Composer is the dependency manager for PHP.

Similar to:

| Language | Package Manager |
| -------- | --------------- |
| PHP      | Composer        |
| Node.js  | npm             |
| Python   | pip             |
| Java     | Maven           |

---

## Step 3: Set Working Directory

```dockerfile
WORKDIR /tmp/
```

### Purpose

Sets:

```text
/tmp/
```

as the working directory.

---

## Step 4: Copy Composer Configuration

```dockerfile
COPY ./src/quote/composer.json composer.json
```

### Purpose

Copies:

```text
composer.json
```

into the container.

### What is composer.json?

Contains:

* PHP dependencies
* Package versions
* Project information

Example:

```json
{
  "require": {
    "open-telemetry/api": "^1.0"
  }
}
```

---

## Step 5: Install Dependencies

```dockerfile
RUN composer install \
    --ignore-platform-reqs \
    --no-interaction \
    --no-plugins \
    --no-scripts \
    --no-dev \
    --prefer-dist
```

### Purpose

Downloads all PHP dependencies.

---

### Option Explanations

#### Ignore Platform Requirements

```text
--ignore-platform-reqs
```

Ignores missing PHP extensions during installation.

---

#### No Interaction

```text
--no-interaction
```

Prevents Composer from asking questions.

Useful for automated Docker builds.

---

#### No Plugins

```text
--no-plugins
```

Disables Composer plugins.

---

#### No Scripts

```text
--no-scripts
```

Prevents execution of package scripts.

---

#### No Development Dependencies

```text
--no-dev
```

Installs only production packages.

Examples skipped:

* Testing frameworks
* Development tools
* Debugging libraries

---

#### Prefer Distribution Packages

```text
--prefer-dist
```

Downloads package archives instead of source code.

Benefits:

* Faster downloads
* Smaller builds

---

# Stage 3 – Runtime Stage

This is the final image that runs the application.

---

## Step 6: Use PHP Runtime Image

```dockerfile
FROM docker.io/library/php:8.4-cli-alpine3.22
```

### Purpose

Uses PHP 8.4 CLI running on Alpine Linux.

### Benefits

* Small image size
* Fast startup
* Lightweight environment

---

## Step 7: Copy Extension Installer Tool

```dockerfile
COPY --from=installer /usr/bin/install-php-extensions /usr/local/bin/
```

### Purpose

Copies the extension installer tool from Stage 1.

Result:

```text
/usr/local/bin/install-php-extensions
```

becomes available.

---

## Step 8: Install Required PHP Extensions

```dockerfile
RUN install-php-extensions opcache pcntl protobuf opentelemetry
```

### Purpose

Installs required PHP extensions.

---

### Opcache

```text
opcache
```

Stores compiled PHP code in memory.

Benefits:

* Faster execution
* Better performance

---

### PCNTL

```text
pcntl
```

Provides process control functionality.

Used for:

* Managing child processes
* Signal handling

---

### Protobuf

```text
protobuf
```

Supports Protocol Buffers.

Used for:

* Efficient data serialization
* gRPC communication

---

### OpenTelemetry

```text
opentelemetry
```

Adds telemetry support.

Collects:

* Traces
* Metrics
* Logs

---

## Step 9: Set Working Directory

```dockerfile
WORKDIR /var/www
```

### Purpose

Sets application directory:

```text
/var/www
```

---

## Step 10: Switch to Non-Root User

```dockerfile
USER www-data
```

### Purpose

Runs the application as:

```text
www-data
```

instead of root.

### Benefits

* Better security
* Reduced permissions
* Safer production deployment

---

## Step 11: Copy Composer Dependencies

```dockerfile
COPY --from=vendor /tmp/vendor/ vendor/
```

### Purpose

Copies downloaded Composer packages from Stage 2.

Result:

```text
/var/www/vendor/
```

contains all required PHP libraries.

---

## Step 12: Copy Application Files

### Application Logic

```dockerfile
COPY ./src/quote/app/ app/
```

Contains business logic of the Quote Service.

---

### Public Files

```dockerfile
COPY ./src/quote/public/ public/
```

Contains public entry points.

Most important file:

```text
public/index.php
```

---

### Source Files

```dockerfile
COPY ./src/quote/src/ src/
```

Contains application source code.

Examples:

* Controllers
* Services
* Utilities

---

## Step 13: Expose Application Port

```dockerfile
EXPOSE ${QUOTE_PORT}
```

### Purpose

Documents which port the service uses.

Example:

```bash
QUOTE_PORT=8080
```

Application listens on:

```text
Port 8080
```

### Note

EXPOSE does not publish the port.

Port mapping happens when running the container.

---

## Step 14: Start the Application

```dockerfile
CMD ["php", "public/index.php"]
```

### Purpose

Defines the command executed when the container starts.

### Executed Command

```bash
php public/index.php
```

### What Happens?

1. PHP starts.
2. Loads application code.
3. Loads Composer dependencies.
4. Starts the Quote Service.
5. Begins handling requests.

---

# Multi-Stage Build Flow

```text
Stage 1 (Installer)
│
├── Provides install-php-extensions tool
│
▼
Stage 2 (Vendor)
│
├── Install Composer dependencies
├── Create vendor directory
│
▼
Stage 3 (Runtime)
│
├── Use PHP 8.4 Alpine
├── Install PHP extensions
├── Copy vendor packages
├── Copy application code
├── Run as www-data user
├── Expose application port
└── Start Quote Service
```

---

# Summary

## Stage 1 – Installer

1. Provides the PHP extension installation utility.

## Stage 2 – Vendor

2. Uses Composer to download PHP dependencies.
3. Installs only production packages.

## Stage 3 – Runtime

4. Uses PHP 8.4 Alpine image.
5. Installs required extensions:

   * Opcache
   * PCNTL
   * Protobuf
   * OpenTelemetry
6. Runs as a non-root user (`www-data`).
7. Copies application code and dependencies.
8. Exposes the Quote Service port.
9. Starts the application using:

```bash
php public/index.php
```

### Final Result

The container runs a PHP-based Quote Service with:

* Composer-managed dependencies
* Protocol Buffer support
* OpenTelemetry observability
* Improved security through non-root execution
* Optimized performance using Opcache
