# Telemetry Data Flow: Detailed Visual Explanation

This document provides a comprehensive visual explanation of the telemetry data flow from Application → OTLP Exporter → Collector → Backends (Jaeger/Prometheus).

## High-Level Flow Diagram

```mermaid
flowchart TD
    subgraph App["Application Services"]
        Flask["Flask UI<br/>(Port 5001)"]
        SpringBoot["Spring Boot Backend<br/>(Port 8080)"]
        Thymeleaf["Thymeleaf UI<br/>(Port 8090)"]
    end
    
    subgraph SDK["OpenTelemetry SDK"]
        Tracer["Tracer<br/>(Creates Spans)"]
        Meter["Meter<br/>(Records Metrics)"]
        Logger["Logger<br/>(Structured Logs)"]
    end
    
    subgraph Exporter["OTLP Exporter"]
        TraceExp["Trace Exporter<br/>(OTLP gRPC/HTTP)"]
        MetricExp["Metric Exporter<br/>(OTLP gRPC/HTTP)"]
        LogExp["Log Exporter<br/>(OTLP gRPC/HTTP)"]
    end
    
    subgraph Collector["OpenTelemetry Collector<br/>(Ports 4317/4318)"]
        Receiver["OTLP Receiver<br/>(Receives telemetry)"]
        Processor["Processors<br/>(Batch, Resource, etc.)"]
        Router["Pipeline Router<br/>(Routes by type)"]
    end
    
    subgraph Backends["Observability Backends"]
        Jaeger["Jaeger<br/>(Port 16686)<br/>Traces"]
        Prometheus["Prometheus<br/>(Port 9090)<br/>Metrics"]
    end
    
    Flask --> Tracer
    Flask --> Meter
    Flask --> Logger
    SpringBoot --> Tracer
    SpringBoot --> Meter
    SpringBoot --> Logger
    Thymeleaf --> Tracer
    Thymeleaf --> Meter
    Thymeleaf --> Logger
    
    Tracer --> TraceExp
    Meter --> MetricExp
    Logger --> LogExp
    
    TraceExp -->|"OTLP Protocol<br/>(gRPC :4317)"| Receiver
    MetricExp -->|"OTLP Protocol<br/>(gRPC :4317)"| Receiver
    LogExp -->|"OTLP Protocol<br/>(HTTP :4318)"| Receiver
    
    Receiver --> Processor
    Processor --> Router
    
    Router -->|"Traces"| Jaeger
    Router -->|"Metrics"| Prometheus
```

## Step-by-Step Breakdown

### Step 1: Application Code Generates Telemetry

**What happens:**
- Application code executes (HTTP requests, database queries, etc.)
- OpenTelemetry SDK instruments the code automatically or manually

**Example - Trace Creation:**
```python
# In Flask application
@app.route("/users")
def get_user():
    # SDK automatically creates a span for this HTTP request
    # OR manually:
    with tracer.start_as_current_span("get_user") as span:
        span.set_attribute("user.id", 123)
        # ... business logic ...
```

**Example - Metric Recording:**
```python
# SDK records metrics
counter.add(1, attributes={"http.route": "/users", "status": "200"})
latency_gauge.record(0.125, attributes={"operation": "get_user"})
```

### Step 2: OpenTelemetry SDK Processes Telemetry

**Internal SDK Processing:**

```mermaid
flowchart LR
    Code["Application Code"] --> SDK["OpenTelemetry SDK"]
    
    subgraph SDK["SDK Components"]
        API["API Layer<br/>(Tracer/Meter/Logger)"]
        Context["Context Manager<br/>(Trace Context)"]
        Resource["Resource<br/>(Service Info)"]
        Processor["Span/Metric Processor"]
    end
    
    API --> Context
    Context --> Resource
    Resource --> Processor
    Processor --> Export["Ready for Export"]
```

**What the SDK does:**
- **Traces**: Creates spans with timing, attributes, and relationships
- **Metrics**: Aggregates metric data points
- **Logs**: Correlates logs with trace context
- **Resource**: Attaches service metadata (service.name, version, etc.)
- **Context**: Manages trace context propagation

### Step 3: OTLP Exporter Sends Data

**OTLP Exporter Configuration:**

```mermaid
flowchart TD
    SDK["SDK Data"] --> Exp["OTLP Exporter"]
    
    subgraph Exp["Exporter Process"]
        Format["Format Data<br/>(OTLP Protocol)"]
        Batch["Batch Data<br/>(Efficiency)"]
        Transport["Transport Layer<br/>(gRPC/HTTP)"]
    end
    
    Format --> Batch
    Batch --> Transport
    Transport -->|"gRPC :4317<br/>HTTP :4318"| Collector["Collector"]
```

**Configuration Example:**
```python
# Python SDK Configuration
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

otlp_exporter = OTLPSpanExporter(
    endpoint="http://otelcol:4317",  # Collector endpoint
    insecure=True  # For development
)

# Environment variables used:
# OTEL_EXPORTER_OTLP_ENDPOINT=http://otelcol:4317
# OTEL_EXPORTER_OTLP_TRACES_PROTOCOL=grpc
# OTEL_EXPORTER_OTLP_METRICS_PROTOCOL=grpc
```

**What gets sent:**
- **Traces**: Span data (name, start/end time, attributes, parent context)
- **Metrics**: Metric data points (counters, gauges, histograms)
- **Logs**: Log records with trace correlation

### Step 4: OpenTelemetry Collector Receives Data

**Collector Architecture:**

```mermaid
flowchart TD
    Apps["Multiple Applications"] -->|"OTLP gRPC/HTTP"| Receiver["OTLP Receiver<br/>(Port 4317/4318)"]
    
    subgraph Collector["Collector Processing"]
        Receiver --> Queue["Internal Queue"]
        Queue --> Processor["Processors"]
        
        subgraph Processors["Processor Types"]
            Batch["Batch Processor<br/>(Groups data)"]
            Resource["Resource Processor<br/>(Adds attributes)"]
            Filter["Filter Processor<br/>(Optional filtering)"]
        end
        
        Processor --> Batch
        Batch --> Resource
        Resource --> Router["Pipeline Router"]
    end
    
    Router -->|"Traces Pipeline"| TraceExp["Jaeger Exporter"]
    Router -->|"Metrics Pipeline"| MetricExp["Prometheus Exporter"]
    
    TraceExp --> Jaeger["Jaeger Backend"]
    MetricExp --> Prometheus["Prometheus Backend"]
```

**Collector Configuration:**
```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317  # Receives gRPC
      http:
        endpoint: 0.0.0.0:4318  # Receives HTTP

processors:
  batch:
    timeout: 10s        # Wait up to 10s to batch
    send_batch_size: 1024  # Send batches of 1024
  resource:
    attributes:
      environment: "production"  # Add to all telemetry

exporters:
  jaeger:
    endpoint: jaeger:14250
  prometheus:
    endpoint: http://prometheus:9090/api/v1/otlp

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

**What the Collector does:**
1. **Receives**: Accepts OTLP data from multiple applications
2. **Processes**: Batches, enriches, filters telemetry
3. **Routes**: Separates traces, metrics, logs into pipelines
4. **Exports**: Sends to appropriate backends

### Step 5: Backends Store and Visualize

**Jaeger (Traces):**

```mermaid
flowchart LR
    Collector -->|"Traces via<br/>OTLP/gRPC"| Jaeger["Jaeger"]
    
    subgraph Jaeger["Jaeger Components"]
        Ingest["Ingestion"]
        Storage["Trace Storage"]
        Query["Query Service"]
        UI["UI<br/>(Port 16686)"]
    end
    
    Ingest --> Storage
    Storage --> Query
    Query --> UI
```

**Prometheus (Metrics):**

```mermaid
flowchart LR
    Collector -->|"Metrics via<br/>OTLP HTTP"| Prometheus["Prometheus"]
    
    subgraph Prometheus["Prometheus Components"]
        Scrape["Scrape/Receive"]
        TSDB["Time Series DB"]
        Query["PromQL"]
        UI["UI<br/>(Port 9090)"]
    end
    
    Scrape --> TSDB
    TSDB --> Query
    Query --> UI
```

## Complete End-to-End Example

**Scenario: User requests `/users` endpoint**

```mermaid
sequenceDiagram
    participant User
    participant Flask as Flask UI
    participant SpringBoot as Spring Boot Backend
    participant DB as PostgreSQL
    participant SDK1 as Flask SDK
    participant SDK2 as SpringBoot SDK
    participant Exporter1 as Flask Exporter
    participant Exporter2 as SpringBoot Exporter
    participant Collector as OTel Collector
    participant Jaeger
    participant Prometheus

    User->>Flask: GET /users
    Flask->>SDK1: Create span "GET /users"
    SDK1->>SDK1: Record start time, attributes
    
    Flask->>SpringBoot: HTTP GET /todos/ (with trace headers)
    SpringBoot->>SDK2: Extract trace context
    SDK2->>SDK2: Continue trace, create child span
    
    SpringBoot->>DB: SELECT * FROM todos
    SDK2->>SDK2: Create span "db.query"
    
    DB-->>SpringBoot: Return data
    SDK2->>SDK2: End db.query span
    
    SpringBoot-->>Flask: Return response
    SDK2->>SDK2: End backend span
    SDK1->>SDK1: End frontend span
    
    SDK1->>Exporter1: Export spans/metrics
    SDK2->>Exporter2: Export spans/metrics
    
    Exporter1->>Collector: OTLP gRPC (spans)
    Exporter1->>Collector: OTLP gRPC (metrics)
    Exporter2->>Collector: OTLP gRPC (spans)
    Exporter2->>Collector: OTLP gRPC (metrics)
    
    Collector->>Collector: Batch & process
    Collector->>Jaeger: Export traces
    Collector->>Prometheus: Export metrics
    
    User->>Jaeger: View traces in UI
    User->>Prometheus: Query metrics
```

## Data Transformation at Each Stage

### 1. Application → SDK:
```
Raw operation → Structured span/metric/log object
```

### 2. SDK → Exporter:
```
SDK object → OTLP protocol buffer format
```

### 3. Exporter → Collector:
```
OTLP protobuf → Network transmission (gRPC/HTTP)
```

### 4. Collector → Backend:
```
OTLP format → Backend-specific format (Jaeger/Prometheus)
```

## Key Points

1. **Asynchronous**: Export happens asynchronously; applications don't wait
2. **Batching**: Data is batched for efficiency
3. **Protocol**: OTLP (OpenTelemetry Protocol) is vendor-neutral
4. **Separation**: Collector separates concerns (receiving vs. routing)
5. **Scalability**: Collector can handle multiple applications
6. **Flexibility**: Can route to multiple backends simultaneously

## Network Flow Details

### Port Usage

- **4317**: OTLP gRPC receiver (standard port for traces/metrics)
- **4318**: OTLP HTTP receiver (alternative protocol)
- **16686**: Jaeger UI (trace visualization)
- **9090**: Prometheus UI (metrics visualization)

### Protocol Details

**OTLP gRPC (Port 4317):**
- Binary protocol over gRPC
- Efficient for high-volume telemetry
- Used for traces and metrics
- Lower latency than HTTP

**OTLP HTTP (Port 4318):**
- JSON/Protobuf over HTTP
- More firewall-friendly
- Used for logs and some metrics
- Easier to debug

### Service Communication

All services communicate over the `todonet` Docker bridge network:
- Services resolve each other by name (e.g., `otelcol`, `jaeger`, `prometheus`)
- Internal DNS handles name resolution
- No need for IP addresses

## Configuration Examples

### Python Application Configuration

```python
# Environment variables
OTEL_EXPORTER_OTLP_ENDPOINT=http://otelcol:4317
OTEL_EXPORTER_OTLP_TRACES_PROTOCOL=grpc
OTEL_EXPORTER_OTLP_METRICS_PROTOCOL=grpc
OTEL_RESOURCE_ATTRIBUTES=service.name=todoui-flask,service.version=1.0.0
OTEL_METRICS_EXPORTER=otlp
```

### Java/Spring Boot Configuration

```properties
# application.properties
otel.exporter.otlp.endpoint=http://otelcol:4317
otel.exporter.otlp.protocol=grpc
otel.resource.attributes=service.name=todobackend-springboot
```

### Docker Compose Environment Variables

```yaml
environment:
  - OTEL_EXPORTER_OTLP_ENDPOINT=http://otelcol:4317
  - OTEL_EXPORTER_OTLP_TRACES_PROTOCOL=grpc
  - OTEL_EXPORTER_OTLP_METRICS_PROTOCOL=grpc
  - OTEL_RESOURCE_ATTRIBUTES=service.name=todobackend-springboot
  - OTEL_METRICS_EXPORTER=otlp,logging-otlp
  - OTEL_LOGS_EXPORTER=none
```

## Troubleshooting Flow

If telemetry isn't appearing in backends, check each stage:

1. **Application**: Is SDK initialized? Are spans/metrics being created?
2. **Exporter**: Is endpoint correct? Is protocol matching?
3. **Network**: Can application reach collector? Check Docker network.
4. **Collector**: Is collector receiving data? Check logs.
5. **Backend**: Is exporter configured correctly? Check collector config.

## Related Documentation

- See `techContext.md` for technology stack details
- See `systemPatterns.md` for architecture patterns
- See `projectbrief.md` for overall repository context

