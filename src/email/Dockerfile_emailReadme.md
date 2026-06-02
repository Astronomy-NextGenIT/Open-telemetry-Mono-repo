# Dockerfile Explanation – Email Service (Ruby)

## What is this Dockerfile?

This Dockerfile builds and runs the Email Service of the OpenTelemetry Demo application.

The Email Service is written in **Ruby** and uses **Bundler** to manage dependencies.

The Dockerfile follows a **Multi-Stage Build** approach:

### Stage 1: Builder Stage

* Installs Ruby dependencies (gems)
* Prepares the application environment

### Stage 2: Runtime Stage

* Copies installed gems
* Copies application code
* Runs the Email Service

---

# Understanding Ruby Components

Before looking at the Dockerfile, understand these important Ruby files:

### Gemfile

Similar to:

```text
Java       → pom.xml
Gradle     → build.gradle
NodeJS     → package.json
Python     → requirements.txt
Ruby       → Gemfile
```

It contains all required Ruby libraries (gems).

Example:

```ruby
gem "sinatra"
gem "grpc"
```

---

### Gemfile.lock

Stores exact versions of gems.

Purpose:

* Consistent builds
* Prevent version conflicts
* Same dependencies everywhere

---

### Bundler

Bundler is Ruby's dependency manager.

Similar to:

```text
Java      → Maven
NodeJS    → npm
Python    → pip
Ruby      → Bundler
```

Command:

```bash
bundle install
```

downloads all required gems.

---

# Stage 1: Builder Stage

## Step 1: Use Ruby Image

```dockerfile
FROM docker.io/library/ruby:3.4.8-alpine3.22 AS builder
```

### What it does

Downloads Ruby 3.4.8 running on Alpine Linux.

Contains:

* Ruby Runtime
* Bundler
* Ruby Development Tools

### Why Alpine?

Benefits:

* Small image size
* Fast downloads
* Better security

---

## Step 2: Copy Dependency Files

```dockerfile
COPY ./src/email/Gemfile Gemfile
COPY ./src/email/Gemfile.lock Gemfile.lock
```

### What it does

Copies:

```text
Gemfile
Gemfile.lock
```

into the container.

### Why copy these first?

Docker layer caching.

If application code changes but dependencies stay the same:

```text
Dependencies
      ↓
Already Cached
      ↓
No Need To Download Again
```

Build becomes much faster.

---

## Step 3: Install Build Packages

```dockerfile
RUN apk update && \
    apk add make gcc musl-dev gcompat
```

### What is apk?

Alpine Package Manager.

Equivalent to:

```text
Ubuntu → apt
RHEL   → yum
Alpine → apk
```

---

### Installed Packages

#### make

Used for building native Ruby gems.

---

#### gcc

GNU C Compiler.

Required when gems contain C extensions.

---

#### musl-dev

Development libraries for Alpine Linux.

Required for compilation.

---

#### gcompat

Compatibility package for Linux libraries.

Helps some gems work correctly on Alpine.

---

### Why are these needed?

Some Ruby gems are not pure Ruby.

Example:

```text
Gem
 ↓
Contains C Code
 ↓
Must Be Compiled
 ↓
Requires gcc + make
```

---

## Step 4: Install Ruby Gems

```dockerfile
RUN bundle install
```

### What it does

Reads:

```text
Gemfile
Gemfile.lock
```

Downloads and installs all required gems.

### Example Flow

```text
Gemfile
     ↓
Bundler
     ↓
Download Gems
     ↓
Install Gems
     ↓
Store In Ruby Bundle Directory
```

Installed gems are typically stored in:

```text
/usr/local/bundle
```

---

# Stage 2: Runtime Stage

## Step 5: Use Fresh Ruby Image

```dockerfile
FROM docker.io/library/ruby:3.4.8-alpine3.22
```

### Why start again?

Builder image contains:

```text
gcc
make
build tools
temporary files
```

These are not needed at runtime.

Starting fresh reduces image size.

---

## Step 6: Copy Installed Gems

```dockerfile
COPY --from=builder /usr/local/bundle/ /usr/local/bundle/
```

### What it does

Copies all previously installed gems.

Result:

```text
Runtime Container
       ↓
Already Has All Ruby Dependencies
```

### Benefit

No need to run:

```bash
bundle install
```

again.

Container starts faster.

---

## Step 7: Set Working Directory

```dockerfile
WORKDIR /email_server
```

### What it does

Creates:

```text
/email_server
```

and makes it the current directory.

All future commands run from here.

---

# Copy Application Files

## Step 8: Copy Views Folder

```dockerfile
COPY ./src/email/views/ views/
```

### What is views?

Contains templates used to generate emails.

Example:

```text
views/
 ├── welcome.erb
 ├── order.erb
 └── confirmation.erb
```

---

### What are ERB Files?

ERB = Embedded Ruby

Used to generate dynamic HTML content.

Example:

```html
Hello <%= customer_name %>
```

becomes:

```html
Hello Harshada
```

---

## Step 9: Copy Ruby Version File

```dockerfile
COPY ./src/email/.ruby-version .ruby-version
```

### What it does

Specifies required Ruby version.

Example:

```text
3.4.8
```

Helps maintain consistency.

---

## Step 10: Copy Gemfile

```dockerfile
COPY ./src/email/Gemfile Gemfile
```

Copies dependency definition file.

---

## Step 11: Copy Gemfile.lock

```dockerfile
COPY ./src/email/Gemfile.lock Gemfile.lock
```

Copies dependency lock file.

---

## Step 12: Copy Main Application

```dockerfile
COPY ./src/email/email_server.rb email_server.rb
```

### What is email_server.rb?

Main application file.

Equivalent to:

```text
Java     → Main.java
.NET     → Program.cs
Go       → main.go
Ruby     → email_server.rb
```

Application execution starts here.

---

# Networking Configuration

## Step 13: Expose Port

```dockerfile
EXPOSE ${EMAIL_PORT}
```

### What it does

Documents the port used by Email Service.

Example:

```text
EMAIL_PORT=8080
```

Other services can connect using this port.

---

# Starting the Application

## Step 14: Application Startup

```dockerfile
ENTRYPOINT ["bundle", "exec", "ruby", "email_server.rb"]
```

This is the most important runtime command.

---

### bundle exec

```bash
bundle exec
```

Runs the application using gems installed by Bundler.

Ensures correct gem versions are used.

---

### ruby

```bash
ruby
```

Starts the Ruby interpreter.

---

### email_server.rb

Application entry point.

---

### Complete Startup Flow

```text
Container Starts
        ↓
bundle exec
        ↓
Load Installed Gems
        ↓
Ruby Interpreter Starts
        ↓
email_server.rb Executes
        ↓
Email Service Starts
```

---

# Overall Dockerfile Flow

```text
Copy Gemfile
        ↓
Copy Gemfile.lock
        ↓
Install Build Tools
        ↓
bundle install
        ↓
Create Runtime Image
        ↓
Copy Installed Gems
        ↓
Copy Email Templates
        ↓
Copy Application Files
        ↓
Expose Port
        ↓
Run Email Service
```

# Key Concepts Used

## Ruby

Programming language used for this service.

## Bundler

Dependency manager for Ruby.

## Gem

Ruby package/library.

## Gemfile

Lists required gems.

## Gemfile.lock

Stores exact gem versions.

## bundle install

Downloads and installs dependencies.

## ERB Templates

Used to generate dynamic email content.

## Multi-Stage Build

Separates build environment from runtime environment.

## ENTRYPOINT

Defines what runs when the container starts.

# Summary

This Dockerfile builds and runs the Ruby-based Email Service. It uses Bundler to manage dependencies and follows a multi-stage build approach to keep the final image small and efficient.

Benefits:

* Smaller final image
* Faster startup
* Dependency caching
* Easy gem management
* Dynamic email template support
* Production-ready container design
