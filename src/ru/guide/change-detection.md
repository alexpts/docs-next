# Особенности отслеживания изменений Vue 2

> Эта страница относится только к Vue 2.x и ниже и предполагает, что вы уже изучили и разобрались с разделом [Реактивность](reactivity.md). Если нет — прочитайте его сначала.

Вследствие ограничений JavaScript есть изменения, которые Vue **не может обнаружить**. Однако существуют способы обойти эту проблему, чтобы сохранить реактивность.

### Для объектов

Vue не может обнаружить добавление или удаление свойства. Поскольку Vue добавляет геттер/сеттер на этапе инициализации экземпляра, то свойство должно присутствовать в объекте `data` чтобы Vue преобразовал его и сделал реактивным. Например:

```js
var vm = new Vue({
  data: {
    a: 1
  }
})
// теперь `vm.a` — реактивное поле

vm.b = 2
// `vm.b` НЕ РЕАКТИВНО
```

Во Vue нельзя динамически добавлять новые корневые реактивные свойства в уже существующий экземпляр. Тем не менее, можно добавить реактивное свойство во вложенные объекты с помощью метода `Vue.set(object, propertyName, value)`:

```js
Vue.set(vm.someObject, 'b', 2)
```

Аналогично можно использовать метод экземпляра `vm.$set`, который является псевдонимом глобального `Vue.set`:

```js
this.$set(this.someObject, 'b', 2)
```

Иногда нужно добавить несколько свойств в существующий объект, например, с помощью `Object.assign()` или `_.extend()`. Если так поступить, то добавленные свойства не станут реактивными. Для решения потребуется создать новый объект, который будет содержать как поля оригинального объекта, так и объекта с добавляемыми свойствами:

```js
// вместо `Object.assign(this.someObject, { a: 1, b: 2 })`
this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
```

### Для массивов

Vue не может отследить следующие изменения в массивах:

1. Прямую установку элемента по индексу: `vm.items[indexOfItem] = newValue`
2. Явное изменение длины массива: `vm.items.length = newLength`

Например:

```js
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // НЕ РЕАКТИВНО
vm.items.length = 2 // НЕ РЕАКТИВНО
```

Решить первую проблему можно любым из двух способов, оба дают эффект аналогичный `vm.items[indexOfItem] = newValue` плюс запускают реактивные обновления состояния приложения:

```js
// Использовать Vue.set
Vue.set(vm.items, indexOfItem, newValue)
```

```js
// Использовать Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```

Можно использовать метод экземпляра [`vm.$set`](https://ru.vuejs.org/v2/api/#vm-set), который является псевдонимом для глобального `Vue.set`:

```js
vm.$set(vm.items, indexOfItem, newValue)
```

Для решения второй проблемы используйте `splice`:

```js
vm.items.splice(newLength)
```

## Объявление реактивных свойств

Поскольку Vue не позволяет динамически добавлять корневые реактивные свойства, то все корневые поля необходимо инициализировать изначально в экземплярах компонента, хотя бы пустыми значениями:

```js
var vm = new Vue({
  data: {
    // объявляем поле message, содержащее пустую строку
    message: ''
  },
  template: '<div>{{ message }}</div>'
})
// впоследствии задаём значение `message`
vm.message = 'Привет!'
```

Если не задать поле `message` в опции data, то Vue выведет предупреждение, что функция отрисовки пытается получить доступ к несуществующему свойству.

Существуют технические причины этого ограничения: оно позволяет исключить целый класс граничных случаев в системе учёта зависимостей, а также упростить взаимодействие компонента с системами проверки типов. Но гораздо важно то, что с этим ограничением становится проще поддерживать код, так как теперь объект `data` можно рассматривать как схему состояния компонента. Код, в котором реактивные свойства компонента перечислены заранее, намного проще для понимания и поддержки.

## Асинхронная очередь обновлений

На всякий случай напомним, что во Vue обновление DOM выполняется **асинхронно**. Каждый раз, когда обнаруживается изменение в данных, создаётся очередь, которая используется в качестве буфера для этого и последующих изменений, происходящих в текущей итерации ("tick") цикла событий. Если один и тот же наблюдатель срабатывает несколько раз, в очередь он попадёт всё равно лишь единожды. Благодаря использованию буфера и устранению дубликатов, вычисления и манипуляции DOM сводятся к минимуму. В следующей итерации цикла событий Vue разбирает очередь и выполняет актуальные (уже не содержащие дубликатов) обновления. На низком уровне для асинхронной постановки задач в очередь используются `Promise.then`, `MutationObserver` и `setImmediate`, а если они недоступны, то `setTimeout(fn, 0)`.

Итак, если выполнить код `vm.someData = 'новое значение'`, компонент не будет отрисован сразу же. Он обновится в следующей итерации при разборе очереди. Чаще всего эту особенность можно не принимать в расчёт, но иногда бывает нужно дождаться состояния, в которое DOM перейдёт после обновления данных. Хотя прямая манипуляция DOM нежелательна, а системы в целом предпочтительнее проектировать так, чтобы в них были первичные данные, иногда всё же её не избежать. Чтобы выполнить какой-нибудь код только после того, как завершится обновление DOM, можно передать коллбэк в метод `Vue.nextTick(callback)` после изменения данных. Он будет вызван после обновления DOM. Например:

```html
<div id="example">{{ message }}</div>
```

```js
var vm = new Vue({
  el: '#example',
  data: {
    message: '123'
  }
})
vm.message = 'новое сообщение' // изменяем данные
vm.$el.textContent === 'новое сообщение' // false
Vue.nextTick(function() {
  vm.$el.textContent === 'новое сообщение' // true
})
```

Существует также метод экземпляра `vm.$nextTick()`, особенно подходящий для использования внутри компонентов, поскольку он не требует обращения к глобальной переменной `Vue`, а также автоматически связывает контекст `this` коллбэка с текущим экземпляром компонента:

```js
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function() {
    return {
      message: 'не обновлено'
    }
  },
  methods: {
    updateMessage() {
      this.message = 'обновлено'
      console.log(this.$el.textContent) // => 'не обновлено'
      this.$nextTick(function() {
        console.log(this.$el.textContent) // => 'обновлено'
      })
    }
  }
})
```

Поскольку `$nextTick()` возвращает Promise, то можно достичь того же, используя новый синтаксис [async/await из ES2017](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Statements/async_function):

 ```js
  methods: {
    updateMessage: async function () {
      this.message = 'обновлено'
      console.log(this.$el.textContent) // => 'не обновлено'
      await this.$nextTick()
      console.log(this.$el.textContent) // => 'обновлено'
    }
  }
```