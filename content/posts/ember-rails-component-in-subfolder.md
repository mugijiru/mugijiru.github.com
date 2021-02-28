+++
title = "ember-rails でコンポーネントをサブフォルダに配置する"
date = 2021-02-28T11:19:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

Ember.js に限らずコンポーネントは増えてくるとサブフォルダに分割して管理したくなるよね。ということでそのあたりの記事。

これも、ES6 Module 対応版と旧来版の両方を書く。
ES6 Module 対応版は何も考えることがないので、この記事は旧来版のためにあるようなものだけど。


## ES6 Module 対応版の場合 {#es6-module-対応版の場合}

上に書いたようにこれは <https://github.com/mugijiru/ember-rails-todo-app/pull/14> に実装してあるけどとっても簡単


### component をサブフォルダに移動 {#component-をサブフォルダに移動}

`components` の下に適当なフォルダを掘ってその中に移動するだけ。


### template をサブフォルダに移動 {#template-をサブフォルダに移動}

`templates/components` の下に適当なフォルダを掘ってその中に移動するだけ。


### template からの呼び出し {#template-からの呼び出し}

template, component をそれぞれ

template
: `templates/components/hoge/fuga.hbs`

component
: `components/hoge/fuga.module.es6`

と配置した場合は
`{{hoge/fuga}}` と書いて呼び出せばいい感じに動く。以上。

こういう感じで動くように [ember-resolver@0.1.21](https://github.com/ember-cli/ember-resolver/tree/v0.1.21) が作られてるっぽいのでとても楽。

Ember.js のドキュメントなどを見ている感じだと多分もっと新しいバージョンでも同じ感じで動くっぽい。というわけで Ember.js@3 にしても多分動きそうなので安心感がある。


## 旧来版の場合 {#旧来版の場合}

これは GlobalsResolver の挙動のおかげでちょっと大変。

と言っても
<https://github.com/mugijiru/ember-rails-todo-app/pull/15>
で実装してある。

今回やりたかったことは、テンプレートとコンポーネントをサブフォルダに移動して扱えるようにすることなので、その実現方法を書いておく


### template からの呼び出し {#template-からの呼び出し}

`{{hoge/fuga}}` と呼び出した際に [GlobalsResolver](https://github.com/emberjs/ember.js/tree/v2.18.2/packages/ember-application/lib/system/resolver.js#L34) でどう解釈されるとかというと
[前の記事]({{< relref "ember-rails-extract-common-libs" >}}) にも書いたように
Hoge という名前空間の FugaComponent を探しに行くようになってるというのが前提。


### component をサブフォルダに移動 {#component-をサブフォルダに移動}

GlobalsRegister の解釈に合わせて
FugaCompnent を Hoge 名前空間に所属させればいいので

```js
Hoge.FugaComponent = Ember.Compnent.extend()
```

という形で定義しておけばいい。

旧来方式だとファイル自体は components の中にあればファイル名も位置も何でもいいはずなので人間がわかりやすいように `components/hoge/fuga.js.es6` として配置したら良い。

また、事前に Hoge という名前空間は必要なので
`components/hoge.js.es6` とファイルで

```js
window.Hoge = Ember.Namespace.create()
```

としておく。

前回の共通ライブラリ切り出しと大体似たお話ですね。


### template をサブフォルダに移動 {#template-をサブフォルダに移動}

これは難しいことは何もなくて
`templates/components/<名前空間>/<コンポーネント名>.hbs`
みたいに配置したら良い。

つまり `Hoge.FugaComponent` の場合は
`templates/components/hoge/fuga.hbs`
と置けばいい。


### さらにネストさせたい場合 {#さらにネストさせたい場合}

試してないけど、
[GlobalsRegister の実装](https://github.com/emberjs/ember.js/blob/e2007b6ecb046fd06f6b43c381e8a1128914ad43/packages/%40ember/application/globals-resolver.js#L221) を見ている感じだと、多分

```js
window.Hoge = Ember.Namesupace.create()
```

```js
Hoge.Fuga = Ember.Namesupace.create()
```

```js
Hoge.Fuga.PiyoComponent = Ember.Component.extend()
```

にみたいな感じに名前空間をネストさせれば大丈夫そう。


## 最後に {#最後に}

前回の共通ライブラリ切り出しよりは簡単でしたね。

ES6 Module 対応版では直感的にやるだけで終わるし、旧来版でも共通ライブラリと大体やること一緒というか、それよりも手順が少ないので、サブフォルダへの移動を先にやった方が良かったかも。

あと、今回も両パターンでやってるので、
ES6 Module 移行前にこちらを実施しても簡単な修正で対応できることがわかりました。やったね。
