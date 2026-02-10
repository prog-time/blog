---
layout: project
title: "tg-logger"
description: "Laravel-пакет для отправки логов в Telegram по топикам"
tags: ["Open Source", "PHP", "Laravel"]
technologies: ["PHP", "Laravel", "Telegram Bot API", "Composer"]
github: "https://github.com/prog-time/tg-logger"
stars: 23
forks: 1
status: active
toc:
  - id: about
    title: О проекте
  - id: install
    title: Установка
  - id: config
    title: Конфигурация
  - id: usage
    title: Использование
  - id: custom-levels
    title: Собственные уровни
---

## О проекте {#about}

Пакет для Laravel, позволяющий отправлять логи в Telegram-группу с разбивкой по топикам (темам).

Каждый уровень лога (error, warning, info и т.д.) можно направить в отдельный топик — это удобно для мониторинга: критические ошибки не теряются среди информационных сообщений.

Packagist — [https://packagist.org/packages/prog-time/tg-logger](https://packagist.org/packages/prog-time/tg-logger)

## Установка {#install}

```bash
composer require prog-time/tg-logger
```

После установки опубликуйте конфигурацию:

```bash
php artisan vendor:publish --tag=tg-logger-config
```

## Конфигурация {#config}

В файле `config/tg-logger.php` задайте токен бота и настройте каналы:

```php
return [
    'bot_token' => env('TELEGRAM_BOT_TOKEN'),

    'channels' => [
        'errors' => [
            'chat_id'  => env('TELEGRAM_CHAT_ID'),
            'topic_id' => env('TELEGRAM_TOPIC_ERRORS'),
        ],
        'warnings' => [
            'chat_id'  => env('TELEGRAM_CHAT_ID'),
            'topic_id' => env('TELEGRAM_TOPIC_WARNINGS'),
        ],
    ],
];
```

Добавьте переменные в `.env`:

```
TELEGRAM_BOT_TOKEN=your_bot_token
TELEGRAM_CHAT_ID=-100xxxxxxxxxx
TELEGRAM_TOPIC_ERRORS=10
TELEGRAM_TOPIC_WARNINGS=20
```

Затем подключите драйвер в `config/logging.php`:

```php
'channels' => [
    'telegram_errors' => [
        'driver'  => 'custom',
        'via'     => \ProgTime\TgLogger\TelegramLogger::class,
        'channel' => 'errors',
        'level'   => 'error',
    ],
],
```

## Использование {#usage}

Используйте стандартный Laravel-фасад `Log`:

```php
use Illuminate\Support\Facades\Log;

Log::channel('telegram_errors')->error('Что-то пошло не так', [
    'user_id' => $user->id,
    'url'     => request()->fullUrl(),
]);
```

Или настройте стек, чтобы ошибки автоматически дублировались в Telegram:

```php
// config/logging.php
'stack' => [
    'driver'   => 'stack',
    'channels' => ['daily', 'telegram_errors'],
],
```

## Собственные уровни {#custom-levels}

Вы можете описать произвольные уровни логирования, расширив базовый форматтер:

```php
use ProgTime\TgLogger\Formatters\BaseFormatter;

class MyFormatter extends BaseFormatter
{
    public function format(array $record): string
    {
        $emoji = match ($record['level_name']) {
            'ERROR'   => '🔴',
            'WARNING' => '🟡',
            default   => '🔵',
        };

        return "{$emoji} <b>{$record['level_name']}</b>\n{$record['message']}";
    }
}
```

Укажите форматтер в конфиге канала:

```php
'formatter' => MyFormatter::class,
```
