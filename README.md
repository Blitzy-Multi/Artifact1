# Artifact1

This repository is intended to host a **Python 3 + [Flask](https://flask.palletsprojects.com/)** re-implementation (port) of an existing **Node.js** server. The objective of the port is **full functional parity**: every endpoint, middleware effect, data operation, validation rule, error/status-code contract, and side effect of the original Node.js project is to be reproduced exactly in Flask, so that existing clients require no changes.

> **In one line:** rewrite the original Node.js server in Python 3 using Flask, preserving all functionality of the original project.

## ⚠️ Project Status / Blocking Prerequisite

**The port is currently blocked: the original Node.js source is not present in this repository.**

A full repository inspection (`git ls-files`, a working-tree search, and `git log`) confirms there is **no Node.js application source** of any kind:

- No `.js`, `.ts`, `.mjs`, or `.cjs` files.
- No `package.json`, `package-lock.json`, or any other Node manifest.
- No Node configuration, `.env`, or server entry point.

**Verified baseline.** `git ls-files` resolves to exactly one tracked file — this `README.md`, whose original first line is the heading `# Artifact1`. The history consists of a single baseline commit, `9e0722a` (full SHA `9e0722ace21443bfac8a1400eab45ceacf9fe8dd`, "Initial commit"). No Node.js source, manifest, configuration, or `.env` exists, and **no Flask application code has been written yet**. The Python dependency manifest (`requirements.txt`), `.gitignore`, and every other file shown under *Planned Project Structure* below are **planned/conditional artifacts** — created only after the original Node.js source is supplied and its dependencies and environment are mapped 1:1.

**Why this blocks the work.** The acceptance criterion is *"preserving all functionalities of the original project."* That set of functionalities is defined **entirely** by the original source. With the source absent, the behavior to replicate cannot be enumerated, implemented, or verified. In keeping with this project's prime directive, **no behavior will be fabricated** — no routes, models, middleware, services, or environment-variable names are invented while the source is missing.

**Required action (to unblock).** Provide the original Node.js source by **committing it into this repository** (the project root, including `package.json` and the server entry point) **or attaching it to the project**.

**What happens once it is supplied.** The Flask application — routes, models, services, schemas, middleware, configuration, and tests — and the final dependency set are derived **1:1** from the original source, following the target stack and structure documented below.

## Target Stack

The target runtime and framework are fixed (Python 3 + Flask). The direct target dependencies are exact pins; the transitive dependencies (`Werkzeug`, `Jinja2`) are noted as Flask-managed lower bounds, not exact pins. None of them are temporary version markers.

| Component | Version | Role |
|-----------|---------|------|
| CPython | 3.12.x | Target language runtime |
| Flask (with the `dotenv` extra) | `Flask[dotenv]==3.1.3` | Web framework (WSGI); bundles `python-dotenv` for `.env` loading |
| Werkzeug | `>=3.1` (transitive via Flask) | WSGI utilities, routing, request/response objects |
| Jinja2 | `>=3.1` (transitive via Flask) | Server-side templating (used only if the original renders views) |
| gunicorn | `gunicorn==26.0.0` | Production WSGI server (Linux) |
| pytest | `pytest==9.0.3` | Test runner for parity tests |

Once the port is unblocked, the definite core stack written to `requirements.txt` will be:

```text
Flask[dotenv]==3.1.3
gunicorn==26.0.0
pytest==9.0.3
```

**Conceptual Node → Python mapping.** These are the conventions the port will follow. They describe how constructs are translated; they do **not** assert that any specific package is in use — capability-specific adapters are added only for dependencies actually found in the original `package.json`.

| Node.js / npm | Python / Flask |
|---------------|----------------|
| Express / Koa / Fastify / `http` server | Flask application (Werkzeug WSGI) |
| `node server.js` / `app.listen(port)` | `wsgi.py` + `gunicorn wsgi:app` |
| `package.json` / npm | `requirements.txt` / `pyproject.toml` + pip |
| Express `Router` | Flask `Blueprint` |
| `app.use(...)` middleware | `before_request` / `after_request` hooks + error handlers |
| `.env` via `dotenv` | `python-dotenv` (Flask `[dotenv]` extra) |

Capability-specific adapters (for example, an ORM, JWT handling, CORS, or request validation) will be selected and pinned **only if** the corresponding capability is present in the original source — mapped from its actual npm dependencies at implementation time.

## Planned Project Structure

The port will use the idiomatic Flask **application-factory** layout shown below. This is the **planned/target** structure to be created once the original source is supplied; none of these files exist yet — the only tracked file today is this `README.md`.

```text
.
├── wsgi.py                  # WSGI entry point: exposes `app = create_app()` (gunicorn target)
├── app/
│   ├── __init__.py          # create_app() application factory
│   ├── config.py            # environment-driven configuration classes
│   ├── extensions.py        # extension singletons (only those the source requires)
│   ├── middleware.py        # before_request/after_request hooks + error handlers
│   ├── routes/              # one Flask Blueprint per Express router/route group
│   │   ├── __init__.py
│   │   └── <resource>.py
│   ├── models/              # ORM/ODM models (only if persistence is present)
│   │   ├── __init__.py
│   │   └── <entity>.py
│   ├── services/            # business logic per service/controller
│   │   └── <service>.py
│   └── schemas/             # request/response validation (only if present)
│       └── <schema>.py
├── tests/
│   ├── conftest.py          # pytest app/client fixtures
│   └── test_<resource>.py   # per-route parity tests
├── requirements.txt         # (planned) Python dependency manifest mapped 1:1 from package.json
├── pyproject.toml           # (optional) project metadata / build configuration
├── .env.example             # documents environment keys (mirrors the original .env 1:1)
└── .gitignore               # (planned) venv, __pycache__, .env, *.pyc
```

The exact `routes/`, `models/`, `services/`, `schemas/`, and `tests/` modules (the `<resource>`, `<entity>`, `<service>`, and `<schema>` template tokens above) are determined **1:1** by the original Node.js source — one Blueprint per Express router, one model per Node model, one service per controller, and one parity test suite per route group.

## Setup

> Applicable once the application scaffold and dependency manifest exist. The steps below describe the intended workflow; `requirements.txt` is generated 1:1 from the original `package.json` once the port is unblocked.

Python **3.12.x** is the target runtime. Create and activate a virtual environment, then install the pinned dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Running the Server

> Intended workflow once `wsgi.py` and the `app/` package exist.

**Development** — run with Flask's built-in server:

```bash
export FLASK_APP=wsgi.py          # Windows (PowerShell): $env:FLASK_APP = "wsgi.py"
flask run
```

**Production (Linux)** — run under gunicorn:

```bash
gunicorn wsgi:app
```

> **Note:** gunicorn relies on Unix-only facilities and does not run on Windows. For local development on Windows, use `flask run` (or a Windows-compatible WSGI server such as `waitress`).

## Environment Variables

Configuration is loaded from a `.env` file via `python-dotenv`, which is bundled through the `Flask[dotenv]` extra. An `.env.example` file will document every required key.

The specific variable names are **not invented here**: they will mirror the original Node.js project's `.env` keys **1:1** once the source is supplied. (`.env` itself is git-ignored; only `.env.example` is committed.)

## Testing

Tests are run with **pytest**:

```bash
pytest
```

The suite asserts **behavioral parity** with the original server — equivalent inputs must produce identical outputs and HTTP status codes — mirroring the original project's jest/mocha tests. The concrete test modules are derived **1:1** from the original source once it is supplied.
