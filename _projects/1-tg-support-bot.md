---
layout: project
title: "tg-support-bot"
description: "Бот технической поддержки в Telegram и ВКонтакте"
tags: ["Open Source", "PHP", "Laravel", "Redis", "PostgreSQL", "Docker", "Node.js + Socket.io",]
technologies: ["PHP", "Laravel", "Redis", "PostgreSQL", "JavaScript", "Docker + Docker Compose", "Node.js + Socket.io",  "Telegram API", "VK API", "OpenAI API"]
github: "https://github.com/prog-time/tg-support-bot"
demo: "https://tg-support-bot.ru"
stars: 180
forks: 39
status: active
---

## О проекте

Телеграм бот для объединения сообщений из Telegram, ВКонтакте и сторонних API источников в единую систему технической поддержки.

Сообщения отправляются в Telegram-группу, где под каждого пользователя создаётся отдельная чат-тема (топик).

Презентация работы бота: [https://youtu.be/hIpYreHOxIk](https://youtu.be/hIpYreHOxIk)

Бот поддерживает все типы сообщений: текст, изображения, файлы, голосовые сообщения, видео, стикеры, контакты и другие медиафайлы.

## Демонстрация

Процесс обработки сообщений:
1. **Получение сообщения:** Пользователь отправляет сообщение боту через Telegram, ВКонтакте, виджет сайта или внешний API
2. **Создание топика:** Бот автоматически находит или создаёт тему (топик) в Telegram-группе для этого клиента
3. **Пересылка в группу:** Сообщение пересылается в соответствующую тему с информацией об отправителе
4. **Ответ менеджера:** Менеджеры отвечают прямо в теме — бот отслеживает их сообщения
5. **Отправка клиенту:** Ответ автоматически пересылается клиенту от имени бота (без раскрытия личности менеджера)

## Как это работает

```
┌─────────────┐         ┌─────────────┐         ┌─────────────────┐
│  Telegram   │────────▶│             │◀────────│   ВКонтакте     │
│   Users     │         │             │         │     Users       │
└─────────────┘         │             │         └─────────────────┘
                        │             │
┌─────────────┐         │   TG Bot    │         ┌─────────────────┐
│  Website    │────────▶│   Server    │◀────────│  External API   │
│   Widget    │         │             │         │    Sources      │
└─────────────┘         │             │         └─────────────────┘
                        └──────┬──────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │  Telegram Group      │
                    │  ┌────────────────┐  │
                    │  │ Topic: User 1  │  │
                    │  ├────────────────┤  │
                    │  │ Topic: User 2  │  │
                    │  ├────────────────┤  │
                    │  │ Topic: User 3  │  │
                    │  └────────────────┘  │
                    └──────────────────────┘
```

## Технологический стек

**Backend:**
- Laravel 12.0+ (PHP 8.2+)
- PostgreSQL (база данных)
- Redis (кэш и очереди)
- Laravel Queue (обработка фоновых задач)

**Frontend & Real-time:**
- Node.js + Socket.io (WebSocket сервер)
- JavaScript (виджет чата)

**External APIs:**
- Telegram Bot API
- VK API
- OpenAI API / DeepSeek / GigaChat (AI)

**DevOps:**
- Docker + Docker Compose
- Nginx (веб-сервер)
- Certbot (SSL сертификаты)

**Monitoring & Logging:**
- Grafana (дашборды)
- Loki (логи)
- Promtail (сбор логов)
- Sentry (error tracking)

**Development:**
- PHPUnit (тестирование)
- PHPStan (статический анализ)
- Laravel Pint (code style)
