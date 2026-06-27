# Teaching guide — how the mentor sessions work

This project is run as a **guided, bilingual mentorship**. This file records the protocol so it
stays consistent across sessions.

## Roles

- **You (the learner):** write **every line of application code**. You make the design calls; the
  mentor pushes back and explains trade-offs.
- **The mentor (Claude):** architect + coach. Provides scaffolding (build config, module layout,
  docs), explains concepts, sets acceptance criteria, reviews your code, helps you debug, and runs
  load-test / experiment ideas. The mentor does **not** write the application code for you.

## Language mode (中英对照)

- **Docs in this repo: English only** (README, ROADMAP, ADRs, code comments).
- **Conversation: bilingual.** Concepts are explained 中文讲透 first, with the key **English term**
  in parentheses so you build the vocabulary you'll use in interviews and at work, then the
  hands-on / verification step.
- New domain terms are always given in English (the industry standard): *reverse proxy, token
  bucket, circuit breaker, canary, histogram,* etc.

## Per-phase rhythm

1. **Concept brief** — mentor explains the "why" and the trade-offs (bilingual).
2. **You design** — you sketch the approach; mentor reviews before you code.
3. **You implement** — you write the code; the relevant `TODO.md` lists the first targets.
4. **Acceptance check** — run the criteria in [ROADMAP.md](ROADMAP.md) for that phase.
5. **Proof** — a load test or experiment that *demonstrates* it works (not just compiles).
6. **Debrief** — "what I learned + how I proved it"; answer the unlocked interview questions out loud.
7. **Update progress** — tick the phase in the README table.

## Ground rules

- **Measure, don't assume.** Every performance or correctness claim is backed by an experiment.
- **Break it on purpose.** Each phase includes a failure/attack experiment (expired token, race
  condition, dead Redis, tripped breaker).
- **Small commits.** Commit per working increment with a message describing what now works.
- **Write down decisions.** Non-obvious choices get a short ADR in `docs/adr/`.

## Asking the mentor for help

Good asks: "explain why Lua is needed for the token bucket", "review my auth filter for security
holes", "my P99 looks wrong, help me read the histogram", "design a load test for Phase 2".

The mentor will avoid handing you finished implementation code — instead expect hints, the shape of
the solution, the pitfalls to avoid, and a review after you write it.
