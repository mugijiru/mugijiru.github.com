+++
title = "GitHub Actions で使ってる Action 達"
categories = ["GitHub"]
draft = true
+++

個人リポジトリでは CI などを GitHub Actions に任せている。で、誰かが作ってくれた色々な Action に助けられている。

というわけでそれらを、より多く助けられてるなってやつから紹介してみようと思う。

正直、自分が何を使ってたかを後から調べる時にこの記事を読めば大体済むよねってなりそうなので書いている部分が大きい。すーぐ忘れるもん。


## actions/checkout {#actions-checkout}

<https://github.com/marketplace/actions/checkout>

多分 GitHub Actions を使ってる人ならみんな使ってる。だって普通チェックアウトするもんね。使うよね。

今のところ自分は単純に checkout するのにしか使ってないが何気にオプションが多い。それらの使い方もしっかり書かれていて良い。使ってないけど。


## lazy-actions/slatify {#lazy-actions-slatify}

<https://github.com/marketplace/actions/slatify>

Slack 通知用の Action ですね。オプションが結構充実している。
CI 終了後に成功/失敗を通知するのに使っている。

元は `homoluctus/slatify` だったので
ember-rails-todo-app とかでまだその設定を残している。変更しないと〜。


## ruby/setup-ruby {#ruby-setup-ruby}

<https://github.com/ruby/setup-ruby>

Ruby やるならこれだよねってやつ。

多数のバージョンをサポートしているしリポジトリに .ruby-version ファイルがあればそれを見てくれる。

Bundler 周りも cache 使ったりとかをいい感じにしてくれてとても便利。


## nanasess/setup-chromedriver {#nanasess-setup-chromedriver}

<https://github.com/marketplace/actions/setup-chromedriver>

CI で Chrome でもテストしたい時に使ってる。
Rails で Capybara でテストする時ですね。

chromedriver のセットアップなので Capybara 使う時はこれでいいんじゃないかなって思ってる。
[rspec 側でも色々書く必要があるのでだるい](https://github.com/mugijiru/ember-rails-todo-app/blob/a98a66ae799eb60f16569c16165d3c456d041e76/spec/support/capybara.rb#L3-L14) けど、まあ Chromedriver だからなのかな。

他のプロジェクトだとテストどうやってるのかな。今度調べてみたいね。


## actions/setup-node {#actions-setup-node}

<https://github.com/marketplace/actions/setup-node-js-environment>

Node.js をセットアップしてくれる。自分は単純な使い方しかしてないけど、何度もお世話になっている。


## reviewdog/action-setup {#reviewdog-action-setup}

<https://github.com/reviewdog/action-setup>

eslint とか Rubocop とかの結果を使って GitHub 上で指摘してくれたりする便利なワンちゃん。

まあ eslint や Rubocop あたりは
[reviewdog/action-eslint](https://github.com/reviewdog/action-eslint) とか後述の [reviewdog/action-rubocop](#reviewdog-action-rubocop) とかを使えたらそっちを使う方が良い。

自分は eslint は yarn を使ってるからかなんかの理由でそっちだと微妙だったので
reviewdog/action-setup で reviewdog のセットアップだけして自前でコマンドを叩く方法を選択している。


## reviewdog/action-rubocop {#reviewdog-action-rubocop}

<https://github.com/marketplace/actions/run-rubocop-with-reviewdog>

最近入れてみた。設定にもよるけど、Rubocop で引っ掛かったところについて
PR のチェックリスト的なやつで教えてくれたり、コメントを入れてくれたりする。
Rubocop 先生に怒られたかったので便利。
