+++
title = "Ember.js のテストでサーバリクエストの Mock する"
date = 2023-08-16T22:44:00+09:00
categories = ["Ember-js"]
draft = false
+++

Ember.js のテストを書こうとすると Model を扱う時にリクエストを飛ばしちゃうのでそれを回避するのをいくつか試したので記事にしてみる

といいつつ2ヶ月ぐらい前に書こうとしていた記事なので実は細かいところは適当に書いているけど、まあほぼ自分用のつもりで書いているので細かいことは気にしない。


## sinon {#sinon}

JS で mock といえば [sinon](https://sinonjs.org/) って勝手に思ってるぐらいには有名な mock ライブラリ。

API へのリクエストを mock するわけじゃないけど、
API にリクエストを飛ばす関数を mock 化できるやつ。


### セットアップ {#セットアップ}

[tests/test-helper.js](https://github.com/mugijiru/ember-rails-todo-app/blob/97f62e8dc820c6b960c1e2167697528fb16d6f70/ember/todo-app/tests/test-helper.js) で [ember-sinon-qunit](https://github.com/elwayman02/ember-sinon-qunit) を使ってセットアップしている。

具体的なセットアップの処理は

```javascript
import { start } from 'ember-qunit'
import setupSinon from 'ember-sinon-qunit'
...
setupSinon()
start()
```

って感じ。まあ実際のコードの方では他のライブラリを使ったりもするためごにゃごにゃやってるけど。


### 使用例 {#使用例}

実際に使ってる箇所は以下のコンポーネントのテストは以下。

<https://github.com/mugijiru/ember-rails-todo-app/blob/97f62e8dc820c6b960c1e2167697528fb16d6f70/ember/todo-app/tests/integration/components/todo-item-test.ts#L279-L282>

```javascript
const mockedItem = mock(this.item)
assert.ok(mockedItem.expects('destroyRecord').once().returns(true))

await click(deleteButton)
```

`this.item` が [app/models/todo-item.ts](https://github.com/mugijiru/ember-rails-todo-app/tree/97f62e8dc820c6b960c1e2167697528fb16d6f70/ember/todo-app/app/models) で用意した Ember.js の model なのだけど、こいつを削除する処理は DELETE リクエストを飛ばしちゃうので mock に差し替えちゃって削除ボタンを押した時に DELETE リクエストが飛ばないようにしている。

メソッドの差し替えをしているので、そのメソッドの中で他に何か処理をしていてもそれも実行されなくなっちゃうからあんまり積極的にこの手口は使いたくない。便利だけどね


## msw {#msw}

[msw](https://mswjs.io/) は API リクエストを受け付ける mock サーバを立てたりするやつ。

sinon みたいに関数を差し替えるのではなく、
API のリクエスト先をこいつに差し替えていい感じのデータを返させるように使うので関数自体の動きはそのまま利用できて嬉しいやつ。


### セットアップ {#セットアップ}

```text
$ npx msw init public
```

とか実行して public ディレクトリに mockServiceWorker.js を生成していたっぽい。
<https://github.com/mugijiru/ember-rails-todo-app/pull/311/commits/cff01d35dc12768e93fbb0cb719da39a016c0a90>


### 使用例 {#使用例}

具体的には以下で使っている。
<https://github.com/mugijiru/ember-rails-todo-app/pull/311/files#diff-86d5570179e37a3d9080bb91efc9f33e337bb94cc115b36eca8f6300a22712ca>

上のコードから適当に最低限こうしたら良さそう、というのを抜き出すと以下のような感じ。

まず最低限以下のものを import する必要がある。

```javascript
import { setupWorker, rest } from 'msw'
import { TestContext } from '@ember/test-helpers'
import { setupTest } from 'todo-app/tests/helpers'
```

msw 関係は、まあ msw を使うので当然として、
`TestContext` はテストファイル内で `this.worker` という変数を使うにあたって必要になる型定義のために入れている。まあ TypeScript じゃなかったら不要。あと `setupTest` は Ember.js のテストを書く上ではいつも必要なやつ。

次に各テストで使う Context の interface を定義する

```javascript
interface Context extends TestContext {
  worker: ReturnType<typeof setupWorker>
}
```

TypeScript で書いている場合はテスト内で `this.worker` として msw の worker を取れるようにするためにテスト中のコンテキストの型定義で worker に `ReturnType<typeof setupWorker>` を指定してあげる必要がある。めんどいけど。

そして module 内の各テストで this.worker で msw の worker にアクセスできるように
before でセットアップする。
beforeEach じゃないので、module に入った時の一発目だけ実行されるのでずっと使い回される。

```javascript
hooks.before(async function (this: Context) {
  this.worker = setupWorker()
  await this.worker.start({ onUnhandledRequest: 'error', quiet: true })
})
```

before で用意したやつは after で後始末もしてあげる

```javascript
hooks.after(function (this: Context) {
  this.worker.stop()
})
```

また各テスト間で依存が生まれると困るので、それぞれのテストが終わった時に影響を残さないように reset してあげることも大事

```javascript
hooks.afterEach(function (this: Context) {
  this.worker.resetHandlers()
})
```

そしてテスト本体では `this.worker.use()` を使ってリクエスト先とその結果を設定する。

以下は `/api/v1/todo_items/1` にリクエストする場合の mock の書き方の例。このあたりの検証をしている [ember-rails-todo-app](https://github.com/mugijiru/ember-rails-todo-app) では [JSON:API](https://jsonapi.org/) を採用しているのでレスポンス形式もその形になっているが、各自の利用中の REST API に合わせれば良い。

```javascript
this.worker.use(
  rest.get("/api/v1/todo_items/1", (_req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.set('Content-Type', 'application/vnd.api+json'),
      ctx.json({
        data: (
          type: 'todo-item',
          id: 1,
          attributes: {
            name: 'item 1',
            is_completed: false,
            created_at: '2021-01-01T00:00:00.000Z',
            updated_at: '2021-01-01T00:00:00.000Z',
          },
        ),
      })
    )
  }
)
```

レスポンスそのものを指定できるので自由度は高いけど、まあ面倒なのである程度共通化してどこでも同じようなものが使えるようにした方が良いはず。

まあ <https://github.com/visiblevc/ember-msw> ではそのあたりもやってくれる雰囲気があるけどこいつ自体が更新されてないので自分は採用していない


## ember-cli-mirage {#ember-cli-mirage}

[ember-cli-mirage](https://www.ember-cli-mirage.com/) は一応 Ember.js のテストでリクエストを mock する定番のやつっぽい。

<https://github.com/mugijiru/ember-rails-todo-app/pull/312>
で試したけど、
TypeScript 対応もされてないしメンテもされてないみたい。また RSpec の実行時にもこちらが動いてしまっていてそれを調整するのも面倒だったので採用を諦めた。

設定さえしておけば

```javascript
this.server.get('/todo_items', () => {
  return {
    data: [
      {
        type: 'todo-item',
        id: 1,
        attributes: {
          name: 'item 1',
          is_completed: false,
          created_at: '2021-01-01T00:00:00.000Z',
          updated_at: '2021-01-01T00:00:00.000Z',
        },
      },
      {
        type: 'todo-item',
        id: 2,
        attributes: {
          name: 'item 2',
          is_completed: false,
          created_at: '2021-01-01T00:00:00.000Z',
          updated_at: '2021-01-01T00:00:00.000Z',
        },
      },
    ],
  }
})
```

という感じにリクエストに対して何が返って来るのか書くだけで良いのは便利なんだけどね〜。まあ msw も似たような感じで使えるので、無視で良さそう。


## (未検証) OpenAPI + Prism {#未検証--openapi-plus-prism}

msw や ember-cli-mirage ではコード内でレスポンスを記述するので、実際の API とはズレる可能性がある。

ところでこの検証をしている [ember-rails-todo-app](https://github.com/mugijiru/ember-rails-todo-app) では
OpenAPI で API を定義しているのでその定義を使って Mock Server を立てられる [Prism](https://meta.stoplight.io/docs/prism/674b27b261c3c-prism-overview) を使ったら
API の変更時にそれを検知できるテストにできそうだな〜なんて思ってる。今度気が向いたら試したい


## 番外: リクエストせずに store にデータを登録 {#番外-リクエストせずに-store-にデータを登録}

Ember.js は store にデータが入っていたらリクエストを飛ばさずにそっちからデータを引っ張り出してくるのでデータの参照だけで良いなら予め store に突っ込んでおくだけで良かったりする

具体的には
<https://github.com/mugijiru/ember-rails-todo-app/pull/311/files#diff-97b466284d9e4c831442301b25e7f3d5fe60b49d2af4b12a8591c866559a7424>
あたりでやっていて `store.createRecord` で store にデータを作成すればそこからデータを取ってくれる。

```javascript
const store = this.owner.lookup('service:store')
const record = store.createRecord('todo-item', {
  name: 'new',
  isCompleted: false,
})
const serializedRecord: any = record.serialize() // eslint-disable-line @typescript-eslint/no-explicit-any
const attributeNames = Object.keys(serializedRecord.data.attributes)
assert.ok(attributeNames.includes('is_completed'))
```

サーバのことを無視して単体での動作を確認したい時には良いかも
