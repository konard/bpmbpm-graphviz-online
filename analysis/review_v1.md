# Анализ сервисов "Graphviz Online"

## Задача анализа

Найти сервисы "graphviz online", которые одновременно:
1. Могут передавать код dot как параметр URL
2. Могут отображать изображения через тег `image="`

## Исследованные сервисы

### 1. dreampuf.github.io/GraphvizOnline

- **URL-параметр**: ✅ Да — код dot передаётся в хэш URL (fragment), например:
  `https://dreampuf.github.io/GraphvizOnline/#digraph%20G%20%7B...%7D`
- **Тег `image="`**: ❌ Нет — не поддерживается

**Причина отказа `image="`**: Сервис использует Graphviz, скомпилированный в WebAssembly (WASM), который работает в браузере. Атрибут `image=` в Graphviz предназначен для чтения файлов с файловой системы, а не загрузки по HTTP. WASM-версия использует виртуальную файловую систему и физически не может загрузить изображения по URL.

Источник: исследование https://github.com/bpmbpm/family-tree/pull/30

### 2. www.devtoolsdaily.com/graphviz

- **URL-параметр**: ✅ Да — поддерживает передачу кода через URL
- **Тег `image="`**: ❌ Нет — та же проблема WASM

### 3. edotor.net

- **URL-параметр**: ❌ Нет — не позволяет передать код схемы через URL-параметр
- **Тег `image="`**: ❌ Нет

### 4. magjac.com/graphviz-visual-editor

- **URL-параметр**: ❌ Нет — не позволяет передать код через URL
- **Тег `image="`**: Частично — использует d3-graphviz с `addImage()`, но требует явного указания размеров

### 5. viz-js.com

- **URL-параметр**: ❌ Нет
- **Тег `image="`**: ❌ Нет (использует viz.js / @viz-js/viz, та же WASM-проблема)

## Вывод

**Ни один из существующих сервисов не поддерживает оба условия одновременно.**

Причина технического ограничения: все существующие браузерные Graphviz-решения используют Graphviz, скомпилированный в WebAssembly. Движок Graphviz при обработке `image="..."` обращается к файловой системе, а виртуальная ФС в WASM не может делать HTTP-запросы.

## Почему family-tree/ver2 отображает узлы корректно, а dreampuf/GraphvizOnline — нет

`family-tree/ver2/index.html` использует библиотеку `@viz-js/viz` с методом `viz.renderString(dotStr, { images: [...] })`.
Опция `images` позволяет передать массив `[{ name: 'url', width: px, height: px }]` прямо в движок WASM
до начала рендеринга — Graphviz получает реальные размеры изображений и правильно вычисляет layout узлов
(с `imagepos=tc`, `labelloc=b`).

`dreampuf/GraphvizOnline` не передаёт `images` — WASM не знает размеры картинок и не может отобразить их.

## Решение (реализовано в ver1/index.html)

Для реализации одновременной поддержки URL-параметров и тега `image="`:

1. Использовать библиотеку **@viz-js/viz** (тот же движок, что и в `family-tree/ver2`) с методом `viz.renderString()`.
2. До вызова рендеринга парсить код dot и извлекать все URL из атрибутов `image="..."`.
3. Загружать каждое изображение через браузер (`new Image()`), чтобы получить реальные размеры.
4. Передавать массив `images` в `viz.renderString()` — это регистрирует размеры в WASM, обеспечивая корректный layout.
5. После рендеринга SVG браузер сам загружает реальные изображения по URL через стандартный `<image href="...">` элемент SVG.

Таким образом, **layout-вычисления** выполняются в WASM (с правильными размерами), а **отображение изображений** — браузером (по HTTP).

Дополнительно в `ver1/index.html` реализованы:
- Выбор движка Graphviz (dot, neato, fdp, sfdp, circo, twopi)
- Кнопки управления масштабом (+, −, Сброс, Вписать)
- Кнопка скачивания SVG

## Ссылки

- [@viz-js/viz: опция images в renderString](https://github.com/nicowillis/viz.js#renderstring)
- [Исследование проблемы в family-tree PR #30](https://github.com/bpmbpm/family-tree/pull/30)
- [Описание проблемы GraphvizOnline vs foto](https://github.com/bpmbpm/family-tree/blob/main/design/problem.md#1-graphvizonlin-vs-foto)
