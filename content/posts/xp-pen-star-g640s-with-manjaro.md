+++
title = "Manjaro Linux にペンタブレット(XP-PEN Star G640S)を設定した"
date = 2021-11-24T15:58:00+09:00
tags = ["Manjaro", "config", "pentablet"]
categories = ["Linux"]
draft = false
+++

今年の前半ぐらいに購入しておいて放置していた
XP-PEN の Star G640S を
Manjaro Linux 環境で使えるか試してみたらとりあえずあっさり動いたというメモ。

まず最初は単に接続して使えるかを試したら、とりあえず動いた。

そんでもって、設定を変更したいな〜と思ったら
[公式にドライバがある](https://www.xp-pen.jp/download-166.html)し、
[AUR にはそれにパッチを当てたやつ](https://aur.archlinux.org/packages/xp-pen-tablet)が転がっていた

というわけで

```sh
yay -S xp-pen-tablet
```

したらそれが入って来た。

その上で `pentablet` というのを rofi から起動すると
XP PEN の設定ソフトが起動したのでそれを使って適当に設定したらその通りに動いて良かった。具体的には、配置の関係上、上下さかさまで使うように設定した。
USB ケーブルの位置がね……。

ちなみに `xp-pen` というのも AUR には転がってるけどもベースとなってるバージョンがちょびっとだけ古いのと人気度的に `xp-pen-tablet` の方が上だったので、そっちを選んでいる。

他にも [Krita](https://krita.org/jp/) ってソフトで試したんだけど、筆圧もちゃんと検知するみたいだしこれは良さそう。とはいえ絵を描いたりはしないので、筆圧は実は要らないんだけども。

まあ普通に使えて良かったので今後使うかもしれない。うん。
