﻿# Плагины

Плагины —  самодостаточная единица кода, которая добавляет во Vue функциональность глобального уровня. Это может быть `object`, предоставляющий метод `install()`, либо `function`.

Нет строго ограниченной области применения для плагинов, но часто распространённые сценарии могут быть такими:

1. Добавление глобальных методов или свойств (например, [vue-custom-element](https://github.com/karol-f/vue-custom-element)).

2. Добавление одного или нескольких глобальных ресурсов: директивы/фильтры/переходы и т.д. (например, [vue-touch](https://github.com/vuejs/vue-touch)).

3. Добавление опций компоненту с помощью глобальной примеси (например, [vue-router](https://github.com/vuejs/vue-router)).

4. Добавление глобальных методов экземпляра, добавляя их в `config.globalProperties`.

5. Библиотека, предоставляющая собственный API и в то же время внедряющая некоторую комбинацию из вышеперечисленного (например, [vue-router](https://github.com/vuejs/vue-router)).

## Создание плагина

Чтобы лучше понять, как создавать собственные плагины Vue.js, создадим упрощённую версию плагина, который отображает локализованные строки (`i18n`).

При добавлении плагина в приложение будет вызываться метод `install`, если указывался объект. При использовании `function` будет вызвана сама функция. В обоих случаях она получает два параметра — объект `app`, являющийся результатом вызова `createApp`, и опции, переданные пользователем.

Начнём с настройки объекта плагина. Рекомендуется создавать его в отдельном файле и экспортировать, как показано ниже, для изолированности и разделения логики.

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    // Код плагина будет здесь
  }
}
```

Нам требуется сделать функцию для перевода ключей, которая будет доступна во всём приложении, поэтому добавим её с помощью `app.config.globalProperties`.

Функция будет получать строку `key`, которую станет использовать для поиска переведённой строки в предоставленных пользователем опциях.

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = key => {
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, options)
    }
  }
}
```

Будем подразумевать, что пользователи при использовании плагина будут передавать объект с переведёнными ключами в параметре `options`. Функция `$translate` будет получать строку ключа вида `greetings.hello`, искать её в предоставленной конфигурации и возвращать переведённое значение — в данном случае, `Bonjour!`

Например:

```js
greetings: {
  hello: 'Bonjour!',
}
```

Плагины также позволяют использовать `inject` для предоставления функции или атрибута пользователям плагина. Например, можно разрешить приложению получать доступ к параметру `options`, чтобы иметь возможность использовать объект с переводами.

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = key => {
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, options)
    }

    app.provide('i18n', options)
  }
}
```

Пользователи плагина теперь смогут использовать `inject['i18n']` в компонентах для получения доступа к объекту.

Кроме того, поскольку есть доступ к объекту `app`, все остальные возможности, такие как использование `mixin` и `directive` доступны в плагине. Подробнее о методе `createApp` и экземпляре приложения можно изучить в [документации API приложения](../api/application-api.md).

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = (key) => {
      return key.split('.')
        .reduce((o, i) => { if (o) return o[i] }, options)
    }

    app.provide('i18n', options)

    app.directive('my-directive', {
      mounted (el, binding, vnode, oldVnode) {
        // какая-то логика ...
      }
      ...
    })

    app.mixin({
      created() {
        // какая-то логика ...
      }
      ...
    })
  }
}
```

## Использование плагина

После инициализации приложения Vue с помощью `createApp()` можно подключить плагин с помощью метода `use()`.

Используем плагин `i18nPlugin`, который создали в разделе [создание плагина](#writing-a-plugin) в демонстрационных целях.

Метод `use()` принимает два параметра. Первый — это устанавливаемы плагин, в данном случае `i18nPlugin`.

Он также автоматически предотвращает подключение одного и того же плагина больше одного раза, поэтому несколько вызовов для установки установят плагин только один раз.

Второй параметр является опциональным и зависит от каждого конкретного плагина. В случае демо-плагина `i18nPlugin`, это будет объект с переведёнными строками.

:::info Информация
При использовании сторонних плагинов, таких как `Vuex` или `Vue Router`, всегда сверяйтесь с документацией, чтобы узнать что ожидает плагин вторым параметром.
:::

```js
import { createApp } from 'vue'
import Root from './App.vue'
import i18nPlugin from './plugins/i18n'

const app = createApp(Root)
const i18nStrings = {
  greetings: {
    hi: 'Hallo!'
  }
}

app.use(i18nPlugin, i18nStrings)
app.mount('#app')
```

Обратите внимание на [awesome-vue](https://github.com/vuejs/awesome-vue#components--libraries) — огромную коллекцию плагинов и библиотек от сообщества Vue.