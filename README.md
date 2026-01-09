# LFS148 - OpenTelemetry Exercise Repository

> **A comprehensive learning codebase for Linux Foundation's LFS148 course on OpenTelemetry observability**

## About

This repository contains structured, hands-on exercises for learning OpenTelemetry observability. It provides practical examples of distributed tracing, metrics collection, and logging with OpenTelemetry across multiple programming languages and frameworks.

## Repository Structure

```text
LFS148-code/
├── exercises/                    # Hands-on exercises
│   ├── automatic-instrumentation/ # Zero-code instrumentation
│   ├── manual-instrumentation-traces/
│   ├── manual-instrumentation-metrics/
│   ├── manual-instrumentation-logs/
│   ├── collector/                 # OTel Collector configuration
│   └── otel-in-action/           # Full-stack observability
├── memory-bank/                   # Comprehensive documentation
│   ├── architecture/              # Architecture deep-dives
│   ├── infographic/               # Visual guides
│   └── *.md                       # Context and progress docs
└── docs/                          # Additional documentation
    └── MULTIPLE_GITHUB_ACCOUNTS_GUIDE.md
```

## Learning Path

### 1. Automatic Instrumentation

Learn zero-code instrumentation using OpenTelemetry agents:

- Java (Spring Boot) with OpenTelemetry Java Agent
- Python (Flask) with `opentelemetry-instrument`
- Environment variable configuration

### 2. Manual Instrumentation

Implement programmatic instrumentation:

- **Traces**: Distributed tracing with spans and context propagation
- **Metrics**: Counters, gauges, and histograms
- **Logs**: Structured logging with trace correlation

### 3. OpenTelemetry Collector

Configure and deploy the OTel Collector:

- Receivers, processors, and exporters
- Pipeline configuration
- Multi-backend routing (Jaeger, Prometheus)

### 4. Full Stack Observability

Build complete observability pipelines:

- Multi-service microservices architecture
- End-to-end tracing across services
- Metrics aggregation and visualization

## Memory Bank Documentation

This repository includes a comprehensive **Memory Bank** (`memory-bank/`) containing detailed documentation:

### Architecture Guides

- **[Collector Architecture](memory-bank/architecture/collectorArchitecture.md)** - Complete guide to OpenTelemetry Collector
- **[Tracing Architecture](memory-bank/architecture/tracingArchitecture.md)** - Distributed tracing concepts and implementation
- **[Metrics Architecture](memory-bank/architecture/metricsArchitecture.md)** - Metrics collection and the Four Golden Signals
- **[Logging Architecture](memory-bank/architecture/loggingArchitecture.md)** - Structured logging with trace correlation
- **[OpenTelemetry Architecture](memory-bank/architecture/openTelemetryArchitecture.md)** - Overall OTel architecture overview
- **[Telemetry Data Flow](memory-bank/architecture/telemetryDataFlow.md)** - End-to-end data flow visualization

### Specialized Guides

- **[Auto-Instrumentation Java Guide](memory-bank/autoInstrumentationJava.md)** - Deep dive into Java bytecode instrumentation
- **[System Patterns](memory-bank/systemPatterns.md)** - Common patterns and best practices
- **[Technical Context](memory-bank/techContext.md)** - Technology stack and setup instructions

### Project Context

- **[Project Brief](memory-bank/projectbrief.md)** - Project overview and objectives
- **[Product Context](memory-bank/productContext.md)** - User experience goals
- **[Active Context](memory-bank/activeContext.md)** - Current work focus
- **[Progress](memory-bank/progress.md)** - Implementation status and roadmap

### Visual Resources

- **[Telemetry Layers Infographic](memory-bank/infographic/)** - Visual representation of OTel layers

## Additional Documentation

### Multiple GitHub Accounts Setup

If you're working with multiple GitHub accounts, see our comprehensive guide:

- **[Complete Guide: Managing Multiple GitHub Accounts](docs/MULTIPLE_GITHUB_ACCOUNTS_GUIDE.md)**
  - SSH key setup for multiple accounts
  - Git configuration management
  - Daily usage examples
  - Troubleshooting guide

## Quick Start

### Prerequisites

- Docker and Docker Compose
- Java 11+ (for Spring Boot exercises)
- Python 3.8+ (for Flask exercises)

### Running an Exercise

1. Navigate to an exercise directory:

   ```bash
   cd exercises/automatic-instrumentation/initial
   ```

2. Start the services:

   ```bash
   docker-compose up
   ```

3. Access the observability backends:

   - **Jaeger UI**: <http://localhost:16686>
   - **Prometheus UI**: <http://localhost:9090>

4. Generate traffic and observe telemetry data

## Technologies Used

- **OpenTelemetry**: Observability framework
- **Java/Spring Boot**: Backend services
- **Python/Flask**: Web applications
- **OpenTelemetry Collector**: Telemetry processing
- **Jaeger**: Distributed tracing backend
- **Prometheus**: Metrics backend
- **Docker**: Containerization

## Resources

- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [Linux Foundation Training](https://training.linuxfoundation.org/)
- [Jaeger Documentation](https://www.jaegertracing.io/docs/)
- [Prometheus Documentation](https://prometheus.io/docs/)

## License

See [LICENSE](LICENSE) file for details.

## Contributing

This is a learning repository. Feel free to:

- Complete exercises and experiment
- Review the memory bank documentation
- Share your learnings and improvements

---

**Note**: This repository includes extensive documentation in the `memory-bank/` directory. These documents provide deep technical insights, architecture explanations, and learning resources beyond the basic exercise code.

