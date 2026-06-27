# gateway-common — shared types go here

Package: `com.learn.gateway.common`

This is a plain library (no Spring Boot app). Things that belong here as the project grows:

- `TraceContext` / trace-id constants (Phase 6).
- A common API error/response model (e.g. `ErrorResponse`) so the gateway returns
  consistent 401/429 bodies.
- Header name constants (`X-Request-Id`, `X-Gray-Tag`, etc.).

You don't need anything here for Phase 0 — start with the gateway and demo services.
Delete this TODO when you add your first shared type.
