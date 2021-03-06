+++
title = "ember-rails から ember-cli-rails へ"
date = 2021-03-06T17:05:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

Ember.js 関係で最も書きたかった記事にやっと辿り着いた。表題の通りで、
ember-rails から ember-cli-rails に置き換える、という記事です。多分長くなる。

やったことはいつも通り [GitHub の PR](https://github.com/mugijiru/ember-rails-todo-app/pull/18) にしています。

PR の Description で「Rails 側ではこうした」「Ember 側ではこうした」みたいに書いているのでここではある程度時系列に沿ったような書き方にしようかな。

完全に時系列通りには書かないので、正確な時系列でどうしたか知りたかったら PR のコミットログを追ってください


## アプリの前提 {#アプリの前提}

これまで作って来た <https://github.com/mugijiru/ember-rails-todo-app> が前提になります。ざっくり内容を書くと

-   ember-rails で Ember.js 2.18 の環境を動かしている
-   Sprockets での ES6 Module 対応済
-   現実世界の複雑さを持ち込むために敢えて以下の手法を導入
    -   Embedded Ember App
    -   Multiple で動かせる構成
    -   一部コンポーネントの共通ライブラリ化
        -   ember-libs という名前で別フォルダに切り出している
    -   Bootstrap の利用
-   複雑さでは以下もありうるが面倒などの理由でやってない
    -   i18n.js での多言語対応
    -   コンポーネント以外の共通ライブラリ化

という感じ。


## ember-rails 用の JS のコードが読まれないようにコメントアウト {#ember-rails-用の-js-のコードが読まれないようにコメントアウト}

<https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/8dd44540bd7d352e497f87a9a12df5ad3cf6efbb>
のあたりのコミット。

本当は後からやった手順だけど、ここで読まれてるコードが邪魔になるので先にコメントアウトしておく方が後の手順でハマらなくて済むのでここに置いといた。

まあ実は ember-cli-rails のアプリが読まれるところで
ember-rails が require されてなければいいだけなので
application.js で require\_tree とかをしなければ良かったりはする。


## Docker 環境への ember-cli の導入 {#docker-環境への-ember-cli-の導入}

Docker でアプリが動くようにしているので、
ember-cli も Docker で動くようにしている。


### Docker で最新 LTS の Node.js が使われるように設定 {#docker-で最新-lts-の-node-dot-js-が使われるように設定}

ember-cli と直接は関係ないけど、Node.js は入れる必要があるのでやってる手順。

とりあえず最新の LTS を入れておく。
Ubuntu で普通に apt から入れると 10 系が入っちゃうので
yarn の apt リポジトリを登録してそこからインストールする。

```Dockerfile
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
&& echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

RUN apt-get update -qq && apt-get install -y nodejs yarn
```


### ember-cli を Global に導入 {#ember-cli-を-global-に導入}

ember-rails で動いているアプリは Ember.js 2.18.2 で動いているので
ember-cli も 2.18.2 を導入する。

```Dockerfile
RUN yarn global add ember-cli@2.18.2
```


## アプリの初期構築 {#アプリの初期構築}

上記手順で導入した ember-cli を使って改めて Ember.js アプリを構築する。ゼロから作っておく方が、より ember-cli-rails に向いた形になるとの判断。


### ember-cli で移植先のアプリの雛形を構築 {#ember-cli-で移植先のアプリの雛形を構築}

`RAILS_ROOT/ember/todo-app` に構築する。

ember-cli-rails の README だと `RAILS_ROOT/frontend` に構築するように書かれているが、複数の Ember.js アプリを平等に扱える形にしたいのと
Ember.js アプリのコード置場を `RAILS_ROOT/frontend` にしていると
Ember.js から別のフレームワークに差し替えが決まって、その移行作業をしている間に

-   frontend に新しいフレームワークで構築しようと思ったら既に Ember.js がいた
-   新しいフレームワークでの実装を修正しようと思って frontend 以下を探していて時間を潰した

ということが起こりそうなので、フレームワーク名は明示しておきたいお気持ち。というわけで `ember` というフォルダの下に更にフォルダを掘っているが、この考え方、あまり合意を得られた試しはない。みんな移行は発生しないつもりなのかな。

ま、とりあえず以下のコマンドを実行したら `RAILS_ROOT/ember/todo-app` に雛形が作成される。

```text
$ ember new todo-app --no-welcome --skip-git --yarn --dir ember/todo-app
```

なお、面倒なので `docker-compose run` とかは省略している。ここより下の部分でも同様に省略しているので、そのあたりは読みながら脳内で補完とかしてください。


#### オプションについて {#オプションについて}

`--no-welcome`
: どうせ後で消すファイルが作られるだけなので出す必要なし

`--skip-git`
: Rails アプリと同じリポジトリに作るので git init は不要

`--yarn`
: yarn を使い慣れてるからそれを指定。ただ `yarn link` に問題があるから `npm` を使う方がいいかも?

`--dir ember/todo-app`
: ember というフォルダの中に構築するので指定する必要あり


### ember-cli-rails-addon の導入 {#ember-cli-rails-addon-の導入}

ember-cli-rails と連携して ember-cli app を動かす時には
ember-cli app 側に [ember-cli-rails-addon](https://github.com/rondale-sc/ember-cli-rails-addon) を入れておく必要があるので、早い段階で追加しておく

```text
$ cd ember/todo-app && ember install ember-cli-rails-addon
```

これを入れておくと CSRF Token のことを意識しないで済むし、ファイルを更新するだけで Rails から読めるように Ember.js app を build してくれたりする。というか、入れてないとそれらがうまく動かなくてハマる。


### active-model-adapter の導入 {#active-model-adapter-の導入}

[active-model-adapter](https://github.com/ember-data/active-model-adapter) は
ActiveModelSerializer の出力をいい感じに Ember.js で扱えるようにする Addon で
ember-rails でも使われている。

というわけでこいつも Rails でいい感じに Ember.js を使うためには必要なので先に入れておく

```text
$ cd ember/todo-app && ember install active-model-adapter
```


## ember-cli-rails の導入と設定 {#ember-cli-rails-の導入と設定}

ここは Rails 側の作業。ひとまず ember-cli-rails の導入に留め、
ember-rails は一旦そのままにしておく。


### ember-cli-rails の導入 {#ember-cli-rails-の導入}

これは単に Gemfile に記載して `bundle install` を叩くだけである

```ruby
gem 'ember-cli-rails'
```

```text
$ bundle
```


### config/initializers/ember.rb で ember-rails の設定 {#config-initializers-ember-dot-rb-で-ember-rails-の設定}

ember-cli-rails で generate コマンドが用意されているのでまずはそれでファイルを生成する

```text
$ rails generate ember:init
```

これで `config/initializers/ember.rb` が作られるの。初期状態は以下の通り。

```ruby
EmberCli.configure do |c|
  c.app :frontend
end
```

それに変更を加えて、以下のようにする

```ruby
EmberCli.configure do |c|
  c.app :todo_app, name: 'todo-app', path: Rails.root.join('ember', 'todo-app'), yarn: true
end
```


#### 引数について {#引数について}

第一引数
: あとで mount する時に使う値

name
: ハイフン繋ぎにしたかったので指定しているが、多分なんでもいい

path
: `ember/todo-app` に構築しているのでそれを見てもらえるように指定

yarn
: yarn を使い慣れてるので指定。ただ yarn link がうまく動かないのでやめた方がいいかも


### config/routes.rb で Ember.js App を Mount {#config-routes-dot-rb-で-ember-dot-js-app-を-mount}

Embedded Ember.js App というわけで
Controller を自前で用意するので、contoller としてそれを指定する。

```ruby
mount_ember_app :todo_app, to: '/ember_cli_todo_items', controller: 'ember_cli_todo_items', action: 'index'
```


### Controller 等の用意 {#controller-等の用意}

移植途中で元のアプリに戻せなくなるのは移行失敗時のリカバリを考えると嫌なのと元の挙動を確認したくなった時のために元の PATH で動く状態にすぐ戻せるようにしておきたい。というわけで別の PATH を用意して、ember-cli で構築したアプリはそこで動くようにする。

```text
$ rails g controller ember_cli_todo_items index
```

あとは ember-rails 実装での Controller, View を参考にしたりして以下の感じに。


#### Controller {#controller}

特にサーバから何かを View に渡す必要はないので基本的に空っぽ。

```ruby
class EmberCliTodoItemsController < ApplicationController
  def index
  end
end
```


#### View {#view}

rootElement を用意して、そこに initializer に渡す data 属性を置いておく。

さらに ember-cli で生成する JS/CSS が読まれるように設定する。
(今回 CSS は書かないけど……)

```haml
#ember-cli-todo-app{ data: { email: current_user.email } }

%base{ href: '/ember_cli_todo_items/' }
= include_ember_script_tags :todo_app
= include_ember_stylesheet_tags :todo_app
```

`%base` は Ember.js のアプリケーションを動かす PATH に合わせる必要があるのと最後の `/` が抜けていると script や stylesheet で正しく PATH 解決できないので注意。

[ember-cli-rails-assets](https://github.com/seanpdoyle/ember-cli-rails-assets) の README を見ていると
include\_ember\_script\_tags とかに追加の引数で
`prepend: '/ember_cli_todo_items/'` とか書いていれば `%base` は使わなくて良さそうだけどまだ試してはいない


## ember-cli で作ったアプリが Rails 上で動くようにする {#ember-cli-で作ったアプリが-rails-上で動くようにする}

Rails 側の設定はここまでで完了しているはずなので次は ember-cli 側の設定を進めて Rails 上で動くようにしていく。


### config/environement.js の設定 {#config-environement-dot-js-の設定}

まず config/environment.js で以下を指定している

```js
modulePrefix: 'todo-app',
rootURL: '/',
locationType: 'hash',
```

rootURL は ember-cli-rails の README 通りに設定していると
`/ember_cli_todo_app` になりそうだがそれを指定すると Ember.js App が読まれた時に URL が
`http://localhost:3000/ember_cli_todo_app/ember_cli_todo_app` というように
`ember_cli_todo_app` が二重に表示されてしまう。

ちゃんと調べられていないが、恐らく README の記載では SPA として Ember が動く想定であって、
`include_ember_script_tags` で読み込まれる Embedded App という想定ではないからと思われる。

locationType は多分 hash にしておく方が
ember-rails からの移行だと URL が変わらなくて良さそう、と思いつつ、深い PATH とかにしてないからか検証はできてない


### app.js の設定 {#app-dot-js-の設定}

あとは app.js の方でも config/environemt から読むようにしたり
rootElement を指定したりしている。

rootElement は config/environment で指定して、
app.js ではそれを利用するのが正しい気はするが、一旦放置。

```js
const TodoApp = Application.extend({
  rootElement: '#todo-app',
  modulePrefix: config.modulePrefix,
  podModulePrefix: config.podModulePrefix,
  locationType: config.locationType,
  rootUrl: config.rootUrl,
  Resolver
});
```


## アプリの移植 {#アプリの移植}

これまでの手順ではとりあえず ember-cli で構築した空っぽの Ember.js アプリが
Rails の指定した PATH 上でとりあえず動くことを主眼に当ててやってきている。

ここからはようやく、既存アプリの実装の移植。いくつかの段階に分かれるから、ここからも長いんだけどね。


### 共通化してない機能のみで起動するようにする {#共通化してない機能のみで起動するようにする}

ember-libs というフォルダに切り出している部分までまとめて対応しようとするとえらく面倒なので、そのあたりを呼び出している部分はコメントアウトなどで呼び出されないようにして、とりあえず最低限の表示がされる程度を目指して移植するフェーズ。

やってることは
<https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/3c31b5bcf86d68ac5db0eca9bb4af410df31c2f1>
のコミットが全てである。

ざっくり説明すると

-   ember-rails で作っていた adapter, component, controller, initializer, model, route, template 等を ember-cli で作ったアプリの適切なディレクトリに配置
    -   router.js は ember-cli 自動生成の雛形に必要な部分だけ移植している
    -   adapter は ActiveModelAdapter を active-model-adapter addon から import するように変更している
-   共通ライブラリに持って行った component の呼び出し部分をコメントアウト

という感じ。これをすることで、不完全ながらも元のアプリと同じものが動くようになる

ちなみにもっと複雑なアプリだと mixin を使っていたりなどするがそれもテキトーに読み込まれないようにするなどで対処したらなんとなーく動く感じになるはず。なんとなーく。

そうそう。ember-cli 対応することで各ファイルの単体テストなんかを書けるようになってるはずだけど元々そんなものを書いてないので、今回もそこまで頑張る必要はないと判断してフロントエンドのテストは一切書いていません。自動生成されたファイルはそのまま追加しているけど。

一応、動作保証は system spec である程度担保しているつもり。
ember-rails の時はそこでしか保証してないしね。


### 共通ライブラリの Addon 化 {#共通ライブラリの-addon-化}

上までの段階だと共通ライブラリにした部分が全然動かないので、当然それを動く状態に持って行く必要がある。

で、その際には、共通ライブラリを addon として構築し直すことをオススメする。なぜなら、なんか無理やり自前の仕組みで動くようにするより公式に提供されてる仕組みに乗っかる方が後々楽そうだからだ。

ember-rails で動かしていた時に自前で解決していたのは
ember-rails だと addon がサポートされてないからというだけの理由だしね。

Addon 化の手順は大体以下の感じ

1.  ember-cli で Addon を generate
2.  共通ライブラリのコンポーネントを Addon に移植
    -   もし共通ライブラリに mixin とかも作っていたら同様に移植すること
3.  Addon をアプリ側で使えるように変更

なお今回の手順では App と同様に Addon のテストを書く、みたいな丁寧な暮らしはしていない。元々書いてないんだし、そこまで頑張る必要もないという判断。

あとやってることは [Addon 作成のチュートリアル](https://cli.emberjs.com/release/writing-addons/intro-tutorial/) に書いていることをベースにしている


#### ember-cli で Addon を generate {#ember-cli-で-addon-を-generate}

<https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/4d6713abfbed3217d65f7382e1f46d341c11d6aa>
でやっていることである

```text
$ cd ember && ember addon my-components --skip-git --yarn
```

というように適当な名前の Addon を作ってるだけ。


#### 共通ライブラリのコンポーネントを Addon に移植 {#共通ライブラリのコンポーネントを-addon-に移植}

-   <https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/cde30b30727d6eb9507b835d009d85759ddff5ee>
-   <https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/4ad2f8a59ccc846a63e6ff31c8f8b53df81d8e42>
-   <https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/30439f21f0659044bb4d2ea80ce68a2f8e0011b7>

あたりでやってる作業。

実際の作業では1つ目を移植してみた段階で、動作確認のためにアプリ側で Addon が使えるように設定していたりする。

ちなみに ember の addon は
app/components のファイルから addon/components のファイルを import してやるみたいなお作法がある。


#### Addon をアプリ側で使えるように変更 {#addon-をアプリ側で使えるように変更}

まずは上の手順で作った my-components という addon を
App 側で読み込めるように package.json の dependencies に以下を書き加える

```json
"my-components": "link:../my-components"
```

ember-cli の公式ドキュメントだと「yarn link を使う」というように書いているがそれだとうまくいかないみたいな Issue が何個か立っているのでドキュメント通りのやりかたは諦めて、それらの Issue の中に書かれている方法を選択した。

npm link だとうまくいきそうな雰囲気もあるので
yarn を使わず npm link にしておけばいい可能性はある。未検証。

まあそれらは置いといて、とにかく Addon が使える状態になったら各コンポーネントでコメントアウトとかで読めなくしていた共通ライブラリの呼び出しを元に戻したり記述を直したりして、元のように動くようにしましょう。


### ember-bootstrap の導入と bootstrap を使った機能を移植 {#ember-bootstrap-の導入と-bootstrap-を使った機能を移植}

ここまでやって、麦汁さんは「わーい動いた〜」と思っていたけどボタンとかをクリックしてみると、Bootstrap 関係のやつが動かない。

そう。元の記述のままだと Bootstrap 関係のやつはメソッド呼び出しでエラーになって動かないのです。というわけでそれらも動くようにしないといけない。

というところで、どうやるのが手っ取り早いかというと
[ember-bootstrap](https://github.com/kaliber5/ember-bootstrap) という Addon が転がっているのでそれをインストールして使うように変更するのが多分手っ取り早い。


#### ember-boostrap のインストール・初期設定 {#ember-boostrap-のインストール-初期設定}

最新版は ember-cli-rails@2.18.2 をサポートしていないので3系を使う必要がある。

```text
$ cd ember/todo-app && ember install ember-bootstrap@3.1.4
```

その上で、元々使っている Bootstrap のバージョンに合わせて
ember-bootstrap でも3系が使われるように設定する。

```text
$ cd ember/todo-app && ember generate ember-bootstrap --bootstrap-version=3
```


#### Bootstrap を使ってる機能の移植 {#bootstrap-を使ってる機能の移植}

<https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/22a3bff502ce993c2f2288623b061a4f38652a29>
でやっていることである。

基本的には、自前で bootstrap 用に DOM を組み立てていたところを
ember-bootstrap の Modal コンポーネント用に書き換えて、開いたりするための挙動を修正するだけである。

ember-boostrap の公式ドキュメントでは Handlebars の書き方が
`<BsModal>` みたいになっていて
3.4 以降でサポートされた Angle Bracket 方式の表記になっているが、
`<>` は `{{}}` に置き換えて
PascalCase を snake-cake にしたりするぐらいで動くので、落ち着いて移植しよう。


### 既存の system spec が新しい PATH で動くことを確認 {#既存の-system-spec-が新しい-path-で動くことを確認}

ここまでやると、全機能を手動で確認できる状態になってるので既存の system spec がアクセスするポイントを新しく作ったアプリの方に変更しテストが通ることを確認すると、ちゃんと移植できたんだなって安心できる

<https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/0c59057ec458edb7cda0febd15585dfc0a916bc1>


## 元の PATH で動くようにする {#元の-path-で動くようにする}


### 元の PATH への再移植 {#元の-path-への再移植}

<https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/334ce5052564a1499de03fb5a6630af3a339af21>
でやっていること。

1.  EmberCliTodoItemsController と TodoItemsController に移植
2.  app/views/ember\_cli\_todo\_items/index.html.haml を app/views/todo\_items/index.html.haml に移植
    -   rootElement に使う ID も `todo-app` に変更
    -   `%base` の href 属性も `/todo_items/` に変更
3.  resources :ember\_cli\_todo\_items を削除
    -   同時に controller, view も消す
4.  mount\_ember\_app で `to` と `controller` の指定を変更
    -   `to` を `/todo_items` に変更
    -   `controller` を `todo_items` に変更
5.  Ember.js 側で rootElement を `#todo-app` にする


### テストの PATH を戻す {#テストの-path-を戻す}

これは
[既存の system spec が新しい PATH で動くことを確認](#既存の-system-spec-が新しい-path-で動くことを確認) でやったことを revert してテストが通ることを確認したら OK


## ember-rails 関連の削除 {#ember-rails-関連の削除}


### ember-rails 用のコードの削除 {#ember-rails-用のコードの削除}

<https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/138ac7b8a76ec0f299edb2d626c9252927647229>
でやってるように
app/assets/javascripts の下にある
ember-rails 関連のコードを全部消すだけ。


### ember-rails 用の設定を削除 {#ember-rails-用の設定を削除}

<https://github.com/mugijiru/ember-rails-todo-app/pull/18/commits/9e036017b7ebee1a84b6f6847d5079a61ca5177c>
でやってるように

-   `config/application.rb`
-   `config/initializers/assets.rb`

の中に ember-rails のために書いた設定を丸っと消しましょう。もう不要なので。


### ember-rails 及びその関連 Gem と決別 {#ember-rails-及びその関連-gem-と決別}

設定も消せたら ember-rails, ember-source も要らないのでさっくり Gemfile から消して bundle install し直しましょう。イエイ。


## GitHub Actions の修正 {#github-actions-の修正}

あとはやり残しとしては
CI でもちゃんとテストが通るようにすること。

このプロジェクトでは GitHub Actions を使ってるのでそのワークフローを修正する


### 最新 LTS の Node.js を使うようにする {#最新-lts-の-node-dot-js-を使うようにする}

Dockerfile のところでもやりましたね。同じようなことをしましょう。とは言っても setup-node という action が公式に提供されているし
[公式ドキュメント](https://docs.github.com/ja/actions/guides/building-and-testing-nodejs) もあるので、それに従って設定するだけで使えるようになる。

```yaml
- name: Use Node.js
  uses: actions/setup-node@v1
  with:
    node-version: 14.x
```


### ember-cli をインストール {#ember-cli-をインストール}

これも似たようなことを Dockerfile でやってるので同じ感じに。

```yaml
- name: install ember-cli
  run: yarn global add ember-cli
```


### Ember Addon 及び Ember App で yarn install {#ember-addon-及び-ember-app-で-yarn-install}

こちらも依存を解決してやる必要があるので。

```yaml
- name: Setup Ember.js Addon
  run: |
    cd ember/my-components
    yarn
- name: Setup Ember.js App
  run: |
    cd ember/todo-app
    yarn
```

ここまでやると
GitHub Actions でもテストが通るし普通に使えるようになる。やったね。


## 最後に {#最後に}

以上の手順で ember-rails から ember-cli-rails への置き換えができます。

現実世界のアプリケーションはこのケースよりもっと複雑でしょうけども、やってやれないことはないはず。

それに ember-cli が使えるようにしておかないと
3系に移行ができないし、つまり、サポート切れのフレームワークを使い続けることになるのでもしまだ ember-rails のアプリが残っていたら頑張ってやっていきましょ。別フレームワークに置き換えるよりは労力はかからないはずですし。
