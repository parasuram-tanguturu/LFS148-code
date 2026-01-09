# Product Context: LFS148 OpenTelemetry Learning Repository

## Why This Project Exists

This repository exists to provide **hands-on, practical exercises** for learning OpenTelemetry observability. Traditional learning materials often focus on theory, but observability requires practical experience with instrumentation, configuration, and visualization tools.

## Problems It Solves

### Educational Challenges

1. **Theory vs. Practice Gap**: Students need working code examples to understand OpenTelemetry concepts
2. **Setup Complexity**: OpenTelemetry requires multiple components (SDKs, collectors, backends) that are difficult to configure from scratch
3. **Learning Progression**: Students need structured exercises that build complexity gradually
4. **Reference Solutions**: Learners need working solutions to compare against their implementations

### Technical Challenges

1. **Instrumentation Approaches**: Demonstrates both automatic (zero-code) and manual (programmatic) instrumentation
2. **Multi-Language Support**: Shows OpenTelemetry patterns in both Java and Python ecosystems
3. **End-to-End Pipelines**: Provides complete observability stacks from applications to visualization tools
4. **Microservices Complexity**: Teaches distributed tracing across multiple services

## How It Works

### Exercise Structure

Each exercise follows a **progressive learning model**:

1. **Initial State** (`initial/` directory):
   - Partial implementation or starter code
   - Students complete the instrumentation
   - Includes setup instructions and hints

2. **Solution State** (`solution/` directory):
   - Complete working implementation
   - Students can reference when stuck
   - Demonstrates best practices

### Learning Flow

```
Student starts with initial/ → Implements solution → Compares with solution/ → Learns patterns
```

### Exercise Categories

1. **Automatic Instrumentation**
   - Zero-code approach using agents
   - Quick setup, minimal changes
   - Demonstrates OpenTelemetry's ease of use

2. **Manual Instrumentation**
   - Programmatic control over instrumentation
   - Custom spans, metrics, and logs
   - Fine-grained observability

3. **Collector Configuration**
   - Data routing and processing
   - Multi-backend support
   - Production-like patterns

4. **Full Stack (OTel in Action)**
   - Complete microservices observability
   - Multiple services and backends
   - Real-world scenario simulation

## User Experience Goals

### For Students

- **Clear Starting Point**: Initial code provides context without being overwhelming
- **Progressive Difficulty**: Exercises build complexity naturally
- **Immediate Feedback**: Docker-based setup allows quick testing
- **Visual Learning**: Integration with Jaeger and Prometheus provides visual feedback
- **Self-Paced Learning**: Can work through exercises independently

### For Instructors

- **Consistent Structure**: All exercises follow same pattern
- **Complete Solutions**: Reference implementations available
- **Docker-Based**: Easy to run and demonstrate
- **Isolated Exercises**: Each exercise is self-contained

## Value Proposition

### For Learners

- **Practical Skills**: Hands-on experience with real OpenTelemetry tools
- **Production Patterns**: Learn patterns used in actual systems
- **Multi-Language**: Understand OTel across different ecosystems
- **Complete Stack**: See full observability pipeline in action

### For Organizations

- **Skilled Engineers**: Trains developers in modern observability practices
- **Standardized Approach**: OpenTelemetry provides vendor-neutral skills
- **Production Ready**: Patterns translate directly to production use

## Success Metrics

A successful learning experience means students can:

- ✅ Instrument applications without assistance
- ✅ Configure OpenTelemetry Collector
- ✅ Visualize traces and metrics
- ✅ Understand distributed tracing concepts
- ✅ Apply patterns to their own projects

## User Journey

1. **Discovery**: Student finds exercise relevant to current learning topic
2. **Setup**: Clones repository, navigates to exercise directory
3. **Exploration**: Reviews initial code, understands requirements
4. **Implementation**: Completes instrumentation following instructions
5. **Testing**: Runs Docker Compose, verifies instrumentation works
6. **Visualization**: Views traces in Jaeger, metrics in Prometheus
7. **Comparison**: Reviews solution to understand best practices
8. **Application**: Applies learned patterns to next exercise or own project
