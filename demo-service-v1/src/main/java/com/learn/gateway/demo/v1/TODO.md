# demo-service-v1 — your code goes here

Package: `com.learn.gateway.demo.v1`

## Phase 0 tasks (you write all of this)

1. `DemoV1Application.java` — `@SpringBootApplication` + `main`.
2. `application.yml` — `server.port=8081`, `spring.application.name=demo-service-v1`.
3. A `@RestController` with at least:
   - `GET /hello` → returns JSON like `{"service":"demo","version":"v1","message":"..."}`.
   - `GET /whoami` → returns which version answered (used later to SEE gray routing).
4. Acceptance: `curl http://localhost:8081/hello` shows `"version":"v1"`.

Delete this TODO once your first class exists.
