---
name: init-python-project
description: Scaffold a new Python project that follows modern best practices, using uv by default to initialize and manage the project (with pip/venv, pipenv, or manual fallbacks if uv isn't available). Sets up ruff (lint + format), mypy type checking, pre-commit, a tests/ folder with pytest, and optionally Docker, a CI workflow, coverage, placeholder structure, and a git repo with .gitignore. Use when the user wants to start/create/bootstrap a new Python project, says "set up a Python project", "init a python repo", "new python project with best practices", or similar.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - AskUserQuestion
  - Bash(uv *)
  - Bash(curl -LsSf https://astral.sh/uv/install.sh -o /tmp/uv-install.sh)
  - Bash(sh /tmp/uv-install.sh)
  - Bash(brew install uv)
  - Bash(python -m venv *)
  - Bash(python3 -m venv *)
  - Bash(python --version)
  - Bash(python3 --version)
  - Bash(python -V)
  - Bash(python3 -V)
  - Bash(pip install *)
  - Bash(pip3 install *)
  - Bash(pipenv --version)
  - Bash(pipenv install *)
  - Bash(pipenv run *)
  - Bash(git *)
  - Bash(mkdir -p *)
  - Bash(ls *)
  - Bash(cat *)
  - Bash(command -v *)
  - Bash(which *)
  - Bash(pwd)
  - Bash(test *)
---

# /init-python-project — Scaffold a modern Python project

This skill sets up a new Python project that follows current best practices.
**uv is the default and recommended project/venv manager**, but the skill can
fall back to pip + venv, pipenv, a manual `.venv`, or skip environment setup
entirely if the user prefers. It is opinionated about quality defaults (ruff +
mypy + pre-commit + pytest) and confirms optional pieces (Docker, CI,
coverage, placeholder structure) before acting.

Be conversational and decisive: state the sensible defaults, confirm them, ask
only the questions that genuinely change the output, then build.

## Steps

### 1. Choose the project/venv manager
Run `command -v uv` (or `uv --version`).

- **If uv is present**, use it — it's the recommended path. Continue.
- **If uv is missing**, ask the user how they want to proceed. Offer:
  1. **Install uv for them** (recommended). Install via the official installer
     (`curl -LsSf https://astral.sh/uv/install.sh | sh`, or `brew install uv`
     if Homebrew is available and they prefer it), then re-check with
     `uv --version`.
  2. **They install uv themselves** — point them at
     https://docs.astral.sh/uv/getting-started/installation/ and let them run
     it in this session with the `!` prefix; wait, re-check, continue.
  3. **Use an alternative manager** — `pip` + `venv` (stdlib), `pipenv`, or a
     **manually created `.venv`**. Confirm which.
  4. **Skip environment setup** — just scaffold files/config and let the user
     wire up an environment later.

**Record the chosen manager**; the steps below are written for uv and include
equivalents for the alternatives (see *Manager equivalents* at the end). When
a manager is skipped, still create all config files and note clearly which
setup commands the user must run themselves.

### 2. Clarify what the project is for
Ask the user, conversationally:
- **What is the project?** (its purpose / what it does) — shapes the package
  name, structure, and dependencies.
- **What name** should the project/package use (default: derive a sensible
  kebab/snake name from the description).
- **Where** should it be created? Run `pwd` and **suggest a default path** —
  a new subdirectory named after the project inside the current working
  directory (e.g. `<cwd>/<name>`). Present it clearly and let the user change
  it to any path they prefer.
- **Is it a library, a CLI, or an application/service?** Affects layout
  (e.g. `--package`/src layout for libraries, an entry point for CLIs/apps)
  and whether Docker makes sense.

### 3. Choose the Python version
**Ask which Python version to target** (default: the latest stable the user
has, or the newest if installing fresh). Then:
- With uv: pin it with `uv python pin <version>` (creating `.python-version`),
  and if that version isn't installed, offer `uv python install <version>`.
- With other managers: confirm the interpreter (`python3 --version`) and write
  a `.python-version` and a `requires-python` in `pyproject.toml` to match.

### 4. State the defaults, then confirm the optional pieces
Tell the user plainly what you'll set up **by default** (let them opt out):
- **Ruff** as the linter **and** formatter.
- **mypy** for static type checking.
- **pre-commit** running ruff (lint + format), mypy, and a set of standard
  hygiene hooks automatically before every commit.
- A **`tests/`** folder with **pytest** added to the dev environment.

Then ask the questions that DO change the output (use AskUserQuestion):
- **Docker?** `Dockerfile`, `docker-compose.yml`, both, or neither?
  (Default: neither.)
- **CI workflow?** A GitHub Actions workflow running lint + type-check + tests
  on push/PR. (Default: yes for libraries/apps that'll live on GitHub; ask.)
- **Test coverage?** Add `pytest-cov` with a coverage config. (Default: yes.)
- **Placeholder structure & files?** Starter scaffolding (package module(s), a
  sample entry point, a sample test, README) or just the bare project? Tailor
  placeholders to the project type from step 2.
- **Git?** Confirm initializing a git repo (default: yes) — you'll add an
  appropriate Python `.gitignore`.

### 5. Safety check before initializing
Before creating anything, verify the target path is safe:
- Run `test -e <path>` / `ls -la <path>`. If it **exists and is non-empty**,
  stop and confirm with the user (pick a different path, or explicitly proceed
  into the existing directory). Never silently overwrite or clobber an
  existing project.

### 6. Initialize the project
With uv, pick the form that matches the project type:
- Application/CLI: `uv init <name>` (add `--package` for an installable CLI
  with an entry point).
- Library: `uv init --lib <name>` (src layout, installable package).

Then work inside the project dir (use absolute paths in subsequent commands to
avoid permission prompts) and confirm `pyproject.toml`, the package module,
and the lockfile workflow exist. For alternative managers, create the dir,
`pyproject.toml`, and package layout yourself and set up the environment with
the chosen tool (see *Manager equivalents*).

### 7. Add tooling to the environment
Add dev dependencies: **ruff, mypy, pytest, pre-commit** (plus **pytest-cov**
if coverage was chosen).
- uv: `uv add --dev ruff mypy pytest pre-commit` (and `uv add --dev pytest-cov` if coverage was chosen)
- Verify with `uv run ruff --version`, `uv run mypy --version`,
- For pip/pipenv, install the same packages as dev dependencies (see *Manager
  equivalents*).

### 8. Configure ruff
Add a `[tool.ruff]` section to `pyproject.toml` with modern defaults: target
the pinned Python version, a reasonable line length, and a useful lint rule
selection (e.g. `E`, `F`, `I` imports, `UP` pyupgrade, `B` bugbear). Configure
both linting and formatting. Keep it readable and lightly commented.

### 9. Configure mypy
Add a `[tool.mypy]` section with sensible-but-not-punishing strictness for a
fresh project (e.g. `python_version` matching the pin, `warn_unused_configs`,
`warn_redundant_casts`, `warn_unused_ignores`, `disallow_untyped_defs`, and
`ignore_missing_imports = true` to start). Note the user can tighten toward
`strict = true` later.

### 10. Configure pre-commit
Create `.pre-commit-config.yaml` with **pinned hook revisions**:
- `ruff-pre-commit`: the `ruff` hook (lint, with `--fix`) and `ruff-format`.
- `mirrors-mypy`: the `mypy` hook.
- `pre-commit-hooks`: standard hygiene hooks (`trailing-whitespace`,
  `end-of-file-fixer`, `check-yaml`, `check-added-large-files`,
  `check-merge-conflict`).
After the git repo exists (step 15), run `pre-commit install` (via the chosen
manager, e.g. `uv run pre-commit install`) so hooks fire on every commit, and
consider `pre-commit autoupdate` to freshen pinned revs. If git isn't being
set up, note that `pre-commit install` requires a git repo.

### 11. Tests folder
Ensure a `tests/` directory exists (with `__init__.py` or relying on pytest
rootdir conventions) and, if placeholders were chosen, a minimal passing
`tests/test_smoke.py`. Add `[tool.pytest.ini_options]` (e.g.
`testpaths = ["tests"]`). If coverage was chosen, add a `[tool.coverage.run]`
section (e.g. `source = ["<package>"]`) and document `--cov` usage.

### 12. Docker (only if requested)
- **Dockerfile**: modern, multi-stage-friendly, using the official `uv` base
  image (or installing the chosen manager) with `uv sync --frozen` / `uv run`.
  Slim Python base, non-root user.
- **docker-compose.yml**: define the app service (and an obvious companion
  like a database only if the user mentions needing one). Keep it minimal.
- Add a `.dockerignore`.

### 13. CI workflow (only if requested)
Create `.github/workflows/ci.yml` that, on push and pull_request: checks out
the repo, sets up the chosen manager (e.g. `astral-sh/setup-uv`), installs
deps, then runs `ruff check`, `ruff format --check`, `mypy`, and `pytest`
(with coverage if enabled). Pin action versions.

### 14. Placeholder structure (only if requested)
Create starter files appropriate to the project type:
- A package module with a `main()` / sample function and, for CLIs/apps, a
  runnable entry point wired in `pyproject.toml` (`[project.scripts]`).
- A `README.md` with a short description and the run/test/lint/type-check
  commands.
- The sample test from step 11.
Keep placeholders minimal and obviously-replaceable.

### 15. Git + .gitignore
- If the user wants git (default yes): `git init`, create a Python
  `.gitignore` (cover `.venv/`, `__pycache__/`, `*.pyc`, `.pytest_cache/`,
  `.ruff_cache/`, `.mypy_cache/`, coverage files (`.coverage`, `htmlcov/`),
  build/dist artifacts, `.env`, and editor/OS cruft; add Docker ignores only
  if Docker was set up).
- Then run `pre-commit install` (via the chosen manager).
- Make an initial commit only if the user wants one (ask) — otherwise leave
  the working tree staged/clean for them to review.

### 16. Validate the scaffold
Before handing off, **prove the setup is green** by running (via the chosen
manager; skip cleanly if env setup was skipped):
- `uv run ruff check .` and `uv run ruff format --check .`
- `uv run mypy .`
- `uv run pytest`
Report the results. If anything fails, fix the scaffold (not by loosening
config blindly) until it passes, or clearly flag what the user needs to
resolve.

### 17. Confirm and hand off
Summarize what was created (paths, key files, defaults applied, optional
pieces included/skipped, and the chosen manager) and give the everyday
commands (uv form shown; translate for the chosen manager):
- Run the app: `uv run <entrypoint>` (or `uv run python -m <package>`)
- Run tests: `uv run pytest`
- Lint/format: `uv run ruff check .` / `uv run ruff format .`
- Type-check: `uv run mypy .`
- Hooks run automatically on `git commit`.

## Manager equivalents
The steps default to uv. For the alternatives, map commands as follows:

- **pip + venv (stdlib):**
  - Create env: `python3 -m venv .venv` then activate it.
  - Init: create the project dir, `pyproject.toml`, and package layout by hand.
  - Add dev deps: `pip install ruff mypy pytest pre-commit pytest-cov` and
    record them under `[dependency-groups]` / an optional `dev` extra (or a
    `requirements-dev.txt`).
  - Run tools directly once the venv is active (`ruff check .`, `mypy .`,
    `pytest`), i.e. drop the `uv run` prefix.
- **pipenv:**
  - Env + deps: `pipenv install --dev ruff mypy pytest pre-commit pytest-cov`.
  - Run tools via `pipenv run <cmd>` instead of `uv run <cmd>`.
- **manual `.venv`:** same as pip + venv, but the user manages activation; be
  explicit about the activation step in the hand-off.
- **skip:** create every config file and the layout, but run no install/venv
  commands — list the exact commands the user should run later to finish.

## Principles
- **uv is the default project manager** — use `uv init`, `uv add`, `uv run`,
  `uv sync`. Only fall back to pip/pipenv/manual when uv is unavailable and the
  user chooses an alternative.
- **Install uv only with consent** — if it's missing, ask first; install it for
  the user if they say yes, otherwise offer the alternatives above.
- **Quality is on by default** — ruff, mypy, pytest, and pre-commit are baked
  in; Docker, CI, coverage, placeholders, and the initial commit are the
  branch points to ask about.
- **Never clobber** — check the target path before initializing.
- **Always validate** — finish by running lint, types, and tests so the user
  receives a known-green scaffold.
- Prefer modern, minimal, well-commented config over kitchen-sink setups the
  user has to delete.
- Use absolute paths in shell commands once inside the new project dir.
