# Golang Observability Stack with Microservices

This project provides a foundation for building microservices using Go, integrated with a comprehensive observability stack. It includes sample microservices (user, auth, frontend) and deploys a suite of tools for metrics, logging, and tracing, orchestrated via Docker Compose.

## Architecture

The project follows a microservices architecture, with each service responsible for a specific domain. These services are instrumented with OpenTelemetry and communicate with an observability backend composed of Prometheus, Loki, and Tempo, all visualized through Grafana. Redpanda serves as a Kafka-compatible message broker.

*   **Microservices:**
    *   **User Service:** Manages user data.
    *   **Auth Service:** Handles authentication and authorization, interacting with the User Service.
    *   **Frontend Service:** Provides the user interface, interacting with the Auth Service.
*   **Observability Stack:**
    *   **OpenTelemetry Collector:** Central component that receives telemetry data (traces, metrics, logs) from services, processes it, and exports it to the appropriate backends.
    *   **Prometheus:** Collects and stores metrics, scraped via the OpenTelemetry Collector or directly from services (though configuration suggests OTel Collector is primary for metrics export).
    *   **Loki:** Aggregates logs from services, ingested via the OpenTelemetry Collector.
    *   **Tempo:** Stores and queries traces, ingested via the OpenTelemetry Collector.
    *   **Grafana:** A powerful visualization tool configured with Prometheus, Loki, and Tempo as data sources, providing dashboards for monitoring.
    *   **Redpanda:** Acts as a message broker, facilitating communication or event streaming between services.

## Key Components and Technologies

*   **Go:** Used for developing the microservices (`user-service`, `auth-service`, `frontend-service`).
*   **gRPC & Protocol Buffers:** Used for inter-service communication. `.proto` files define the service interfaces.
*   **OpenTelemetry:** Standard for instrumenting services to generate telemetry data.
*   **Docker & Docker Compose:** Used for containerizing and orchestrating all services and the observability stack.

### Services

*   **User Service (`services/user/`)**:
    *   Exposes a gRPC API defined in `user_service.proto`.
    *   Persists user data using SQLite (`DB_PATH`).
    *   Instrumented for OpenTelemetry.
*   **Auth Service (`services/auth/`)**:
    *   Exposes a gRPC API defined in `auth_service.proto`.
    *   Utilizes JWT for secret management.
    *   Interacts with the User Service to validate users.
    *   Instrumented for OpenTelemetry.
*   **Frontend Service (`services/frontend/`)**:
    *   Likely serves a web UI.
    *   Interacts with the Auth Service to handle authentication flows.
    *   Instrumented for OpenTelemetry.

### Observability Stack

*   **OpenTelemetry Collector (`configs/otel-collector/otel-collector.yaml`)**:
    *   Receives telemetry data via OTLP (gRPC/HTTP) on ports 4317/4318.
    *   Exports metrics to Prometheus (`http://prometheus:9090/api/v1/write`).
    *   Exports traces to Tempo (`http://tempo:4418`).
    *   Exports logs to Loki (`http://loki:3100/otlp`).
*   **Prometheus (`configs/prometheus/prometheus.yml`)**:
    *   Configured with minimal settings, primarily for storage. Receives metrics via OTLP write endpoint from the collector.
    *   Accessible at `http://localhost:9090`.
*   **Loki (`configs/loki/loki.yml`)**:
    *   Aggregates logs.
    *   Uses file-based storage for chunks.
    *   Accessible at `http://localhost:3100`.
*   **Tempo (`configs/tempo/tempo.yml`)**:
    *   Stores and queries distributed traces.
    *   Receives traces via OTLP, Jaeger, Zipkin.
    *   Also sends metrics to Prometheus.
    *   Accessible via Tempo query API at `http://localhost:3200`.
*   **Grafana (`configs/grafana/provisioning/`)**:
    *   Provides dashboards for visualizing metrics, logs, and traces.
    *   Configured with Prometheus, Loki, and Tempo as datasources.
    *   Features derived fields for seamless navigation between logs and traces.
    *   Accessible at `http://localhost:3000` (anonymous access enabled).
*   **Redpanda (`docker-compose.yaml`)**:
    *   Runs a Kafka-compatible message broker accessible on port 9092.
    *   `redpanda-console` provides a UI at `http://localhost:8080`.

## Setup and Running

### Prerequisites

*   Docker
*   Docker Compose

### Building Services and Generating gRPC Code

1.  **Generate Protocol Buffers Go code:**
    ```bash
    make gen-services-proto
    ```
    This command uses `protoc` to generate Go gRPC client and server code from the `.proto` files in the `services/user` and `services/auth` directories.

2.  **Build Docker Images:**
    The `docker-compose.yaml` file is configured to build the Docker images for the services.

### Running the Application

1.  **Start all services:**
    ```bash
    docker-compose up --build
    ```
    This command will build the Docker images (if not already built) and start all the services, including the microservices and the observability stack.

### Accessing the Application

*   **Grafana:** `http://localhost:3000` (Anonymous access, default role: Admin)
*   **Prometheus:** `http://localhost:9090`
*   **Loki:** `http://localhost:3100`
*   **Tempo:** `http://localhost:3200`
*   **User Service:** `http://localhost:8080` (gRPC)
*   **Auth Service:** `http://localhost:8081` (gRPC)
*   **Redpanda Console:** `http://localhost:8080`

### Dashboards

Pre-configured Grafana dashboards are available in the `dashboards/` directory:
*   `auth-service-dashboard.json`
*   `frontend-dashboard.json`
*   `otel-collector.json`
*   `user-service-dashboard.json`

These can be imported into Grafana for monitoring.

## File Structure Overview

```
/
├── .dockerignore
├── .gitignore
├── build_services.sh       # (Potentially for building Go binaries or Docker images)
├── docker-compose.yaml     # Orchestrates all services
├── Makefile                # For generating Go gRPC code from .proto files
├── README.MD               # This documentation file
├── configs/                # Configuration files for observability stack and services
│   ├── grafana/
│   │   └── provisioning/
│   │       ├── dashboards/
│   │       └── datasources/
│   ├── loki/
│   ├── otel-collector/
│   ├── prometheus/
│   ├── promtail/
│   ├── redpanda/
│   └── tempo/
├── dashboards/             # Grafana dashboard JSON files
├── img/                    # Project images
├── services/               # Source code for microservices
│   ├── auth/               # Auth service
│   ├── frontend/           # Frontend service
│   └── user/               # User service
└── volumes/                # Persistent data volumes for services
    ├── grafana/
    ├── loki/
    ├── prometheus/
    ├── services/
    └── tempo/
```
