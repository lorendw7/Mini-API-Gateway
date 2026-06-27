# Mini API Gateway

A **production-grade API Gateway built from scratch**, as a guided learning project.
The goal is to cross from *"I can write REST endpoints with Spring Boot"* to
*"I can design and implement infrastructure."*

> **Learning contract:** the author writes **every line of application code** by hand.
> This repository's scaffolding (Maven layout, build config, docs) exists to support
> that — not to replace it. Each module ships with a `TODO.md` describing exactly what
> *you* implement next.

---

## What you will build

Every external request flows through this pipeline:

```
                        ┌─────────────────────────── gateway-server ───────────────────────────┐
client ──▶  request ──▶ │ JWT auth ─▶ rate limit ─▶ gray router ─▶ resilience ─▶ proxy/forward  │ ──▶ demo-service-v1 (stable)
                        │   401          429        v1 / v2        timeout/CB                    │ ──▶ demo-service-v2 (canary)
                        └───────────────────────────────────────────────────────────────────────┘
                                          │ trace-id injected, metrics emitted
                                          ▼
                               Prometheus  ──▶  Grafana (QPS · P99 · reject-rate)
```

- **JWT authentication** — invalid requests are rejected with `401` before anything else runs.
- **Rate limiting** — Redis + Lua token bucket; over-limit requests get `429`. Rules are pushed
  dynamically via Nacos with **no restart**.
- **Gray / canary routing** — split traffic to `v1` (stable) and `v2` (canary) by header tag or
  traffic percentage, adjustable at runtime via an admin endpoint (seconds to take effect).
- **Resilience** — timeout, retry with backoff + jitter, circuit breaker, bulkhead (Resilience4j).
- **Observability** — Prometheus metrics (QPS, P99 latency, reject rate), a Grafana dashboard, and a
  Trace ID injected at the gateway and propagated through every downstream service.
- **Reusable, published starter** — the rate limiter is extracted into a Spring Boot Starter and
  **published to a Maven repository**, so any project adopts it by coordinates alone (one
  dependency), just like an official Spring component.

By the end you can argue, from first-hand experience, about rate-limiting algorithm trade-offs,
JWT security boundaries, and gray-release rollback strategies — because you wrote, broke, and
load-tested all of it.

---

## Tech stack

| Area | Choice |
|------|--------|
| Language / build | Java 21, Maven (multi-module, Maven Wrapper) |
| Framework | Spring Boot 3.4.x, Spring Cloud 2024.0.x (Spring Cloud Gateway, reactive WebFlux) |
| Rate limiting | Redis + Lua, then extracted as a Spring Boot Starter |
| Dynamic config | Nacos |
| Resilience | Resilience4j |
| Observability | Micrometer → Prometheus, Grafana, OpenTelemetry / W3C `traceparent` |
| Testing | JUnit 5, Testcontainers, reactor-test, load tests (k6 / JMeter) |

**Gateway approach — ladder:** Phase 0 starts with a hand-written reactive reverse proxy (raw
WebFlux + `WebClient`) so the mechanics are clear, then refactors onto Spring Cloud Gateway.

---

## Module layout

```
mini-api-gateway/
├── pom.xml                          # parent / aggregator: versions, BOM imports, module list
├── mvnw / mvnw.cmd / .mvn/          # Maven Wrapper — no global Maven install needed
├── gateway-common/                  # shared types: trace context, error model, header constants
├── gateway-server/                  # THE gateway (hand-written proxy → Spring Cloud Gateway)
├── demo-service-v1/                 # stable downstream backend (port 8081)
├── demo-service-v2/                 # canary downstream backend (port 8082)
├── ratelimit-spring-boot-starter/   # Phase 7: extracted rate limiter, published to a Maven repo
└── docs/
    ├── ROADMAP.md                   # the full 9-phase curriculum, acceptance criteria, interview Qs
    └── teaching-guide.md            # how the bilingual mentor sessions work
```

---

## Prerequisites

- **JDK 21** (already present: `java -version` → 21).
- **Maven** is *not* required globally — use the bundled wrapper (`./mvnw`).
- Later phases add: **Redis** (Phase 2), **Nacos** (Phase 3), **Prometheus + Grafana** (Phase 6).
  Docker Compose files for these are introduced when each phase needs them.

---

## Getting started

```bash
# from the repo root
./mvnw -B validate        # sanity-check the reactor (works even before you write code)
./mvnw -B compile         # compiles once you have started writing classes
```

> On Windows PowerShell use `.\mvnw.cmd` instead of `./mvnw`.

Then open [`docs/ROADMAP.md`](docs/ROADMAP.md) and start **Phase 0**. Each module's `TODO.md`
tells you the first class to write.

### Phase 0 quick goal

1. Write `DemoV1Application` + a `/hello` endpoint on port `8081` (`demo-service-v1`).
2. Write `DemoV2Application` + a `/hello` endpoint on port `8082` (`demo-service-v2`).
3. Write `GatewayServerApplication` + a hand-written reactive reverse proxy on port `8080`
   (`gateway-server`) that forwards `/**` to `demo-service-v1`.
4. Verify: `curl http://localhost:8080/hello` returns v1's response **through** the gateway.

---

## Progress

| Phase | Topic | Status |
|------:|-------|--------|
| 0 | Foundations & hand-written reverse proxy | ☐ |
| 1 | JWT authentication (401) | ☐ |
| 2 | Rate limiting (Redis + Lua, 429) | ☐ |
| 3 | Dynamic config (Nacos) | ☐ |
| 4 | Gray / canary routing | ☐ |
| 5 | Resilience (Resilience4j) | ☐ |
| 6 | Observability (Prometheus / Grafana / tracing) | ☐ |
| 7 | Extract & publish rate-limit Spring Boot Starter (to Maven) | ☐ |
| 8 | Hardening, load testing & chaos | ☐ |

See [`docs/ROADMAP.md`](docs/ROADMAP.md) for the detailed plan, acceptance criteria, and the
interview questions each phase unlocks.

## License

See [LICENSE](LICENSE).
