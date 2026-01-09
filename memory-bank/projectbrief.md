# Project Brief: LFS148 OpenTelemetry Exercise Repository

## Overview

This repository is a learning codebase for **LFS148** (Linux Foundation course) focused on OpenTelemetry observability. It serves as a comprehensive exercise repository for students learning distributed tracing, metrics collection, and logging with OpenTelemetry.

## Purpose

The repository provides structured, hands-on exercises that teach:

- **Automatic Instrumentation**: Zero-code instrumentation using OpenTelemetry agents
- **Manual Instrumentation**: Programmatic instrumentation using OpenTelemetry SDKs
- **OpenTelemetry Collector**: Configuration and deployment of the OTel Collector
- **Full Stack Observability**: Complete microservices observability implementation

## Learning Objectives

1. Understand OpenTelemetry concepts and architecture
2. Implement automatic instrumentation for Java (Spring Boot) and Python (Flask) applications
3. Implement manual instrumentation for traces, metrics, and logs
4. Configure and deploy OpenTelemetry Collector
5. Integrate observability data with backends (Jaeger, Prometheus)
6. Build end-to-end observability pipelines for microservices

## Repository Structure

The repository follows a consistent exercise pattern:

```
exercises/
├── automatic-instrumentation/
│   ├── initial/          # Starter code for students
│   └── solution/         # Complete solution reference
├── manual-instrumentation-traces/
├── manual-instrumentation-metrics/
├── manual-instrumentation-logs/
├── collector/
└── otel-in-action/       # Comprehensive full-stack exercise
```

Each exercise directory contains:

- `initial/`: Starting point code with partial implementation
- `solution/`: Complete working solution for reference

## Target Audience

- Students enrolled in LFS148 course
- Developers learning OpenTelemetry observability
- Engineers implementing observability in microservices architectures

## Success Criteria

Students should be able to:

- Instrument applications automatically and manually
- Configure OpenTelemetry Collector for data routing
- Visualize traces in Jaeger
- Query metrics in Prometheus
- Understand distributed tracing across microservices
- Correlate logs, traces, and metrics

## Project Scope

**In Scope:**

- Exercise code and solutions
- Docker-based development environment
- Sample microservices applications (Todo app)
- OpenTelemetry Collector configurations
- Integration with observability backends

**Out of Scope:**

- Course materials and lectures (external)
- Production deployment strategies
- Advanced performance optimization
- Security hardening

## Key Principles

1. **Progressive Learning**: Exercises build from simple to complex
2. **Hands-On Practice**: Students implement solutions themselves
3. **Reference Solutions**: Solutions available for comparison and learning
4. **Real-World Patterns**: Exercises reflect actual production scenarios
5. **Technology Diversity**: Covers both Java and Python ecosystems
