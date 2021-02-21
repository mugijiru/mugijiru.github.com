+++
title = "ember-rails で書いた Web アプリを GitHub で公開した"
date = 2021-02-21T15:40:00+09:00
draft = false
+++

[この間の記事]({{< relref "ember-rails-in-2021" >}}) で書いたように
ember-rails で簡単なアプリケーションを作ってた。よくある TODO アプリである。

{{< figure src="/ox-hugo/screenshot-ember-rails-todo-app.png" >}}

先週時点では「テストとかなくてもいいから動けばいいだろ」って気持ちだったけどなんとなーくテストを追加したくなったりあんまり慣れてない docker-compose 対応してみたりしていたのと平日はこのプログラムに触れてなかったので、結構日が空いてしまった。

まあ、それはともかくとして、とりあえず <https://github.com/mugijiru/ember-rails-todo-app/> に置いておいた現時点の最新コミットで [v1.1.1](https://github.com/mugijiru/ember-rails-todo-app/tree/v1.1.1) のタグを振ってるやつは自分の知ってる一番古いスタイルで書かれてる状態にしてある。


## 使ってる Gem {#使ってる-gem}

-   Ember.js 関係
    -   ember-rails
    -   ember-source
    -   jquery-rails
        -   Ember.js は 2 系まで jquery に依存しているので
            -   よく見ると ember-rails の依存に入ってるから書かなくて良かったな……
    -   active\_model\_serializers 0.9
        -   0.9 系じゃないとうまく動かないっぽい
-   CSS framework
    -   bootstrap-sass
        -   レガシー感の演出のため敢えてこれにしている
-   テスト関係
    -   rspec-rails
    -   factory\_bot\_rails
    -   database\_rewinder
    -   capybara
    -   selenium-webdriver

あたり。


## レガシー感の演出 {#レガシー感の演出}

レガシー感を出すために bootstrap-sass(Bootstrap3系になる)を使ったりはしているがあまり特別なものは使ってない。

また ember-rails で ember アプリのソースコードを generate すると
es6 module を使ったようなコードが出力されるけど、これも敢えてレガシー感を出すために module を使わない形式に書き直している。

よりレガシー感を出すために CoffeeScript にするという手もあったけど、さすがにそこまでは頑張りたくないw
もう何年も触ってないよ CoffeeScript...

そしてページ全体を Ember.js にはしないでページの一部を Ember.js にする [埋め込み](https://guides.emberjs.com/v2.18.0/configuring-ember/embedding-applications/) 形式を採用している。既存のアプリに Ember.js を後乗せした感の演出である。実際、構築時には一時的に普通の Rails App として動くようにしていた。

他にこだわったところは、今回は単一のアプリケーションしか動かしてないけど
[Multiple Ember Application](https://github.com/emberjs/ember-rails#multiple-ember-application)
の作法に則って、Ember アプリケーションを追加で乗せられるようにしている。これにより「この画面も Ember 化しようず」という流れで
Ember アプリが複数動いてる状態により近くなったんじゃないかなと。実際今回動いてるのは1つだから、ちょっと違うけどね。。。


## 最後に {#最後に}

ここから段々と最新の Ember.js を使えるように寄せていくつもり。
