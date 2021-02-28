+++
title = "ember-rails でコンポーネントを共通ライブラリとして切り出す"
date = 2021-02-28T10:45:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

ember-rails を使って1つの Rails アプリの上に複数の Ember.js アプリケーションを動かしていると各アプリで同じようなコンポーネントを使っていたり、あるいは同じようなコンポーネントが必要だというのに気付いて共通ライブラリとして実装したくなることがある。あるんだよ。

というわけで、その共通化を2パターンでやってみた。
2パターンというのは 旧来の書き方の場合と
ES6 Module 対応版の場合とである。

なおいずれのパターンもサーバへのデプロイはやってないのでもしかしたらサーバ環境では動かないかもしれないがご容赦を。


## 旧来版 {#旧来版}

先に答えを出すと
<https://github.com/mugijiru/ember-rails-todo-app/pull/13>
に実装した通りである。


### template からの呼び出し {#template-からの呼び出し}

template で `{{ember-libs/button}}` と書いた場合に
Resolver には `component:ember-libs/button` として解釈するように要求されるっぽい。これはソースからではなく、挙動的に確かめただけ。


### コンポーネントの探索 {#コンポーネントの探索}

旧来の書き方の場合に探索に使われるのが GlobalsResolver というやつ。

この GlobalsResolver というやつは [コメント](https://github.com/emberjs/ember.js/blob/e2007b6ecb046fd06f6b43c381e8a1128914ad43/packages/%40ember/application/globals-resolver.js#L59-L76) にも書かれてるように
`component:ember-libs/button` と渡されたら、
GlobalsResolver は `EmberLibs.ButtonComponent` として解釈するようになっている。つまり EmberLibs という名前空間の ButtonComponent を探しに行くようになっている。


### 名前空間の定義 {#名前空間の定義}

というわけで、まずは [ember-libs/ember-libs.js.es6](https://github.com/mugijiru/ember-rails-todo-app/pull/13/files#diff-e1803bb0635866bc90975a1321dbfa6d20be59e76ec3d7b80c8acc4656f8af9fR6) に書いてるように

```js
window.EmberLibs = Ember.Namespace.create()
```

と書くことで
EmberLibs という名前空間を定義してやる。

一応 `ember-libs/ember-libs.js.es6` では require の順番として
ember はそこで定義している実装を使うので先に require してそのファイルで定義している名前空間を components で使うので components を require するより前に
require\_self をしている。


### 共通コンポーネントの記述 {#共通コンポーネントの記述}

各コンポーネントはその名前空間の下に入るように書けばいい。例えば [ember-libs/components/button.js.es6](https://github.com/mugijiru/ember-rails-todo-app/pull/13/files#diff-9f9be147342dc470d8f0cba8a06a55a210550e01b22502bd6e0aff0d029ae38cR1) に書いてるように

```js
EmberLibs.ButtonComponent = Ember.Component.extend()
```

というように書いてやれば動く。


### config.handlerbars.templates\_root の設定 {#config-dot-handlerbars-dot-templates-root-の設定}

templates を ember-libs/templates に入れるので
Rails 側の設定で `config.handlebars.templates_root` に `ember-libs/templates` を追加するのを忘れずに。
ember-rails の設定例に従っていれば [config/application.rb](https://github.com/mugijiru/ember-rails-todo-app/pull/13/files#diff-c1fd91cb1911a0512578b99f657554526f3e1421decdb9e908712beab57e10f9R34) に設定があるはず。


### 利用側の設定 {#利用側の設定}

あとは [todo-app/application.js.es6](https://github.com/mugijiru/ember-rails-todo-app/pull/13/files#diff-2cb7f9d0c761533d0e2b01e0b7e6f4a34529c7b52f9a13c7493b2629251bccd8R9) に書いてるようにこの共通コンポーネントを使いたいアプリ側で

```js
//= require ember-libs/ember-libs
```

としてやるだけでさくっと使えるようになる。


### 他の type について {#他の-type-について}

試してないけど mixin や service ぐらいなら同じノリでいけるんじゃないかなと思ってる。
model もいけそう。名前空間が変わるだけだし、その呼び出しも難しくないし、大体なんとかなりそう。


### 余談: 名前空間を分けない場合 {#余談-名前空間を分けない場合}

上のようなやりかたをしているのは、名前空間を分けたいってのが先だったので、各アプリで名前空間を分ける必要がなければ、全部のアプリで

```js
window App = Ember.Application.create()
```

とかしちゃって

`ember-libs/components/button.js.es6` では普通に書く場合と同じように

```js
App.ButtonComponent = Ember.Component.extend()
```

みたいにしておいて require したら `{{button}}` で使える。個人的には、名前空間が混ざるとどっちかが上書きされたりしそうで怖くて嫌だけど。


## ES6 Module 対応版の場合 {#es6-module-対応版の場合}

最初に答えを出すと
<https://github.com/mugijiru/ember-rails-todo-app/pull/12>
で実装したやつ。


### コンポーネントの探索 {#コンポーネントの探索}

ES6 Module で書かれている Ember Application では基本的に単一の名前空間しか持たないようである。また、使用される Resolver が [ember-resolver@0.1.21](https://github.com/ember-cli/ember-resolver/tree/v0.1.21) となっている。

こいつは `component:ember-libs/button` と渡って来た時の解釈が GlobalsResolver と異なっている。この ember-resolver の場合は、アプリケーションの下の `components/ember-libs/button` を探しに行く。

なのだけど今回はそんなところを探しに行って欲しくないので、
regsiter を Ember.js で自動的に解決して対応してもらうのではなく
[ember-libs/ember-libs.module.es6](https://github.com/mugijiru/ember-rails-todo-app/pull/12/files#diff-029812c538a995224fcf19bfa24f65558246c054aea77c95ec1f4a404b4f5256R1) に書いているように、自前で

```js
application.register()
```

して対応することにした。


### コンポーネントの register {#コンポーネントの-register}

基本的には以下のように書いておけば Button コンポーネントは動くようになる。

```js
import Button from './components/button';

application.register('component:ember-libs/button', Button);
```

が、コンポーネントが増えていった際に全部そうやって書くのはアホらしい。というわけで、自動的に解決するようにした。


### コンポーネントの auto register {#コンポーネントの-auto-register}

ES6 Module 対応して import している場合に ember-rails では実際はどんな形に transpile されるかというとどうやら requirejs の機能で読み込んだりしているらしい。

で export されているファイルは `requirejs.entries` に含まれているのでそこから必要なものを探し出して
`application.register` に対し、解釈してほしい名前で渡してクラスを渡しておけば
template で `{{ember-libs/button}}` とした時に require したクラスのインスタンスとして動いてもらえる。

という感じで自動的に register する処理を [メソッドにして](https://github.com/mugijiru/ember-rails-todo-app/pull/12/files#diff-029812c538a995224fcf19bfa24f65558246c054aea77c95ec1f4a404b4f5256R4) おけば、利用側はそれを呼び出すだけでセットアップが済む


### config.handlebars.templates\_root の設定 {#config-dot-handlebars-dot-templates-root-の設定}

やはりこちらの場合も templates を `ember-libs/templates` に入れるので
Rails 側の設定で `config.handlebars.templates_root` に `ember-libs/templates` を追加するのを忘れずに。
ember-rails の設定例に従っていれば [config/application.rb](https://github.com/mugijiru/ember-rails-todo-app/pull/12/files#diff-c1fd91cb1911a0512578b99f657554526f3e1421decdb9e908712beab57e10f9R34) に設定があるはず。


### 利用側の設定 {#利用側の設定}

アプリ側では [initializers/resolve-common-libs に書いている](https://github.com/mugijiru/ember-rails-todo-app/pull/12/files#diff-97468a821d4c12c1b223617fba29257a5b1e00553a1b8e8f403ee99864756ebaR4) ように
initializer で

```js
EmberLibs.registerAll()
```

を叩くだけでいい感じに使えるようになる。


### 他の type について {#他の-type-について}

試してないけど、component でやってみた所感。

mixin はどうせ明示的に import して使うので関係なさそう。
service は、component と同じやりかたでいけそうな気がする。
model もいけそうなので user model を共通化するような用途がありそう。

controller もいけそうだけどそれに付随する route からどう呼ばれるかが難しそう。


### 余談: 他の方法について {#余談-他の方法について}

多分 EmberEngine とか EmberAddon の仕組みを使って似たようなことはできそうな気はする。

だけど ember-rails で Engine や Addon を使うというのはそれはそれでかなり大変かと思われるので今回はそれを動かすようなことはしてない。

より正確にいうと、それしか方法がないかもと思って途中まで調べたけど、厳しそうだったので今回は上述の方法にしておいた。


## 最後に {#最後に}

旧来版と ES6 Module 対応版とで実装方法は異なるがどちらでも同じような使い勝手でコンポーネントを共通ライブラリとすることができることがわかった。

両方のパターンが使えることがわかったので、
ES6 Module 対応版への移行がまだでも躊躇せずライブラリを分割できそう。
