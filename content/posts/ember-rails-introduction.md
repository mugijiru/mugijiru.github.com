+++
title = "ember-rails アプリケーション保守入門"
date = 2021-05-24T09:02:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

ちょっとした事情で Ember.js 入門的なサムシングをすることになったけどどうせなら公開情報にしちゃえって気持ちになったので資料化する前にブログの記事にしちゃうぞ、というエントリです。


## 想定する読者 {#想定する読者}

携わっているシステムが ember-rails を利用して作られているために令和になっても 2018 年にサポートが切れてしまった Ember.js@2.18 のアプリケーションをなんとか動かし続けないといけない哀れな子羊たち


## 記事を書いている人 {#記事を書いている人}

Rails と Ember.js と Vue.js での開発をしたことがある人。
React や Angular はやったことない。なので比較には Rails と Vue.js を出しがち


## この記事で書かないこと {#この記事で書かないこと}

既に存在してしまっている ember-rails アプリケーションの保守をする人達向けの記事なので、
ember-rails を使って新規で Ember.js アプリを構築する、みたいなことは書きません。

それに今更 ember-rails で構築するのはよろしくないですし、
Rails で Ember.js やりたいなら [ember-cli-rails](https://github.com/thoughtbot/ember-cli-rails) にしておいた方が良いですよ。

どうしても構築から知りたい人は
[ember-rails-todo-app で ember-rails アプリを構築したあたりのコミット](https://github.com/mugijiru/ember-rails-todo-app/compare/9f190efb6320e19a80768b0c6a37e1929e9c4146...b18270e90c694f14e0fac5df2cb9aadad41376c5)
を見たりとかしたらなんとなくわかるかもだけどまああんまり要らないよね。


## Ember.js と ember-rails の簡単な紹介 {#ember-dot-js-と-ember-rails-の簡単な紹介}


### Ember.js とは {#ember-dot-js-とは}

[Ember.js](https://emberjs.com/) はWebフロントエンドMVCフレームワークの1種。昔流行ったよね WebフロントエンドMVC。残念ながら最近はもう主流ではない。

Ruby on Rails の影響を受けてるようなので、
Rails エンジニアが取っ付きやすいフレームワークになっている。多分。


### ember-rails とは {#ember-rails-とは}

[ember-rails](https://github.com/emberjs/ember-rails) は Ember.js をいい感じに Rails と連携してくれる素敵な Gem です。これを使うと Rails 上で動く Ember.js アプリケーションが簡単に作れる。

Ember.js 自体が [ember-source](https://rubygems.org/gems/ember-source/versions/2.18.1?locale=ja) という Gem を提供してくれていたのでそれらのソースも依存関係で入って来てくれる便利なやつ。

だけどそういう蜜月も Ember.js@2.18 の頃までのお話なので今から新規で Rails の上で Ember.js を触りたい子は大人しく [ember-cli-rails](https://github.com/thoughtbot/ember-cli-rails) を使いましょう。


## Rails 側の設定 {#rails-側の設定}


### Gem の install {#gem-の-install}

```ruby
gem 'ember-rails'
gem 'active_model_serializers', '~> 0.9.0'
```

多分このあたりを入れておけば良い。
active\_model\_serializers は
[ember-rails の gemspec でバージョン指定がされてない](https://github.com/emberjs/ember-rails/blob/3ff45327d1320376715b365f0319192a35dc1d56/ember-rails.gemspec#L16) けど
Ember.js 側で使う active-model-adapter が 0.9 系対応なのでバージョンをこちらで固定している。

実は ember-rails-todo-app 1.x では
[jquery-rails や ember-source も指定している](https://github.com/mugijiru/ember-rails-todo-app/blob/21241d30fe4f9987760bd18bc2341b1943de26fe/Gemfile#L39-L42) けど多分わざわざ指定しないでいい気もする。


### config.handlebars.template\_root {#config-dot-handlebars-dot-template-root}

ember-rails ではこれで指定されているフォルダが Ember.js の Template として利用される。デフォルト設定だと `app/assets/javascriptes/templates` が指定されている。

複数の Ember.js App が動くようになってる場合には
config/application.js あたりで複数設定されているはず。

もし新しくテンプレートを放り込む場所を増やしたかったらここを弄る必要あり


## Ember.js の基礎 {#ember-dot-js-の基礎}


### EmberObject {#emberobject}

Ember.js では EmberObject というものが全体的に使われている。

こいつは Ember.js で使う Class 的なやつのベースとなる機能を提供する。例えば Computed Property や Observer がそれにあたる。

以下では EmberObject のよく使う機能について
Vue.js の props, data, computed, watch, methods あたりと比較しながら解説する。


#### 通常のプロパティ {#通常のプロパティ}

Vue.js における data にあたるもの。

EmberObject は、JS の Object でもあるので、普通に JS Object のプロパティとして定義したら Vue.js の data 的に扱える。

```js
export default Ember.Object({
  width: 200,
  height: 200
})
```

参照したり更新したりする時は

```js
this.get('width') // 参照
this.set('width', 300) // 更新
```

のような書き方になります。正直だるい。

で、上で「Vue.js でいう data にあたるもの」と書いたが、
Vue.js でいう props と data 的な区別は特にないので、外から簡単に書き換えられる値でもある。

<!--list-separator-->

-  3系では

    3系だと EmberObject は使わなくなって JS の Native Class なので

    ```js
    width = 200
    height = 200

    hoge() {
      this.width = 200
    }
    ```

    みたいな感じに書ける。

    また Vue.js でいう props 的なのは
    `this.args` の中に閉じ込められるし、値が変に上書きされることもない。便利。


#### Computed Property {#computed-property}

Vue.js の Computed Property のように通常のプロパティから新しい値を導出してプロパティとして使えるようにするやつ。
Vue.js のそれと同様にキャッシュされるしリアルタイムに反映される。

書き方は Vue.js よりちょっとだるくて

```js
fullName: Ember.computed('firstName', 'lastname', function () {
  return `${this.get('firstName')} ${this.get('lastName')}`
})
```

みたいな感じ。
Vue.js だと

```js
computed: {
  fullName() { return `${this.firstName} ${this.lastName}` }
}
```

で済むのになって思うことはある。

ただ、別名でも取れるようにしたり(alias)とか、全部足したり(sum)とかのよくあるやつは関数が定義済なので a と b を足した c が欲しい場合は

```js
c: Ember.computed.sum('a', 'b')
```

とか書けるのでそれは便利かもしれない。
<https://api.emberjs.com/ember/2.18/modules/@ember%2Fobject#functions-computed>

<!--list-separator-->

-  3系では

    3.15 以降だと
    [Tracked Property](https://guides.emberjs.com/release/upgrading/current-edition/tracked-properties/) というのを使うのが推奨されている。まだ使えるけどね Computed も。

    \`@tracked\` というデコレータが付与されたプロパティは値の変更が検知されるようになる。

    また 3.15 以降では JS の Native Class になるので
    Tracked Property と Getter を使って

    ```js
    @tracked firstName
    @tracked lastName
    get fullName() { return `${this.firstName} ${this.lastName}` }
    ```

    みたいに書ける。便利。


#### Observer {#observer}

Vue.js の watch のようなやつ。値を監視して、変更があったら関数が実行されるやつ。

```js
Ember.Observer('hoge', function () {
  console.log('"hoge" is changed!')
})
```

みたいな感じで使う。

<!--list-separator-->

-  3系では

    Observer もまだ使えるけど、
    Tracked Property を使ってなんとかするのが推奨されている。

    上の例みたいにログだけ残したい時どうしたらいいんだろうね。


#### Methods {#methods}

これは普通に関数を書けばいいだけ。

```js
hoge () {
  // なんらかの処理
}
```

`methods` とかいうところに書かなくていい分 Vue.js より楽ではという気もする。


### Handlebars {#handlebars}

Ember.js で採用されているテンプレートエンジンです。変数の呼び出しに `{{foo}}` みたいな感じにするやつです。

Ember.js ではコンポーネントを呼び出す時にも
`{{my-component}}` みたいに使います。

上記の書き方はインラインコンポーネントの呼び出し方でブロックコンポーネントは

```hbs
{{#my-block-component}}
  ...
{{/my-block-component}}
```

みたいな感じになります。

変数を渡す時は

`{{my-component hoge=fuga}}`

みたいな感じ。
Vue.js と比べて見た目以外はそんなに変わらないかなって。


### Ember.js のフォルダ・ファイル構成 {#ember-dot-js-のフォルダ-ファイル構成}

ここでは ember-rails を前提として話します。

| フォルダ・ファイル    | 概要                                            |
|--------------|-----------------------------------------------|
| adapters              | サーバとの通信周りを設定するコードを置く場所    |
| components            | コンポーネント定義の JS ファイルを入れるところ  |
| controllers           | Rails でもお馴染みの Controller                 |
| initializers          | 初期化用コードを入れるところ                    |
| instance-initializers | アプリ起動直後の初期化コードを入れるところ      |
| helpers               | Rails でもお馴染みに Helper                     |
| mixins                | Rails だと Concern にあたるものを置くところ     |
| models                | Rails でもお馴染みの Model                      |
| routes                | Controller 実行前に Model などを取得したりするところ |
| routor.js             | Routing を設定するところ。Vue.js の Router と似てるかも? |
| serializers           | サーバと通信する前に形式変換するコードを置く場所 |
| templates             | Handlebars という HTML テンプレートを置く場所。Rails でいう views |
| views                 | 使わない。Ember.js@1系の名残りかと              |


## Ember.js アプリにアクセスした時の処理の流れ {#ember-dot-js-アプリにアクセスした時の処理の流れ}

ざっくり書くと

1.  EMBER\_ROOT/application.js.es6 のコード実行
2.  EMBER\_ROOT/{APP\_NAME}.module.es6
3.  initializer 実行
4.  instance-initializer 実行
5.  routor/routes の実行
    1.  beforeModel()
    2.  model()
    3.  afterModel()
6.  Controller の実行
7.  templates のレンダリング

という流れ。これは Ember.js@3.26 でも大体似てる。1,2 あたりがちょっと違うけど。


### EMBER\_ROOT/application.js.es6 のコード実行 {#ember-root-application-dot-js-dot-es6-のコード実行}

単にこのあたりで色々なものを require したり
Ember.js App を呼び出したりしているのでここからスタートだよねってだけ。

Sprockets で require した順に実行されるので、以下の順序は大事っぽい。

```js
//= require ./environment
//= require ember
//= require ember-data
//= require ember-rails/application
//= require active-model-adapter
```


### EMBER\_ROOT/{APP\_NAME}.module.es6 {#ember-root-app-name-dot-module-dot-es6}

require でアプリケーション本体の色々なコードを読み込んだり設定を読み込んだりしている部分。

require される対象は Class みたいなものなのであまり読込順は関係なさそう。

module 化されてない場合は
`require router` と `require_self` は最後じゃないとまずそうだけども。


### initializer 実行 {#initializer-実行}

Ember.js App の起動時に実行される処理。

initializers フォルダに配置されていて

```js
export function initializer(application) {
  ...
}
```

みたいに書かれているファイルが実行される。アプリケーション初期化用のコード(ex: Websocket の初期化)とか
DI で組込みたい service の injection のコードを入れておくと良いっぽい。

initializer が複数ある場合の実行順はどうなってるかわからんけど
[「これより先に」「これより後で」みたいな指定](https://guides.emberjs.com/v2.18.0/applications/initializers/#toc%5Fspecifying-initializer-order) はできる。


### instance-initializer 実行 {#instance-initializer-実行}

Ember.js App が起動直後に動く処理。

instance-initializers フォルダに配置されていて

```js
export function instanceInitializer(application) {
  ...
}
```

みたいに書かれているファイルが実行される。インスタンス化・起動後に動くので、
A/B テストみたいに見ている人毎に違う条件を埋め込むとかしたい時に使うのが良いらしい。


### routor/routes の実行 {#routor-routes-の実行}

初期化処理が済んだら routor が実行される。

```js
Router.map(function () {
  this.route('root', { path: '/' })
  this.route('posts', { path: '/posts' }, function () {
    this.route('new')
  })
})
```

みたいに書かれていて、
path にマッチした route が実行されるようになっている。
Vue Router なんかもそういう作りだよね。

Ember.js の Router は Vue Router とは違って直接コンポーネントが呼び出されるのではなく
routes 以下にある Route が呼び出されるようになっている。読まれるファイルは `this.route` の第一引数にマッチしたやつですね。

上の例では `/` というところにアクセスした場合には
`routes/root.module.es6` が実行されるという仕組み。


#### beforeModel() {#beforemodel}

`model()` が実行されるより前に動く処理。ログイン状態チェックしてログインしてなければ別のルートに飛ばす、みたいに、あまりアプリケーションのModelと関係ない処理をする時に使えば良さそう。

正直あまり出番ない。


#### model() {#model}

その画面で使う Model を取得する処理。ここで return された値が Controller や Template で model としてアクセスできるようになる。

また Promise を返した場合は、それが解決された時の戻値を model として使えるようにしてくれる。

store.findAll などは Promise を返すが、
Promise の解決結果を Model とする挙動のおかげで

```js
model() {
  return this.store.findAll('posts')
}
```

とした場合に Controller などでは `this.get('model')` で `posts` が取得できるようになっている。

<!--list-separator-->

-  複数の model

    1画面で複数の Model を使いたい場合には

    ```js
    return {
      foo: hoge,
      bar: fuga
    }
    ```

    みたいに書けば良い。

    Controller とかで `this.get('model.foo')` みたいにして扱える。


#### afterModel(model) {#aftermodel--model}

model を取得した後で、画面を表示する前にやっておきたい処理を書くところ。第一引数で model が渡されてくるので、
model の状態を見て別の route に飛ばすとか、controller に値を放り込むとかの処理ができる。


#### 入れ子になった Route {#入れ子になった-route}

上の例にも書いたけど Route はネストさせることもできる。

例えば `/posts/new` という route の場合は画面表示的には `/posts` というレイアウトの中に登録フォームを表示する、みたいな時に使う

このあたりは [Vue Router の Nested Route](https://router.vuejs.org/ja/guide/essentials/nested-routes.html) と似てるよね。どっちが先か知らんけど。

で、そうやってネストさせている場合は、一番外側の Route から順に実行されるような感じになっている。

つまり上記の例だと `posts.module.es6` の `model()` なんかが実行された後に
`/posts/new.module.es6` の内容が実行される。

上位の path の route が既に実行されているので下位の path で上位の model 情報を使いたい場合、上記の例だと `/posts/new` で `posts` の model 情報を使いたい場合には
[modelFor()](https://api.emberjs.com/ember/3.26/classes/Route/methods/modelFor?anchor=modelFor) という関数で `const posts = modelFor('posts')` みたいに取得できる。


### Controller の実行 {#controller-の実行}

Route での処理が済んだら Controller の処理が実行される。というか Route から [setupController](https://api.emberjs.com/ember/release/classes/Route/methods/setupController?anchor=setupController) というので呼び出される。

呼び出される Controller は route と同じように path にマッチしたやつになる。つまり `/posts` という path なら
`controllers/posts.module.es6` で定義している Controller が呼び出される。

Controller では Template で使う値を設定したり Action を定義したりする。


### Template のレンダリング {#template-のレンダリング}

Controller の処理が済んだら Template のレンダリングに入る。呼び出される Template は Controller の path と一致する。

つまり `/posts` という path なら `templates/posts.hbs` が呼び出される。

Template は Handlebars 形式で記述する。


## Ember.js の Component {#ember-dot-js-の-component}

Ember.js の開発で一番弄る機会の多いのが Component です。

components フォルダに JS のロジックが書かれたファイルが置かれ、対応するテンプレートは、
templates/components フォルダに Handlebars で記述した hbs ファイルとなる。

JS のロジックが書かれる Component は [EmberObject](#emberobject) がベースなのでそれらの機能が使える。さらに Component 用の機能が追加されている。


### 値の渡し方 {#値の渡し方}

[Handlebars](#handlebars) の方でも書いたけど
`{{my-component hoge=fuga}}` というように書けば良いです。

そうすると MyComponent 内で
`this.get('hoge')` で値が取得できるようになり、そのテンプレート内で `{{hoge}}` で参照できるようになります。


### Component のタグのカスタマイズ {#component-のタグのカスタマイズ}

古い Ember.js では Component を呼び出すと
Wrapper となる div が挿入されてその中に Template で書いた記述が入るようになっています。


#### tagName {#tagname}

Wrapper のタグを指定する。
ul の下にリストアイテムをコンポーネントとして配置したい時は
`tagName: 'li'` とかやったりする。


#### classNames {#classnames}

Wrapper タグに付与する class のリスト。こちらは固定の値になる。


#### classNameBindings {#classnamebindings}

Wrapper タグに、条件に応じて付与する class のリスト的な。
Vue.js の class の Binding に似ていて

```js
isActive: false
classNameBindings: ['isActive:active']
```

みたいに書くと isActive が true の時に \`active\` というクラスが付与される。

Vue.js だと

```html
<div :class="{ active: isActive }">
```

となるので書き方が逆だけどね。


#### 3系では {#3系では}

3.15 以降では Vue のコンポーネント同様に余計な Wrapper が入らなくなります。なので tagName, classNames, classNameBindings とは全ておさらばとなります。


### Actions {#actions}

テンプレート内のボタンを叩いた時に動かしたい関数なんかを記述するところ。

```js
actions: {
  hoge(){
    // なんか処理を書く
  }
}
```

みたいな感じで書く。


#### Template で直接使う場合 {#template-で直接使う場合}

Template から直接アクションを呼びたい時は

```hbs
<button type="}button" {{action "hoge"}}>
```

みたいに書くと、そのボタンを押した時に hoge アクションが動く。
<https://guides.emberjs.com/v2.18.0/components/triggering-changes-with-actions/#toc%5Fdesigning-the-child-component>


#### 下位のコンポーネントに渡す場合 {#下位のコンポーネントに渡す場合}

```hbs
{{child-component foo=(action "hoge")}}
```

というように書いて渡すと、child-component では
foo というメソッドが定義されていて、その実体は親コンポーネントで定義した hoge アクション、という状態になる

なので child-component 内のアクションを

```nil
actions: {
  fuga() {
    this.get('foo')()
  }
}
```

というように書いて child-component のテンプレートで fuga アクションを実行したら親コンポーネントの hoge アクションが実行される、みたいな挙動になる

参考: <https://guides.emberjs.com/v2.18.0/components/triggering-changes-with-actions/#toc%5Fpassing-the-action-to-the-component>


### LifeCycle Hooks {#lifecycle-hooks}

[なんかめっちゃ Hook が用意されている。](https://guides.emberjs.com/v2.18.0/components/the-component-lifecycle/)


#### 初期化時 {#初期化時}

1.  init
2.  didReceiveAttrs
3.  willRender
4.  didInsertElement
5.  didRender

init と didRender あたりは割と使うかな?
didRender は再描画でも使うので両方に適用したい時に使う感じ。

didInsertElement もたまに使う気がする。


#### 再描画時 {#再描画時}

1.  didUpdateAttrs
2.  didReceiveAttrs
3.  willUpdate
4.  willRender
5.  didUpdate
6.  didRender

didRender は割と使う。
didUpdate は初期化時には動かしたくないけど didRender 的に使いたい時に使う感じ。他はあまり使った記憶なし。


#### 破壊時 {#破壊時}

1.  willDestroyElement
2.  willClearRender
3.  disDestroyElement

このあたりはあまり使った記憶がない


#### Hook の使い方 {#hook-の使い方}

使い方は2種類あって

```js
init () {
  this._super(...arguments) // 通常の処理をさせておく
  // ここに特別な処理を書く
}
```

というように hook 関数を Override する方法と

```js
hoge: Ember.on('init', functoin () {
  // ここに特別な処理を書く
})
```

というように [on メソッド](https://api.emberjs.com/ember/2.18/functions/@ember%2Fobject%2Fevented/on) を使う方法がある。

Rails の Model で

```ruby
after_save do
  # ここに特別な処理を書く
end
```

と書くか

```ruby
after_save :hoge

private
def hoge
  # ここに特別な処理を書く
end
```

と書くかの違いみたいなものですね。

私は後者の方が好み。間違えて二重定義したりしなさそうだし `this._super(...arguments)` を呼び忘れもしなさそうなので。

まあ前者の方が `this._super` の呼び出しタイミングをズラせる自由さはあるけどね。


### Block Component {#block-component}

[Block Component](https://guides.emberjs.com/v2.18.0/components/block-params/) では

my-component のテンプレートを

```hbs
<div>
  {{yield}}
  ↑親から来たやつ↑
</div>
```

みたいに書いてる状態で親から

```hbs
{{#my-component}}
  hogehoge
{{/my-component}}
```

みたいに呼び出すと

```html
<div>
  hogehoge
  ↑親から来たやつ↑
</div>
```

という感じになるやつ。

Rails のテンプレートの yield と似てる気がするよね。あと Vue.js の slot にも似てる気がするよね。複数 Slot みたいなことはできなさそうだけども。

本当はもうちょっと複雑なこともできるんだけど入門記事だしそこまで書くのはだるいので割愛。というか [Block Component](#block-component) のリンク先読んで。


### フォルダ階層分け {#フォルダ階層分け}

最新の Ember.js だと素直にいけるのに ember-rails かつ古い書き方だと苦労する問題の1つ。

[ember-rails でコンポーネントをサブフォルダに配置する]({{< relref "ember-rails-component-in-subfolder" >}}) に全部書いているけどざっくり書くと
template で `{{hoge/fuga}}` というように呼び出した場合に
GlobalResolver では Hoge という名前空間の Fuga というコンポーネントを探しに行くようになっている。

つまりコンポーネントを
`App.HogeFugaComponent` ではなく
`Hoge.FugaComponent` として定義する必要がある。そして名前空間 Hoge を `window.Hoge = Ember.Namespace.create()` みたいにして用意しておく必要もある。だるい。

試したことはないけど、さらに1階層増やすとしたら、また同じようにすることになりそうでさらにだるそう。


### 3系でのテンプレートの配置 {#3系でのテンプレートの配置}

3.15 以降では components に hbs を置けるようになるよ。
JS ファイルと場所が近くて便利。

さらに [Pods layout](https://cli.emberjs.com/release/advanced-use/project-layouts/#podslayout) という構成が使えて機能群毎にまとめるのが便利になる。

Pods layout はコンポーネントだけではなくて
model, controller とかの構成もフォルダに閉じ込められるので多分便利。まだ試したことないけど。


## Model(Ember Data) {#model--ember-data}

大体 Route で Controller にセットされる子。
Rails の Model に似せて作られている。

[EmberObject](#emberobject) ベースで作られているので、当然そのあたりの機能も使える。
computed とかね。


### Store の操作 {#store-の操作}

データを取得したりまとめて消したりする際に使う。
Rails でいうと ActiveRecord モデルの Class Method を叩く感じ。

参考: <https://api.emberjs.com/ember-data/2.18/classes/DS.Store>

以下ではよく使うものをピックアップして簡単な説明をしています。


#### findAll/peekAll {#findall-peekall}

どちらも対象のレコードを全件取得するメソッド。

違いは peekAll の方はフロントエンドの Store 内の検索に留まり、
findAll はデータがなければサーバにまで取りに行く。

`store.findAll('user')` みたいに使う。


#### findRecord/peekRecord {#findrecord-peekrecord}

findAll/peekAll のレコード単体版。違いも同じ。

`store.findRecord('user', 1)` とすると
id が 1 の User を探しに行く。


#### query/queryRecord {#query-queryrecord}

findAll ではクエリパラメータを付けられないので検索画面などは作れない。そんな場合は query を使うとパラメータを付与してリクエストを投げられる。それの単体レコード版が queryRecord となる。


#### create {#create}

```js
this.store.create('user', { firstName: 'Taro', lastName: 'Yamada' })
```

みたいな感じでデータを作成できる。


#### push/pushPayload {#push-pushpayload}

push は Object から Store にレコードを作る処理。

```js
store.push({
  {
    "user": {
      "id": 1,
      "first_name": "Taro",
      "last_name": "Yamada"
    }
  }
});
```

とかやるとレコードが Store 上に作られる。無理矢理サーバから持って来たデータを Store に突っ込む時に使う。

pushPayload はそれの便利版で

```nil
{
  "users": [{
    "id": 1,
    "first_name": "Taro",
    "last_name": "Yamada"
  }],
  "posts": [{
    "id": 1,
    "title": "Awesome Blog Post",
    "body": "..."
  }]
}
```

みたいなのを受け取ると users と posts をそれぞれ複数レコード登録してくれる。

<!--list-separator-->

-  注意

    push/pushPayload の例で書いている JSON は ember-rails で使う active-model-adapter 前提で書いています。デフォルトだと JSON:API 形式を受け付けるようになっています。


### Model の操作 {#model-の操作}

peekRecord などで取得した Model に対しての操作。更新したり削除したりはこれで行う。

参考: <https://api.emberjs.com/ember-data/2.18/classes/DS.Model>

以下ではよく使うものをピックアップして簡単な説明をしています。


#### save {#save}

取得したデータは set で値を書き換えて save で保存できる。

```js
const user = this.store.findRecord('user', 1)
user.set('firstName', 'Taro')
user.save()
```

save は Promise を返すので、保存後に何か処理をしたければ

```js
user.save().then((savedUser) => { /* 保存成功時の処理 */ }, (error) => { console.log(error) })
```

みたいなことができる。


#### destroyRecord {#destroyrecord}

```js
const user = this.store.findRecord('user', 1)
user.destroyRecord()
```

みたいな感じで削除できる。

save と同様に callback も受け付けている。


#### unloadRecord {#unloadrecord}

unloadRecord はサーバの実データは消しに行かずフロントエンドの Store から捨てるだけ。


#### reload {#reload}

サーバからデータを読み直す。


### Model の定義 {#model-の定義}


#### attr {#attr}

`firstName: DS.attr()` みたいに定義するやつ。
API から取って来た値をそのまま放り込む感じである。


#### belongsTo {#belongsto}

`user: DS.belongsTo('user')`
って書いたら `this.get('user')` と指定した時に userId から適切なデータを探してくる感じ。
Rails と似てますね。

`user: DS.belongsTo('user', { async: false })`
としておけばサーバには探しに行かなくなるっぽい。対応するエンドポイントを用意してない時なんかはそうしておいた方が良い。


#### hasMany {#hasmany}

`posts: DS.hasMany('posts')`
って書いたら関連する posts を `this.get('posts')` で取ってこれるようになる。
Rails と似てますね。

`posts: DS.hasMany('posts, { async: false })`
としておけばサーバには探しに行かなくなる。
hasMany の関連先のレコードが多い時なんかはまとめて取らない方がいいのでこの方が便利。

その場合、別で取得する方法を考えた方が良い。


#### 3系では {#3系では}

-   `@attr firstName`
-   `@belongsTo('user') user`
-   `@hasMany('post') posts`

みたいにデコレータで定義する。
option の async などをつける場合は `@belongsTo('user', { async: false }) user` みたいな感じ。


## Adapter/Serializer {#adapter-serializer}

どっちも普段あまり触らないところなので簡単に。

Adapter がサーバへのアクセス方法を色々設定するところ。どのホストにアクセスするかとか、どの PATH にアクセスするかとか
HTTP Header で何か渡すならそれを設定するとか。

Serializer がデータ形式を変換するところ。
JS 側では camelCase のキーを、サーバに渡す時には dashlize したり逆に dashlize されてるキーを貰う時は camelize したり。


## Controller {#controller}

Rails の Controller とは結構違う役割の子。
Rails ベースで考えると混乱する。

Route と関連する特別な Component みたいな扱いなので
Component の扱いをベースに触った方が良い。

Component に近い立場なので各種 hook だったり actions だったりが使える。

tagName や className などは使えるか調べたこともないのでわからない。

コンポーネントツリーの根っこの Component みたいな感じなので大分カオスになりがちなやつ。

正直そのカオスをどう解消したらいいかまだ分かってないが、恐らくここではページ全体に関するロジックだけを詰めておき、
Component に直接 Service Object を Inject して、その Service Object にロジックを詰めておいた方が良いんじゃないかなと考えている。どこかで試してみたい。


## Service Object {#service-object}

各 Component や Controller に Inject して使う便利な Object です。
Vue.js でいうと Vuex ぐらいどこでも使えて便利。つまり無闇にあちらこちらに Inject するとそれはそれでカオスになる系。

アプリケーション全体で使うような、例えば EC サイトだと shopping-cart を Service で実装しておいて各 Component で Inject する、みたいな感じで使う。


## 色々な罠達 {#色々な罠達}


### Ember.js の Guide {#ember-dot-js-の-guide}

ember-rails だと Ember.js@2.18 までなので
<https://guides.emberjs.com/v2.18.0/>
などの古いバージョンのガイド読みましょう。

っていうか古いバージョンのも全部残っててありがたいよね。

ただ ember-rails でも古い書き方をしていると
import とか使えないのでそのあたりは読みにくいと思う。
import とかを使ってないのは 1.10 ぐらいの書き方だから……。


### active\_model\_serializers という Gem の 0.10 系で動きません>< {#active-model-serializers-という-gem-の-0-dot-10-系で動きません}

ember-rails では active\_model\_adapter というのを Ember.js 側で使っています。そしてそれに適合する active\_model\_serializers gem は 0.9系となっています。というわけで 0.9 系を使いましょう。


### コンポーネント呼び出し階層が深いので Controller の Action を叩くのがしんどいです {#コンポーネント呼び出し階層が深いので-controller-の-action-を叩くのがしんどいです}

その Action を Controller じゃなくて Service に移動して
Component に Service を Inject して使うようにしたら良いかもしれません。

Vue.js でいうと Vuex をあちらこちらから叩くとカオスになるように、
Service の Inject もやりすぎるとそれはそれでカオスになりそう。


### mixins に色々置いて共通化ウェーイ {#mixins-に色々置いて共通化ウェーイ}

それあなた Rails でも Concerns に置いて後々苦労したりしませんでしたか?
あんまり Mixin に頼るとつらいので Service に逃がすなど検討しましょう


### ember コマンドを使いたい {#ember-コマンドを使いたい}

ember-cli-rails に乗り換えましょう。
[ember-rails から ember-cli-rails へ]({{< relref "migrate-ember-rails-to-ember-cli-rails" >}}) でその方法書いてるので参考にどうぞ。


### Addon を入れたい! 便利そう! {#addon-を入れたい-便利そう}

ember-rails では無理です。
ember-cli-rails に乗り換えましょう。


### Webpack 使えないの? {#webpack-使えないの}

ember-rails なら諦めましょう。そうじゃない場合も諦めた方がいいです。

[Embroider](https://github.com/embroider-build/embroider) というプロジェクトで Webpack とか Rollup とかと連携できるように頑張ってるみたいですがまだ v1 がリリースされてないのでプロダクションで使うには罠がまだまだ多そうです。


### Ember.js の単体テストを書きたい……! {#ember-dot-js-の単体テストを書きたい}

ember-rails でしたら諦めてください。

ember-cli なら QUnit または Mocha あたりでテストが書けるので
ember-cli-rails に乗り換えましょう。

Ember.js 自体に黒魔術が多いようで Jest ではテストが書けませんので Jest 派はお疲れ様でした


### CSS も Ember.js 内で書きたい! {#css-も-ember-dot-js-内で書きたい}

ember-rails なら諦めて `RAILS_ROOT/app/assets/stylesheets` あたりで書きましょう。おつかれさまでした。

ember-cli を使っていたら `EMBER_ROOT/app/styles/` 以下に置けるらしいよ。
<https://cli.emberjs.com/release/advanced-use/stylesheets/>


### Resolver のソースってどれ? {#resolver-のソースってどれ}

ember-rails をお使いで、かつ古い書き方をしているみなさんは
ember-source の中にある GlobalResolver をお読み下さい。以上。

ember-cli で Ember.js アプリを構築している先進的な皆様は
Ember.js の中にある GlobalResolver ではなくて ember-resolver という Addon を見ましょう。ただそいつはそいつで GlobalResolver を継承しているので結局 GlobalResolver のソースも読まないといけないという罠がある。

まあ v4 あたりでは継承やめるって話あるけどね


## 最後に {#最後に}

ember-rails のまま保守し続けるよりは
ember-cli-rails に移行して3系にした方が幸せになれそうなので是非ご検討ください。

そのあたりの対応方法は、私が書いた記事ですが、

-   [ember-rails から ember-cli-rails へ]({{< relref "migrate-ember-rails-to-ember-cli-rails" >}})
-   [ember-cli-rails の Ember.js を 2.18 から 3.4 にアップデート]({{< relref "update-emberjs-2.18to3.4" >}})
-   [Ember.js@3.4 から最新の 3.26 に上げた]({{< relref "update-emberjs-3_4-to-latest" >}})

あたりが参考になるかもです。
