+++
title = "Manjaro Linux のディスプレイ周りの設定を弄った"
date = 2021-11-24T14:01:00+09:00
tags = ["Manjaro", "config", "display"]
categories = ["Linux"]
draft = false
+++

Manjaro Linux の設定を弄ってる系の記事です。まあまだ2記事目だけど。

家には外部ディスプレイが存在するけどまあ必ずいつも繋いでるわけでもないので、両方の状態に対応できるようにしたいよねって思ってた。

それをするには [XRandR](https://www.x.org/wiki/Projects/XRandR/) とそれの設定を保存したりするのに [autorandr](https://github.com/phillipberndt/autorandr) を使えばいいというのがわかった。ありがとう [Arch Wiki](https://wiki.archlinux.jp/index.php/Xrandr)。今確認していると autorandr ではなくて [xlayoutdisplay](https://github.com/alex-courtis/xlayoutdisplay) とやらでも良さそうだがね……。

まあそれはさておき、ひとまずはディスプレイに繋いでる状態をなんとかしたかったので最初は

```sh
xrandr --output eDP-1 --mode 2880x1620 --right-of DP-2
```

というコードを .xprofile に書いていた。
eDP-1 がノート PC のディスプレイで DP-2 が外部接続しているディスプレイである。つまり eDP-1 は 2880x1620 の解像度で出力し
DP-2 の右側に配置する、といった感じである。

DP-2 自体の設定はどこかにいったが、まあ解像度を弄ってるぐらいなのでとりあえずヨシ。

とりあえずこれにより、ディスプレイを繋いでる状態は良かったんだけどもディスプレイを切り離した時に、仮想デスクトップ2が切り離した方に表示されてるような認識をされていたので困ってしまった。

調べたところ

```sh
xrandr --output eDP-1 --mode 2880x1620 --output DP-2 --off
```

を実行することで DP-2 つまり外部ディスプレイをオフにできることがわかった。

というわけで、手動でコマンドを叩く前提にはなるが一旦解決はした。

そしてこれらそれぞれの状態を autorandr で保存するためまずは繋いでる状態で

```sh
autorandr --save docked
```

として保存し、切り離して先のコマンドを実行し、ノートのディスプレイだけを使える状態にした上で

```sh
autorandr --save mobile
```

と実行することで、それぞれの状態を

-   docked
-   mobile

として記録しておいた。

これにより、ディスプレイを繋いだり外したりした時に望んだ通りの設定に自動で切り替わるようになった(切り替わりは今記事を書きながら試した)

というわけで、切り離した時に表示調整を自分でする必要がなくなった。便利。
