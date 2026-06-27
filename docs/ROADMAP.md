# Roadmap — Mini API Gateway

A 9-phase curriculum. Each phase follows the same loop:

> **build → measure → understand → interview-ready**

You write all code. For each phase: a concept brief, what *you* build, **acceptance criteria**
(objectively verifiable), a **proof step** (load test / experiment), and the **interview questions**
it unlocks. Treat acceptance criteria as a definition of done before moving on.

Legend: ☐ todo · ◐ in progress · ☑ done

---

## Phase 0 — Foundations & a hand-written reverse proxy ☐

**Concept.** A gateway is, at its core, a *reverse proxy*: it receives a client request, decides
what to do with it, forwards it to a backend, and relays the response. Before using any framework
magic, build the smallest possible version so the data flow is concrete.

**You build**
- `demo-service-v1` (port 8081) and `demo-service-v2` (port 8082): each a Spring MVC app with
  `GET /hello` and `GET /whoami` that report their own version.
- `gateway-server` (port 8080): a hand-written reactive reverse proxy on WebFlux using `WebClient`,
  forwarding `/**` to `demo-service-v1`.

**Acceptance**
- `curl :8081/hello` → `version=v1`; `curl :8082/hello` → `version=v2`.
- `curl :8080/hello` returns v1's body **through** the gateway (method, headers, body preserved).

**Proof.** Forward a `POST` with a JSON body and a custom header; confirm both arrive downstream.

**Interview unlock.** What a gateway/reverse proxy does · blocking vs non-blocking I/O · why a
gateway benefits from reactive (event loop, many idle connections) while backends may stay blocking.

---

## Phase 1 — JWT authentication (401) ☐

**Concept.** Stateless auth: the gateway validates a signed token instead of looking up a session.
Get the signature, expiry, and key handling right — this is a security boundary.

**You build**
- A tiny token issuer (for testing): sign a JWT, first with **HMAC (HS256)**, then **RS256**.
- A gateway auth filter that runs **first**: missing/invalid/expired token → `401` with a clean
  error body; valid token → attach claims (e.g. `userId`, `roles`) to the request context for
  downstream filters.
- A small allow-list of public paths (e.g. health, token issue) that skip auth.

**Acceptance**
- No token → `401`. Tampered token → `401`. Expired token → `401`. Valid token → forwarded.
- Claims are available to later filters (you'll key rate limiting on `userId` in Phase 2).

**Proof / security experiments.** Reproduce and defend against: the `alg=none` attack; the
RS256→HS256 key-confusion attack; clock-skew handling on `exp`/`nbf`.

**Interview unlock.** Stateless vs session auth · JWT revocation problem & mitigations · refresh
tokens · where keys live (JWKS) · symmetric vs asymmetric signing.

---

## Phase 2 — Rate limiting (Redis + Lua, 429) ⭐ ☐

**Concept.** Protect backends from overload. Compare algorithms, then make it correct under
concurrency and across multiple gateway instances.

**You build**
- **2a (single node, in memory):** implement and compare *fixed window*, *sliding window log*,
  *sliding window counter*, *token bucket*, and *leaky bucket*. Note each one's burst behavior and
  memory cost.
- **2b (distributed):** a **token bucket in a Redis Lua script** so the check-and-decrement is
  atomic. Key by `userId` / route / IP.
- **2c:** return `429` with a `Retry-After` header; make limits configurable per route.

**Acceptance**
- Under a configured limit of N req/s, the (N+1)th within the window gets `429`.
- Two gateway instances sharing one Redis enforce the limit **globally**, not per instance.

**Proof.** Load test with k6/JMeter at 2× the limit; confirm the allowed rate matches config and
there are **no race conditions** (atomicity holds under concurrency).

**Interview unlock.** Algorithm trade-offs (burst vs smoothness, accuracy vs memory) · why Lua
(atomicity / round-trip reduction) · distributed rate-limiting accuracy · token vs leaky bucket.

---

## Phase 3 — Dynamic config with Nacos ☐

**Concept.** Change rate-limit rules in production **without redeploying**. Config is pushed and
applied live.

**You build**
- Externalize rate-limit rules to Nacos; a listener updates in-memory rules on change.
- Sensible fail-safe defaults if Nacos is unreachable.

**Acceptance**
- Edit a limit in Nacos → new limit takes effect within seconds, no restart, no dropped traffic.

**Proof.** While a load test runs, tighten the limit in Nacos and watch the `429` rate rise live.

**Interview unlock.** Config push vs pull · long polling · config versioning & rollback · fail-open
vs fail-closed defaults.

---

## Phase 4 — Gray / canary routing ☐

**Concept.** Send a slice of traffic to a new version to de-risk releases.

**You build**
- Header-based routing (`X-Gray-Tag: v2` → v2) **and** percentage-based routing (e.g. 10% → v2).
- An admin endpoint to change the split at runtime (seconds to take effect).
- Optional: consistent hashing so a given user sticks to one version.

**Acceptance**
- With a 10% split, ~10% of requests hit v2 (verify via `/whoami`); flip to 0% to "roll back"
  instantly.

**Proof.** Drive steady traffic; change the ratio live and confirm the v1/v2 distribution shifts
within seconds.

**Interview unlock.** Canary vs blue-green · rollback strategy · sticky sessions / consistent
hashing · traffic shifting and load balancing.

---

## Phase 5 — Resilience (Resilience4j) ☐

**Concept.** A gateway must fail gracefully. One slow backend must not exhaust the gateway.

**You build**
- Per-route **timeout**, **retry** (bounded, with backoff + jitter), **circuit breaker**,
  **bulkhead** (concurrency isolation), and a **fallback** response.

**Acceptance**
- A slow/erroring downstream trips the breaker; the gateway returns a fast fallback instead of
  hanging; the breaker recovers (half-open) when the backend heals.

**Proof.** Inject latency/errors into a downstream; observe breaker state transitions and that
gateway latency stays bounded.

**Interview unlock.** Cascading failure · thundering herd · why naive retries amplify outages ·
why jitter · bulkhead isolation.

---

## Phase 6 — Observability (Prometheus / Grafana / tracing) ☐

**Concept.** You can't operate what you can't see. Expose the RED signals and trace every request.

**You build**
- Micrometer metrics → a Prometheus scrape endpoint: **QPS**, **P99 latency** (histogram),
  **reject rate** (401/429).
- A Grafana dashboard with those three curves.
- **Trace ID**: inject at the gateway, propagate downstream; upgrade to OpenTelemetry / W3C
  `traceparent` so a single trace spans gateway + services.

**Acceptance**
- Grafana shows live QPS, P99, and reject-rate curves under load.
- One client request produces one trace id visible in gateway **and** downstream logs.

**Proof.** Run a load test; read P99 off Grafana; correlate a single request across services by
its trace id.

**Interview unlock.** RED method · histogram vs summary · why P99 ≠ average · context propagation ·
sampling.

---

## Phase 7 — Extract & publish a Spring Boot Starter ☐

**Concept.** Turn the rate limiter into a reusable library any project adopts with one dependency,
then **publish it to a Maven repository** so it's consumed by coordinates alone — exactly like an
official Spring starter (`implementation 'com.example:ratelimit-spring-boot-starter:1.0.0'`).

**You build**

*7a — Extract the starter*
- `ratelimit-spring-boot-starter`: auto-configuration, `@ConditionalOn...` beans,
  `@ConfigurationProperties`, registered via
  `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`.
- Consume it from `gateway-server` with a single dependency + a few properties.

*7b — Make it publishable (release engineering)*
- Complete POM metadata required by repositories: `name`, `description`, `url`, `licenses`,
  `developers`, `scm`.
- Attach **sources** and **javadoc** jars (`maven-source-plugin`, `maven-javadoc-plugin`).
- **GPG-sign** the artifacts (`maven-gpg-plugin`).
- Adopt **semantic versioning** (`MAJOR.MINOR.PATCH`) and drop the `-SNAPSHOT` suffix for a release.

*7c — Publish*
- First publish to your **local repo**: `./mvnw install` (lands in `~/.m2`).
- Then publish to a **remote** repository — pick one and document why:
  - **Maven Central** via the Sonatype Central Portal (`central-publishing-maven-plugin`) — needs a
    namespace you own (verified reverse-domain `groupId`).
  - **GitHub Packages** (`distributionManagement` + a PAT) — simplest for a personal project.
  - A private **Nexus/Artifactory** — the company setup.

**Acceptance**
- Adding the starter dependency + properties enables rate limiting with **zero** glue code; removing
  the dependency cleanly disables it.
- After `./mvnw install`, the artifact resolves from `~/.m2` by coordinates only.
- After remote publish, the artifact resolves in a **clean environment** (empty local `.m2`) straight
  from the repository, sources/javadoc and signature present.

**Proof.** Wire the starter into a throwaway second app — pulling it **only by coordinates** (no
local module path) — and prove rate limiting works there with nothing but the dependency + properties.

**Interview unlock.** Spring auto-configuration internals · conditional beans · starter packaging ·
semantic versioning & backward compatibility · what a Maven repository requires to publish (POM
metadata, sources/javadoc, GPG signing) · SNAPSHOT vs release · Central / GitHub Packages / private
Nexus trade-offs.

---

## Phase 8 — Hardening, load testing & chaos ☐

**Concept.** Prove the whole thing under stress and failure.

**You build**
- A full load-test campaign across the pipeline; document SLOs (target P99, max reject rate).
- Chaos experiments: kill Redis, kill a downstream, kill one gateway instance.

**Acceptance**
- Documented capacity numbers (sustained QPS at target P99).
- Defined, observed behavior for each failure (e.g. Redis down → fail-open or fail-closed, your
  choice, justified).

**Proof.** The campaign report: throughput curve, P99 under load, and how each injected failure
was handled.

**Interview unlock.** Capacity planning · graceful degradation · fault injection · SLO/SLA/SLI.

---

## Cross-cutting (every phase)

- **Testing:** unit tests; **Testcontainers** for Redis/Nacos integration tests; concurrency tests
  for the rate limiter; `reactor-test` for the reactive paths.
- **Docs:** keep a short **ADR** (architecture decision record) per non-obvious choice, and a
  per-phase "what I learned + how I proved it" note.
- **Each phase ends** by updating the progress table in the [README](../README.md).
