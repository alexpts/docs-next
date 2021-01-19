# Обработка крайних случаев

> Подразумевается, что вы уже изучили и разобрались с разделом [Основы компонентов](component-basics.md). Если нет — прочитайте его сначала.

:::tip Примечание
Возможности на этой странице документируют обработку крайних случаев — необычных ситуаций, когда требуются некоторые исключения из правил Vue. Обратите внимание, все они имеют недостатки или ситуации, когда могут быть опасными. Об этом отмечено в каждом случае, поэтому помните о них, когда собираетесь использовать какую-либо из этих возможностей.
:::

## Контролирование обновлений

Благодаря системе реактивности, Vue всегда знает когда требуется выполнить обновление (если используете корректно). Однако есть крайние случаи, когда может потребоваться принудительное обновление, даже если никакие реактивные данные не изменились. Могут быть и другие случаи, когда требуется предотвращать ненужные обновления.

### Принудительное обновление

Если столкнулись с тем, что нужно принудительное обновление во Vue, то в 99.99% случаев где-то совершили ошибку. Например, полагаетесь на состояние, которое не отслеживается системой реактивности Vue, добавляя новое свойство в `data` после создания компонента.

Однако, если все возможные варианты исключены и это оказалась крайне редкая ситуация, связанная с необходимостью принудительного обновления вручную, то это можно сделать с помощью [`$forceUpdate`](../api/instance-methods.md#forceupdate).

### Дешёвые статические компоненты с помощью `v-once`

Отрисовка простых HTML-элементов во Vue происходит очень быстро, но иногда бывают компоненты, в которых **очень-очень много статичного содержимого**. Для таких случаев, можно убедиться, что он будет вычислен только один раз, а потом закэшируется, указав директиву `v-once` на корневом элементе, например так:

```js
app.component('terms-of-service', {
  template: `
    <div v-once>
      <h1>Условия использования</h1>
      ... много-много статичного содержимого ...
    </div>
  `
})
```

:::tip Совет
Повторимся ещё раз, старайтесь не злоупотреблять этим шаблоном. Хоть это и удобно в тех редких случаях, когда приходится отрисовывать много статичного содержимого, это попросту не нужно, если только на самом деле не заметили замедления в отрисовке — кроме того, это может добавить путаницы в будущем. Представьте себе разработчика, который не знаком с `v-once` или пропустил её использование в шаблоне. Можно потратить часы, выясняя почему шаблон не обновляется корректно.
:::