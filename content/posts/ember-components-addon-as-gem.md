+++
title = "自作の Ember.js Addon を ember-rails 用に無理やり Gem 化した"
date = 2021-03-26T01:49:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

[前回の記事]({{< relref "my-ember-libs-to-npm" >}}) で Ember.js の共通コンポーネントを詰めた Addon を
NPM package にしたわけですが、今度はそいつを割と無理やり Gem 化して
ember-rails でも使えるようにしたよ、というお話です。


## 目的 {#目的}

Ember.js の Addon として切り出した共通コンポーネントを同一ソースで ember-rails でも使いたいな〜、使えるようにしたいな〜、という目的。

ほら。
ember-rails で1つの Rails アプリケーションの上に複数の Ember.js アプリを動かしていて一部ずつ ember-cli-rails 移行を進めていたらどうしても混ざる時期あるじゃないですか。

そういう時に共通コンポーネントは同一ソースで両方で動かせると多分便利じゃないですか。


## ember-components の Gem 化 {#ember-components-の-gem-化}

Gem にして ember-rails でも使えるようにするために色々やりました。こんなにやらないといけないのかってぐらいやった気がします。。。


### Component の書き方を古い方式に戻した {#component-の書き方を古い方式に戻した}

ember-rails だとどうも

```js
import Component from '@ember/component'

export default Component.extend({})
```

という書き方だと読み込んでくれないようなので全部以下のように書き換えている。

```js
import Ember from 'ember

export default Ember.Component.extend({})
```

で、この変更を加えると eslint に怒られるので新しい記述を要求する eslint のルールをオフにしてあげる必要がある。悲しい。

```js
rules: {
  'ember/new-module-imports': 'off'
},
```


### components を ember-rails で読み込めるようにする {#components-を-ember-rails-で読み込めるようにする}

ember-libs というフォルダに共通コンポーネントとして分割した時も同じようなことをしたんだけど
ember-rails に components を読み込ませるためのコードをこのリポジトリに用意してある。

[lib/ember/components/templates/ember-components.js](https://github.com/mugijiru/ember-components/blob/bfbcda1c31a8bdf0efcb6aeaa0fb15efaccc5a7a/lib/ember/components/templates/ember-components.js)

やってることは、
requirejs で読み込まれてるファイルを調べて component を見つけ次第
`application.register` するだけのコードである。このコードは後で利用側から実行されるようにする。


### addon 以下のファイルを vendor/assets 以下にコピー、変更する Raketask 作成 {#addon-以下のファイルを-vendor-assets-以下にコピー-変更する-raketask-作成}

ここでやってることは

-   上で用意した ember-rails に読み込ませるためのコードをコピー。
-   Rails が読んでくれるところにファイルを置きたいのでaddon 以下のファイルを vendor/assets/javascripts 以下にコピー
-   ember-rails で module として読み込んでほしいので拡張子を `.module.es6` に変更
-   `import layout` などの Addon 用記述があるとエラーになるのでそれらの記述を強制排除

となっている。

後者2つは実装都合上、まとめてやっている


#### ファイルのコピー {#ファイルのコピー}

addon 以下に入っていても Rails 的には通常読み込めないので
`vendor/assets/javascripts` 以下にファイルをコピーしてあげている。あと上の手順で作った ember-rails に読み込ませるためのコードもコピーしている。

<https://github.com/mugijiru/ember-components/blob/bfbcda1c31a8bdf0efcb6aeaa0fb15efaccc5a7a/Rakefile#L14-L18>

```ruby
path = 'vendor/assets/javascripts/ember-components'
FileUtils.mkdir_p(path)
FileUtils.cp('lib/ember/components/templates/ember-components.js', "#{path}.module.es6")
FileUtils.cp_r('addon/templates', path)
FileUtils.cp_r('addon/components', path)
```

多分 `app/assets/javascripts` 以下でもいいんだろう。というかそっちの方が良さそうな気もするけど、
`app` は Ember.js 側で使っているので、それと混ざると嫌だなということで避けている。


#### addon 用の記述削除 & 拡張子の変更 {#addon-用の記述削除-and-拡張子の変更}

component に関しては addon での component 作成のお作法に従い
`import layout` とか書いているけど
ember-rails ではその記述はむしろ不要になるというか
hbs を import できない問題が発生するのでそれらの行を強制的に削除する処理を入れている。

また、それと同時に ember-rails で ES6 module として読み込めるように拡張子を `.module.es6` にしている。

方法としては、ファイルを `.js` から `.module.es6` にコピーしつつ不要な行を消してそれが済んだら `.js` ファイルを消すという手法を取ってる。結構、無理やり感がある。

<https://github.com/mugijiru/ember-components/blob/bfbcda1c31a8bdf0efcb6aeaa0fb15efaccc5a7a/Rakefile#L19-L33>

```ruby
Dir["#{path}/components/*.js"].each do |file_path|
  File.open(file_path, 'r')
  basename = File.basename(file_path, '.js')
  File.open("#{path}/components/#{basename}.module.es6", 'w') do |write_f|
    File.open(file_path, 'r') do |read_f|
      read_f.each do |line|
        next if line =~ /^\s*import layout/
        next if line =~ /^\s*layout,/

        write_f.puts line
      end
    end
  end
end
FileUtils.rm(Dir.glob("#{path}/components/*.js"))
```


### Rails Engine 化 {#rails-engine-化}

Rails Engine として組み込んで使えるように
`lib` 以下にちょろちょろコードを書いている。

-   [lib/ember/components.rb](https://github.com/mugijiru/ember-components/blob/bfbcda1c31a8bdf0efcb6aeaa0fb15efaccc5a7a/lib/ember/components.rb)
-   [lib/ember/components/version.rb](https://github.com/mugijiru/ember-components/blob/bfbcda1c31a8bdf0efcb6aeaa0fb15efaccc5a7a/lib/ember/components/version.rb)
-   [lib/ember/components/engine.rb](https://github.com/mugijiru/ember-components/blob/bfbcda1c31a8bdf0efcb6aeaa0fb15efaccc5a7a/lib/ember/components/engine.rb)

ほとんど「Rails Engine のお作法」ってだけのコードだけど上に書いたファイルをコピーしたりする時の
PATH を取得するための便利メソッドとして以下を生やしている。

```ruby
def self.root
  Pathname(__FILE__).join('../../..')
end
```


### gemspec 修正 {#gemspec-修正}

Gem として GitHub Packages に登録するので当然 .gemspec ファイルを用意している。
[ember-components.gemspec](https://github.com/mugijiru/ember-components/blob/bfbcda1c31a8bdf0efcb6aeaa0fb15efaccc5a7a/ember-components.gemspec)

一応 GitHub Packages に出すためのお作法として

```ruby
spec.metadata["allowed_push_host"] = "https://rubygems.pkg.github.com"
```

というように push できるホストをしていしたり

```ruby
spec.metadata["github_repo"] = "ssh://github.com/mugijiru/ember-components.git"
spec.metadata["git_repo"] = "ssh://github.com/mugijiru/ember-components.git"
```

というようにリポジトリを指定していたりする。

[同じリポジトリへの複数パッケージ公開](https://docs.github.com/ja/packages/guides/configuring-rubygems-for-use-with-github-packages#publishing-multiple-packages-to-the-same-repository) の記述を読む限り
github\_repo だけ指定あれば良さそうな気もするが
git\_repo があっても特に害もないだろうということでとりあえず入れている。

あとは gem に含めたいファイルとして

```ruby
spec.files = Dir[
  'lib/**/*',
  'vendor/**/*',
  'README.md',
  'LICENSE.md'
]
```

としている。
lib 以下は Rails Engine として組込むために必要だし
vendor 以下には ember-rails で読み込める形に変換したファイルがあるので
gem に含める必要がある。


### GitHub Actions での Gem 登録 {#github-actions-での-gem-登録}

[NPM Package にした時]({{< relref "my-ember-libs-to-npm" >}}) と同様に
Tag を打ってそれからリリースを作ったら Gem が登録されるように
GitHub Actions を設定している。


#### Gem の build {#gem-の-build}

publish する前に以下のようにして Rake Task を実行している。

```yaml
- name: Build gem
  run: |
    bundle exec rake clean_assets generate_assets build
```

clean\_assets は説明してなかったけど `vendor/assets/javascripts` 以下を真っ新にするだけの処理。

で、generate\_assets が
[addon 以下のファイルを vendor/assets 以下にコピー、変更する Raketask 作成](#addon-以下のファイルを-vendor-assets-以下にコピー-変更する-raketask-作成)
のあたりで書いた、コピーしたり中身を弄ったりしている処理。

最後の build は Gem を作ったことある人ならわかるはずだけど
gemspec の記述に従って gem ファイルを生成する処理。これを実行する pkg 以下に `ember-components-x.y.z.gem` みたいなファイルが作られる。


#### Publish {#publish}

上の手順で gem はできたので、あとはそれを GitHub Packages に登録するだけである。そのための step が以下。

```yaml
- name: Publish to RubyGems
  run: |
    mkdir -p $HOME/.gem
    touch $HOME/.gem/credentials
    chmod 0600 $HOME/.gem/credentials
    printf -- "---\n:github: Bearer ${{ secrets.GITHUB_TOKEN }}\n" > $HOME/.gem/credentials
    gem push --key github --host https://rubygems.pkg.github.com/mugijiru pkg/*.gem
```

まずは [Authenticating with a personal access token](https://docs.github.com/en/packages/guides/configuring-rubygems-for-use-with-github-packages#authenticating-with-a-personal-access-token) の手順に従って
`~.gem/credentials` に
`github: Bearer ${{ secrets.GITHUB_TOKEN }}`
の記述が入るようにしている。

それをすると GitHub Packages の認証が通るようになるので

```text
gem push --key github --host https://rubygems.pkg.github.com/mugijiru pkg/*.gem
```

を実行することで Gem として登録ができる。


## ember-rails アプリケーションから Gem 化した Addon の読み込んで利用する {#ember-rails-アプリケーションから-gem-化した-addon-の読み込んで利用する}

<https://github.com/mugijiru/ember-rails-todo-app/pull/51>
の PR でやっていることである。

PR では途中色々ごちゃごちゃやってるけど、ここでは最終結果に基いて説明をする。


### Gem を bundle install できるようにする {#gem-を-bundle-install-できるようにする}

まずは bundle install で組込めないと何も始まらないので
Gemfile に以下を追加する。

```ruby
source "https://rubygems.pkg.github.com/mugijiru" do
  gem "ember-components"
end
```

さらに手元のマシンで以下のコマンドを実行して、
bundle install の際に GitHub Packages への認証が通るようにする。

```text
$ bundle config --local https://rubygems.pkg.github.com/mugijiru mugijiru:XXXXXX
```

`XXXXXX` には Gem をインストールできるパーソナルアクセストークンを設定すること。

こうしておけば

```text
$ bundle install
```

で無事に自作 Gem の ember-components がインストールできる

Docker を使ってる場合は以下のようにして
Docker 内で bundle config が設定された状態で `bundle` を実行する必要あり

```text
$ docker-compose run rails bash -c "bundle config --local https://rubygems.pkg.github.com/mugijiru mugijiru:XXXXXX && bundle"
```


### templates\_root への登録 {#templates-root-への登録}

ember-rails は Rails 側で templates\_root を設定してあげる必要がある。

というわけで config/application.rb で
`ember-components/templates` が templates\_root として認識されるように記述する。

```ruby
config.handlebars.templates_root = %w[todo-app/templates ember-components/templates]
```


### sprockets で ember-rails を読み込む {#sprockets-で-ember-rails-を読み込む}

Gem として読み込めるようになったので
Sprockets で以下のようにして require してあげると
Gem の `vendor/assets/javascrips/ember-components` に生成したファイルが
ember-rails アプリ側で認識されるようになる。

```js
//= require ember-components
```


### Ember.js に component を register する {#ember-dot-js-に-component-を-register-する}

require するだけだと Ember.js ではまだ使えないので
Gem 内の Componentを登録する必要がある。

が、基本的な処理は
[components を ember-rails で読み込めるようにする](#components-を-ember-rails-で読み込めるようにする) のところで書いたので、
ember-rails 側では initializers に以下のような内容のファイルを置けば良い。

```js
import EmberComponents from 'ember-components';

export function initialize(application) {
  EmberComponents.registerAll(application);
}

export default {
  name: 'register-ember-components',
  initialize: initialize
};
```

実質的にやってることは
Gem 内のスクリプトに定義している registerAll メソッドを叩いているだけ。

本当はこういう処理すらなしに使えるのがベストだけどそこまでうまくやる方法は見つけられず……。


### 利用箇所の修正 {#利用箇所の修正}

これは component の prefix を `my-` から `mg-` に変えたから発生している作業なので本質的には不要な作業。

とにかく `my-button` のような古い prefix になっているところを
`mg-button` というように新しい prefix に置き換えるだけの簡単なお仕事。


### GitHub Actions の修正 {#github-actions-の修正}


#### setup-ruby で ember-components を bundle install できるようにする {#setup-ruby-で-ember-components-を-bundle-install-できるようにする}

GitHub Actions の CI でも bundle install をしているのでそこでもインストールが正常に行われるようにしてあげないといけない。

```yaml
- uses: ruby/setup-ruby@v1
  env:
    BUNDLE_HTTPS://RUBYGEMS__PKG__GITHUB__COM/MUGIJIRU/: "mugijiru:${{ secrets.NPM_AUTH_TOKEN }}"
  with:
    bundler-cache: true
```

のように `bundle config` で設定したのと同じようなものを
env で設定してあげるとインストールができる。

NPM\_AUTH\_TOKEN なのは、NPM Package にした時に使ったやつが丁度いいスコープを持っていたから流用しちゃった。てへぺろっ。


#### assets:precompile {#assets-precompile}

Gem の作りが悪いのか、
rspec を流す前に

```text
$ bin/rails assets:precompile
```

を流さないと component の template がテスト環境でで読まれない。というわけで GitHub Actions で rspec を実行する前にその手順を挟んでいる。

<https://github.com/mugijiru/ember-rails-todo-app/blob/4acafe0fd741fd24dc4e6bc69d98df5cbb68ef0e/.github/workflows/ci.yml#L32>

ちなみにこれは手元で rspec を流す時も同じなのでちゃんと手元のマシンでも precompile してあげましょう。だるい。


### 旧共通ライブラリの削除 {#旧共通ライブラリの削除}

`app/assets/javascripts/ember-libs` に配置していたファイルは不要なのでさっくりと

```text
$ rm -rf app/assets/javascripts/ember-libs
```

して

```js
//= require_tree ../ember-libs
```

としている行が残っていればそれも削除すること。

`config/application.rb` で templates\_root として
`ember-libs/templates` を追加している場合はそれも削除しておくこと。まあこれは残っててもエラーにならないけどね。


## 旧スタイルの ember-rails アプリケーションでも Gem 化 Addon を利用する {#旧スタイルの-ember-rails-アプリケーションでも-gem-化-addon-を利用する}

<https://github.com/mugijiru/ember-rails-todo-app/pull/52>
でやっていること。

まあ正直 module 化しているやつとほとんどやってることは変わらない。

変わってる点は、registerAll の呼び出し方ぐらいで
TodoApp という Ember.js アプリケーションが入ってる変数がグローバル空間に収まっているので
application.js.es6 の方で直接以下のように書いている。

```js
import EmberComponents from 'ember-components';
EmberComponents.registerAll(TodoApp);
```

他は module 化しているパターンと一緒なので割愛。


## 最後に {#最後に}

という手順で
NPM Package にした Ember.js Addon を若干無理やりながらも ember-rails で使えるようにすることができました。

まあ mixin とかは試してないのと
Component をサブフォルダに分割していたりするともうちょっと手をかけないといけなさそうだけどとりあえず動いたから許して。

正直、無理やり感が結構あるので普通のプロダクトに適用するのは厳しい感じある。
