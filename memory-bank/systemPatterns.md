# System Patterns: LFS148 OpenTelemetry Architecture

## Architecture Overview

The repository demonstrates **microservices observability** patterns using OpenTelemetry. The architecture follows a distributed system model with multiple services communicating over HTTP, all instrumented for observability.

## System Architecture

### High-Level Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Flask UI      │────▶│  Spring Boot    │────▶│   PostgreSQL    │
│  (Port 5001)    │     │   Backend       │     │   Database      │
│                 │     │  (Port 8080)    │     │   (Port 5432)   │
└────────┬────────┘     └────────┬────────┘     └─────────────────┘
         │                       │
         │                       │
         ▼                       ▼
    ┌─────────────────────────────────────┐
    │  OpenTelemetry Collector           │
    │  (Ports 4317/4318)                 │
    └──────────────┬──────────────────────┘
                   │
         ┌─────────┴─────────┐
         │                   │
         ▼                   ▼
    ┌─────────┐         ┌──────────┐
    │ Jaeger  │         │Prometheus│
    │(16686)  │         │  (9090)  │
    └─────────┘         └──────────┘
```

### Component Relationships

1. **Frontend Services** (Flask UI, Thymeleaf UI)
   - Make HTTP requests to backend
   - Instrumented with OpenTelemetry SDK
   - Propagate trace context in headers

2. **Backend Service** (Spring Boot)
   - Receives requests from frontend
   - Processes business logic
   - Queries database
   - Instrumented with OTel SDK

3. **OpenTelemetry Collector**
   - Receives OTLP data from all services
   - Processes and routes telemetry
   - Exports to backends (Jaeger, Prometheus)

4. **Observability Backends**
   - **Jaeger**: Stores and visualizes traces
   - **Prometheus**: Stores and queries metrics

## Exercise Organization Pattern

### Directory Structure Pattern

Each exercise follows this consistent structure:

```
exercise-name/
├── initial/
│   ├── [application code]
│   ├── docker-compose.yml
│   ├── otel-collector-config.yml (if applicable)
│   └── README.md
└── solution/
    ├── [complete application code]
    ├── docker-compose.yml
    ├── otel-collector-config.yml (if applicable)
    └── README.md
```

### Exercise Categories

#### 1. Automatic Instrumentation

**Pattern**: Zero-code instrumentation using agents

- **Initial**: Application without instrumentation
- **Solution**: Application with automatic instrumentation enabled
- **Key Pattern**: Environment variable configuration

#### 2. Manual Instrumentation

**Pattern**: Programmatic SDK-based instrumentation

- **Traces**: Manual span creation, context propagation
- **Metrics**: Custom instruments, recording values
- **Logs**: Structured logging with trace correlation

#### 3. Collector Configuration

**Pattern**: Centralized telemetry processing

- **Receivers**: OTLP input configuration
- **Processors**: Data transformation (batch, resource)
- **Exporters**: Backend routing (Jaeger, Prometheus)

#### 4. Full Stack (OTel in Action)

**Pattern**: Complete microservices observability

- Multiple services with different technologies
- End-to-end tracing across service boundaries
- Metrics aggregation and visualization
- Load generation for realistic scenarios

## Instrumentation Patterns

### Automatic Instrumentation Pattern

**Java (Spring Boot)**:

- Uses OpenTelemetry Java agent
- Configured via environment variables
- Zero code changes required
- Automatic HTTP, database, and framework instrumentation

**Python (Flask)**:

- Uses `opentelemetry-distro` package
- Auto-instrumentation via `opentelemetry-instrument` command
- Environment variable configuration
- Automatic HTTP and framework instrumentation

### Manual Instrumentation Pattern

**Trace Instrumentation**:

```python
tracer = create_tracer("service-name", "version")
with tracer.start_as_current_span("operation-name") as span:
    span.set_attribute("key", "value")
    # business logic
```

**Metric Instrumentation**:

```python
meter = create_meter("service-name", "version")
counter = meter.create_counter("request_count")
counter.add(1, attributes={"route": "/users"})
```

**Log Correlation**:

```python
span = get_current_span()
trace_id = format_trace_id(span.get_span_context().trace_id)
logger.info(f"[trace_id={trace_id}] Operation completed")
```

### Context Propagation Pattern

**HTTP Request Propagation**:

1. Extract trace context from incoming headers
2. Attach context to current execution
3. Inject context into outgoing requests
4. Detach context on request completion

**Implementation**:

```python
# Extract from incoming request
ctx = extract(request.headers)
token = context.attach(ctx)

# Inject into outgoing request
headers = {}
inject(headers)
requests.get(url, headers=headers)

# Cleanup
context.detach(token)
```

## Data Flow Patterns

### Telemetry Data Flow

```
Application Code
    │
    ├─▶ OpenTelemetry SDK
    │       │
    │       ├─▶ Traces → OTLP Exporter
    │       ├─▶ Metrics → OTLP Exporter
    │       └─▶ Logs → OTLP Exporter
    │
    └─▶ OTLP Protocol (gRPC/HTTP)
            │
            ▼
    OpenTelemetry Collector
            │
            ├─▶ Processors (batch, resource, etc.)
            │
            ├─▶ Jaeger Exporter → Jaeger Backend
            └─▶ Prometheus Exporter → Prometheus Backend
```

### Trace Flow Across Services

```
User Request
    │
    ▼
Flask UI (creates root span)
    │
    ├─▶ HTTP Request with trace context headers
    │
    ▼
Spring Boot Backend (continues trace)
    │
    ├─▶ Database Query (child span)
    │
    └─▶ Response with trace context
    │
    ▼
Flask UI (completes root span)
    │
    ▼
All spans sent to Collector → Jaeger
```

## Configuration Patterns

### OpenTelemetry Collector Configuration

**Standard Structure**:

```yaml
extensions:
  # Extension configurations

receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 10s
  resource:
    attributes:
      environment: "development"

exporters:
  jaeger:
    endpoint: jaeger:14250
  prometheus:
    endpoint: 0.0.0.0:8889

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch, resource]
      exporters: [jaeger]
    metrics:
      receivers: [otlp]
      processors: [batch, resource]
      exporters: [prometheus]
```

### Application Configuration Pattern

**Environment Variables**:

- `OTEL_EXPORTER_OTLP_ENDPOINT`: Collector endpoint
- `OTEL_RESOURCE_ATTRIBUTES`: Service identification
- `OTEL_SERVICE_NAME`: Service name
- `OTEL_METRICS_EXPORTER`: Metrics exporter list
- `OTEL_LOGS_EXPORTER`: Logs exporter list

## Design Patterns

### Service Identification Pattern

Each service sets resource attributes:

- `service.name`: Unique service identifier
- `service.version`: Application version
- `deployment.environment`: Environment (dev/prod)

### Span Naming Convention

- **HTTP Routes**: Use route path (e.g., `/users`, `/todos`)
- **Operations**: Use operation name (e.g., `get_user`, `create_todo`)
- **Database**: Use query type (e.g., `db.query`, `db.insert`)

### Attribute Naming Convention

Follow OpenTelemetry semantic conventions:

- `http.request.method`: HTTP method
- `http.route`: Route path
- `http.response.status_code`: Status code
- `db.system`: Database type
- `db.operation`: Database operation

## Integration Patterns

### Docker Compose Pattern

- **Network**: Single bridge network for all services
- **Service Discovery**: Docker DNS resolution
- **Dependencies**: `depends_on` for startup order
- **Environment**: Shared environment variables

### Multi-Backend Pattern

Collector routes to multiple backends:

- Traces → Jaeger
- Metrics → Prometheus
- Logs → (configurable, often disabled in exercises)

## Error Handling Patterns

### Span Error Recording

```python
span = get_current_span()
try:
    # operation
except Exception as e:
    span.record_exception(e)
    span.set_status(Status(StatusCode.ERROR))
    raise
```

### Metric Error Tracking

```python
error_counter.add(1, attributes={
    "error.type": type(e).__name__,
    "http.route": request.path
})
```

## Best Practices Demonstrated

1. **Consistent Service Naming**: All services use semantic service names
2. **Context Propagation**: Proper trace context propagation across services
3. **Resource Attributes**: Rich metadata for filtering and grouping
4. **Sampling**: (Implicitly via collector configuration)
5. **Batch Processing**: Efficient telemetry export via batching
6. **Separation of Concerns**: Collector handles routing, services focus on instrumentation
