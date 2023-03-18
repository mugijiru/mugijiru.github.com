+++
title = "Emacs で動く Kibela クライアントを作ってる"
date = 2023-03-18T15:58:00+09:00
categories = ["Emacs", "Kibela"]
draft = false
+++

最近 Emacs 上で動く Kibela のクライアントを作り始めたからちょっと紹介してみる


## Kibela とは {#kibela-とは}

[Kibela](https://kibe.la/) は知識共有やドキュメント管理を目的とした SaaS 型のナレッジ管理ツールの一種。

記事を書く時は Markdown エディタで書くこともできるしリッチテキストエディタで書くこともできるので
Markdown に不慣れな方も Markdown で書きたい人も同じ記事をそれぞれが好きなエディタで編集することができるあたり便利。

無料で使えるコミュニティプランもあるので個人でも気軽に試せるのも便利。

そして [API が公開されている](https://support.kibe.la/hc/ja/articles/360035089312-Kibela%E3%81%AEWeb-API%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) ので、自分でクライアントも作れるわけですよ。


## Emacs で動く Kibela クライアント kibela.el {#emacs-で動く-kibela-クライアント-kibela-dot-el}

[kibela.el](https://github.com/mugijiru/emacs-kibela) は Emacs 上で動く Kibela クライアントです。
Kibela を普段使ってるので、それを Emacs から触れると便利だなあと思って作っています。

上述の通り Kibela は API を公開しているのでそれを利用しています。


### 機能 {#機能}

今の kibela.el は以下の機能を備えています。使い方は README に書いているはず

-   デフォルトグループの記事一覧表示
    -   40件毎の表示します。
    -   `<`, `>` でのページ送りに対応
-   記事の新規作成
    -   記事テンプレートから作成することもできます。
    -   投稿先の変更機能は未実装です
-   記事の閲覧・編集
    -   記事一覧から閲覧画面へ遷移し、閲覧画面から編集モードに遷移できます。
    -   閲覧画面で `C-c C-c C-o` と入力することで記事をブラウザで開くこともできます。
        -   kibela.el ではできないこともブラウザで開いたら操作できる、という逃げ。
    -   こちらも投稿先の変更機能は未実装です
-   チームの切り替え


### 動画キャプチャ {#動画キャプチャ}


#### テンプレートからの記事の投稿 {#テンプレートからの記事の投稿}

<div class="org-youtube"><iframe width="600" height="337" src="https://www.youtube.com/embed/_FaUlP1rHMw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></div>


#### 記事一覧から閲覧・編集 {#記事一覧から閲覧-編集}

<div class="org-youtube"><iframe width="600" height="337" src="https://www.youtube.com/embed/BNsHa4peUmY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></div>


### 今後やりたいこと {#今後やりたいこと}

自分が普段使う上で「これができると Emacs 上でいつもの作業が済んで楽だな〜」ということから作っていくつもりです。

今ぱっと思い付くのはこのあたりかな

-   いいね! の表示/投稿
    -   これができると新しい記事を適当に眺めてリアクションしていくことができて便利
-   グループ一覧の表示とデフォルトグループ以外の記事一覧表示
    -   これができると各グループの記事を追えて便利
-   通知の表示
    -   メンション飛ばされているのとかも Emacs から確認できると便利そうなので
-   検索
    -   古い記事は記事一覧からは流石に探せないので
-   埋め込み画像の表示
    -   対応が面倒なので後回しにしていますが表示したい
    -   動画についてはあまり考えてない
-   フォルダツリー
    -   あったらフォルダから記事を絞り込むことができて便利かも
-   絵文字の表示や補完
    -   あると記事書く時に使えて便利。まあなくてもさほど困らないけど
-   下書き保存・編集
    -   あったら便利。なくてもまあいいけど

ま、飽きるまでやるつもり。すぐ飽きそうな気もするけど。
