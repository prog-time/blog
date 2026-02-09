---
layout: post
title: "Как подключить плагин WYSIWYG в Strapi"
date: 2025-03-04
categories: ["strapi"]
tags: ["javascript", "Strapi"]
---
Для добавления WYSIWYG редактора в Strapi вам необходимо сделать несколько действий. В данной записи я расскажу "по шагам", как настроить поддержку поля с редактором WYSIWYG в ваше приложение на Strapi.

## Установка плагина WYSIWYG

Для начала вам нужно установить плагин для WYSIWYG.

```
npm install @_sh/strapi-plugin-ckeditor
```

## Регистрация плагина в системе Strapi

Далее вам необходимо зарегистрировать ваш плагин в системе. Для этого переходим в **./config/plugins.js** где вам необходимо прописать данные о ранее добавленном плагине.

Если вы ранее не подключали сторонние плагины, то код будет выглядеть примерно так:

```
module.exports = () => ({
  'ckeditor5': {
    enabled: true,
    resolve: './node_modules/@_sh/strapi-plugin-ckeditor',
  },
});
```

Теперь вам необходимо перезапустить сборку и запустить ваш Strapi проект

```
npm run build
npm run develop
```
