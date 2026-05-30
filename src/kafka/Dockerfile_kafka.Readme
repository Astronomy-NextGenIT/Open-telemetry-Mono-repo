# Dockerfile Explanation – Kafka Service

This Dockerfile is used to create a Kafka broker container with OpenTelemetry Java Agent integration. The container runs Apache Kafka and automatically sends telemetry data (metrics, traces, and logs) to an OpenTelemetry Collector.

---

# Complete Dockerfile

```dockerfile
# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0

FROM apache/kafka:3.9.1

USER root
ARG OTEL_JAVA_AGENT_VERSION

USER appuser

ADD --chown=appuser:appuser https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v$OTEL_JAVA_AGENT_VERSION/opentelemetry-javaagent.jar /tmp/opentelemetry-javaagent.jar

ENV KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER
ENV KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
ENV KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS=0
ENV KAFKA_PROCESS_ROLES=controller,broker
ENV KAFKA_NODE_ID=1
ENV KAFKA_METADATA_LOG_SEGMENT_MS=15000
ENV KAFKA_METADATA_MAX_RETENTION_MS=60000
ENV KAFKA_METADATA_LOG_MAX_RECORD_BYTES_BETWEEN_SNAPSHOTS=2800
ENV KAFKA_AUTO_CREATE_TOPICS_ENABLE=true
ENV KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
ENV KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1
ENV KAFKA_OPTS="-javaagent:/tmp/opentelemetry-javaagent.jar -Dotel.jmx.target.system=kafka-broker"
ENV CLUSTER_ID=ckjPoprWQzOf0-FuNkGfFQ
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

* Show ownership of the code.
* Define the software license.
* Do not affect Docker image execution.

---

## 2. Base Image

```dockerfile
FROM apache/kafka:3.9.1
```

### Purpose

Uses the official Apache Kafka Docker image as the base image.

### What is Kafka?

Apache Kafka is a distributed event-streaming platform used for:

* Real-time data streaming
* Message queues
* Log aggregation
* Event-driven applications

### Version

```text
Kafka Version: 3.9.1
```

This image already contains:

* Java Runtime
* Kafka Broker
* Kafka Controller
* Kafka startup scripts

---

## 3. Switch to Root User

```dockerfile
USER root
```

### Purpose

Temporarily switches to the root user.

### Why?

Root privileges may be needed for certain image build operations.

Examples:

* Installing packages
* Downloading files
* Modifying system directories

---

## 4. Build Argument

```dockerfile
ARG OTEL_JAVA_AGENT_VERSION
```

### Purpose

Defines a build-time variable.

### Example

During build:

```bash
docker build \
--build-arg OTEL_JAVA_AGENT_VERSION=2.15.0 \
-t kafka-service .
```

Value becomes:

```text
2.15.0
```

inside the Docker build process.

---

## 5. Switch Back to Application User

```dockerfile
USER appuser
```

### Purpose

Runs the remaining steps as a non-root user.

### Why?

Security best practice.

Benefits:

* Reduced permissions
* Better container security
* Prevents accidental system modifications

---

## 6. Download OpenTelemetry Java Agent

```dockerfile
ADD --chown=appuser:appuser https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v$OTEL_JAVA_AGENT_VERSION/opentelemetry-javaagent.jar /tmp/opentelemetry-javaagent.jar
```

### Purpose

Downloads the OpenTelemetry Java Agent directly from GitHub.

### Example URL

```text
https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v2.15.0/opentelemetry-javaagent.jar
```

### Stored Location

```text
/tmp/opentelemetry-javaagent.jar
```

### Ownership

```text
Owner = appuser
Group = appuser
```

### Why?

The Java Agent automatically collects:

* Traces
* Metrics
* Logs

without changing Kafka source code.

---

# Kafka Environment Variables

The following variables configure Kafka behavior.

---

## 7. Controller Listener

```dockerfile
ENV KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER
```

### Purpose

Defines the listener used for Kafka controller communication.

### Controller Role

Responsible for:

* Managing brokers
* Topic metadata
* Partition assignments

---

## 8. Listener Protocol Mapping

```dockerfile
ENV KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
```

### Purpose

Maps listeners to communication protocols.

### Meaning

| Listener   | Protocol  |
| ---------- | --------- |
| CONTROLLER | PLAINTEXT |
| PLAINTEXT  | PLAINTEXT |

### PLAINTEXT

No encryption and no authentication.

Commonly used in:

* Local development
* Test environments

---

## 9. Consumer Group Rebalance Delay

```dockerfile
ENV KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS=0
```

### Purpose

Removes startup delay for consumer groups.

### Result

Consumers begin processing immediately instead of waiting.

Useful for:

* Demos
* Labs
* Development environments

---

## 10. Kafka Roles

```dockerfile
ENV KAFKA_PROCESS_ROLES=controller,broker
```

### Purpose

Runs this container as both:

1. Controller
2. Broker

### Controller

Manages cluster metadata.

### Broker

Stores and serves messages.

### Benefit

Single-node Kafka setup.

---

## 11. Node ID

```dockerfile
ENV KAFKA_NODE_ID=1
```

### Purpose

Assigns a unique ID to the Kafka node.

### Value

```text
Node ID = 1
```

Required for KRaft mode.

---

## 12. Metadata Log Segment Duration

```dockerfile
ENV KAFKA_METADATA_LOG_SEGMENT_MS=15000
```

### Purpose

Creates a new metadata log segment every:

```text
15000 milliseconds = 15 seconds
```

---

## 13. Metadata Retention

```dockerfile
ENV KAFKA_METADATA_MAX_RETENTION_MS=60000
```

### Purpose

Retains metadata logs for:

```text
60000 milliseconds = 60 seconds
```

---

## 14. Snapshot Frequency

```dockerfile
ENV KAFKA_METADATA_LOG_MAX_RECORD_BYTES_BETWEEN_SNAPSHOTS=2800
```

### Purpose

Controls when metadata snapshots are created.

### Benefit

* Faster recovery
* Reduced startup time

---

## 15. Auto Topic Creation

```dockerfile
ENV KAFKA_AUTO_CREATE_TOPICS_ENABLE=true
```

### Purpose

Automatically creates topics when applications publish to non-existing topics.

### Example

Application sends message to:

```text
orders
```

If topic doesn't exist:

```text
Kafka automatically creates it.
```

---

## 16. Offset Topic Replication

```dockerfile
ENV KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
```

### Purpose

Sets replication factor for consumer offset storage.

### Value

```text
1
```

Because this is a single-node Kafka cluster.

---

## 17. Transaction Log Replication

```dockerfile
ENV KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=1
```

### Purpose

Sets replication factor for transaction logs.

### Value

```text
1
```

Suitable for development environments.

---

# OpenTelemetry Configuration

## 18. Kafka JVM Options

```dockerfile
ENV KAFKA_OPTS="-javaagent:/tmp/opentelemetry-javaagent.jar -Dotel.jmx.target.system=kafka-broker"
```

### Purpose

Adds OpenTelemetry monitoring to Kafka.

### Option 1

```text
-javaagent:/tmp/opentelemetry-javaagent.jar
```

Loads the OpenTelemetry Java Agent.

### Option 2

```text
-Dotel.jmx.target.system=kafka-broker
```

Enables Kafka-specific JMX metrics collection.

### Collected Metrics

Examples:

* Broker metrics
* Topic metrics
* Consumer metrics
* Producer metrics
* JVM metrics

---

## 19. Kafka Cluster ID

```dockerfile
ENV CLUSTER_ID=ckjPoprWQzOf0-FuNkGfFQ
```

### Purpose

Defines the unique Kafka cluster identifier.

### Why Needed?

KRaft mode requires a cluster ID.

Used for:

* Metadata management
* Broker registration
* Cluster identification

---

# Flow of Execution

```text
Container Starts
       │
       ▼
Kafka Image Loads
       │
       ▼
OpenTelemetry Agent Downloaded
       │
       ▼
Kafka Configuration Applied
       │
       ▼
Single Node Cluster Created
       │
       ▼
Controller + Broker Started
       │
       ▼
OpenTelemetry Monitoring Enabled
       │
       ▼
Kafka Ready For Messages
```

# Summary

This Dockerfile:

1. Uses Apache Kafka 3.9.1 as the base image.
2. Downloads the OpenTelemetry Java Agent.
3. Runs Kafka using a non-root user.
4. Configures Kafka in KRaft mode.
5. Runs both Controller and Broker in a single container.
6. Enables automatic topic creation.
7. Configures replication settings for a single-node setup.
8. Enables Kafka monitoring through OpenTelemetry.
9. Sets a unique Kafka Cluster ID.
10. Starts a Kafka broker capable of sending telemetry data to an OpenTelemetry Collector.
