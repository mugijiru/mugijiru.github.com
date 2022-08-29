+++
title = "Kibela の記事を検索できる Ivy 拡張を作った"
date = 2022-08-29T09:53:00+09:00
categories = ["Emacs"]
draft = false
+++

Emacs の [ivy](https://github.com/abo-abo/swiper) を使って [Kibela](https://kibe.la/) の記事を検索できる拡張 [ivy-kibela](https://github.com/mugijiru/ivy-kibela) を作りました。

まあ自分用に作っただけなので Melpa にも登録してなかったりと、ちゃんとしてない部分は色々ありますが。


## 作った理由 {#作った理由}

現職では情報共有ツールとして Kibela を使っています。そして Kibela の記事を探したい時ってのは大体 Emacs でプログラムを書いている時なので
Emacs 内からさくっと検索できると便利だな〜と思って作りました。


## インストール方法 {#インストール方法}

README に一応書いているけど、さらっとしすぎているので一旦こっちに厚めに書きます。


### el-get {#el-get}

多分

```emacs-lisp
(el-get-bundle mugijiru/ivy-kibela)
```

で入れられるんじゃないかな。もしダメだったら

<https://github.com/mugijiru/.emacs.d/blob/master/recipes/ivy-kibela.rcp>

に el-get 用のレシピを置いてるのでそれを使ってください。


### マニュアルインストール {#マニュアルインストール}

依存関係に

-   request
-   graphql
-   ivy

があるのでそれらを事前に入れておいてください。そして

```text
$ git clone git@github.com:mugijiru/ivy-kibela.git
```

で Clone してきて

```emacs-lisp
(add-to-list 'load-path "CloneしたPATH")
```

みたいな感じで load-path に突っ込んだら使えるようになるはずです


### その他 {#その他}

Melpa への登録は大変そうだったので登録していません。なので package.el とかは使えません。

登録したい気持ちはなくはないけど、頑張る気力がない……。


## 使い方 {#使い方}

設定とかは [README](https://github.com/mugijiru/ivy-kibela#%E8%A8%AD%E5%AE%9A) に割とちゃんと書いているつもりなので省略。

コマンドは

-   ivy-kibela
-   ivy-kibela-recent
-   ivy-kibela-search

の3つを用意しています。


### ivy-kibela {#ivy-kibela}

ivy-kibela はデフォルトでは ivy-kibela-recent と同じ動作をします。設定でその動作を ivy-search と同じ動作をするように切り替えられます。よく使う方の動作を設定しておくと便利かもしれません。

裏事情的には、単に実装当初は ivy-kibela-recent 相当の処理しかなかったので互換性のために用意されているだけだったりはします。


### ivy-kibela-recent {#ivy-kibela-recent}

ivy-kibela-recent は直近投稿された記事情報を取得して絞り込むためのコマンドです。直近 100 件の記事を取得し、そこから ivy のインターフェースで絞り込んで Enter を叩くとブラウザでその記事を開きます。

情報を取得してから絞り込むので、
API を叩くのは1回で済むし、絞り込み時に wait が発生しません。そのため、直近の記事を検索する時はこちらを利用するのをオススメします。

また [ivy-migemo](https://github.com/ROCKTAKEY/ivy-migemo) を導入していたら

```emacs-lisp
(with-eval-after-load 'ivy-kibela
  (add-to-list 'ivy-re-builders-alist '(ivy-kibela . ivy-migemo--regex-plus) t))
```

とかしておくと migemo も使えます。
[ivy-migemo](https://github.com/ROCKTAKEY/ivy-migemo) 便利。


### ivy-kibela-search {#ivy-kibela-search}

ivy-kibela-search は ivy のインターフェースから
Kibela の検索 API を叩くコマンドです。
3文字以上入力した場合に検索 API にリクエストを飛ばし、その結果から記事・コメントを選択し Enter を叩くとブラウザで選択した記事・コメントを開きます。

3文字以上を入力した場合に API を叩くようになっているため例えば "React" と type した場合には "Rea", "Reac", "React" と
3回 API を叩くことになり、
[検索コスト](https://github.com/kibela/kibela-api-v1-document#1%E6%99%82%E9%96%93%E3%81%94%E3%81%A8%E3%81%AB%E6%B6%88%E8%B2%BB%E3%81%A7%E3%81%8D%E3%82%8B%E3%82%B3%E3%82%B9%E3%83%88) を多く消費するかもしれません。

また、検索には Kibela の API を使う関係上
ivy-migemo は利用できません。


### ivy-kibela-recent と ivy-kibela-search の使い分け {#ivy-kibela-recent-と-ivy-kibela-search-の使い分け}

私は「最近書かれたあのあたりの記事を開きたいな〜」という時に ivy-kibela-recent を使い、「あーあの記事古いよな」って時には ivy-kibela-search を使う、という感じで使い分けています。

ivy-kibela-recent を使うとローカルでの絞り込みなのでストレスが少ないのと ivy-migemo も使えるので絞り込みやすくて便利。「今開発しているやつの情報」は大体新しいのでこっちで十分だったりする。

ただし直近 100 件しか取れないので、古い記事を調べたい時は ivy-kibela-search を使う、という感じの使い方をしています。

記事を探すのが楽になったので自分的には便利。
