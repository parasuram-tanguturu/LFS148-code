# Metrics Architecture: ASCII Visual Diagram

> **Visual ASCII representation** of the OpenTelemetry Metrics Architecture for producing telemetry. This diagram complements the detailed documentation in `metricsArchitecture.md`.

```text
═══════════════════════════════════════════════════════════════════════════════════
                    OPEN TELEMETRY METRICS ARCHITECTURE
                    Producing Telemetry - Full Details
═══════════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────────┐
│                          LAYER 1: APPLICATION LAYER                             │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                          APPLICATION CODE                                │  │
│  │                                                                          │  │
│  │  • Executes business logic                                              │  │
│  │  • Calls metrics API to record measurements                             │  │
│  │  • Does NOT know about SDK implementation details                       │  │
│  │                                                                          │  │
│  │  Example:                                                                │  │
│  │    meter = metrics.get_meter("my-service")                               │  │
│  │    counter = meter.create_counter("requests_total")                      │  │
│  │    counter.add(1, attributes={"method": "GET", "status": "200"})        │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ calls
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          LAYER 2: API LAYER                                    │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                         METRICS API                                      │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  Key Functions:                                                    │  │  │
│  │  │  • metrics.get_meter(name, version)                                │  │  │
│  │  │  • meter.create_counter(name, ...)                                │  │  │
│  │  │  • meter.create_histogram(name, ...)                               │  │  │
│  │  │  • meter.create_observable_gauge(name, callbacks, ...)            │  │  │
│  │  │  • counter.add(value, attributes)                                 │  │  │
│  │  │  • histogram.record(value, attributes)                            │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  │                                                                          │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  NoOpMeterProvider (Default)                                       │  │  │
│  │  │  • Used when NO SDK is configured                                  │  │  │
│  │  │  • Prevents errors when SDK not initialized                        │  │  │
│  │  │  • All API calls succeed but produce NO telemetry                 │  │  │
│  │  │  • Allows gradual adoption of observability                       │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    │                               │
         implements │                               │ register global provider
                    │                               │ (replaces NoOpMeterProvider)
                    ▼                               ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          LAYER 3: SDK LAYER                                    │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                      METER PROVIDER                                      │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  Responsibilities:                                                 │  │  │
│  │  │  • Implements metrics API interface                                │  │  │
│  │  │  • Registers itself as global provider                             │  │  │
│  │  │  • Creates and manages Meter instances                             │  │  │
│  │  │  • Stores configuration:                                           │  │  │
│  │  │    - Resource attributes                                           │  │  │
│  │  │    - MetricReaders (list)                                          │  │  │
│  │  │    - Views (customize metrics)                                     │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  │                                                                          │  │
│  │  Configuration:                                                          │  │
│  │    provider = MeterProvider(                                            │  │
│  │        resource=resource,                                               │  │
│  │        metric_readers=[reader1, reader2],                              │  │
│  │        views=[view1, view2]                                            │  │
│  │    )                                                                    │  │
│  │    metrics.set_meter_provider(provider)  # Register                    │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                    │                                            │
│                    ┌───────────────┼───────────────┐                            │
│                    │               │               │                            │
│         creates    │               │               │                            │
│                    ▼               ▼               ▼                            │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                            METER                                         │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  Responsibilities:                                                 │  │  │
│  │  │  • Creates instruments (Counter, Histogram, Gauge, etc.)          │  │  │
│  │  │  • Stores configuration from MeterProvider                         │  │  │
│  │  │  • Manages instrument lifecycle                                    │  │  │
│  │  │                                                                     │  │  │
│  │  │  Created per: name + version                                       │  │  │
│  │  │  Example: meter = metrics.get_meter("my-service", "1.0.0")         │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  │                                                                          │  │
│  │                    ┌──────────────────────┐                              │  │
│  │                    │   INSTRUMENTS        │                              │  │
│  │                    └──────────────────────┘                              │  │
│  │                                                                          │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  SYNCHRONOUS INSTRUMENTS                                           │  │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │  │  │
│  │  │  │   COUNTER    │  │UP-DOWN COUNTER│  │  HISTOGRAM   │          │  │  │
│  │  │  │              │  │              │  │              │          │  │  │
│  │  │  │ Monotonic: ✓ │  │ Monotonic: ✗ │  │ Monotonic: ✗ │          │  │  │
│  │  │  │ Additive: ✓  │  │ Additive: ✓  │  │ Additive: ✗  │          │  │  │
│  │  │  │              │  │              │  │              │          │  │  │
│  │  │  │ Use Cases:   │  │ Use Cases:   │  │ Use Cases:   │          │  │  │
│  │  │  │ • Requests   │  │ • Connections│  │ • Latency    │          │  │  │
│  │  │  │ • Errors     │  │ • Queue size │  │ • Size       │          │  │  │
│  │  │  │ • Events     │  │ • Cache size │  │ • Duration   │          │  │  │
│  │  │  │              │  │              │  │              │          │  │  │
│  │  │  │ Operations:  │  │ Operations:  │  │ Operations:  │          │  │  │
│  │  │  │ counter.add( │  │ counter.add( │  │ histogram.   │          │  │  │
│  │  │  │   value,     │  │   value,     │  │   record(    │          │  │  │
│  │  │  │   attributes)│  │   attributes)│  │   value,     │          │  │  │
│  │  │  │              │  │   (can be -)│  │   attributes)│          │  │  │
│  │  │  └──────────────┘  └──────────────┘  └──────────────┘          │  │  │
│  │  │                                                                 │  │  │
│  │  │  Characteristics:                                               │  │  │
│  │  │  • Invoked inline with application logic                        │  │  │
│  │  │  • Can access current context (trace context)                    │  │  │
│  │  │  • Record measurements immediately when called                   │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  │                                                                          │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  ASYNCHRONOUS INSTRUMENTS                                            │  │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │  │  │
│  │  │  │ OBSERVABLE   │  │ OBSERVABLE   │  │ OBSERVABLE   │          │  │  │
│  │  │  │   COUNTER    │  │UP-DOWN COUNTER│  │    GAUGE     │          │  │  │
│  │  │  │              │  │              │  │              │          │  │  │
│  │  │  │ Monotonic: ✓ │  │ Monotonic: ✗ │  │ Monotonic: ✗ │          │  │  │
│  │  │  │ Additive: ✓  │  │ Additive: ✓  │  │ Additive: ✗  │          │  │  │
│  │  │  │              │  │              │  │              │          │  │  │
│  │  │  │ Use Cases:   │  │ Use Cases:   │  │ Use Cases:   │          │  │  │
│  │  │  │ • Bytes sent │  │ • Memory     │  │ • CPU usage  │          │  │  │
│  │  │  │ • Operations │  │ • Threads     │  │ • Temp       │          │  │  │
│  │  │  │              │  │              │  │ • Disk usage │          │  │  │
│  │  │  │              │  │              │  │              │          │  │  │
│  │  │  │ Operations:  │  │ Operations:  │  │ Operations:  │          │  │  │
│  │  │  │ Callback     │  │ Callback     │  │ Callback     │          │  │  │
│  │  │  │ returns:     │  │ returns:     │  │ returns:     │          │  │  │
│  │  │  │ Observation( │  │ Observation( │  │ Observation( │          │  │  │
│  │  │  │   value,     │  │   value,     │  │   value,     │          │  │  │
│  │  │  │   attributes)│  │   attributes)│  │   attributes)│          │  │  │
│  │  │  └──────────────┘  └──────────────┘  └──────────────┘          │  │  │
│  │  │                                                                 │  │  │
│  │  │  Characteristics:                                               │  │  │
│  │  │  • Register callback functions                                  │  │  │
│  │  │  • Callbacks invoked only on demand (when Reader collects)      │  │  │
│  │  │  • Cannot access current context directly                        │  │  │
│  │  │  • Useful for system metrics                                     │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                    │                                            │
│                    ┌───────────────┼───────────────┐                            │
│                    │               │               │                            │
│         report     │               │               │ callback (async)            │
│         (sync)     │               │               │                            │
│                    ▼               ▼               ▼                            │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                         MEASUREMENT                                       │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  Structure:                                                         │  │  │
│  │  │  • value: number                                                   │  │  │
│  │  │  • attributes: {key: value}                                        │  │  │
│  │  │                                                                     │  │  │
│  │  │  Example:                                                          │  │  │
│  │  │    Measurement(                                                    │  │  │
│  │  │      value=1,                                                      │  │  │
│  │  │      attributes={"method": "GET", "status": "200"}                │  │  │
│  │  │    )                                                               │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                    │                                            │
│                                    │ aggregated by                               │
│                                    ▼                                            │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                        METRIC READER                                      │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  Responsibilities:                                                 │  │  │
│  │  │  • Periodically collects measurements from instruments             │  │  │
│  │  │  • Aggregates measurements into metrics                            │  │  │
│  │  │  • Triggers export via MetricExporter                              │  │  │
│  │  │  • Manages collection intervals                                    │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  │                                                                          │  │
│  │  Types:                                                                  │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  PeriodicExportingMetricReader                                    │  │  │
│  │  │  • Collects periodically (e.g., every 5 seconds)                   │  │  │
│  │  │  • Exports via exporter                                           │  │  │
│  │  │  • Push-based                                                      │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  PrometheusMetricReader                                           │  │  │
│  │  │  • Exports in Prometheus format                                   │  │  │
│  │  │  • Pull-based (exposes HTTP endpoint)                             │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                    │                                            │
│                                    │ exports via                                 │
│                                    ▼                                            │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                       METRIC EXPORTER                                    │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  Responsibilities:                                                 │  │  │
│  │  │  • Receives metrics from MetricReader                              │  │  │
│  │  │  • Formats metrics according to protocol                           │  │  │
│  │  │  • Sends metrics to backend                                       │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  │                                                                          │  │
│  │  Types:                                                                  │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  OTLPMetricExporter                                               │  │  │
│  │  │  • Exports via OTLP protocol (gRPC/HTTP)                          │  │  │
│  │  │  • Sends to Collector (port 4317)                                 │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  ConsoleMetricExporter                                            │  │  │
│  │  │  • Exports to console (stdout)                                    │  │  │
│  │  │  • For debugging                                                  │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  PrometheusMetricExporter                                         │  │  │
│  │  │  • Exports in Prometheus format                                   │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                    │                                            │
│                                    │ exports                                    │
│                                    ▼                                            │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                         BACKEND SYSTEMS                                  │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  • OpenTelemetry Collector (OTLP)                                 │  │  │
│  │  │  • Prometheus                                                      │  │  │
│  │  │  • Console (stdout)                                               │  │  │
│  │  │  • Other observability backends                                    │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │                            VIEWS                                         │  │
│  │  ┌────────────────────────────────────────────────────────────────────┐  │  │
│  │  │  Purpose: Customize metrics generated by SDK                       │  │  │
│  │  │                                                                     │  │  │
│  │  │  Capabilities:                                                     │  │  │
│  │  │  • Rename metrics                                                  │  │  │
│  │  │  • Drop metrics                                                    │  │  │
│  │  │  • Change aggregation (histogram buckets)                          │  │  │
│  │  │  • Filter attributes                                               │  │  │
│  │  │                                                                     │  │  │
│  │  │  Example:                                                          │  │  │
│  │  │    View(                                                           │  │  │
│  │  │      instrument_type=Counter,                                      │  │  │
│  │  │      instrument_name="traffic_volume",                            │  │  │
│  │  │      name="request_count"  # Rename                               │  │  │
│  │  │    )                                                               │  │  │
│  │  └────────────────────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════════════════════════
                            DATA FLOW DIAGRAMS
═══════════════════════════════════════════════════════════════════════════════════

SYNCHRONOUS INSTRUMENT FLOW:
───────────────────────────────────────────────────────────────────────────────────
Application
    │
    │ counter.add(1, attributes={"method": "GET"})
    ▼
Instrument (Counter)
    │
    │ Create Measurement(value=1, attributes={"method": "GET"})
    ▼
Measurement
    │
    │ Report immediately
    ▼
MetricReader
    │
    │ Aggregate measurements over collection period
    ▼
MetricReader (on interval, e.g., every 5 seconds)
    │
    │ Export metrics
    ▼
MetricExporter
    │
    │ Format metrics (OTLP)
    ▼
Backend (Collector/Prometheus)


ASYNCHRONOUS INSTRUMENT FLOW:
───────────────────────────────────────────────────────────────────────────────────
MetricReader (on collection interval)
    │
    │ Collect (invoke callback)
    ▼
Instrument (ObservableGauge)
    │
    │ Invoke callback function
    ▼
Callback Function
    │
    │ Return [Observation(value, attributes)]
    ▼
Observation
    │
    │ Convert to Measurement
    ▼
Measurement
    │
    │ Report
    ▼
MetricReader
    │
    │ Aggregate measurements
    ▼
MetricReader (on interval)
    │
    │ Export metrics
    ▼
MetricExporter
    │
    │ Format metrics (OTLP)
    ▼
Backend (Collector/Prometheus)

═══════════════════════════════════════════════════════════════════════════════════
                            METRIC STRUCTURE
═══════════════════════════════════════════════════════════════════════════════════

resource_metrics [array]
│
└─── resource_metric
     │
     ├─── resource
     │    └─── attributes
     │         ├─── telemetry.sdk.language: "python"
     │         ├─── host.name: "ebdcf73e98c0"
     │         ├─── service.name: "app.py"
     │         └─── service.version: "0.1"
     │
     └─── scope_metrics [array]
          │
          └─── scope_metric
               │
               ├─── scope
               │    ├─── name: "app.py"
               │    ├─── version: "0.1"
               │    └─── schema_url: ""
               │
               └─── metrics [array]
                    │
                    └─── metric
                         │
                         ├─── name: "process.memory.usage"
                         ├─── description: "total amount of memory used"
                         ├─── unit: "By"
                         │
                         └─── data
                              │
                              ├─── data_points [array]
                              │    │
                              │    └─── data_point
                              │         ├─── attributes: {}
                              │         ├─── start_time_unix_nano: 1713351214657218000
                              │         ├─── time_unix_nano: 1713351214657419300
                              │         └─── value: 673140736
                              │
                              ├─── aggregation_temporality: 2 (DELTA)
                              │    • 1 = CUMULATIVE (sum since start)
                              │    • 2 = DELTA (sum since last collection)
                              │
                              └─── is_monotonic: false
                                   • true for counters
                                   • false for gauges, histograms, up-down counters

═══════════════════════════════════════════════════════════════════════════════════
                    INSTRUMENT SELECTION DECISION TREE
═══════════════════════════════════════════════════════════════════════════════════

What are you measuring?
    │
    ├─ Is it an EVENT/OPERATION in your code?
    │  │
    │  ├─ YES → SYNCHRONOUS INSTRUMENT
    │  │        │
    │  │        ├─ Only increases?
    │  │        │  └─→ Counter
    │  │        │      Examples: http_requests_total, errors_total
    │  │        │
    │  │        ├─ Can increase/decrease?
    │  │        │  └─→ UpDownCounter
    │  │        │      Examples: active_connections, queue_size
    │  │        │
    │  │        └─ Distribution of values?
    │  │           └─→ Histogram
    │  │              Examples: latency, response_size
    │  │
    │  └─ NO → ASYNCHRONOUS INSTRUMENT
    │           │
    │           ├─ Only increases?
    │           │  └─→ Observable Counter
    │           │      Examples: system_bytes_sent_total
    │           │
    │           ├─ Can increase/decrease?
    │           │  └─→ Observable UpDownCounter
    │           │      Examples: memory_usage, active_threads
    │           │
    │           └─ Current state (non-additive)?
    │              └─→ Observable Gauge
    │                 Examples: cpu_utilization, temperature

═══════════════════════════════════════════════════════════════════════════════════
                    PROVIDER REGISTRATION FLOW
═══════════════════════════════════════════════════════════════════════════════════

Application Starts
    │
    ├─ SDK Configured?
    │  │
    │  ├─ NO → NoOpMeterProvider (Default)
    │  │        │
    │  │        └─→ Metrics API (No telemetry produced)
    │  │
    │  └─ YES → Create MeterProvider
    │            │
    │            ├─ Configure Provider:
    │            │  • Resource (service metadata)
    │            │  • MetricReaders (list)
    │            │  • Views (customize metrics)
    │            │
    │            ├─ Register Provider:
    │            │  metrics.set_meter_provider(provider)
    │            │
    │            └─→ Replace NoOpMeterProvider
    │                 │
    │                 └─→ Metrics API (Active telemetry)

═══════════════════════════════════════════════════════════════════════════════════
                    KEY TAKEAWAYS
═══════════════════════════════════════════════════════════════════════════════════

1. Three-Layer Separation: Application → API → SDK provides clean separation
2. NoOpMeterProvider Safety: Default provider prevents errors when SDK isn't configured
3. Provider Registration: MeterProvider must be registered to replace NoOpMeterProvider
4. Instruments vs Spans: Instruments are reusable objects, spans are one-time events
5. Sync vs Async: Two distinct patterns - sync for operations, async for system state
6. Measurements: Intermediate data structure between instruments and metrics
7. MetricReader: Handles periodic collection and aggregation
8. Views: Unique to metrics for customizing how instruments become metrics
9. Resource Attributes: Applied to all metrics from a service
10. Aggregation: Metrics aggregate measurements over time periods

═══════════════════════════════════════════════════════════════════════════════════
```

## Related Documentation

- **`metricsArchitecture.md`**: Detailed documentation explaining the metrics architecture
- **`openTelemetryArchitecture.md`**: Overall OpenTelemetry architecture (Specification, SDK, API)
- **`telemetryDataFlow.md`**: How telemetry flows from applications to backends
- **`tracingArchitecture.md`**: Tracing architecture (compare similarities and differences)
