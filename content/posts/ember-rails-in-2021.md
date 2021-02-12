+++
title = "2021年にもなって ember-rails で新規アプリを書いてみてる"
date = 2021-02-13T01:15:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

少し思うところがあって、
2021 年になったというのに [ember-rails](https://github.com/emberjs/ember-rails) を使って新規で Web アプリを書いている。


## ember-rails とは? {#ember-rails-とは}

ember-rails は
Ember.js という Web フロントエンド MVC なフレームワークを
Rails といい感じに連携してくれて快適な Web アプリケーション開発体験を提供してくれるものであった。

過去形なのは、ember-rails は Rails3 とか 4 とかの時代に主に使われていて既にメンテナンスされてないのと、今はそれよりも良い [ember-cli-rails](https://github.com/thoughtbot/ember-cli-rails) というのがあるから。


## 今 ember-rails を使うと何がつらいか {#今-ember-rails-を使うと何がつらいか}

色々つらい。


### まずメンテナンスが止まってる {#まずメンテナンスが止まってる}

なので Rails 6 で動くかがわからない。多分、試している人はいないし、自分もそこまで試す気力はない。


### Ember.js のサポートが 2.18.2 までとなっている。 {#ember-dot-js-のサポートが-2-dot-18-dot-2-までとなっている}

より詳細に話すと
ember-rails が依存している Gem である ember-source で本当は 3.0.0.beta.2 まで出てるんだけど、β版のことは無視する。
<https://rubygems.org/gems/ember-source/versions/2.18.2>

で、その 2.18.2 は既にサポートされてないバージョンである。

サポートされてないバージョンを使うのはセキュリティ面でもまずいしもはや情報もあまり落ちてないので苦行である。

Ember.js 公式サイトのドキュメントが過去のバージョンのものも残されているのでそれを頼りにするしかない。というか公式で残しててくれてありがとう。それがないと何もできないよ。


### ember-rails だと Ember.js の addon が導入できない {#ember-rails-だと-ember-dot-js-の-addon-が導入できない}

例えば Handlebars でロジックを書く上でとても基本的な比較用のヘルパーを提供してくれる [ember-truth-helper](https://github.com/jmurphyau/ember-truth-helpers) が使えない。これが使えないはめっちゃ不便で、それをなんとかするために同じようなコードを自前で用意するハメになる。

他にも [ember-community-russia/awesome-ember](https://github.com/ember-community-russia/awesome-ember) に載っている色々なものが使えないわけだ。つらいどころか悲しくなってくる。


### 自動テストが書けない {#自動テストが書けない}

Ember.js は QUnit で自動テストができるようになっているのだが
ember-rails だとそれも使えない。すなわちフロントエンドのコンポーネントの単体テストが書けないのである。


## それでも ember-rails を使いたい方には {#それでも-ember-rails-を使いたい方には}

どうして素直に ember-cli-rails や ember-cli そのものを使おうとしないのかはわからないけどどうしても ember-rails の世界に住みたいのであれば
[discourse](https://github.com/discourse/discourse) のソースを参考にしたら良いと思う。

どうやら [ember-cli に乗り換える方針で動いているよう](https://github.com/discourse/discourse/pull/11932) だが今日時点の Gemfile には未だに discourse-ember-source などの記述が残っている状態であり、まだ完全移行はできてない様子。

[discourse-ember-source](https://rubygems.org/gems/discourse-ember-source/versions/3.12.2.2) は 3.12 系まで追従していたようなのでそこまでは discourse の真似をすれば使えるだろう。

また彼らは ember-rails を使いながら qunit でのテストもできるようにしているようである。正直マジか頑張ったなって気持ち。ちょっとどうやって動かしているのかはわからない。あんまり調べる気力もない。なんとなくわかったことは ES6 の module システムを活用して頑張ってる雰囲気があることである。

他にも addon も使えるようにしている様子でもあるが、これもちょっとよくわかってない。あまり adoon が使われてる気もしないが……。

ともかく ember-rails を独自に拡張した上で色々頑張っているようである。すごい。それでももう ember-cli-rails に乗り換えようとしているようなので今から ember-rails の世界に住もうとするのはやめた方がいいはず。

あ、よく見ると Rails は 6.0 系だ。ってことは少なくとも discourse-ember-rails なら Rails 6.0 でも動くわけか。なるほど。


## で、なぜ自分は ember-rails で新規アプリを書いているか {#で-なぜ自分は-ember-rails-で新規アプリを書いているか}

マゾなので、敢えてその環境で新規アプリを用意しておいてそこから ember-cli-rails に移行する、みたいなことをしてみたいから。

本当は自分で ember-rails なアプリを書くつもりはなかったんだけどサンプルになるようなアプリが探せなかったってのもある。これが Yak Shaving か〜と思いながら粛々と小さなアプリを書いていくのであった

できたらまた記事にする。アプリ自体は公開しないけど、ソースは GitHub に上げるつもり。
