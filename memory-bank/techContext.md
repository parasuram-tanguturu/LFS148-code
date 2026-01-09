# Technical Context: LFS148 OpenTelemetry Repository

## Technology Stack

### Application Languages & Frameworks

#### Java/Spring Boot

- **Framework**: Spring Boot 3.2.2
- **Java Version**: 21
- **Build Tool**: Maven
- **Key Dependencies**:
  - `spring-boot-starter-web`: Web framework
  - `spring-boot-starter-actuator`: Metrics and health endpoints
  - `spring-boot-starter-data-jpa`: Database access
  - `springdoc-openapi-starter-webmvc-ui`: API documentation
- **Databases**: H2 (dev), PostgreSQL (prod)
- **Services**:
  - `todobackend-springboot`: REST API backend
  - `todoui-thymeleaf`: Java-based UI

#### Python/Flask

- **Framework**: Flask 3.0.3
- **Python Version**: Python 3.x
- **Package Manager**: pip
- **Key Dependencies**:
  - `opentelemetry-api==1.26.0`: OpenTelemetry API
  - `opentelemetry-sdk==1.26.0`: OpenTelemetry SDK
  - `opentelemetry-distro==0.47b0`: OpenTelemetry distribution
  - `requests==2.32.3`: HTTP client
  - `Faker==28.4.1`: Test data generation
- **Services**:
  - `todoui-flask`: Python-based UI

### Observability Stack

#### OpenTelemetry Components

- **OpenTelemetry Collector**: `otel/opentelemetry-collector-contrib:0.109.0`
  - Receives OTLP data (gRPC ports 4317, HTTP port 4318)
  - Processes and routes telemetry data
  - Configurable via YAML

- **OpenTelemetry SDKs**:
  - **Python**: `opentelemetry-api`, `opentelemetry-sdk`, `opentelemetry-distro`
  - **Java**: Spring Boot Actuator with OTel integration

#### Observability Backends

- **Jaeger**: Distributed tracing backend
  - Image: Configurable via `${JAEGERTRACING_IMAGE}`
  - UI Port: 16686
  - OTLP Receiver: Enabled
  - Prometheus integration for metrics

- **Prometheus**: Metrics storage and querying
  - Image: Configurable via `${PROMETHEUS_IMAGE}`
  - Port: Configurable via `${PROMETHEUS_SERVICE_PORT}`
  - OTLP write receiver enabled
  - Exemplar storage enabled

### Infrastructure

#### Containerization

- **Docker**: Container runtime
- **Docker Compose**: Multi-container orchestration
  - Network: `todonet` (bridge network)
  - Services: Applications, databases, observability stack

#### Supporting Services

- **PostgreSQL**: Database for Spring Boot applications
- **Echo Server**: Test endpoint (`ealen/echo-server:0.9.2`)
- **Load Generator**: Custom container for generating traffic

## Development Setup

### Prerequisites

- Docker and Docker Compose installed
- Java 21+ (for local Java development)
- Python 3.x (for local Python development)
- Maven 3.x (for Java builds)
- Git (for cloning repository)

### Environment Variables

Key environment variables used across services:

- `OTEL_EXPORTER_OTLP_ENDPOINT`: OTLP endpoint URL
- `OTEL_EXPORTER_OTLP_TRACES_PROTOCOL`: Protocol for traces (grpc/http)
- `OTEL_EXPORTER_OTLP_METRICS_PROTOCOL`: Protocol for metrics (grpc/http)
- `OTEL_RESOURCE_ATTRIBUTES`: Service name and attributes
- `OTEL_METRICS_EXPORTER`: Metrics exporter configuration
- `OTEL_LOGS_EXPORTER`: Logs exporter configuration
- `SPRING_PROFILES_ACTIVE`: Spring profile (dev/prod)
- `POSTGRES_*`: PostgreSQL connection details

### Build & Run

#### Java Services

```bash
cd exercises/automatic-instrumentation/initial/todobackend-springboot
mvn clean package
docker build -t todobackend .
```

#### Python Services

```bash
cd exercises/automatic-instrumentation/initial/todoui-flask
pip install -r requirements.txt
docker build -t todoui-flask .
```

#### Docker Compose

```bash
cd exercises/otel-in-action
docker-compose up -d
```

## Technical Constraints

### Version Compatibility

- OpenTelemetry SDK versions must be compatible across services
- Collector version must support required processors/exporters
- Spring Boot version determines Java version requirements

### Port Requirements

- **4317**: OTLP gRPC receiver (standard)
- **4318**: OTLP HTTP receiver (standard)
- **16686**: Jaeger UI
- **9090**: Prometheus UI (default)
- **8080**: Spring Boot backend
- **8090**: Thymeleaf UI
- **5001**: Flask UI
- **5432**: PostgreSQL
- **6000**: Echo server

### Network Architecture

- All services on `todonet` bridge network
- Services communicate via service names (Docker DNS)
- External access via published ports

## Dependencies

### Python Dependencies

```
Faker==28.4.1
Flask==3.0.3
opentelemetry-api==1.26.0
opentelemetry-distro==0.47b0
opentelemetry-sdk==1.26.0
requests==2.32.3
```

### Java Dependencies

Managed via Maven POM files:

- Spring Boot BOM (parent)
- Spring Boot Starters (web, actuator, data-jpa)
- Database drivers (H2, PostgreSQL)
- SpringDoc OpenAPI

## Configuration Files

### OpenTelemetry Collector

- **Location**: `exercises/*/otel-collector-config.yml`
- **Format**: YAML
- **Sections**: extensions, receivers, processors, exporters, service

### Docker Compose

- **Location**: `exercises/*/docker-compose.yml` or `docker-compose.yaml`
- **Services**: Application services, databases, observability stack
- **Networks**: Bridge network for service communication

### Application Configuration

- **Spring Boot**: `application.properties`, `application-{profile}.properties`
- **Python**: Environment variables, code-based configuration

## Development Workflow

1. **Local Development**: Run services locally with OTel SDK
2. **Docker Development**: Use Docker Compose for full stack
3. **Testing**: Verify instrumentation via Jaeger/Prometheus UIs
4. **Comparison**: Compare implementation with solution directory

## Technical Patterns

### Instrumentation Patterns

- **Automatic**: Agent-based, zero code changes
- **Manual**: SDK-based, programmatic control
- **Hybrid**: Automatic with manual enhancements

### Data Flow

```
Application → OTLP Exporter → Collector → Backends (Jaeger/Prometheus)
```

### Context Propagation

- W3C Trace Context headers
- OpenTelemetry context propagation
- Cross-service trace correlation
