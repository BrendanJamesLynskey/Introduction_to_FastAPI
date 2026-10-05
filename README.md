# ⚡ Introduction to FastAPI

An interactive Reveal.js presentation covering **FastAPI** — the modern Python web framework where type hints become the API. Pydantic v2 validation, Starlette / ASGI, `Depends()` injection, automatic OpenAPI 3.1, async-first with a graceful sync escape hatch.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_FastAPI/)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | Pitch and the type → validate → inject → async → document → deploy flow |
| 02 | Topics | Map of foundations, behaviour, integration, production |
| 03 | What Is FastAPI? | Origins, scope, where it sits in the stack, who uses it |
| 04 | Why FastAPI? | Type-driven design, performance, ergonomics, honest trade-offs |
| 05 | Installation | `fastapi[standard]`, the first endpoint, project layout |
| 06 | Path operations | Path / query / body / header / cookie params, response models |
| 07 | Pydantic v2 | Models, field validators, model validators, serialisation, common types |
| 08 | Dependency injection | `Depends()`, sub-deps, `yield` cleanup, attaching at handler / router / app |
| 09 | Async vs sync | When each runs, classic foot-guns, CPU-bound work, threadpool |
| 10 | Authentication | OAuth2 password / JWT, scopes, API keys, what FastAPI gives you vs you bring |
| 11 | Errors & middleware | HTTPException, custom handlers, request-ID + timing, CORS, TrustedHost |
| 12 | Background, streaming, WebSockets | `BackgroundTasks`, SSE / NDJSON, WS, LLM streaming |
| 13 | Database | SQLAlchemy 2.0 async, SQLModel, Alembic migrations, repository pattern |
| 14 | Configuration | Pydantic Settings — typed env / .env, secrets, dependency overrides for tests |
| 15 | Testing | `TestClient`, async tests with httpx, the killer `dependency_overrides` |
| 16 | OpenAPI | Tags, summaries, custom schema, client generation, hide internal routes |
| 17 | Lifespan | `asynccontextmanager`, app.state, health vs ready, graceful shutdown |
| 18 | Performance | Workers vs concurrency, loop blocking, `orjson`, profiling |
| 19 | Deployment | Dockerfile, K8s probes, where to host, things not to ship |
| 20 | Observability | structlog, Prometheus, OpenTelemetry, hygiene |
| 21 | Security hardening | Required middleware, headers, body limits, rate limiting, hygiene |
| 22 | FastAPI vs Flask, Django REST, Litestar, Starlette | Capability matrix and verdicts |
| 23 | Production recipes | Repository + service shape, outbox, versioned routers, feature flags |
| 24 | Summary | Take-aways, checklist, next steps, further reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Speaker notes | `S` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Outfit + Plus Jakarta Sans + Fira Code

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

FastAPI documentation — fastapi.tiangolo.com · FastAPI source — github.com/fastapi/fastapi · Pydantic — docs.pydantic.dev · Starlette — starlette.io · Uvicorn — uvicorn.org · Pydantic Settings — docs.pydantic.dev/latest/concepts/pydantic_settings · OpenAPI Specification — openapis.org · OpenTelemetry — opentelemetry.io · structlog — structlog.org · prometheus-fastapi-instrumentator — github.com/trallnag/prometheus-fastapi-instrumentator

## License

Educational use. Code examples provided as-is.
