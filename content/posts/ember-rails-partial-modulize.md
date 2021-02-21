+++
title = "古い ember-rails App で一部ファイルを ES6 Module 化"
date = 2021-02-21T22:41:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

## これは何? {#これは何}

ember-rails を古いスタイルで書いておいてそれをモダン化していく企画の第一弾の記事。


## 何をしたのか {#何をしたのか}

今回は ES6 module を使ってない ember-rails アプリケーションで一部のファイルだけ ES6 Module にしてみた。


## 何が嬉しい? {#何が嬉しい}

今回扱ってるアプリケーションのサイズはとても小さいのでまとめて置き換えることも可能というか、ぶっちゃけ [古いスタイルに書き換えた PR](https://github.com/mugijiru/ember-rails-todo-app/pull/7) を revert するだけで
ES6 Module 化できたりする。

しかし、世の中に潜んでいる、レガシー化した ember-rails のプロジェクトでは全部まとめて ES6 Module にするのはファイル数が多過ぎて困難かと考え、敢えて一部のファイルだけ ES6 Module 化する方法を探してみた。


## どうやったらできるの? {#どうやったらできるの}

簡単に言うと
ES6 Module 形式で書いたやつを import して
Ember.js Application の Namespace に放り込めばいいだけ。


### Example {#example}

`app/assets/javascripts/ember-app/components/foo.module.es6` ってファイル名で

```js
import Ember from 'ember';

export default Ember.Component.extend({});
```

のように書いておいて

`app/assets/javascripts/ember-app/application.js.es6` の末尾にでも

```js
import FooComponent from 'ember-app/components/foo';

EmberApp.FooComponent = FooComponent;
```

のように書いたら、一応 module 形式で書けるし、それを window.EmberApp で用意した Ember.js Application で使えるって感じ。

拡張子が `.module.es6` というのがポイントで、そうしておくと
[ember-es6\_template](https://github.com/tricknotes/ember-es6%5Ftemplate) という Gem が自動的に ES6 の module として判定してくれるようになっている
<https://github.com/tricknotes/ember-es6%5Ftemplate/blob/c1c7b8d23be7669a0aa6c5f9c71b916a3799f9a6/lib/ember/es6%5Ftemplate/sprockets.rb#L10>


## 関連 PR {#関連-pr}

実際に動くコードは以下の PR で用意した。
<https://github.com/mugijiru/ember-rails-todo-app/pull/8>

PR では `modules` フォルダにさらに components フォルダを掘ってその中にファイルを配置している。

その方が全部移行できた後にまるっと置き換えするのに楽そうだからだ。

また import して Namespace に放り込む処理も別ファイルに追い出している。これも、完全移行が済んだら不要になるファイルなので消しやすさを重視して分割しておいた。

CI でテストも通しているしバッチリだと思う。デプロイできるようにはしてないからサーバで動くかは確認してないけど、ま、大丈夫だろ
