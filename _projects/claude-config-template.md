---
layout: project
title: "claude-config-template"
description: "Шаблон-репозиторий для хранения конфигурации Claude Code (скилы, агенты, команды, хуки, MCP) в Git и сборки ~/.claude/ через симлинки"
tags: ["Open Source", "Claude Code", "Shell", "Make", "Python"]
technologies: ["Claude Code", "Shell", "Bash", "Make", "Python", "GitHub Actions", "Markdown", "YAML"]
github: "https://github.com/prog-time/claude-config-template"
stars: 4
forks: 0
status: active
---

## О проекте

`claude-config-template` — репозиторий-шаблон для хранения личной (или командной) конфигурации [Claude Code](https://docs.claude.com/en/docs/claude-code) как кода: скилы, агенты, команды, хуки и MCP-серверы лежат в Git, а директория `~/.claude/` собирается из них с помощью симлинков.

Идея простая: `~/.claude/` — это такой же код, и его место в Git. Шаблон решает три повседневные задачи:

- конфигурация хранится в репозитории и легко переносится на любую машину;
- команда работает с единым набором агентов, скилов и команд;
- набор линтеров (в pre-push хуке и GitHub Actions) ловит ошибки во frontmatter, YAML и JSON ещё до попадания в `main`.

## Возможности

- **Симлинки вместо копирования** — `make install` создаёт ссылки из `~/.claude/` в репозиторий, поэтому правки подхватываются мгновенно, а встроенные скилы Anthropic не затираются.
- **Поэлементная линковка** — каждый скил, агент, команда и хук линкуется отдельным элементом, без перетирания существующих файлов в `~/.claude/`.
- **Чистый uninstall** — `make uninstall` удаляет ровно те ссылки, которые создал `make install`, и не трогает посторонние файлы.
- **Каркас нового скила** — `make new-skill name=my-skill desc="..."` создаёт SKILL.md с готовым frontmatter, который сразу проходит линтер.
- **`make doctor`** — проверяет, на месте ли симлинки, установлены ли нужные версии Python, ruff, shfmt и других линтеров, корректно ли читаются конфиги.
- **Линтеры в pre-push и CI** — markdownlint, yamllint, shellcheck, shfmt, ruff, codespell, jsonschema и опциональный gitleaks; один и тот же набор скриптов работает локально и в GitHub Actions.

## Структура репозитория

```
skills/      пользовательские скилы (SKILL.md + ресурсы)
agents/      субагенты (отдельные .md с frontmatter)
commands/    слеш-команды
mcp/         примеры конфигов MCP-серверов
hooks/       PreToolUse / PostToolUse и прочие хуки
docs/        конвенции, гайды, changelog
scripts/     утилиты: линтер, генератор новых скилов
linting/     pre-push-хук и набор проверок
```

## Установка

```bash
git clone https://github.com/prog-time/claude-config-template.git
cd claude-config-template
make install
```

После этого в `~/.claude/` появятся симлинки на файлы из репозитория. Существующие файлы и встроенные скилы не перетираются.

## Использование

- Кладёте новый скил в `skills/my-skill/SKILL.md` — Claude Code видит его при следующем запуске.
- Меняете правила агента в `agents/<name>.md` — изменения подхватываются мгновенно, потому что `~/.claude/agents/<name>.md` — это симлинк.
- Перенос на другую машину — `git clone` + `make install`. Pre-push хук подключается автоматически.
- Если что-то сломалось — `make doctor` подскажет, какой симлинк потерян и какого линтера не хватает.
