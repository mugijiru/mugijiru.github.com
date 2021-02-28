+++
title = "ember-rails でユーザー情報を Rails から inject"
date = 2021-02-28T13:10:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

フロントエンドのフレームワークを使っていて、そのフレームワークで Server Side Rendering をしてない時に
API 経由でデータ渡すよりも表示用の HTML 経由で直接データを渡したい時がある。

ember-rails を使ってる時もそれはあって、今回は Haml 経由で Ember.js に情報を渡して表示する方法を書いてみた。もちろん旧来版と ES6 Module 対応版の両方で実装している。


## 旧来版 {#旧来版}

<https://github.com/mugijiru/ember-rails-todo-app/pull/16> で実装したやつ。


### おおまかな実装内容 {#おおまかな実装内容}

こちらは名前空間に Ember.js の外からアクセスできるので
Haml 内に JavaScript を埋め込んで Ember に渡すというちょっと乱暴なことができる。

今回は email を todo-items テンプレート内で表示したかったので
[Ember.js の呼び出し元の haml](https://github.com/mugijiru/ember-rails-todo-app/pull/16/files#diff-69c2e4b0a6040f2873e963c79265340fd97c099e1ea1a7fbf579902259126e3fR1) 内で

```js
:javascript
  TodoApp.register('session:current-user', Ember.Object.extend({ email: '#{current_user.email}' }));
  TodoApp.inject('controller:todo-items', 'current-user', 'session:current-user');
```

と書いてみた。

以下にもう少し詳細に書いてみる。


### ユーザー情報の登録 {#ユーザー情報の登録}

```js
TodoApp.register('session:current-user', Ember.Object.extend({ email: '#{current_user.email}' }));
```

という記述で JavaScript の中に Haml での Ruby のコード呼び出し機能を用いて
email を EmberObject を継承したクラスにぶち込んでいる。

正直 `:javascript` で書いて Ruby のコードを呼び出すのは結構乱暴だとは思うけどできちゃうのでやっちゃった。


### コントローラへの inject {#コントローラへの-inject}

アプリケーションに `session:current-user` として登録できたので、後はもう

```js
TodoApp.inject('controller:todo-items', 'current-user', 'session:current-user');
```

として controller に inject することができる。


### template での表示 {#template-での表示}

inject された controller の template で `{{current-user.email}}` と記述するだけでそのユーザーのメアドが表示される。以上。


## ES6 Module 対応版 {#es6-module-対応版}

<https://github.com/mugijiru/ember-rails-todo-app/pull/17> で実装したやつ。


### おおまかな実装内容 {#おおまかな実装内容}

こちらは旧来版とは違って名前空間は隠蔽されているため
Haml で JavaScript を書いて埋め込むなんて荒技はできない。

だけどまあそんなことをしなくても
data 属性に情報を埋めておいてそれを initializer で取得して使えばいいだけである。


### Haml へのデータ埋め込み {#haml-へのデータ埋め込み}

Haml の方では

```haml
#todo-app{ data: { email: current_user.email } }
```

こんな感じにデータを埋めておく。それを Ember.js の initializer で取得して処理してあげれば良い。


### initializer でのデータの取得 {#initializer-でのデータの取得}

まずはデータを

```js
const currentUser = Ember.Object.extend({
  email: document.querySelector(application.rootElement).dataset.email
});
```

という感じで取得して適当な変数に放り込んでおく。ま、大体普通の JavaScript なので何も難しいことはない。


### アプリケーションへの登録 {#アプリケーションへの登録}

上で取得したデータをアプリケーションから見れるように登録してあげる必要があるので以下のように `application.register()` でデータを登録する。

```js
application.register('session:current-user', currentUser);
```


### controller への inject {#controller-への-inject}

上に書いた感じで application に登録してしまえば、後は旧来版と同じように

```js
application.inject('controller:todo-items', 'current-user', 'session:current-user');
```

という感じで設定できる。


### template での表示 {#template-での表示}

あとは旧来版と同じく
inject された controller の template で `{{current-user.email}}` と記述するだけでそのユーザーのメアドが表示されると。うん、簡単でしたね。


## 最後に {#最後に}

API を経由せずに Ember.js にデータを渡す方法が旧来版と ES6 Module 対応版の両方で書けることがわかったので、旧来版から移行しようとした時もすぐ書き直せそうで安心。

ES6 Module 対応版の方は、
ember-rails から ember-cli-rails とかに乗り換えてもそのまま使えそうだしね。
