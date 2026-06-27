# demo-service-v2 — your code goes here

Package: `com.learn.gateway.demo.v2`

## Phase 0 tasks (you write all of this)

1. `DemoV2Application.java` — `@SpringBootApplication` + `main`.
2. `application.yml` — `server.port=8082`, `spring.application.name=demo-service-v2`.
3. Same endpoints as v1 BUT report `"version":"v2"` so traffic splitting is visible.
4. Acceptance: `curl http://localhost:8082/hello` shows `"version":"v2"`.

Delete this TODO once your first class exists.
