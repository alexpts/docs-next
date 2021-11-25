# Спецификация синтаксиса SFC

## Введение

Файл `*.vue` — пользовательский формат файла, использующий HTML-подобный синтаксис для описания компонента Vue. Каждый файл `*.vue` состоит из трёх типов языковых секций верхнего уровня: `<template>`, `<script>` и `<style>`, а также могут быть дополнительные пользовательские секции:

```vue
<template>
  <div class="example">{{ msg }}</div>
</template>

<script>
export default {
  data() {
    return {
      msg: 'Привет мир!'
    }
  }
}
</script>

<style>
.example {
  color: red;
}
</style>

<custom1>
  Это может быть например документация к компоненту.
</custom1>
```

## Языковые секции

### `<template>`

- В каждом файле `*.vue` может быть не более одной секции `<template>` верхнего уровня.

- Содержимое будет извлечено и передано `@vue/compiler-dom`, где предварительно скомпилируется в JavaScript render-функцию и присоединено к экспортируемому компоненту в качестве его опции `render`.

### `<script>`

- В каждом файле `*.vue` может быть не более одной секции `<script>` (за исключением случаев использования [`<script setup>`](sfc-script-setup.md)).

- Скрипт выполняется как ES-модуль.

- **Экспорт по умолчанию**  должен быть объектом опций компонента Vue, либо обычным объектом, либо как возвращаемое значение [defineComponent](global-api.md#definecomponent).

### `<script setup>`

- В каждом файле `*.vue` может быть не более одной секции `<script setup>` (не считая обычных секций `<script>`).

- Секция предварительно обрабатывается и используется в качестве функции компонента `setup()`, что означает, что она будет выполняться **для каждого экземпляра компонента**. Привязки верхнего уровня в `<script setup>` автоматически объявляются в шаблоне. Более подробную информацию можно найти на [специальной странице документации про `<script setup>`](sfc-script-setup.md).

### `<style>`

- В одном файле `*.vue` может быть несколько секций `<style>`.

- Секция `<style>` может иметь атрибуты `scoped` или `module` (подробнее см. в разделе [возможности стилей SFC](sfc-style.md)), для помощи в инкапсуляции стилей для текущего компонента. Несколько секций `<style>` с разными режимами инкапсуляции могут использоваться и совместно в одном компоненте.

### Пользовательские секции

Дополнительные пользовательские секции могут быть добавлены в файл `*.vue` для любых нужд, требуемых для проекта, например секцию `<docs>`. Некоторые реальные примеры использования пользовательских секций:

- [Gridsome: `<page-query>`](https://gridsome.org/docs/querying-data/)
- [vite-plugin-vue-gql: `<gql>`](https://github.com/wheatjs/vite-plugin-vue-gql)
- [vue-i18n: `<i18n>`](https://github.com/intlify/bundle-tools/tree/main/packages/vite-plugin-vue-i18n#i18n-custom-block)

Обработка пользовательских секций зависит от инструментарий — если требуется создать собственные интеграции пользовательских секций, обратитесь к разделу [инструментарий SFC](sfc-tooling.md#интеграция-пользовательских-блоков) для более подробной информации.

## Автоматическое определение `name`

Однофайловые компоненты автоматически определяют имя компонента по его  **имени файла** в следующих случаях:

- Форматирование предупреждений при разработке
- Инспектирование в DevTools
- Рекурсивная ссылка на самого себя. Например, файл с именем `FooBar.vue` может ссылаться на себя как `<FooBar/>` в своём шаблоне. Это имеет более низкий приоритет, чем явно зарегистрированные/импортированные компоненты.

## Пре-процессоры

В секциях можно объявить язык пре-процессора с помощью атрибута `lang`. Наиболее распространённый случай — использование TypeScript для секции `<script>`:

```html
<script lang="ts">
  // используем TypeScript
</script>
```

Атрибут `lang` можно применить к любой секции — например можно использовать [SASS](https://sass-lang.com/) в секции `<style>` и [Pug](https://pugjs.org/api/getting-started.html) в секции `<template>`:

```html
<template lang="pug">
p {{ msg }}
</template>

<style lang="scss">
  $primary-color: #333;
  body {
    color: $primary-color;
  }
</style>
```

Обратите внимание, что интеграция с пре-процессорами может отличаться в зависимости от используемого инструментария. Смотрите примеры в соответствующих документациях:

- [Vite](https://vitejs.dev/guide/features.html#css-pre-processors)
- [Vue CLI](https://cli.vuejs.org/ru/guide/css.html#%D0%BF%D1%80%D0%B5-%D0%BF%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D0%BE%D1%80%D1%8B)
- [webpack + vue-loader](https://vue-loader.vuejs.org/ru/guide/pre-processors.html#%D0%B8%D1%81%D0%BF%D0%BEn%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BF%D1%80%D0%B5-%D0%BF%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D0%BE%D1%80%D0%BE%D0%B2)

## Импорты через src

Если предпочитаете разделять компоненты `*.vue` на несколько файлов, то можно использовать атрибут `src` для импорта внешнего файла для языковой секции:

```vue
<template src="./template.html"></template>
<style src="./style.css"></style>
<script src="./script.js"></script>
```

Импорты через `src` следуют тем же правилам разрешения путей, что и запросы модулей webpack, что означает:

- Относительные пути должны начинаться с `./`
- Можно импортировать ресурсы из зависимостей npm:

```vue
<!-- импорт файла из установленного npm-пакета "todomvc-app-css" -->
<style src="todomvc-app-css/index.css">
```

Импорты через `src` также работают с пользовательскими секциями, например:

```vue
<unit-test src="./unit-test.js">
</unit-test>
```

## Комментарии

Внутри секции следует использовать синтаксис комментариев используемого языка (HTML, CSS, JavaScript, Pug, и т.д.). Для комментариев верхнего уровня следует использовать синтаксис HTML-комментариев: `<!-- какой-то комментарий верхнего уровня -->`