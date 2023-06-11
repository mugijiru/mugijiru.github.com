+++
title = "Ember.js の Integration test で Cannot read properties of undefined (reading 'Symbol(TAG_COMPUTE)') と怒られる時は this が漏れている"
date = 2023-06-11T15:02:00+09:00
draft = false
+++

<https://github.com/glimmerjs/glimmer-vm/issues/1369> の通りなのですが、ちょっとハマったので備忘録に。


## ダメなコード例 {#ダメなコード例}

```js
module("Integration | Component | Hoge", function (hooks) {
  setupRenderingTest(hooks)

  test("renders", function (assert) {
    const item = store.peekRecord('todo-item', 1)
    this.item = item

    // ここの引数がダメ
    await render(hbs(`<HogeComponent @item={{item}}>`))
  })
}
```


## 修正版 {#修正版}

一度 this に詰めてそれを参照させる必要があった

```js
module("Integration | Component | Hoge", function (hooks) {
  setupRenderingTest(hooks)

  test("renders", function (assert) {
    const item = store.peekRecord('todo-item', 1)
    this.item = item

    // this.item を渡す
    await render(hbs(`<HogeComponent @item={{this.item}}>`))
  })
}
```


## おまけ {#おまけ}

これを TypeScript でやる場合は <https://docs.ember-cli-typescript.com/ember/testing#the-testcontext> に書かれているように
Context の interface を用意したりする必要がある

```js
interface Context extends TestContext {
  item: TodoItemModel | null;
}

module("Integration | Component | Hoge", function (hooks) {
  setupRenderingTest(hooks)

  // this: Context が必要
  test("renders", function (this: Context, assert) {
    const item = store.peekRecord('todo-item', 1)
    this.item = item

    await render(hbs(`<HogeComponent @item={{this.item}}>`))
  })
}
```
