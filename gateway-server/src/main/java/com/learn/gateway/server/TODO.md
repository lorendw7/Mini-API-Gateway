# gateway-server — your code goes here

This `target` package is `com.learn.gateway.server`. Spring Boot scans components
from the package of your `@SpringBootApplication` class downward, so put that main
class **here** (or higher).

## Phase 0 — your very first task (you write all of this)

1. Create `GatewayServerApplication.java` with `@SpringBootApplication` and a `main`
   method that calls `SpringApplication.run(...)`.
2. Add `src/main/resources/application.yml`; set `server.port=8080`.
3. Build a **hand-written reactive reverse proxy**:
   - A `@RestController` or a `WebFilter` / `RouterFunction` that catches all paths.
   - Use `org.springframework.web.reactive.function.client.WebClient` to forward the
     incoming request (method, path, headers, body) to a downstream base URL
     (e.g. `http://localhost:8081` = demo-service-v1).
   - Copy the downstream response (status, headers, body) back to the caller.
4. Acceptance: `curl http://localhost:8080/hello` returns demo-service-v1's response.

Delete this TODO file once you've created your first real class here.
