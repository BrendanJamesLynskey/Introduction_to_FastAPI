# Introduction to FastAPI

Full slide content in Markdown format. Each slide is separated by `---`.

---

## Slide 01 — Title

# Introduction to FastAPI

**A modern Python framework where type hints become the API**

Pydantic v2 · Starlette / ASGI · Depends() · OpenAPI · ReDoc

> Type hints drive validation, serialisation, dependency injection and OpenAPI generation — one declaration, four behaviours, no decorators stack to learn.

Type · Validate · Inject · Async · Document · Deploy

---

## Slide 02 — Topics

### Foundations
- What FastAPI is — origins, Sebastián Ramírez, Starlette + Pydantic
- Why FastAPI — type-driven design, performance, ergonomics
- Installation, the first endpoint, `uvicorn`
- Path operations — methods, path / query / body params
- Pydantic v2 — models, validators, field types

### Behaviour
- Dependency injection with `Depends()`
- Sub-dependencies, generators, `yield` for cleanup
- Async vs sync — when each runs and where
- Authentication — OAuth2 password, JWT, API keys
- Authorisation — scopes, dependencies as policies
- Background tasks, streaming, file uploads

### Integration
- Database — SQLAlchemy 2.0 async, Tortoise, SQLModel
- Pydantic Settings — typed config from env / .env
- Middleware & lifespan events
- WebSockets & SSE
- Testing — `TestClient`, async tests, fixtures
- OpenAPI — tags, descriptions, ReDoc, custom schema

### Production
- Performance — uvicorn, gunicorn, workers, async pitfalls
- Deployment — Docker, K8s, ASGI servers compared
- Observability — logging, metrics, OpenTelemetry
- Security — CORS, HTTPS, rate limiting, request size
- Comparisons — Flask, Django REST, Litestar, Starlette
- Production patterns — lifespan, dependency overrides

---

## Slide 03 — What Is FastAPI?

FastAPI is a Python web framework created by **Sebastián Ramírez (tiangolo)** in 2018. It builds on **Starlette** (ASGI toolkit) and **Pydantic v2** (data validation in Rust) to turn type hints into HTTP behaviour.

### The two-line pitch

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    email: str
    name: str
    age: int | None = None

@app.post('/users', response_model=User, status_code=201)
async def create_user(user: User) -> User:
    return user
# → validation, deserialisation, serialisation, OpenAPI,
#   Swagger UI, ReDoc — all from the type hints alone.
```

### What FastAPI owns
- **Routing** — path operations, path / query / body params
- **Validation** — via Pydantic v2 (Rust-backed, fast)
- **Serialisation** — response models, exclude / include
- **DI** — `Depends()` with sub-deps and yield-cleanup
- **Docs** — OpenAPI 3.1, Swagger UI, ReDoc, schema export
- **WebSockets**, **SSE**, **background tasks**
- Test client (`httpx`-based)

### Where it sits in the stack

| Layer | Component |
|-------|-----------|
| App | **FastAPI** — routes, DI, OpenAPI |
| HTTP | **Starlette** — ASGI app, middleware, WebSocket |
| Validation | **Pydantic v2** — `pydantic-core` (Rust) |
| Server | Uvicorn / Hypercorn / Granian (ASGI) |
| Process mgr | Gunicorn (with uvicorn workers) / systemd |

### What it is not
Not a full-stack framework. No ORM (you bring SQLAlchemy / SQLModel / Tortoise). No template engine (you bring Jinja2). No admin / auth UI (you bring your own). It's a *web API* framework.

---

## Slide 04 — Why FastAPI?

Three claims, each backed by something real: **type-driven design**, **performance close to Node / Go**, **developer ergonomics** from one declaration doing four jobs.

### One declaration, four behaviours

```python
@app.get('/items/{item_id}')
def read_item(
    item_id: int,                          # path — coerced & validated
    q: str | None = None,                  # query — optional
    skip: int = 0, limit: int = 100,       # query — defaults
    user: User = Depends(current_user),    # injected
):
    ...

# → routing, type coercion, validation, OpenAPI param entries,
#   DI, automatic error responses with location info — one signature.
```

### Performance
- Pydantic v2 core is in Rust — validation is > 10× v1
- Async-first — one event loop, thousands of in-flight requests
- Comparable to Node / Go for I/O-bound APIs in independent benchmarks
- Sync paths run in a thread pool — mixing sync and async is graceful

### What you give up — honestly
- **No batteries** — you assemble ORM, migrations, auth from libraries
- **Async-first culture** — sync-only DBs need extra care
- **Pydantic learning curve** — v1 vs v2 differences still trip teams up
- **DI is opinionated** — `Depends()` is great, but unusual if you've come from Spring / DI containers

### When FastAPI is the right tool
- JSON / OpenAPI HTTP APIs — the original sweet spot
- ML / LLM model servers — PyTorch, transformers, vLLM
- Internal microservices in a Python house
- Anywhere you want OpenAPI as a contract for free
- Background-task / webhook receivers

### When something else fits better
- Server-rendered HTML apps with auth, admin, ORM → **Django**
- Tiny single-page proof-of-concepts → **Flask**
- Need msgspec speed and stricter type discipline → **Litestar**

---

## Slide 05 — Installation & First Endpoint

### Install

```bash
# the recommended bundle — FastAPI + uvicorn + extras
uv add 'fastapi[standard]'

# minimal — just the framework
uv add fastapi
uv add 'uvicorn[standard]'

# pip equivalent
pip install 'fastapi[standard]'
```

### The first app

```python
# app/main.py
from fastapi import FastAPI

app = FastAPI(title='Hello API', version='0.1.0')

@app.get('/healthz')
def health() -> dict:
    return {'ok': True}

@app.get('/echo/{msg}')
async def echo(msg: str) -> dict:
    return {'msg': msg}
```

### Run it

```bash
# dev: hot reload, single worker
uv run fastapi dev app/main.py
# → http://127.0.0.1:8000
# → http://127.0.0.1:8000/docs   (Swagger UI)
# → http://127.0.0.1:8000/redoc  (ReDoc)

# prod-style: uvicorn directly
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### Project layout that scales

```text
app/
├── main.py              # creates FastAPI(), includes routers
├── api/
│   ├── deps.py          # shared dependencies
│   ├── v1/
│   │   ├── users.py     # APIRouter
│   │   └── items.py
├── core/
│   ├── config.py        # pydantic Settings
│   ├── security.py
│   └── logging.py
├── schemas/             # pydantic models (DTOs)
├── models/              # SQLAlchemy ORM models
├── services/            # domain logic
├── db/                  # session, migrations
└── tests/
```

### Anatomy of a path operation
- **Decorator** — `@app.get/.post/.put/.patch/.delete` sets method & path
- **Function name** — appears as the OpenAPI `operationId`
- **Parameters** — FastAPI inspects type + default & classifies as path / query / body / dep
- **Return type** — used as `response_model` if not overridden; drives serialisation

---

## Slide 06 — Path Operations — Params & Responses

Parameters are classified by their **type** and **where they're declared**. FastAPI never guesses — it follows a single rule: `Path`, `Query`, `Body`, `Header`, `Cookie` overrides; otherwise scalar → query, model → body.

### The classification rule

```python
from fastapi import Query, Path, Header, Cookie, Body
from typing import Annotated

@app.get('/users/{user_id}/posts')
def list_posts(
    user_id: Annotated[int, Path(ge=1)],
    page:    Annotated[int, Query(ge=1)] = 1,
    size:    Annotated[int, Query(ge=1, le=100)] = 20,
    q:       Annotated[str | None, Query(max_length=64)] = None,
    x_request_id: Annotated[str | None, Header()] = None,
    session: Annotated[str | None, Cookie()] = None,
):
    ...
```

### Body — one model or many fields

```python
from pydantic import BaseModel

class ItemCreate(BaseModel):
    name: str
    price: float

@app.post('/items')
def create(item: ItemCreate):     # whole body
    ...

@app.post('/items/multi')
def create2(
    item: ItemCreate,
    user: UserStub,
    importance: Annotated[int, Body(ge=1, le=5)] = 1,
):
    # body becomes { "item": ..., "user": ..., "importance": ... }
    ...
```

### Response models & status

```python
from fastapi import status

class ItemRead(BaseModel):
    id: int
    name: str
    price: float

@app.post('/items',
          response_model=ItemRead,
          status_code=status.HTTP_201_CREATED,
          tags=['items'],
          summary='Create an item',
          response_description='The created item')
def create(item: ItemCreate) -> ItemRead:
    return ItemRead(id=42, **item.model_dump())
```

### Multiple responses, error shapes

```python
from fastapi import HTTPException

class Error(BaseModel):
    code: str
    message: str

@app.get('/items/{id}',
         response_model=ItemRead,
         responses={
           404: {'model': Error, 'description': 'Not found'},
           409: {'model': Error, 'description': 'Conflict'},
         })
def get(id: int):
    if id > 1000:
        raise HTTPException(404, detail='not found')
    return ItemRead(id=id, name='x', price=1.0)
```

### The Annotated-first style
FastAPI 0.95+ recommends `Annotated[T, Query(...)]` over `= Query(...)` default values — lets type checkers see the real type, plays nicely with stricter mypy / pyright settings, and reads better in long signatures.

---

## Slide 07 — Pydantic v2 — The Validation Engine

Pydantic v2 (released 2023) rewrote the validation core in Rust. It is what makes FastAPI fast, strict, and informative — and learning Pydantic is most of learning FastAPI.

### Models & field constraints

```python
from pydantic import BaseModel, Field, EmailStr, HttpUrl
from datetime import datetime
from decimal import Decimal
from typing import Literal

class User(BaseModel):
    id:       int
    email:    EmailStr
    name:     str = Field(min_length=1, max_length=120)
    role:     Literal['user', 'admin'] = 'user'
    avatar:   HttpUrl | None = None
    balance:  Decimal = Field(decimal_places=2)
    created:  datetime
    metadata: dict[str, str] = {}

# parse / validate
u = User.model_validate(payload)        # raises ValidationError
u = User.model_validate_json(raw_bytes) # parse JSON directly
```

### Field validators

```python
from pydantic import field_validator, model_validator

class Order(BaseModel):
    qty:  int
    unit_price: Decimal
    total: Decimal

    @field_validator('qty')
    @classmethod
    def positive(cls, v):
        if v <= 0: raise ValueError('qty must be positive')
        return v

    @model_validator(mode='after')
    def check_total(self):
        if self.total != self.qty * self.unit_price:
            raise ValueError('total mismatch')
        return self
```

### Serialisation — `model_dump`

```python
u.model_dump()              # → dict
u.model_dump(mode='json')   # JSON-compatible (Decimal/datetime as str)
u.model_dump_json()         # → bytes / str
u.model_dump(exclude={'metadata'})
u.model_dump(include={'id', 'email'})
u.model_dump(by_alias=True, exclude_none=True)
```

### Common types you'll reach for

| Type | Use |
|------|-----|
| `EmailStr` | Validated email (needs `email-validator`) |
| `HttpUrl` · `AnyHttpUrl` | URL with scheme/host parsing |
| `UUID4` · `UUID7` | UUID with version check |
| `SecretStr` | Hidden in `repr` / dump |
| `conlist` · `conint` | Constrained list / int |
| `Annotated[..., AfterValidator(f)]` | Plug a function in |
| `RootModel[list[X]]` | Top-level list / scalar |
| `Field(discriminator='kind')` | Tagged union |

---

## Slide 08 — Dependency Injection with `Depends()`

FastAPI's signature feature: anything that can be a parameter can be a **dependency**. Dependencies compose, can have their own dependencies, and can yield to clean up.

### The basic shape

```python
from fastapi import Depends

def common_paginate(page: int = 1, size: int = 20) -> dict:
    return {'page': page, 'size': size}

@app.get('/items')
def list_items(p: Annotated[dict, Depends(common_paginate)]):
    return {'page': p['page'], 'size': p['size']}

def get_db() -> Session: ...
@app.get('/u/{id}')
def by_id(id: int, db: Annotated[Session, Depends(get_db)]):
    ...
```

### Yield — setup / teardown

```python
def get_db():
    db = SessionLocal()
    try:
        yield db          # the value passed to the handler
    finally:
        db.close()        # runs after response is sent
```

### Sub-dependencies and policies

```python
def current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    db:    Annotated[Session, Depends(get_db)],
) -> User:
    user = decode_and_load(token, db)
    if not user: raise HTTPException(401)
    return user

def require_admin(
    user: Annotated[User, Depends(current_user)],
) -> User:
    if user.role != 'admin': raise HTTPException(403)
    return user

@app.delete('/users/{id}')
def remove(id: int, _admin: Annotated[User, Depends(require_admin)]):
    ...
```

### Where to attach dependencies
- **Per-handler** — `def f(x = Depends(dep))`
- **Per-router** — `APIRouter(dependencies=[Depends(verify_api_key)])`
- **Per-app** — `FastAPI(dependencies=[...])` for global guards
- **As policy** — raise `HTTPException` in the dep; never reach the handler

### Don't fight the cache
Dependencies are *cached per request*: if `get_db` is in three deps in the same request, you get one session. To opt out: `Depends(get_db, use_cache=False)`.

---

## Slide 09 — Async vs Sync — When Each Runs

FastAPI runs **async path operations on the event loop**. Sync ones run in a **thread pool** (anyio). Mixing is fine; getting the boundary right is the difference between fast and stalled.

### The decision tree
- **All your I/O is async-aware** (httpx, asyncpg, aiokafka) → declare `async def` and `await` normally
- **Some I/O is sync-only** (psycopg2, requests, openai sync client) → declare the handler `def`; FastAPI runs it in the threadpool
- **Never** mix blocking I/O inside an `async def` — one stalled handler stalls every other request on the loop

### Async handler — fully async stack

```python
import httpx

@app.get('/weather/{city}')
async def weather(city: str):
    async with httpx.AsyncClient(timeout=5) as c:
        r = await c.get(f'https://api.weather/{city}')
        r.raise_for_status()
        return r.json()
```

### Sync handler — blocking work

```python
import requests

@app.get('/legacy/{q}')
def legacy(q: str):              # def, not async def
    # runs in threadpool — main loop unblocked
    r = requests.get(f'https://legacy/{q}', timeout=5)
    return r.json()
```

### Run sync work from async

```python
from fastapi.concurrency import run_in_threadpool
from anyio import to_thread

@app.post('/render')
async def render(req: RenderReq):
    img = await run_in_threadpool(make_pdf, req)
    img2 = await to_thread.run_sync(make_pdf, req)
    return {'size': len(img)}
```

### The classic foot-gun

```python
# BAD — time.sleep blocks the event loop
@app.get('/slow')
async def slow():
    time.sleep(2)         # blocking!
    return {'ok': True}

# GOOD
import asyncio
@app.get('/slow')
async def slow():
    await asyncio.sleep(2)
    return {'ok': True}
```

### CPU-bound? Off the loop entirely
- **Threads** for short I/O wrappers (the threadpool default is 40 workers)
- **Process pool** (`concurrent.futures`) for genuinely CPU-bound work — one process per core
- **Background queue** (Celery, RQ, dramatiq, arq) when the work is > ~1 second or needs retries
- **GPU inference** — async handler that `await`s the model client; never block the loop on `.generate()`

---

## Slide 10 — Authentication

FastAPI ships **security utilities** — classes that double as dependencies and OpenAPI Security Schemes. The framework parses the right header / query / cookie; you decide how to verify.

### OAuth2 password / JWT

```python
from fastapi.security import OAuth2PasswordBearer
from jose import jwt, JWTError

oauth2_scheme = OAuth2PasswordBearer(tokenUrl='/auth/token')

def current_user(token: Annotated[str, Depends(oauth2_scheme)]) -> User:
    try:
        payload = jwt.decode(token, settings.JWT_KEY,
                             algorithms=['HS256'],
                             audience=settings.AUD,
                             issuer=settings.ISS)
    except JWTError:
        raise HTTPException(401, 'invalid token')
    return User(**payload['user'])

@app.get('/users/me', response_model=User)
def me(user: Annotated[User, Depends(current_user)]):
    return user
```

### Scopes

```python
from fastapi.security import SecurityScopes, OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl='/auth/token',
    scopes={'read': 'Read', 'write': 'Modify'})

def require_scopes(
    scopes: SecurityScopes,
    token: Annotated[str, Depends(oauth2_scheme)],
):
    payload = jwt.decode(token, ...)
    have = set(payload.get('scope', '').split())
    for s in scopes.scopes:
        if s not in have: raise HTTPException(403)
    return payload

@app.delete('/items/{id}')
def remove(id: int,
           _: Annotated[dict, Security(require_scopes, scopes=['write'])]):
    ...
```

### API keys

```python
from fastapi.security import APIKeyHeader

api_key = APIKeyHeader(name='X-API-Key', auto_error=False)

def require_api_key(
    key: Annotated[str | None, Depends(api_key)],
    db:  Annotated[Session, Depends(get_db)],
) -> APIKey:
    if not key: raise HTTPException(401)
    row = db.query(APIKey).filter_by(hash=sha256(key)).first()
    if not row or row.revoked: raise HTTPException(401)
    return row
```

### What FastAPI gives you
- **Security utilities** — `OAuth2*`, `APIKey*`, `HTTPBearer`, `HTTPBasic`
- **OpenAPI security schemes** auto-populated; "Authorize" button in Swagger
- **Composability** — the security dep can have its own deps (DB, cache, logger)

### What you bring
- Token signing / verification — `python-jose`, `authlib`, or hit your IdP's JWKS
- Password hashing — `passlib[argon2]` or `argon2-cffi` directly
- Refresh-token rotation, idle timeouts, revocation lists
- Rate limiting on the auth endpoints — FastAPI doesn't ship one

---

## Slide 11 — Errors, Exception Handlers & Middleware

### HTTPException & custom errors

```python
from fastapi import HTTPException
from fastapi.responses import JSONResponse

class NotFoundError(Exception):
    def __init__(self, what: str): self.what = what

@app.exception_handler(NotFoundError)
async def not_found(_req, exc: NotFoundError):
    return JSONResponse(404, content={
      'error': {'code': 'not_found',
                'message': f'{exc.what} not found'}})

@app.get('/users/{id}')
def get_user(id: int):
    user = repo.find(id)
    if not user: raise NotFoundError('user')
    return user
```

### Override the validation error shape

```python
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
async def validation_error(req, exc):
    return JSONResponse(422, content={
      'error': {
        'code': 'validation_failed',
        'details': exc.errors(),
        'request_id': req.state.request_id,
      }
    })
```

### Middleware

```python
from fastapi import Request
from uuid import uuid4
import time

@app.middleware('http')
async def request_id_and_timing(request: Request, call_next):
    rid = request.headers.get('x-request-id') or str(uuid4())
    request.state.request_id = rid

    t0 = time.perf_counter()
    response = await call_next(request)
    dur_ms = (time.perf_counter() - t0) * 1000
    response.headers['x-request-id'] = rid
    response.headers['server-timing'] = f'app;dur={dur_ms:.1f}'
    return response
```

### CORS & trusted host

```python
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware

app.add_middleware(CORSMiddleware,
  allow_origins=['https://app.example.com'],
  allow_credentials=True,
  allow_methods=['GET','POST','PUT','PATCH','DELETE'],
  allow_headers=['authorization','content-type'])

app.add_middleware(TrustedHostMiddleware,
  allowed_hosts=['api.example.com', '*.example.com'])
```

### Order matters
Middleware added later runs *outermost*. Put request-ID and timing *last* in code so they wrap everything — including CORS rejections.

---

## Slide 12 — Background, Streaming & WebSockets

Three patterns for non-classical request/response: **`BackgroundTasks`** for fire-and-forget after the response, **`StreamingResponse`** for chunked output, and **WebSockets** for full-duplex.

### Background tasks

```python
from fastapi import BackgroundTasks

@app.post('/users')
def create_user(
    body: UserCreate,
    bg: BackgroundTasks,
    mailer: Annotated[Mailer, Depends(get_mailer)],
):
    user = users.create(body)
    bg.add_task(mailer.send_welcome, user.email)
    return user
# → response sent immediately; mailer runs after.
# Good for: emails, webhooks, audit logs, <1s work.
# For real retry / scheduling: use Celery / arq / dramatiq.
```

### Streaming — SSE / NDJSON

```python
from fastapi.responses import StreamingResponse
import json, asyncio

async def gen_events():
    for i in range(10):
        yield f'event: tick\ndata: {json.dumps({"i": i})}\n\n'
        await asyncio.sleep(1)

@app.get('/stream')
async def stream():
    return StreamingResponse(gen_events(),
                             media_type='text/event-stream',
                             headers={'Cache-Control': 'no-cache',
                                      'X-Accel-Buffering': 'no'})

# NDJSON
async def gen_rows():
    async for r in db.stream(query):
        yield json.dumps(r).encode() + b'\n'

@app.get('/users.ndjson')
async def ndjson():
    return StreamingResponse(gen_rows(),
                             media_type='application/x-ndjson')
```

### WebSockets

```python
from fastapi import WebSocket, WebSocketDisconnect

@app.websocket('/ws')
async def ws(socket: WebSocket):
    await socket.accept()
    try:
        async for msg in socket.iter_text():
            await socket.send_text(f'echo: {msg}')
    except WebSocketDisconnect:
        ...
```

### LLM-streaming pattern

```python
@app.post('/chat')
async def chat(req: ChatReq):
    async def tokens():
        async for delta in llm.stream(req.messages):
            yield f'data: {json.dumps({"delta": delta})}\n\n'
        yield 'data: [DONE]\n\n'
    return StreamingResponse(tokens(),
        media_type='text/event-stream',
        headers={'Cache-Control': 'no-cache',
                 'X-Accel-Buffering': 'no'})
```

### Pitfalls
- `BackgroundTasks` blocks worker shutdown — long jobs go in a queue, not here
- SSE behind nginx without `X-Accel-Buffering: no` → silent client
- WS handlers don't auto-reconnect — client must handle it
- `StreamingResponse` + a `response_model` doesn't work — you're past Pydantic

---

## Slide 13 — Database — SQLAlchemy 2.0 & Friends

FastAPI doesn't bundle an ORM. Three real choices: **SQLAlchemy 2.0 async** (the default), **SQLModel** (Pydantic + SQLAlchemy by tiangolo), **Tortoise ORM** (Django-ish async).

### SQLAlchemy 2.0 async

```python
# db.py
from sqlalchemy.ext.asyncio import (
    create_async_engine, async_sessionmaker, AsyncSession)
from sqlalchemy.orm import DeclarativeBase

engine = create_async_engine(settings.DATABASE_URL,
                             pool_size=10, max_overflow=20)
SessionLocal = async_sessionmaker(engine, expire_on_commit=False)

class Base(DeclarativeBase): ...

# deps.py
async def get_db() -> AsyncIterator[AsyncSession]:
    async with SessionLocal() as s:
        yield s

# usage
@app.get('/users/{id}', response_model=UserRead)
async def get_user(id: int,
                   db: Annotated[AsyncSession, Depends(get_db)]):
    user = await db.get(User, id)
    if not user: raise HTTPException(404)
    return user
```

### Migrations — Alembic

```bash
uv add alembic
uv run alembic init alembic
uv run alembic revision --autogenerate -m 'init'
uv run alembic upgrade head
```

### SQLModel — one class, two roles

```python
from sqlmodel import SQLModel, Field

class User(SQLModel, table=True):
    id:    int | None = Field(default=None, primary_key=True)
    email: str        = Field(index=True, unique=True)
    name:  str
# → this is BOTH the SQLAlchemy table and the Pydantic model
#   used in request / response bodies.

# pros: less duplication for simple CRUD
# cons: ORM and DTO concerns become tangled in larger apps
```

### Patterns that pay off
- **DTOs separate from ORM** — `UserCreate / UserRead / UserDB`; never expose the table directly
- **Repository functions** over the session — `users.by_email(db, email)` is testable; chained ORM calls in handlers aren't
- **Transactions** — `async with db.begin():` at the boundary
- **Pool sizing** — `pool_size` per worker × workers < DB max_connections

### Don't
- Don't share a session across requests — one per request, scoped via `Depends`
- Don't call sync ORM from `async def` handlers
- Don't ship without a connection-pool monitor

---

## Slide 14 — Configuration — Pydantic Settings

Twelve-factor config, but typed. **`pydantic-settings`** reads env vars / `.env` / secrets dirs, validates types and defaults at boot, and surfaces missing values as *startup errors*, not runtime ones.

### Settings model

```python
from pydantic import Field, SecretStr, AnyHttpUrl, PostgresDsn
from pydantic_settings import BaseSettings, SettingsConfigDict
from typing import Literal

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file='.env', env_prefix='APP_',
        env_nested_delimiter='__', case_sensitive=False)

    env:        Literal['dev','staging','prod'] = 'dev'
    debug:      bool = False
    database_url: PostgresDsn
    redis_url:    str
    jwt_key:      SecretStr
    cors_origins: list[AnyHttpUrl] = []
    log_level:    Literal['DEBUG','INFO','WARNING','ERROR'] = 'INFO'

    smtp__host: str | None = None
    smtp__port: int = 587
    smtp__user: str | None = None
    smtp__pass: SecretStr | None = None

settings = Settings()
```

### Use from a dependency

```python
from functools import lru_cache

@lru_cache
def get_settings() -> Settings:
    return Settings()

@app.get('/config')
def cfg(s: Annotated[Settings, Depends(get_settings)]):
    return {'env': s.env, 'log_level': s.log_level}
```

### Why this is load-bearing
- Type errors at *boot* — the pod that can't start is the one you want
- `SecretStr` hides secrets in `repr` / logs / `model_dump`
- `.env` for local dev, real env vars for k8s — same code path
- Tests override with `app.dependency_overrides[get_settings] = lambda: TestSettings()`

### Don't
- Don't call `Settings()` at import time at module top — it makes tests harder
- Don't log secrets — even with `SecretStr`, custom `__str__` in error paths can leak
- Don't conflate config and feature flags

---

## Slide 15 — Testing

FastAPI ships a **`TestClient`** built on httpx. The interesting feature is **`app.dependency_overrides`** — swap any `Depends` dependency in tests for a stub, with no patching.

### Sync tests with TestClient

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_health():
    r = client.get('/healthz')
    assert r.status_code == 200
    assert r.json() == {'ok': True}

def test_create_user():
    r = client.post('/users', json={'email': 'a@x.io', 'name': 'A'})
    assert r.status_code == 201
    assert r.json()['email'] == 'a@x.io'
```

### Async tests with httpx

```python
import pytest
from httpx import AsyncClient, ASGITransport
from app.main import app

@pytest.mark.asyncio
async def test_async():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport,
                           base_url='http://test') as c:
        r = await c.get('/healthz')
        assert r.status_code == 200
```

### Dependency overrides — the killer feature

```python
def fake_db():
    return InMemoryDB()

def fake_user():
    return User(id=1, email='t@t', role='admin')

app.dependency_overrides[get_db]      = fake_db
app.dependency_overrides[current_user] = fake_user

# now every endpoint sees the fakes — no monkeypatch
client = TestClient(app)
r = client.get('/users/me')
assert r.status_code == 200

app.dependency_overrides.clear()
```

### What to fake, what to keep real
- **Database**: real Postgres (Testcontainers) for integration; SQLite for fast unit; never mock
- **Auth**: override `current_user`; don't mint real tokens in unit tests
- **External HTTP**: `respx` (httpx) or `pytest-httpserver`
- **Queues / mailers**: in-memory fake; assert calls in the test

---

## Slide 16 — OpenAPI & Documentation

OpenAPI generation is the **point** of FastAPI — not a side feature. The schema you serve at `/openapi.json` is the same schema Swagger UI / ReDoc render, the same one you ship to clients.

### What's free
- `/openapi.json` — OpenAPI 3.1
- `/docs` — Swagger UI
- `/redoc` — ReDoc
- Path / query / body / header param entries
- Request bodies and response schemas
- Security schemes from the security utilities
- Example payloads from `Field(examples=[...])`

### Tags, summaries, deprecation

```python
@app.post('/items',
          tags=['items'],
          summary='Create an item',
          description='Accepts a JSON body...',
          response_description='The created item',
          status_code=201,
          deprecated=False)
def create(item: ItemCreate): ...

# group tags
app = FastAPI(openapi_tags=[
    {'name': 'items', 'description': 'Catalogue items'},
    {'name': 'users', 'description': 'Account ops'},
])
```

### Customise the schema

```python
from fastapi.openapi.utils import get_openapi

def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    schema = get_openapi(title='My API', version='1.2.3', routes=app.routes)
    schema['info']['x-logo'] = {'url': 'https://example/logo.png'}
    schema['servers'] = [{'url': 'https://api.example.com'}]
    app.openapi_schema = schema
    return schema

app.openapi = custom_openapi
```

### Export & client generation

```bash
uv run python -c \
  'from app.main import app; import json; print(json.dumps(app.openapi()))' \
  > openapi.json

# typescript types for the frontend
npx openapi-typescript openapi.json -o api.ts

# python client
uv run datamodel-code-generator --input openapi.json --output client.py
```

### Hide what shouldn't be public
- Internal / admin routes — `include_in_schema=False`
- `response_model_exclude_none=True` for cleaner output schemas
- Mount Swagger UI behind admin auth in prod, or disable with `FastAPI(docs_url=None)`

---

## Slide 17 — Lifespan & Startup / Shutdown

Where to open a DB pool, warm a model, connect to Redis, register Prometheus collectors. The modern pattern is **`asynccontextmanager` + `FastAPI(lifespan=...)`**.

### Lifespan context manager

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # —— startup ——
    app.state.db    = create_engine_pool()
    app.state.redis = await aioredis.from_url(settings.REDIS_URL)
    app.state.model = load_model('llama-3-8b')
    log.info('startup complete')

    yield                            # app runs

    # —— shutdown ——
    await app.state.redis.close()
    await app.state.db.dispose()
    log.info('shutdown complete')

app = FastAPI(lifespan=lifespan)
```

### Sharing state with handlers

```python
def get_redis(req: Request):
    return req.app.state.redis

@app.get('/cache/{k}')
async def cache(k: str, r: Annotated[Redis, Depends(get_redis)]):
    return {'value': await r.get(k)}
```

### Health vs ready

```python
@app.get('/healthz')         # liveness
def healthz(): return {'ok': True}

@app.get('/readyz')          # readiness
async def readyz(req: Request):
    if app.state.draining:
        return JSONResponse(503, {'ok': False})
    try:
        async with req.app.state.db.connect() as c:
            await c.execute(text('select 1'))
        await req.app.state.redis.ping()
    except Exception:
        return JSONResponse(503, {'ok': False})
    return {'ok': True}
```

### Graceful shutdown
- Uvicorn handles SIGTERM — calls the lifespan teardown after in-flight requests drain
- Set `app.state.draining = True` so `/readyz` goes 503 first — LB stops sending traffic
- K8s: `terminationGracePeriodSeconds > longest_request`; `preStop` hook of `sleep 5` avoids the LB race

> Avoid the deprecated `@app.on_event('startup')` / `'shutdown'` decorators. Use lifespan — that's what the docs recommend.

---

## Slide 18 — Performance

FastAPI is rarely the bottleneck. Wins live in **workers vs concurrency**, **avoiding loop blocking**, **payload size & serialisation**, and **downstream parallelism**.

### Workers & processes

```bash
# single process, multi-coroutine — great for I/O
uvicorn app.main:app --host 0.0.0.0 --port 8000

# multi-process — one event loop per worker
uvicorn app.main:app --workers 4

# production: gunicorn manages uvicorn workers
gunicorn app.main:app \
  -k uvicorn.workers.UvicornWorker \
  -w 4 -b 0.0.0.0:8000 \
  --timeout 60 --graceful-timeout 30 \
  --max-requests 10000 --max-requests-jitter 1000
```

Rule of thumb: `workers` ≈ `2 × CPU`. In K8s prefer *one* worker per pod — the orchestrator scales pods.

### Newer ASGI servers
- **Granian** — Rust-backed; faster than uvicorn on raw I/O
- **Hypercorn** — HTTP/2, HTTP/3, trio support
- For LLM streaming, uvicorn + `--http httptools` is usually plenty

### Avoid loop blocking
- **No** `time.sleep`, `requests`, blocking `open()` in `async def`
- **Use** `asyncio.sleep`, `httpx.AsyncClient`, `aiofiles`
- If you must call sync code — `await run_in_threadpool(fn, ...)`
- Tune `anyio` threadpool: `BackgroundExecutor(max_workers=...)`

### Serialisation
- Pydantic v2 is fast — usually the fastest part of your handler
- Switch to `orjson` via `ORJSONResponse` for > 2× on big lists
- `response_model_exclude_unset=True` — trims default fields from the wire
- Stream large lists as NDJSON — constant memory

```python
from fastapi.responses import ORJSONResponse
app = FastAPI(default_response_class=ORJSONResponse)
```

### If you're still slow
- Profile with `py-spy record` against the prod-like image
- Watch the connection pool — pool exhaustion looks identical to "FastAPI is slow"
- Measure p50 / p95 / p99 separately — a slow tail is usually a slow downstream

---

## Slide 19 — Deployment

The shape that ships: **multi-stage Dockerfile**, **non-root user**, **distroless or slim base**, **healthchecks wired to `/healthz` & `/readyz`**.

### Dockerfile (uv-based)

```dockerfile
ARG PY=3.12
FROM python:${PY}-slim-bookworm AS base
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1

FROM base AS deps
RUN pip install --no-cache-dir uv
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev

FROM base AS runtime
RUN useradd -r -u 10001 app
WORKDIR /app
COPY --from=deps /app/.venv /app/.venv
COPY app/ ./app/
USER app
ENV PATH=/app/.venv/bin:$PATH

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD python -c "import urllib.request,sys; \
                 sys.exit(0 if urllib.request.urlopen('http://localhost:8000/healthz').status==200 else 1)"

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Kubernetes basics

```yaml
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: api
        image: ghcr.io/me/api:1.2.3
        ports: [{containerPort: 8000}]
        livenessProbe:
          httpGet: {path: /healthz, port: 8000}
          periodSeconds: 10
        readinessProbe:
          httpGet: {path: /readyz, port: 8000}
          periodSeconds: 5
          failureThreshold: 3
        resources:
          requests: {cpu: "200m", memory: "256Mi"}
          limits:   {cpu: "1",    memory: "512Mi"}
        lifecycle:
          preStop:
            exec: {command: ["sleep", "5"]}
```

### Where to host

| Platform | Notes |
|----------|-------|
| K8s (EKS / GKE / AKS) | One worker per pod; HPA |
| Cloud Run / Lambda + Mangum | Stateless; cold starts; no WebSockets on Lambda |
| Render / Fly / Railway | Easy buttons; great for staging |
| VPS + nginx + systemd | Cheap, predictable; gunicorn + uvicorn |

### Don't ship
- As `root`
- With `--reload`
- With `/docs` public if the API is non-public
- Without HTTPS at the proxy layer

---

## Slide 20 — Observability

Three pillars, same as everywhere: **structured logs**, **metrics**, **distributed traces**. Minimum competent setup is **structlog + prometheus-fastapi-instrumentator + OpenTelemetry**.

### Structured logs (structlog)

```python
import structlog

structlog.configure(
  processors=[
    structlog.contextvars.merge_contextvars,
    structlog.processors.add_log_level,
    structlog.processors.TimeStamper(fmt='iso'),
    structlog.processors.JSONRenderer(),
  ],
  wrapper_class=structlog.stdlib.BoundLogger,
)
log = structlog.get_logger()

@app.middleware('http')
async def log_request(req, call_next):
    rid = req.headers.get('x-request-id') or str(uuid4())
    structlog.contextvars.bind_contextvars(request_id=rid)
    t0 = time.perf_counter()
    resp = await call_next(req)
    log.info('http_request',
             method=req.method, path=req.url.path,
             status=resp.status_code,
             dur_ms=round((time.perf_counter()-t0)*1000, 1))
    return resp
```

### Metrics — Prometheus

```python
from prometheus_fastapi_instrumentator import Instrumentator
Instrumentator().instrument(app).expose(app, endpoint='/metrics')
```

### Tracing — OpenTelemetry

```python
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.sdk.resources import Resource
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

provider = TracerProvider(resource=Resource.create(
    {'service.name': 'api', 'service.version': '1.2.3'}))
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
trace.set_tracer_provider(provider)

FastAPIInstrumentor.instrument_app(app)
```

### Hygiene
- Redact `authorization`, `cookie`, `x-api-key` in logs
- Never log request / response bodies in prod
- Bound metric cardinality — never tag by user ID
- Sample traces 1–5% with tail-sampling on errors
- Bind `request_id` via `contextvars` — appears on every line

### What to look at first
1. p95 / p99 per route
2. Error rate per route
3. DB pool wait time
4. Downstream call duration histograms

---

## Slide 21 — Security Hardening

FastAPI is a thin layer; security is mostly composing the right middleware, validating at the boundary, and not trusting input.

### Required middleware

```python
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from fastapi.middleware.httpsredirect import HTTPSRedirectMiddleware
from starlette.middleware.gzip import GZipMiddleware

app.add_middleware(HTTPSRedirectMiddleware)
app.add_middleware(TrustedHostMiddleware,
  allowed_hosts=['api.example.com','*.example.com'])
app.add_middleware(CORSMiddleware,
  allow_origins=settings.cors_origins,
  allow_credentials=True,
  allow_methods=['GET','POST','PUT','PATCH','DELETE'],
  allow_headers=['authorization','content-type'])
app.add_middleware(GZipMiddleware, minimum_size=1024)
```

### Security headers

```python
@app.middleware('http')
async def security_headers(req, call_next):
    resp = await call_next(req)
    resp.headers['Strict-Transport-Security'] = \
      'max-age=31536000; includeSubDomains; preload'
    resp.headers['X-Content-Type-Options'] = 'nosniff'
    resp.headers['Referrer-Policy'] = 'no-referrer'
    resp.headers['X-Frame-Options'] = 'DENY'
    resp.headers['Content-Security-Policy'] = \
      "default-src 'none'; frame-ancestors 'none'"
    return resp
```

### Body size & rate limiting

```python
from starlette.middleware.base import BaseHTTPMiddleware

class BodySizeLimit(BaseHTTPMiddleware):
    def __init__(self, app, max_bytes: int):
        super().__init__(app); self.max = max_bytes
    async def dispatch(self, req, call_next):
        cl = int(req.headers.get('content-length', 0))
        if cl > self.max:
            return JSONResponse(413, {'error': 'too large'})
        return await call_next(req)

app.add_middleware(BodySizeLimit, max_bytes=128 * 1024)

# rate limiting: slowapi (limits + redis)
from slowapi import Limiter
limiter = Limiter(key_func=lambda r: r.client.host,
                  storage_uri=settings.redis_url)
app.state.limiter = limiter

@limiter.limit('5/minute')
@app.post('/auth/login')
async def login(req: Request, body: Login): ...
```

### Input & auth hygiene
- **Validate every field** with Pydantic
- **Parameterise SQL** — never f-string a user value
- **Hash passwords** with argon2 — `passlib[argon2]`
- **Verify JWTs strictly** — `iss / aud / exp / nbf`
- **Constant-time compare** for tokens — `secrets.compare_digest`

### Don't
- Echo unsanitised user input in errors / headers / logs
- Trust `X-Forwarded-For` without configuring trust at the proxy
- Disable CORS «temporarily» with `'*'`

---

## Slide 22 — FastAPI vs Flask, Django REST, Litestar, Starlette

| Criterion | FastAPI | Litestar | Starlette | Flask | Django REST |
|-----------|---------|----------|-----------|-------|-------------|
| Style | Type hints + DI | Type hints + DI | ASGI toolkit | Decorators + context | Class-based views |
| Validation | Pydantic v2 | msgspec / Pydantic / attrs | None (BYO) | None (BYO) | DRF serializers |
| Speed | Fast | Faster (msgspec) | Fast (raw) | Sync, slower | Sync, slowest |
| Async | Native | Native | Native | Limited | Limited (3.x) |
| OpenAPI | Built-in | Built-in | BYO | BYO (Spectree, etc.) | drf-spectacular |
| Batteries | Few | Few | None | Few | Many (admin/ORM) |
| Maturity | Mature (2018) | Active (2023+) | Mature | Mature | Very mature |
| Best for | JSON APIs, ML servers | Throughput, stricter typing | Building frameworks | Tiny apps | Server-rendered + admin |

### Choose FastAPI when
- JSON API or ML / LLM model server
- You want OpenAPI for free
- You like Pydantic v2's validation feel
- Most Python web jobs in 2026

### Choose Litestar when
- You want msgspec speed and stricter typing discipline
- You prefer a richer DI / plugin model
- You're starting a fresh greenfield project

### Choose Django / Flask when
- **Django + DRF**: server-rendered HTML + admin + ORM in one
- **Flask**: tiny scripts, glue services, minimum surface area

---

## Slide 23 — Production Recipes

### Repository + service shape

```python
# schemas/users.py — DTOs
class UserCreate(BaseModel):
    email: EmailStr; name: str
class UserRead(BaseModel):
    id: int; email: EmailStr; name: str

# repositories/users.py
async def create(db, data: UserCreate) -> UserRead:
    row = User(**data.model_dump())
    db.add(row); await db.commit(); await db.refresh(row)
    return UserRead.model_validate(row, from_attributes=True)

# services/users.py
async def signup(db, queue, data: UserCreate) -> UserRead:
    if await users_repo.by_email(db, data.email):
        raise HTTPException(409)
    user = await users_repo.create(db, data)
    await queue.enqueue('emails.welcome', user.id)
    return user

# api/v1/users.py
@router.post('/', response_model=UserRead, status_code=201)
async def signup_endpoint(
    body: UserCreate,
    db: Annotated[AsyncSession, Depends(get_db)],
    q:  Annotated[Queue,        Depends(get_queue)],
):
    return await users_service.signup(db, q, body)
```

### Outbox pattern

```python
async def signup(db, data):
    async with db.begin():
        user = await users_repo.create(db, data)
        await db.execute(insert(Outbox).values(
            kind='emails.welcome',
            payload={'user_id': user.id}))
    return user
```

### Versioned routers

```python
from fastapi import APIRouter

v1 = APIRouter(prefix='/v1')
v1.include_router(users_router, prefix='/users', tags=['v1.users'])
v1.include_router(items_router, prefix='/items', tags=['v1.items'])

v2 = APIRouter(prefix='/v2')
v2.include_router(users_router_v2, prefix='/users', tags=['v2.users'])

app.include_router(v1)
app.include_router(v2)
```

### Feature flags via dependency overrides

```python
def feature_x_enabled() -> bool:
    return get_settings().flags.feature_x

@app.get('/x')
def x(enabled: Annotated[bool, Depends(feature_x_enabled)]):
    if not enabled: raise HTTPException(404)
    ...
# tests can flip the flag with dependency_overrides[feature_x_enabled]
```

---

## Slide 24 — Summary & Next Steps

### Why FastAPI earns its keep
- **Type hints become the API** — one declaration drives validation, serialisation, DI and OpenAPI
- **Async-first** with a graceful sync escape hatch via the threadpool
- **Pydantic v2 is fast** — the Rust core leaves serialisation off the hot list
- **DI via `Depends()`** is small enough to hold in your head, big enough to compose policies
- **OpenAPI for free** — clients, contract tests, mock servers all fall out

### Take-aways
- Use `Annotated[T, Query/Path/Body(...)]` for type-checker-friendly signatures
- One `Depends()` chain per concern: DB, current user, scope guard
- Lifespan over deprecated `on_event`
- DTOs separate from ORM — `UserCreate / UserRead / UserDB`
- `app.dependency_overrides` in tests, not monkeypatch
- Pydantic Settings for config; fail at boot, not at request time

### Production checklist
- Lifespan opens/closes pools; `/healthz` + `/readyz` wired
- HTTPS + TrustedHost + CORS allow-list + CSP
- Rate limit on auth endpoints; body-size cap globally
- Structured logs (structlog), Prometheus, OpenTelemetry
- Non-root container, distroless / slim base, K8s probes & preStop
- Tests use `TestClient` + `dependency_overrides`

### Next steps
- Build a small CRUD service: FastAPI + SQLAlchemy 2.0 async + Alembic + uvicorn + Docker
- Wire structlog + Prometheus + OTel; ship dashboards
- Generate a TypeScript client from `openapi.json`
- Add a Testcontainers Postgres test suite
- Try Litestar on a side project — see what stricter typing buys you

### Essential reading
- [fastapi.tiangolo.com](https://fastapi.tiangolo.com/) — the canonical tutorial
- [Pydantic docs](https://docs.pydantic.dev/) — Pydantic is half the framework
- [Starlette docs](https://www.starlette.io/) — what's underneath
- [uvicorn docs](https://www.uvicorn.org/) — the server you ship behind
