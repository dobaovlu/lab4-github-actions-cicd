# Notes app — Lab 4 starting point (Python + FastAPI)

This is the end state of Lab 3 **plus** what Lab 4 expects you to bring:
a passing test suite and a configured linter. Use it only if your team did not finish
Lab 3 or has no test/linter yet.

It is the starting point for Lab 4, **not** a solution to it: there is no
`.github/workflows/` folder — writing the workflow is the lab.

## What is in the folder

| File | Purpose |
|------|---------|
| `app/main.py` | The app: `/`, `/health` (platform health check), `/health/db` |
| `Dockerfile` | Reads `PORT` (`${PORT:-8000}`), binds `0.0.0.0`, `exec` for clean shutdown |
| `requirements.txt` | **Runtime** dependencies — the only ones installed in the image |
| `requirements-dev.txt` | Runtime + `pytest`, `httpx2`, `ruff` — what CI installs |
| `pyproject.toml` | Configuration for `ruff` (linter) and `pytest` |
| `tests/conftest.py` | Sets a dummy `DATABASE_URL` so the app can be imported without a database |
| `tests/test_health.py` | Two real tests: `/health` and `/` |
| `compose.yml`, `db/init.sql`, `.env.example` | Local stack with PostgreSQL (Week 4) |

## 1. Put it in your team repository

Copy every file in this folder into the **root** of your team repository — including the
hidden files `.dockerignore`, `.gitignore`, `.env.example` and the folders `app/`, `db/`,
`tests/`. Then, in Git Bash:

    git switch -c lab4-starter
    git add .
    git commit -m "Lab 4: start from the Lab 4 starter"
    git push -u origin lab4-starter
    # open a pull request on GitHub and merge it

## 2. Check it works on your machine (before the lab)

    python -m venv .venv
    source .venv/Scripts/activate      # Git Bash on Windows
    # source .venv/bin/activate        # macOS / Linux
    pip install -r requirements-dev.txt

    ruff check .                       # expect: All checks passed!
    pytest -q                          # expect: 2 passed

    podman build -t app:dev .
    podman run --rm -p 3000:8000 -e DATABASE_URL=postgresql://unused@localhost/none app:dev
    # second Git Bash window:
    curl -s http://localhost:3000/health     # {"status":"ok"}

These are exactly the commands your workflow will run on the GitHub runner.
If they fail here, they will fail there.

## 3. Render (if you have no live service yet)

Do Lab 3 Tasks B–D with this code: New → Web Service → Docker, branch `main`,
health check path `/health`, environment `DATABASE_URL` = any placeholder
(e.g. `postgresql://app:unused@localhost:5432/appdb`), `APP_ENV` = `production`.
Do **not** set `PORT`.

## 4. Then follow the Lab 4 handout from Task A

| Handout says | In this starter |
|--------------|-----------------|
| Install step (Task A) | `pip install -r requirements-dev.txt` — already what the handout uses |
| Deliberate failure (Task C) | change `200` to `201` in `tests/test_health.py` |
| Visible change (Task D) | change `APP_VERSION` in `app/main.py` |

Never commit `.env`. It is already listed in `.gitignore` and `.dockerignore`.
