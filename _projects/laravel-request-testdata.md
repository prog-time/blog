---
layout: project
title: "laravel-request-testdata"
description: "Laravel пакет для автоматической генерации тестовых данных на основе правил валидации"
tags: ["Open Source", "PHP", "Laravel"]
technologies: ["PHP", "Laravel", "Composer"]
github: "https://github.com/prog-time/laravel-request-testdata"
stars: 11
forks: 0
status: active
---

## О проекте

PHP-библиотека для автоматической генерации тестовых данных на основе правил валидации Laravel Request.

Пакет анализирует правила валидации в FormRequest-классах и автоматически создаёт корректные тестовые данные, что упрощает написание тестов и снижает количество boilerplate-кода.

Packagist — [https://packagist.org/packages/prog-time/laravel-request-testdata](https://packagist.org/packages/prog-time/laravel-request-testdata)

## Установка

```bash
composer require prog-time/laravel-request-testdata
```

## Использование

Для генерации тестовых данных используйте статический метод `generate` класса `RequestDataGenerator`. Метод принимает объект Request и возвращает массив с тестовыми данными:

```php
$request = new ExampleRequest();
$testData = RequestDataGenerator::generate($request);
```

## Кастомизация тестовых данных

Если в Request-классе определить метод `requestTestData`, его значения будут использованы при генерации данных:

```php
class ExampleRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'email' => 'required|email',
            'age' => 'required|integer|min:18',
        ];
    }

    public function requestTestData(): array
    {
        $faker = \Faker\Factory::create();
        return [
            'email' => $faker->email(),
            'age' => 25,
        ];
    }
}
```
