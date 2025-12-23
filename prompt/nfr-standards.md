# 🏗️ Hardcore Non-Functional Requirements (NFR) Framework Standards

## 1. Identity & Security
- **Isolation**: Every design must specify how data is isolated between different entities or tenants.
- **Privacy**: PII and sensitive data must be handled using industry-standard masking or encryption techniques at the earliest possible stage.
- **Auth**: Use standardized authentication and authorization patterns.

## 2. Observability & Maintainability
- **Telemetry**: Systems must be designed to expose health checks, metrics, and logs in standard formats (e.g., Prometheus-compatible).
- **Tracing**: Complex asynchronous flows must include correlation IDs or Trace IDs for debugging.

## 3. Reliability & Scale
- **Decoupling**: Time-intensive or heavy operations must be decoupled via asynchronous patterns (queues, streams).
- **Resilience**: Designs must include error handling strategies (retries, circuit breakers, fallbacks).
- **Idempotency**: All side-effect-producing operations must be designed to be idempotent.

## 4. Operational Excellence
- **Resource Management**: Designs must account for cost, performance, and resource limits.
- **Auditability**: Critical state changes must be recorded in an audit trail.
