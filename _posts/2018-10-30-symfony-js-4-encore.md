---
layout: post
title:  "Encore"
date:   2018-10-30 18:30:00 +0300
categories: symfony tutor encore js
---

# Webpack Encore: A Party for your Assets


### 1. Installing Encore

```bash
composer require asset
```

Вынесли класс Helper  в отдельный файл.

```bash
yarn add @symfony/webpack-encore --dev
```


### 2. Our First Encore

Encore упрощает настройку webpack под симфони.

Пример настройки webpack.config.js взять из документации 
[https://symfony.com/doc/current/frontend/encore/simple-example.html](https://symfony.com/doc/current/frontend/encore/simple-example.html)

Там указываем точку входа и куда билдить

Запускать можно так

```bash
./node_modules/.bin/encore dev --watch
```

или

```bash
yarn run encore dev --watch
```

Для десктопных уведомлений и сборке

```
yarn add webpack-notifier --dev
```


### 3. Require Outside Libraries

Подгружаем библиотеки через require.

```bash
yarn add jquery --dev
yarn add sweetalert2 --dev
```


### 4. Component Organization

Меняем entry point.

И используем модульную систему webpack'a modules.

```js
module.exports = RepLogApp;
```


### 5. Multiple Pages / Entries

Несколько entry point.


### 6. jQuery Plugins / Bootstrap

Bootstrap добавляет плагины для jQuery. 
Но использует глобальный jQuery.
Можно решить 2 способоми:

1. `window.jQuery = $;`
1. `Encore.autoProvidejQuery()`

Но для некоторых старых плагинов jQuery надо сделать так `global.$ = $;`.  global - специально для Webpack


### 7. Require CSS!?

Encore.addEntry() подключает все require() в выходной файл. Если это js - все будут в одном файле. Если добавится css - он будет в .css файле.

Но при импорте есть проблема с относительными путями в background-image. Но Webpack это решает сам. Копируя картинки в новую папку.

Замечание про require(). Когда указываешь без . вначале, библиотека ищется в node_modules. Если указать просто название библиотеки, 
то в этой библиотеке смотрится файл package.json и в нем раздел `main`. 

Тоже со шрифтами.


### 8. Handling Images with the CopyPlugin

Можно вынести assets из public.
Ну кроме картинок, которые мы используем с тегом <img>

Используем 
```
yarn add copy-webpack-plugin --dev
```
И копируем статику.


### 9. Sass & Sourcemaps

Добавил loader `yarn add sass-loader node-sass --dev`

И включил его `Encore.enableSassLoader()`

Так же включаем SourceMaps `Encore.enableSourceMaps(!Encore.isProduction())`


### 10. Integrating FOSJsRoutingBundle

В документации сказано, как правильно можно подключить FosJs 
[https://symfony.com/doc/master/bundles/FOSJsRoutingBundle/usage.html](https://symfony.com/doc/master/bundles/FOSJsRoutingBundle/usage.html)


### 11. ES6 Import & Export

Заменяем module.export на import export ES6.

Можно еще делать multi export
 
```js
import {Helper, foo} from './RepLogHelper';
```

Или так
```js
import isEqual from 'lodash.isequal';
```


### 12. Building for Production

В продакшне должно быть все оптимизировано и минифицировано.

Первая проблема - jQuery подключается в каждый файл.

Для решения перед `.addEntry()` устанавливаем `.createSharedEntry('layout', './assets/js/layout.js')`.
То есть те модули, что есть в shared не будут включены в остальные entries.

Почему-то надо перед shared при подключении файла еще добавить manifest
```html
<script src="{{ asset('build/manifest.js') }}"></script>
```

Последний вопрос - Как деплоить assets на продакшн.
Надо ли ставить node на продакшн.
Это зависит от многого. Можно посмотреть курс по Ansistrano.

### 13. Asset Versioning & Cache Busting

Для этого и нужен manifest. Когда используем cache в имени файла Симфони должна знать, какой хэш файл брать.

**Послесловие.**

Есть еще `enableReactPreset()`  or `enableVueLoader()`