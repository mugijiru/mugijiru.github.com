+++
title = "GitHub Actions で使ってる Action 達"
date = 2021-05-30T09:50:00+09:00
categories = ["GitHub"]
draft = false
+++

個人リポジトリでは CI などを GitHub Actions に任せている。で、誰かが作ってくれた色々な Action に助けられている。

というわけでそれらを、より多く助けられてるなってやつから紹介してみようと思う。

正直、自分が何を使ってたかを後から調べる時にこの記事を読めば大体済むよねってなりそうなので書いている部分が大きい。すーぐ忘れるもん。


## actions/checkout {#actions-checkout}

<https://github.com/marketplace/actions/checkout>

多分 GitHub Actions を使ってる人ならみんな使ってる。だって普通チェックアウトするもんね。使うよね。

今のところ自分は単純に checkout するのにしか使ってないが何気にオプションが多い。それらの使い方もしっかり書かれていて良い。

普通に使う分には

```yaml
- uses: actions/checkout@v2
  with:
    ref: ${{ github.head_ref }}
    submodules: true
```

だけで良い。これで checkout される。楽。

自分の場合は submodule が必要だった際に

```yaml
with:
  submodules: true
```

としてみたり、
pull\_request で動かしたいけど、その hash ではなく branch をチェックアウトしたい時に

```yaml
with:
  ref: ${{ github.head_ref }}
```

としたりなどしてた。


## lazy-actions/slatify {#lazy-actions-slatify}

<https://github.com/marketplace/actions/slatify>

Slack 通知用の Action ですね。オプションが結構充実している。
CI 終了後に成功/失敗を通知するのに使っている。

Slack の方に Incoming Webhook を用意してそこを使ってメッセージを飛ばせるようにするやつ。

自分の場合はこんな感じで、成功失敗に関わらず通知を飛ばしていて、ただ失敗の場合は here mention が飛ぶようにしている。

```yaml
- name: Notify slack build result
  uses: lazy-actions/slatify@master
  if: always()
  with:
    type: ${{ job.status }}
    job_name: '*Build*'
    mention: 'here'
    username: 'GitHub Actions'
    mention_if: 'failure'
    channel: '#develop'
    url: ${{ secrets.SLACK_WEBHOOK }}
    commit: true
    token: ${{ secrets.GITHUB_TOKEN }}
```

ここではやってないけど絵文字も指定できるので便利。まあ Webhook 側に仕込んでいたらあまり使わないんだけども。

元は `homoluctus/slatify` だったので
ember-rails-todo-app とかでまだその設定を残している。変更しないと〜。


## ruby/setup-ruby {#ruby-setup-ruby}

<https://github.com/ruby/setup-ruby>

Ruby やるならこれだよねってやつ。

多数のバージョンをサポートしているしリポジトリに .ruby-version ファイルがあればそれを見てくれる。

Bundler 周りも cache 使ったりとかをいい感じにしてくれてとても便利。

というわけで今のところ自分は以下の感じで間に合ってる。

```yaml
- name: Use ruby
  uses: ruby/setup-ruby@v1
  with:
    bundler-cache: true
```


## nanasess/setup-chromedriver {#nanasess-setup-chromedriver}

<https://github.com/marketplace/actions/setup-chromedriver>

CI で Chrome でもテストしたい時に使ってる。
Rails で Capybara でテストする時ですね。

chromedriver のセットアップなので Capybara 使う時はこれでいいんじゃないかなって思ってる。
[rspec 側でも色々書く必要があるのでだるい](https://github.com/mugijiru/ember-rails-todo-app/blob/a98a66ae799eb60f16569c16165d3c456d041e76/spec/support/capybara.rb#L3-L14) けど、まあ Chromedriver だからなのかな。

とりあえず workflow 側ではこんな程度でしか使ってない。

```yaml
- name: use-chromedriver
  uses: nanasess/setup-chromedriver@master
```

他のプロジェクトだとテストどうやってるのかな。今度調べてみたいね。


## actions/setup-node {#actions-setup-node}

<https://github.com/marketplace/actions/setup-node-js-environment>

Node.js をセットアップしてくれる。自分は以下のような単純な使い方しかしてないけど、何度もお世話になっている。

```yaml
- name: Use Node.js
  uses: actions/setup-node@v1
  with:
    node-version: 14.x
```


## reviewdog/action-setup {#reviewdog-action-setup}

<https://github.com/reviewdog/action-setup>

eslint とか Rubocop とかの結果を使って GitHub 上で指摘してくれたりする便利なワンちゃん。

まあ eslint や Rubocop あたりは
[reviewdog/action-eslint](https://github.com/reviewdog/action-eslint) とか後述の [reviewdog/action-rubocop](#reviewdog-action-rubocop) とかを使えたらそっちを使う方が良い。

自分は eslint は yarn を使ってるからか自前の NPM ライブラリを使ってるからかで、
action-eslint だとうまく動かせなかったので
reviewdog/action-setup で reviewdog のセットアップだけして自前でコマンドを叩く方法を選択している。

というわけで

```yaml
- uses: reviewdog/action-setup@v1
  with:
    reviewdog_version: latest
```

でとりあえず使えるようにして

```yaml
- name: Run reviewdog
  env:
    REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    NPM_TOKEN: ${{ secrets.NPM_AUTH_TOKEN }}
  run: |
    yarn --silent run eslint -f rdjson . | reviewdog -f=rdjson -reporter=github-pr-review
```

みたいなことをしている。

Ember.js のテンプレートもそのうち対応しないとなあ……。


## reviewdog/action-rubocop {#reviewdog-action-rubocop}

<https://github.com/marketplace/actions/run-rubocop-with-reviewdog>

最近入れてみた。設定にもよるけど、Rubocop で引っ掛かったところについて
PR のチェックリスト的なやつで教えてくれたり、コメントを入れてくれたりする。
Rubocop 先生に怒られたかったので便利。

```yaml
- name: rubocop
  uses: reviewdog/action-rubocop@v1
  with:
    rubocop_version: gemfile
    rubocop_flags: -a
    rubocop_extensions: rubocop-rails:gemfile
    github_token: ${{ secrets.GITHUB_TOKEN }}
    reporter: github-pr-review
    fail_on_error: true
```

とりあえずこんな感じの設定で使ってる。基本的に Gemfile の設定を使って、後は何か見つかったら review として飛ばしてくれるようにしている。

rubocop\_flags の `-a` は要るのかな……。少なくとも eslint の方は `--fix` するとかえって思ったような挙動にならなかったような……。まいっか。しばらくそのまま運用しよう。
