# Section 11: Observability and Monitoring - A Detailed Guide

This section implements a full **observability stack** to monitor the health and behavior of the microservices. Observability is crucial for debugging and maintaining a distributed system and is often described by three pillars: **Metrics**, **Traces**, and **Logs**.

---

### 1. OpenTelemetry: The Tracing Standard

*   **What it is:** An open-source framework and standard for generating, collecting, and exporting telemetry data (traces, metrics, and logs). In this project, it is primarily used for **distributed tracing**. A trace follows a single request as it moves from one microservice to another, which is essential for pinpointing bottlenecks and errors in a distributed system.

*   **How it's configured:**
    1.  **Java Agent:** The `opentelemetry-javaagent` dependency is added to each microservice's `pom.xml`. This agent attaches to the Java Virtual Machine (JVM) at startup.
    2.  **Automatic Instrumentation:** The agent automatically "instruments" the code. This means it detects when an HTTP request comes in or goes out, when a database query is made, etc., and it creates "spans" (units of work) for each of these operations. It also injects and propagates a unique `Trace ID` across service calls.
    3.  **Service Naming:** Each microservice is given a unique name via an environment variable in `docker-compose.yml` so that traces can be correctly attributed.
        ```yaml
        accounts:
          environment:
            OTEL_SERVICE_NAME: "accounts"
        ```
    The agent is configured to export this trace data to **Tempo**.

---

### 2. Tempo: The Trace Storage System

*   **What it is:** A high-volume, distributed tracing backend developed by Grafana. Its only job is to receive trace data from OpenTelemetry agents, store it efficiently, and make it available for querying.

*   **How it's configured:**
    1.  **Docker Service:** A `tempo` service is defined in the `docker-compose.yml` file.
    2.  **Grafana Integration:** The `datasource.yml` file for Grafana automatically adds Tempo as a data source, pointing to the Tempo container's address (`http://tempo:3100`). This allows Grafana to query and visualize the stored traces.

---

### 3. Prometheus: The Metrics Database

*   **What it is:** A time-series database and monitoring system. It periodically "scrapes" (pulls) numerical metrics from configured endpoints, stores them, and allows you to query them using its powerful query language, PromQL.

*   **How it's configured:**
    1.  **Metrics Exposure:** Each microservice includes the `micrometer-registry-prometheus` dependency. This enables an endpoint at `/actuator/prometheus` where the service exposes its internal metrics (like request counts, latency, JVM memory) in a format Prometheus understands.
    2.  **Scrape Configuration:** A `prometheus.yml` file explicitly tells Prometheus which services to monitor.
        *File: `docker-compose/observability/prometheus/prometheus.yml`*
        ```yaml
        scrape_configs:
          - job_name: 'accounts'
            metrics_path: '/actuator/prometheus'
            static_configs:
              - targets: [ 'accounts:8080' ]
          # ... other services
        ```
    3.  **Grafana Integration:** The `datasource.yml` file for Grafana adds Prometheus as a data source, pointing to `http://prometheus:9090`.

---

### 4. Loki: The Log Aggregator

*   **What it is:** A log aggregation system inspired by Prometheus. Instead of indexing the full text of logs (which is expensive), Loki only indexes a small set of labels (metadata) for each log stream, like the service name and container name. This makes it very efficient for storing and querying logs from many services.

*   **How it's configured:**
    1.  **Log Collection (Alloy):** An **Alloy** agent is run as a Docker container. Its configuration file (`alloy-local-config.yaml`) tells it to discover all other running Docker containers, collect their log output, and forward it to Loki.
        *File: `docker-compose/observability/alloy/alloy-local-config.yaml`*
        ```yaml
        loki.source.docker "flog_scrape" {
            host       = "unix:///var/run/docker.sock"
            targets    = discovery.docker.flog_scrape.targets
            forward_to = [loki.write.default.receiver]
        }
        loki.write "default" {
            endpoint {
                url = "http://gateway:3100/loki/api/v1/push"
            }
        }
        ```
    2.  **Grafana Integration:** The `datasource.yml` file adds Loki as a data source. It also cleverly configures a "derived field" that automatically detects Trace IDs in the log lines and turns them into clickable links that jump directly to the corresponding trace in Tempo.

---

### 5. Grafana: The Unified Dashboard

*   **What it is:** The centerpiece of the observability stack. Grafana is an open-source platform for data visualization, monitoring, and analysis. It does not store any data itself; instead, it connects to various data sources to query and visualize their data.

*   **How it's configured:**
    1.  **Docker Service:** A `grafana` service is defined in `docker-compose.yml`.
    2.  **Automatic Provisioning:** A `datasource.yml` file is mounted into the Grafana container. When Grafana starts, it automatically reads this file and configures its connections to **Prometheus**, **Tempo**, and **Loki**. This means the entire stack is ready to use immediately after startup, with no manual configuration required.

This setup provides a powerful, integrated experience where you can seamlessly pivot between metrics, logs, and traces to get a complete picture of your system's behavior.
