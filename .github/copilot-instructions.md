# Copilot instructions for this repository

## Repository overview

This repo is a hands-on DevOps learning project for beginners. The core application is a small Flask service in `app/`; the rest of the repo wires it into Docker, Compose, CI/CD, Terraform, and monitoring.

- `app/app.py`: Flask app with `/`, `/health`, and `/metrics` endpoints.
- `app/tests/test_app.py`: the canonical behavior tests for the app.
- `compose.yaml`: defines the app + Redis stack, with optional Prometheus/Grafana via the `monitoring` profile.
- `.github/workflows/ci.yml`: the real validation pipeline used by CI.
- `scripts/start.sh`: starts the lab stack and prints the local/Codespaces URLs.
- `iac/`: Terraform examples for the infrastructure lab.
- `monitoring/`: Prometheus and Grafana config.

## Build, test, and validation commands

This project uses Python + pytest for app logic and Docker/Compose for integration checks. There is no dedicated lint target in the repo.

### Local app setup

```bash
cd app
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt -r requirements-dev.txt
```

### Run the full test suite

```bash
cd app
python -m pytest -q
```

### Run one test or a subset

```bash
cd app
python -m pytest tests/test_app.py -k "health_is_ok" -q
```

You can also target a specific test node:

```bash
cd app
python -m pytest tests/test_app.py::test_metrics_are_exposed -q
```

### Smoke-test Docker image locally

```bash
docker build -t devops-demo:local app
docker run -d --name demo -p 8000:8000 devops-demo:local
curl -fsS http://localhost:8000/health
```

### Start the full lab stack

```bash
bash scripts/start.sh
```

Or use Compose directly with monitoring enabled:

```bash
docker compose --profile monitoring up -d --build
```

Stop it with:

```bash
bash scripts/start.sh stop
```

## Architecture and execution model

The app intentionally stays tiny so the DevOps lessons are easy to follow:

- The Flask app is a single module: `app/app.py`.
- Visitor counting uses Redis when `REDIS_HOST` is set (the Compose setup does this); otherwise it falls back to a process-local counter.
- Prometheus metrics are exposed at `/metrics`; the project uses the `prometheus_client` library, and the metrics are asserted in tests.
- CI follows a standard pattern: install dependencies, run pytest, then build a Docker image and verify the container responds on `/health`.
- The Docker image starts the app with `gunicorn --bind 0.0.0.0:8000 app:app` rather than `flask run`.
- Local labs and operational scripts are intentionally separated from the application code: `scripts/` handles startup/cleanup, `monitoring/` contains observability config, and `iac/` contains Terraform examples.

## Codebase conventions and project-specific patterns

- Keep app behavior aligned with the tests in `app/tests/test_app.py`; `/health` must return JSON `{"status": "ok"}` and `/metrics` must include `app_requests_total`.
- The repo is primarily educational and intentionally simple: prefer minimal, explicit changes over framework-heavy abstractions.
- Use the app folder as the operational root for Python commands; `app/pytest.ini` sets `testpaths = tests` and `pythonpath = .`.
- When changing dependencies, update `app/requirements.txt` and `app/requirements-dev.txt` together and keep versions pinned to match the lab environment.
- The Docker and Compose workflows are part of the learning experience: if a change affects startup or health checks, validate with both `pytest` and a container-level `/health` check.
- `scripts/start.sh` and `compose.yaml` are the authoritative ways to run the full stack locally, especially when working with Grafana/Prometheus on ports 3000 and 9090.
- Prefer repository docs (`README.md`, `docs/*`, and `labs/*`) for environment-specific setup details and expected commands; this repo emphasizes guided lab flows over custom tooling.

## Working with this repo

When adding or modifying app code:

1. Change the Flask app in `app/app.py` or related Python files.
2. Validate with the smallest relevant pytest target first.
3. If the change touches startup, health, or container behavior, rerun a Docker smoke test and/or the compose stack.
4. Keep the project’s beginner-friendly, lab-oriented structure intact.

## MCP server configuration

If you want, I can help wire up MCP servers relevant to this project type (for example, a browser or web automation server for the app, or a container/infra server for Docker/Terraform workflows).
