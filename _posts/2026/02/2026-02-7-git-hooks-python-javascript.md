---
layout: post
title: "Git-хуки для автоматической проверки качества кода"
date: 2026-02-7
categories: ["open source", "git-hooks"]
description: "Готовые git-хуки для Python и JavaScript: автоматическая проверка линтинга, форматирования и качества кода перед коммитом."
keywords: "git hooks, Python, JavaScript, линтинг, форматирование, автоматизация, pre-commit"
---

Большинство фейлов в CI — это мелочи: забытый `console.log`, форматирование, линт, сломанный импорт, файл без теста. Такие ошибки не должны доезжать до сборки или код-ревью.

Git-хуки позволяют запускать проверки прямо во время `git commit` и блокировать коммит, если были обнаружены нарушения.

Все скрипты находятся здесь: [https://github.com/prog-time/git-hooks](https://github.com/prog-time/git-hooks)

## Как это работает

### Структура скриптов

Каждый скрипт — обычный `.sh` файл. Для каждого типа проверки существует 2 версии:

1. **Скрипт для конкретных файлов**  
   Например: `bash python/check_flake8.sh $FILES`

2. **Скрипт для всего проекта**  
   Например: `bash python/check_flake8_all.sh`

### Коды возврата

- `0` — всё хорошо, коммит проходит
- `1` — есть ошибки, коммит блокируется

### Применение

- Версия с передачей файлов → `pre-commit` (проверка файлов в Git индексе)
- Версия `*_all.sh` → `pre-push` или CI (проверка всего проекта)

### Пример использования в pre-commit

```bash
#!/bin/bash

set -e

# получаю все измененные файлы
ALL_FILE_ARRAY=()
while IFS= read -r line; do
    ALL_FILE_ARRAY+=("$line")
done < <(git diff --cached --name-only --diff-filter=ACM || true)

bash scripts/python/check_flake8.sh $ALL_FILE_ARRAY
bash scripts/js/check_eslint_all.sh
```

## JavaScript/TypeScript: Форматирование и линтинг

Базовый уровень защиты кода заключается в том, чтобы сразу приводить его к единому стилю и ловить очевидные ошибки ещё до того, как они попадут в репозиторий.

### ESLint

Скрипт `check_eslint_all.sh` — обёртка над ESLint, которая проверяет и автоматически исправляет ошибки по всему проекту.

**Возможности:**
- Запускает `npx eslint --fix` по директориям из переменной `LINT_DIRS`
- Автоисправимые проблемы чинятся автоматически
- При неисправимых ошибках (неиспользуемые переменные, сломанные импорты) — блокирует коммит

**Пример скрипта:**

```bash
#!/bin/bash
# ------------------------------------------------------------------------------
# Runs ESLint with --fix on the entire project (app/, components/, lib/, types/).
# No file arguments required — always checks all configured directories.
# Exits 1 if ESLint fails to fix issues, 0 on success.
# ------------------------------------------------------------------------------

set -e

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(cd "$SCRIPT_DIR/../.." && pwd)"

cd "$PROJECT_ROOT"

# Directories to check
LINT_DIRS=("app/" "components/" "lib/" "types/")

echo "=== ESLint (full project) ==="

echo "Fixing ESLint issues..."
if ! npx eslint "${LINT_DIRS[@]}" --fix; then
    echo "ERROR: Failed to fix ESLint issues"
    exit 1
fi
echo "ESLint issues fixed!"

exit 0
```

### Prettier

Для форматирования есть два варианта:

- `check_prettier_all.sh` — форматирует весь проект
- `check_prettier.sh` — только конкретные файлы

В `pre-commit` обычно используется второй вариант — форматируются только изменённые файлы. Это быстрее и не создаёт лишних диффов.

Скрипт прогоняет `prettier --write`, поэтому разработчику не нужно думать о пробелах, переносах и кавычках — стиль применяется автоматически.

Скрипты: [https://github.com/prog-time/git-hooks/tree/main/javascript](https://github.com/prog-time/git-hooks/tree/main/javascript)

### TypeScript (TSC)

Скрипт `check_tsc_all.sh` запускает проверку типов без сборки. Для настройки используется `tsconfig.check.json`.

Это полезно для больших проектов: можно случайно «уронить» типизацию в другом модуле, и обычный линт это не поймает.

**Пример скрипта:**

```bash
#!/bin/bash
# ------------------------------------------------------------------------------
# Runs TypeScript type checking on the entire project using tsconfig.check.json.
# No file arguments required — checks all files configured in tsconfig.
# Exits 1 if type check fails, 0 on success.
# ------------------------------------------------------------------------------

set -e

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(cd "$SCRIPT_DIR/../.." && pwd)"

cd "$PROJECT_ROOT"

# TypeScript config
TS_CONFIG="tsconfig.check.json"

echo "=== TypeScript ==="
echo "Running type check..."
if ! npx tsc --project "$TS_CONFIG"; then
    echo "ERROR: TypeScript check failed"
    exit 1
fi
echo "TypeScript check passed!"

exit 0
```

**Конфигурация `tsconfig.check.json`:**

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "noUncheckedIndexedAccess": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

## Python: Линтинг и статический анализ

Для Python набор Git-хуков выполняет ту же роль, что и для JavaScript/TypeScript, но с учётом особенностей экосистемы: динамическая типизация и разнообразие стилей кодирования требуют чуть более строгой проверки.

### Flake8 (линтинг)

Скрипт `check_flake8.sh` проверяет `.py` файлы на соответствие PEP8 и код-стайлу.

**Что ловит Flake8:**
- Лишние пробелы
- Неверные отступы
- Слишком длинные строки
- Нарушения соглашений по именованию функций и переменных

**Пример скрипта:**

```bash
#!/bin/bash
# ----------------------------------------
# Python Code Style Checker
#
# This script checks Python files for style
# issues using flake8. It runs directly on
# the host (no Docker required).
# ----------------------------------------

if [ $# -eq 0 ]; then
    echo "No files to check"
    exit 0
fi

PY_FILES=()
CHECKED_FILES=0
HAS_ERRORS=0

for file in "$@"; do
    # Skip non-Python files
    if [[ ! "$file" =~ \.py$ ]]; then
        continue
    fi

    # Skip if file doesn't exist
    if [ ! -f "$file" ]; then
        continue
    fi

    PY_FILES+=("$file")
done

# Check if there are any Python files to check
if [ ${#PY_FILES[@]} -eq 0 ]; then
    echo "No Python files to check"
    exit 0
fi

# Run flake8 linter on each file
for file in "${PY_FILES[@]}"; do
    CHECKED_FILES=$((CHECKED_FILES + 1))

    OUTPUT=$(flake8 --max-line-length=120 "$file" 2>&1)
    EXIT_CODE=$?

    if [ $EXIT_CODE -ne 0 ]; then
        HAS_ERRORS=1
        echo "Style errors in: $file"
        echo "$OUTPUT"
        echo ""
    fi
done

# Final result
if [ $HAS_ERRORS -ne 0 ]; then
    echo "----------------------------------------"
    echo "ERROR: Code style check failed!"
    echo "Total files checked: $CHECKED_FILES"
    echo "Fix the errors above before committing."
    exit 1
fi

echo "Code style check passed! ($CHECKED_FILES files checked)"
exit 0
```

**Пример вывода ошибок:**

```
Style errors in: app/services/user_service.py
app/services/user_service.py:42:1: E302 expected 2 blank lines, found 1
app/services/user_service.py:67:80: E501 line too long (132 > 120 characters)
```

### Mypy (статический анализ)

Скрипт `check_mypy.sh` выполняет статический анализ типов.

**Особенности:**
- Игнорирует тесты и вспомогательные файлы
- Фокусируется только на продакшн-коде
- Выявляет потенциальные баги, которые в динамическом Python остаются незамеченными до выполнения кода

**Пример скрипта:**

```bash
#!/bin/bash

# ------------------------------------------------------------
# Runs mypy static analysis for changed Python files locally.
# Accepts file paths as args and checks only app/*.py files.
# ------------------------------------------------------------

if [ $# -eq 0 ]; then
    echo "No files to check"
    exit 0
fi

PY_FILES=()

for file in "$@"; do
    # Skip non-Python files
    if [[ ! "$file" =~ \.py$ ]]; then
        continue
    fi

    # Only files inside app/
    if [[ ! "$file" =~ ^app/ ]]; then
        continue
    fi

    # Skip if file doesn't exist
    if [ ! -f "$file" ]; then
        continue
    fi

    PY_FILES+=("$file")
done

if [ ${#PY_FILES[@]} -eq 0 ]; then
    echo "No Python files to check"
    exit 0
fi

OUTPUT=$(mypy \
    --pretty \
    --show-error-codes \
    --ignore-missing-imports \
    --follow-imports=skip \
    "${PY_FILES[@]}" 2>&1)

EXIT_CODE=$?

if [ $EXIT_CODE -ne 0 ]; then
    echo "----------------------------------------"
    echo "ERROR: Static analysis failed!"
    echo "----------------------------------------"
    echo "$OUTPUT"
    echo "----------------------------------------"
    echo "Total files checked: ${#PY_FILES[@]}"
    exit 1
fi

echo "Static analysis passed! (${#PY_FILES[@]} files checked)"
exit 0
```

### Зачем нужна связка Flake8 + Mypy

- **Flake8** следит за стилем и базовыми ошибками, экономя время на код-ревью
- **Mypy** ловит ошибки типов, которые не видны при обычном запуске Python

Вместе они создают первый фильтр качества: код не попадёт в коммит, пока не будет корректен по стилю и типам.

## Тесты и проверка покрытия

Следующий уровень защиты — убедиться, что новый код не только корректен по стилю и типам, но и покрыт тестами.

### JavaScript/TypeScript: проверка тестов

Процесс проверки тестирования делится на 2 стадии:

1. **Проверка наличия теста**
2. **Запуск теста для изменённого функционала**

#### Проверка наличия тестов

Скрипт `check_tests_exist.sh` проверяет наличие тестов для каждого `.ts` файла и блокирует коммит, если тест отсутствует.

**Правила:**
- Для файла `app/api/v1/roles/route.ts` должен существовать тест `tests/app/api/v1/roles/route.test.ts`
- Исключения задаются в переменной `SKIP_PATTERNS`

**Пример скрипта:**

```bash
#!/bin/bash
# ------------------------------------------------------------------------------
# Checks that each staged TypeScript source file has a corresponding test file.
# Receives files as arguments or reads from git staged files if none provided.
# Exits 1 if any source file is missing its tests/...test.ts counterpart.
# ------------------------------------------------------------------------------

set -e

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(cd "$SCRIPT_DIR/../.." && pwd)"

cd "$PROJECT_ROOT"

# Patterns to skip (no tests required)
SKIP_PATTERNS=("tests/" ".test.ts" ".config.ts" ".config.mjs" "types/" ".d.ts" "layout.tsx" "page.tsx" "loading.tsx" "error.tsx" "globals.css" "providers/" "components/ui/" "prisma/")

should_skip() {
    local file="$1"
    for pattern in "${SKIP_PATTERNS[@]}"; do
        [[ "$file" == *"$pattern"* ]] && return 0
    done
    return 1
}

# Get test path: source.ts -> tests/source.test.ts
get_test_path() {
    local file="$1"
    local base="${file%.ts}"
    base="${base%.tsx}"
    echo "tests/${base}.test.ts"
}

# Get files to check
FILES=()
if [ $# -eq 0 ]; then
    while IFS= read -r line; do
        [[ "$line" == *.ts || "$line" == *.tsx ]] && FILES+=("$line")
    done < <(git diff --cached --name-only --diff-filter=ACM 2>/dev/null || true)
    [ ${#FILES[@]} -eq 0 ] && echo "No staged TypeScript files" && exit 0
else
    for arg in "$@"; do
        [[ "$arg" == *.ts || "$arg" == *.tsx ]] && FILES+=("$arg")
    done
fi

echo "=== Test Coverage Check ==="
echo ""

missing=()
found=()
skipped=()

for file in "${FILES[@]}"; do
    [ ! -f "$file" ] && continue
    should_skip "$file" && { skipped+=("$file"); continue; }

    test_path=$(get_test_path "$file")

    if [ -f "$test_path" ]; then
        found+=("$file")
    else
        missing+=("$file → $test_path")
    fi
done

[ ${#found[@]} -gt 0 ] && echo -e "Has tests:" && printf '  %s\n' "${found[@]}" && echo ""
[ ${#skipped[@]} -gt 0 ] && echo -e "Skipped:" && printf '  %s\n' "${skipped[@]}" && echo ""

if [ ${#missing[@]} -gt 0 ]; then
    echo -e "Missing tests:"
    printf '  %s\n' "${missing[@]}"
    echo ""
    echo -e "ERROR: ${#missing[@]} file(s) missing tests"
    exit 1
fi

echo -e "All files have tests!"
```

#### Запуск тестов

Дополнительные скрипты:
- `check_tests.sh` — находит и запускает тесты для изменённых файлов
- `check_tests_all.sh` — запускает все тесты проекта

Ссылки на скрипты:
- [check_tests.sh](https://github.com/prog-time/git-hooks/blob/main/javascript/check_tests.sh)
- [check_tests_all.sh](https://github.com/prog-time/git-hooks/blob/main/javascript/check_tests_all.sh)

### Python: проверка тестов

Для работы с Python используется похожий набор скриптов:

- `find_tests.sh` — проверяет наличие тест-файла
- `check_pytest.sh` — запускает тесты для изменённых файлов

Эти проверки делают pre-commit не просто линтером, а **локальным гарантом качества кода**: даже если разработчик забудет написать тест, коммит не пройдёт.

## Docker-варианты скриптов

Для Python-проектов, которые используют специфичные зависимости или системные библиотеки, иногда запуск линтеров и тестов на локальной машине даёт ошибки, которых нет в контейнере.

### Структура Docker-скриптов

Для большинства скриптов есть вариации, которые должны запускаться через Docker. Каждый такой скрипт имеет суффикс `_in_docker`.

**Варианты запуска:**

- **Локальный** — `check_flake8.sh`, `check_mypy.sh`, `check_pytest.sh` (работает на хосте)
- **Docker** — `check_flake8_in_docker.sh`, `check_mypy_in_docker.sh`, `check_pytest_in_docker.sh` (внутри контейнера)

### Пример запуска в Docker

```bash
docker exec app_dev mypy /app/services/auth.py
```

**Вывод ошибок:**

```
app/services/auth.py:15: error: Incompatible return value type
```

**Если контейнер не запущен:**

```
ERROR: Container 'app_dev' is not running
Start it with: docker-compose -f docker/dev/docker-compose.yml up -d
```

### Преимущества Docker-скриптов

- **Консистентность окружения** — проверки проходят в том же окружении, что и продакшн
- **Без зависимости от локальной машины** — версии Python, системных библиотек или расширений не влияют на результат
- **Прямые пути в терминале** — ошибки отображаются так же, как если бы вы работали локально
- **Лёгкая интеграция с pre-commit** — просто замените локальный скрипт на Docker-версию

## Гибкость и тестируемость сборки

Все скрипты разделены на отдельные подгруппы. Вы можете подключать только те проверки, которые реально нужны вашему проекту, и собирать **собственную подборку хуков**.

### Примеры конфигураций

- Только линтинг и Prettier для фронтенда
- Только Flake8 и Mypy для Python
- Все проверки сразу

### Автотесты для скриптов

В проекте есть автотесты для всех shell-скриптов. Процесс использования:

1. Склонировать репозиторий: `git clone https://github.com/prog-time/git-hooks.git`
2. Настроить конфигурацию под свои директории, инструменты и правила
3. Прогнать версию через встроенные тесты

Такой подход делает сборку **надёжной и предсказуемой**: вы точно знаете, что хуки сработают так, как задумано.

## Итого

Релиз 2.0.0 превращает pre-commit в полноценный фильтр качества кода, который работает **ещё до того, как изменения попадут в репозиторий**.

### Ключевые возможности

**JavaScript/TypeScript:**
- ESLint
- Prettier
- Проверка типов через TypeScript
- Запуск тестов (полный и по изменённым файлам)
- Проверка наличия тестов

**Python:**
- Flake8
- Mypy
- Pytest (локально и в Docker)
- Проверка наличия тестов с контролем дубликатов

**Docker-поддержка:**
- Скрипты корректно работают в контейнере
- Автоматически маппят пути
- Выводят понятные ошибки

**Гибкость:**
- Скрипты разделены на подгруппы
- Можно подключать только нужные проверки
- Собирать свою подборку под конкретный проект

**Тестируемость:**
- Все скрипты покрыты автотестами
- Можно прогонять свою конфигурацию для проверки надёжности

## Ссылки

**Репозиторий:** [https://github.com/prog-time/git-hooks](https://github.com/prog-time/git-hooks)

Если вам понравилась эта сборка и она оказалась полезной для вашей команды — не забудьте поставить ⭐ на GitHub!

Если вы используете эту сборку и находите, что что-то можно улучшить, буду рад вашим **Pull Request**! Любые предложения по новым хукам, исправлениям или оптимизации скриптов помогут сделать проект ещё полезнее для всех.
