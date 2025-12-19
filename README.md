
# Spring-Service-Mesh-Ready-Application

## Overview

This project is a **multi-microservice demonstration** of **service mesh-ready applications** built with **Spring Boot 3.x**. It showcases how to design and configure Spring Boot services to work seamlessly with modern service meshes like **Istio** and **Envoy** — enabling advanced traffic management, security, observability, and resilience without modifying application code.

The focus is on **best practices for mesh integration**: proper health endpoints, standardized metrics, distributed tracing propagation, sidecar compatibility, and non-blocking I/O considerations.

## Real-World Scenario (Simulated)

In cloud-native environments using **Istio** or other Envoy-based service meshes:
- Applications run as sidecar proxies (Envoy) intercept all inbound/outbound traffic.
- The mesh provides mTLS, traffic routing (canary, mirroring), circuit breaking, retries, timeouts, fault injection, and observability.
- Applications must expose correct health, metrics, and tracing headers for the mesh to function properly.
- Services should avoid hardcoding network logic and be ready for dynamic routing and security policies.

We simulate a typical e-commerce flow (Product → Order → Payment) with services fully prepared for Istio injection: proper liveness/readiness, Prometheus metrics, OpenTelemetry tracing headers, and graceful shutdown.

## Microservices Involved

| Service                | Responsibility                                                                 | Port  |
|------------------------|--------------------------------------------------------------------------------|-------|
| **eureka-server**      | Service discovery (Netflix Eureka) — optional, shows coexistence with mesh     | 8761  |
| **product-service**    | Manages product catalog, exposes mesh-ready endpoints                          | 8081  |
| **order-service**      | Orchestrates orders, calls downstream via REST                                 | 8082  |
| **payment-service**    | Processes payments, simulates external integration                             | 8083  |

All services are **sidecar-ready**: no host/port hardcoding, proper HTTP headers, health endpoints, and metrics.

## Tech Stack

- Spring Boot 3.x
- Spring Web/WebFlux (configurable — shows both blocking and non-blocking)
- Spring Boot Actuator (liveness, readiness, Prometheus metrics)
- Micrometer + Prometheus format
- OpenTelemetry (or Sleuth) for tracing header propagation
- Spring Cloud OpenFeign (with mesh-aware headers)
- Spring Cloud Netflix Eureka (coexists with mesh discovery)
- Lombok
- Maven (multi-module)
- Docker & Docker Compose
- Kubernetes + Istio manifests (provided for real deployment)

## Docker Containers

```yaml
services:
  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"

  product-service:
    build: ./product-service
    depends_on:
      - eureka-server
    ports:
      - "8081:8081"
    environment:
      - MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE=health,info,metrics,prometheus
      - MANAGEMENT_ENDPOINT_HEALTH_PROBES_ENABLED=true  # Kubernetes probes

  order-service:
    build: ./order-service
    depends_on:
      - eureka-server
    ports:
      - "8082:8082"

  payment-service:
    build: ./payment-service
    depends_on:
      - eureka-server
    ports:
      - "8083:8083"
```

Run with: `docker-compose up --build`

## Service Mesh Readiness Features

| Feature                        | Implementation Details                                                  |
|--------------------------------|-------------------------------------------------------------------------|
| **Kubernetes Probes**          | `/actuator/health/liveness` and `/actuator/health/readiness`           |
| **Prometheus Metrics**         | `/actuator/prometheus` with JVM, HTTP, and custom metrics               |
| **Tracing Headers**            | OpenTelemetry/Sleuth auto-propagates `traceparent`, `tracestate`, B3    |
| **Sidecar Compatibility**      | No localhost assumptions, no socket binding, graceful shutdown         |
| **mTLS Ready**                 | Uses HTTP (mesh terminates TLS), no cert management in app             |
| **Envoy Header Forwarding**    | Accepts `x-request-id`, `x-forwarded-for`, etc.                         |
| **Non-blocking Option**        | Configurable WebFlux version for high concurrency                      |

## Key Features

- Kubernetes-native liveness/readiness probes
- Prometheus-compatible metrics endpoint
- Distributed tracing header propagation
- Graceful shutdown and startup hooks
- No hard-coded ports/hosts (uses discovery)
- Custom business metrics for mesh observability
- Istio manifest examples (VirtualService, DestinationRule, Gateway)
- Sidecar injection ready (no port conflicts)

## Expected Endpoints (All services)

| Endpoint                              | Purpose (Mesh Usage)                             |
|---------------------------------------|--------------------------------------------------|
| GET `/actuator/health/liveness`       | Liveness probe — container restart if DOWN       |
| GET `/actuator/health/readiness`      | Readiness probe — traffic routing control        |
| GET `/actuator/prometheus`            | Scrape target for Prometheus + Istio telemetry   |
| GET `/actuator/metrics`               | Detailed JVM and custom metrics                  |
| GET `/api/*`                          | Business endpoints — traffic managed by mesh     |

## Architecture Overview (With Istio)

```
Clients
   ↓
Istio Ingress Gateway
   ↓
Envoy Sidecar (per pod)
   ↓
Spring Boot Service (Product/Order/Payment)
   ├── Actuator → health/metrics → Envoy → Prometheus/Kiali
   └── Tracing headers → Envoy → Jaeger/Tempo
```

**Mesh Benefits Enabled**:
- mTLS between services
- Traffic shifting (canary)
- Circuit breaking/retries
- Fault injection testing
- Observability dashboards (Kiali, Jaeger)

## How to Run

### Local (Without Istio)
1. `docker-compose up --build`
2. Access Eureka: `http://localhost:8761`
3. Test health: `curl http://localhost:8081/actuator/health/readiness`
4. Scrape metrics: `curl http://localhost:8081/actuator/prometheus`

### With Istio (Kubernetes)
1. Deploy provided `k8s/` manifests
2. Enable automatic sidecar injection (`istio-injection=enabled`)
3. Deploy services → Envoy sidecars injected
4. Use Kiali → visualize service graph
5. Use Jaeger → trace requests across services
6. Apply VirtualService → test canary routing

## Testing Mesh Readiness

1. Local: health endpoints return correct status
2. Prometheus scrapes `/prometheus` successfully
3. Tracing headers propagated in logs
4. In Istio: traffic flows, mTLS active, metrics in Prometheus
5. Apply fault injection → service resilient
6. Scale order-service → traffic distributed

## Skills Demonstrated

- Designing applications for service mesh integration
- Kubernetes liveness/readiness best practices
- Exposing Prometheus metrics correctly
- Distributed tracing header propagation
- Graceful lifecycle management
- Coexistence of traditional discovery (Eureka) with mesh
- Production cloud-native application patterns

## Provided Istio Resources (k8s/)

- Deployment with sidecar injection
- VirtualService (routing, timeouts)
- DestinationRule (circuit breaker, outlier detection)
- Gateway (ingress)
- ServiceEntry (external calls if needed)

## Future Extensions

- Full OpenTelemetry instrumentation
- Istio authorization policies
- Rate limiting with Envoy
- Mutual TLS client certificates
- WebSocket support through mesh
- Chaos testing with Istio fault injection

