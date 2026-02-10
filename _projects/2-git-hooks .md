---
layout: project
title: "git-hooks"
description: "Git Hooks Collection"
tags: ["Open Source", "git", "shell", "bash"]
technologies: ["git", "shell", "bash", "eslint", "prettier", "pytest", "flake8", "shell-script", "bash-script", "phpstan", "pint"]
github: "https://github.com/prog-time/git-hooks"
stars: 17
forks: 2
status: active
---

## About

A collection of reusable git hook scripts for automating code quality checks and developer workflows.

## Features

- **JavaScript/TypeScript**: ESLint, Prettier, TypeScript type checking, Vitest test runner, test existence validation
- **Python**: Flake8 linting, Mypy static analysis, Pytest runner, service test validation (local and Docker modes)
- **PHP/Laravel**: PHPStan analysis with progressive error reduction, Pint code style fixing, test coverage validation
- **Docker**: Dockerfile linting with Hadolint
- **Shell**: Script validation with ShellCheck
- **Git**: Branch naming conventions, automatic task ID injection in commits

## Available Scripts

### JavaScript / TypeScript

| Script | Description |
|--------|-------------|
| `javascript/check_eslint_all.sh` | Runs ESLint with `--fix` across the entire project (`app/`, `components/`, `lib/`, `types/`). |
| `javascript/check_prettier.sh` | Runs Prettier on staged `.ts` files passed as arguments. |
| `javascript/check_prettier_all.sh` | Runs Prettier on all TypeScript files using glob patterns. |
| `javascript/check_tsc_all.sh` | Runs TypeScript type checking via `tsconfig.check.json`. |
| `javascript/check_vitest.sh` | Runs Vitest for specified files or the full suite. Supports `--watch` and `--coverage`. |
| `javascript/check_vitest_all.sh` | Runs the full Vitest test suite. |
| `javascript/check_tests.sh` | Runs Vitest only for changed/staged TypeScript files, auto-discovers matching test files. |
| `javascript/check_tests_all.sh` | Runs the full Vitest test suite (simple wrapper). |
| `javascript/check_test_coverage.sh` | Runs Vitest with optional `--watch` and `--coverage` flags. |
| `javascript/check_tests_exist.sh` | Validates that each staged TypeScript source file has a corresponding `tests/...test.ts` file. |

### Python

| Script | Description |
|--------|-------------|
| `python/check_flake8.sh` | Runs Flake8 locally on Python files (line length 120). |
| `python/check_flake8_in_docker.sh` | Runs Flake8 inside the `app_dev` Docker container. |
| `python/check_mypy.sh` | Runs Mypy locally on changed `app/*.py` files. |
| `python/check_mypy_in_docker.sh` | Runs Mypy inside the Docker container, maps host/container paths. |
| `python/check_pytest.sh` | Runs the full Pytest suite locally with `PYTHONPATH=./app`. |
| `python/check_pytest_in_docker.sh` | Runs Pytest inside the Docker container, converts container paths to host paths. |
| `python/find_test.sh` | Validates that each service in `app/services/` has exactly one corresponding test file. |

### PHP

| Script | Description |
|--------|-------------|
| `php/check_phpstan.sh` | Runs PHPStan with progressive error reduction. Tracks error count per file and allows commits only when errors decrease. |
| `php/laravel/check_pint.sh` | Runs Laravel Pint, auto-fixes code style issues, and stages corrected files. |
| `php/find_test.sh` | Validates that modified PHP classes have corresponding unit tests. |
| `php/start_test_in_docker.sh` | Runs PHPUnit tests inside Docker for modified files. |

### Git Workflow

| Script | Description |
|--------|-------------|
| `git/check_branch_name.sh` | Validates branch names match pattern: `{type}/{task-id}_{description}` |
| `git/preparations/add_task_id_in_commit.sh` | Prepends task ID from branch name to commit message. |
| `git/preparations/prepare-commit-description.sh` | Appends list of changed files to commit message. |

### Docker

| Script | Description |
|--------|-------------|
| `docker/check_hadolint.sh` | Lints Dockerfiles using Hadolint. |

### Shell

| Script | Description |
|--------|-------------|
| `shell/check_shellcheck.sh` | Validates shell scripts using ShellCheck (via Docker). |
| `scripts/check_shellcheck.sh` | Validates shell scripts using ShellCheck (local). |


## Requirements

- Bash 4.0+
- Git
- jq
- Node.js with npx (for JavaScript/TypeScript hooks)
- Python 3, Flake8, Mypy, Pytest (for Python hooks)
- PHP 8.1+ with Composer (for PHP hooks)
- Docker (optional, for Docker-based hooks)
- ShellCheck (for shell validation)
