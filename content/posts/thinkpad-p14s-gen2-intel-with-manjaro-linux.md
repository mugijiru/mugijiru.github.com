+++
title = "ThinkPad P14s Gen 2 (第11世代インテル) に Manjaro Linux を入れてみている"
date = 2021-10-10T19:01:00+09:00
tags = ["Manjaro", "config", "skk"]
categories = ["Linux"]
draft = false
+++

先月、「えいやっ」で
[Lenovo ThinkPad P14s Gen 2 (第11世代インテル)](https://www.lenovo.com/jp/ja/notebooks/thinkpad/p-series/ThinkPad-P14s-Gen-2-14%E2%80%9D-Intel/p/22WSP14P4S2) を購入した。詳細は面倒だから省くけどそれなりの構成にしている。なお AMD 版にしなかったのは Linux をちゃんと動かせるか不安だったから。

で、そのマシンに Manjaro Linux を載せて色々設定している。この記事は、そのあたりでどういう設定を書いているかの備忘録的なそれです。

だってさ〜、絶対忘れるじゃん。だからある程度書いておこうかなって。そしてその上でまた後で入れ直ししてそれを見ながらセットアップし直そうかなと。そしたら多分その時に見逃していた部分も修正できそうだし。

というわけでなんか適当につらつらと書いていく。

ま、Twitter につぶやいていたことを記憶で補完しながら書いている感じだけどね。

Twitter でのつぶやきだと Twilog で

-   <https://twilog.org/mugijiru/date-211003>
-   <https://twilog.org/mugijiru/date-211009>

あたりに該当する部分。まだ2日しか触れてないのよな。うん。


## Manjaro Linux のインストール {#manjaro-linux-のインストール}

<https://manjaro.org/download/> から Xfce 版のやつをダウンロードして
Windows PC で DVD に iso ファイルを書き込んでそれを使ってセットアップしてた。

その際に
<https://www.mimir.yokohama/useful/0023-installing-manjaro.html>
を参考にした。まあパーティションどうしたらいいんだっけ以外は特に悩まなかったけど。

あとは光学ドライブは大昔に購入した外付けのやつなので荷物の奥底から引っ張り出して「うわー USB type B のケーブルはどこだ〜!」「電源アダプタもねえぞー!」とか一人で騒いでた。みんなも機器を仕舞う時はそれに使うケーブルなども一緒に仕舞うようにしような。

あとパーティションは自動で切ってるけど
swap 領域はメモリ拡張すると足らないのでなんとかする必要がありそう。


## Font 設定 {#font-設定}

デフォルトの Terminal である xfce4-terminal のフォントが何故か文字間が広くて気持ち悪いので Sazanami と IPA フォントを入れた記憶がある。

history を漁ると

```text
$ pacman -S ttf-sazanami
$ pacman -S otf-ipafont
```

が出て来たのでとりあえずそうやって入れているっぽい。

多分 Sazanami を入れてるのは
<https://applecom.blog.jp/archives/31595846.html>
の影響かな。日本語環境構築したかったから見ていた記憶がある。

で、その記事でも後から IPA フォントを入れてるからその流れに乗って両方を入れてる感じがある。


### やってないことだけど補足 {#やってないことだけど補足}

今 Arch Wiki のフォントのページ
<https://wiki.archlinux.jp/index.php/%E3%83%95%E3%82%A9%E3%83%B3%E3%83%88#.E6.97.A5.E6.9C.AC.E8.AA.9E>
を調べてると otf-ipaexfont とか良さそうな雰囲気あるな。

あとはコーディングする上では ttf-mplus を入れておけばとりあず固定幅で使えそうなので一安心できそう。

ちなみに普段他の環境では Ricty diminished を使っているのでそこに戻る可能性もあるけど、新しい環境ということで新しいフォントにしてみるのも面白そうなので、どうするかはわからない。


## Terminal(xfce4-terminal) の設定 {#terminal--xfce4-terminal--の設定}

上に書いたようにフォントが気に食わなくて別のフォントを入れてその後適当に弄ってたけど、結局 Monospace 16 に戻った上で文字間は悪くない感じになってしまった。

あれ、これ最初の設定だよね?? なんで文字間の無駄に広いのが直ったの?? ってなってるけどまあ悪くないのでそのままで。

あとは背景が半透明だったけど個人的には半透明な Terminal は、カッコいいけど後ろ側が気になってしょうがないので

-   背景
    -   指定なし(単色を使用する)

という設定にすることで透明度をゼロにしている。

あと背景が透けるようにすると処理が重くなるってのもあるよね。


## Web ブラウザ周りの調整 {#web-ブラウザ周りの調整}

Manjaro の Xfce 版には標準で Firefox が入っていた。私の普段使いの Web ブラウザも Firefox なので丁度良い。

まずは言語設定がアメリカ英語だったので日本語に変更。英語設定のまま使い続けられるほど強くないのです。

次にフォントを IPA Pゴシックを基本的に使うように変更。

そして Firefox Sync で同期をするといつも使ってるアドオンが入っていい感じ。

多分それ以上は何もしてないかな。ここは楽だった。

なおその後アップデートしたら言語設定が吹っ飛んでてまた設定し直すことになった。


## 日本語入力(fcitx5-skk) {#日本語入力--fcitx5-skk}

日本語入力周りは最近の Linux では fcitx というのを使うらしいのでそれの中の SKK を使うやつを選択した。

というわけで
<http://neko-mac.blogspot.com/2021/06/archlinuxskk.html>
を参考に設定をした。

```text
$ sudo pacman -S fcitx5-skk
```

あとは fcitx5-configtool も必要なので入れた。

```text
$ sudo pacman -S fcitx5-configtool
```

これを別途入れないと設定ツールが起動できないのだが、
Not Found 的なエラーが出るわけでもなく単に起動できないだけなので何が問題なのか気付きにくかった。別途インストールが必要だったが、それに気付くまで2,3時間溶かした気がする。

そんでもって fcitx5-configtool を

```text
$ fcitx5-configtool
```

で起動して右側の Available Input Method から SKK を探して左側の Current Input Method に入るようにすると
Ctrl + SPC で日本語を SKK で入力できるようになる。

さらに StickyShift を愛用しているのでそのあたりの設定もしている。設定ファイルについては先に紹介した記事の内容そのままなので割愛。後で GitHub にも設定ファイルを上げるのでここに書いてなくても困らないはずだし。

設定ファイルの準備ができたら
configtool を再度立ち上げて
Addons タブ内の Input Method ってところの SKK の設定ボタンをクリックし、
Rule を Default から先程作った StickyShift に切り替える。

このあたりの手順も先に紹介した記事にある手順そのままだけどまあ人の記事は消えることもあるのでちょっとここにも書かせてもらいました。向こうの方が画像キャプチャもあって親切なんだけどね。

まあともかくこれで日本語でもググることができるようになった。

あとはよくわからんけど [.xprofile](https://github.com/mugijiru/dotfiles/blob/ead701cd58dc2596d8fb883641a1d793ccddd3ed/.xprofile#L1-L4) に

```text
export DefaultIMModule=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

とか書いているな。

[ArchWiki の Fcitx5 の記事内にあるインプットメソッドモジュールの環境変数の設定](https://wiki.archlinux.jp/index.php/Fcitx5#.E3.82.A4.E3.83.B3.E3.83.97.E3.83.83.E3.83.88.E3.83.A1.E3.82.BD.E3.83.83.E3.83.89.E3.83.A2.E3.82.B8.E3.83.A5.E3.83.BC.E3.83.AB.E3.81.AE.E7.92.B0.E5.A2.83.E5.A4.89.E6.95.B0.E3.81.AE.E8.A8.AD.E5.AE.9A)
での記述を見てると、俺が書いてる場所や内容がちょっと違うのでなんか俺が間違えてるかも。

[ArchWiki の Fcitx の記事内にあるインプットメソッドモジュールの環境変数の設定](https://wiki.archlinux.jp/index.php/Fcitx#.E3.82.A4.E3.83.B3.E3.83.97.E3.83.83.E3.83.88.E3.83.A1.E3.82.BD.E3.83.83.E3.83.89.E3.83.A2.E3.82.B8.E3.83.A5.E3.83.BC.E3.83.AB.E3.81.AE.E7.92.B0.E5.A2.83.E5.A4.89.E6.95.B0.E3.81.AE.E8.A8.AD.E5.AE.9A)
の方の記述に似ているけど、これ `Fcitx5` じゃなくて `Fcitx` の記事だから多分古いんだよな……。


## i3wm の導入 {#i3wm-の導入}

ちゃんとしたタイル型WMを使いたい、というのが Linux にした理由の1つ。というわけで i3wm を入れることにした。正確には i3-gaps なので fork 版なのだけども。

実は昔使っていた Awesome にしようかとも思っていたのだけど最終更新が一昨年だったのであまり活発じゃないのかな〜と思って
i3 にすることにした。

i3wm を使うならコミュニティで提供されている i3 版をインスコした方が良いのではというのもあるんだけど気付くのが遅かったのと、そっちの設定でミスって起動しなくなった時に Xfce を動かすことでとりあえず乗り切る、みたいなことも考えている。

でまあそんなわけで i3 周りを入れることにしたので、以下をインストールしてある。

```text
$ sudo pacman -S i3-gaps
$ sudo pacman -S i3status
$ sudo pacman -S i3blocks
$ sudo pacman -S i3lock
$ sudo pacman -S dmenu
```


### i3-gaps {#i3-gaps}

i3 の fork 版で i3 より機能が豊富らしい。違いがどこにあるかはまだ認識してない。

Mod キーとの組み合わせで色々な操作ができたりして、操作感は Awesome と似ているかな。
Awesome を使っていたころも Windows キーを Mod キーとして使っていたので。

設定ファイルである `~/.config/i3/config` は今度 GitHub に上げておこう。

変更点は、初期設定だと `jkl;` を左下上右としていたのが気に食わなかったので
Vim と同じ hjkl にして、その影響で h が従来の目的では使えなくなったので水平分割、垂直分割をそれぞれ `:` とか `;` にしている。けど `-` `|` にした方がわかりやすいかもね

あとはファイル末尾で exec しているあたりが変更点だが、この記事の後の記述でそのあたりについては触れているのでここでは取り上げない。


### i3status {#i3status}

ステータスバーに表示されるやつで、デフォルトはこいつみたい。デフォルト設定だと文字だらけの表示で正直わかりやすくはないかなと思う。一応少しはカスタマイズを試みるつもりだが、すぐ乗り換えそうな気もしている。


### i3blocks {#i3blocks}

なんかあるから入れただけで何も設定してない。
i3status の代わりに使うと良いらしい。
i3status のデフォルト表示はあまり気に入ってないのでそのうち使うことになりそう。


### i3lock {#i3lock}

画面ロック機能を提供してくれるやつ。
xss-lock と一緒に使うものらしくて
xss-lock を入れてない状態だとエラーを吐いていたので

```text
$ sudo pacman -S xss-lock
```

としたらロックされるようになった。

ロック画面は真っ白な画面でそこで適当にキーを叩いたりすると真ん中に丸が出て来るというよくわからない感じ。

真ん中に丸が出てない状態でパスワードを入力して Enter するとロックが解除されるっぽい。が、わかりづらいのもあって今は指紋認証でロック解除できるようにしてある。その詳細は後程。


### dmenu {#dmenu}

デフォルト設定だと Mod + d で起動するランチャ。画面上部のバーに横並びで候補が表示されてそこから絞り込むことができるがあまり見易くはない。

あとは解像度を高めにしているから文字が小さくて困ってるけど
<https://wiki.archlinux.jp/index.php/Dmenu#.E3.83.95.E3.82.A9.E3.83.B3.E3.83.88>
を見るとそこは変更できそう。

でも [rofi](https://github.com/davatorium/rofi) とかに乗り換える方が良いかもしれない。


### 気になるけどやってないこと {#気になるけどやってないこと}

Alt + Tab での切替を Mac でも Windows でもやってるのでどうしても手癖でそれをやってしまう。

というわけでそれを実現できそうな以下の記事が気になる。
<https://scrapbox.io/tamago324vim/i3wm%5F%E3%81%A7%5FAlt+Tab%5F%E3%81%A7%E3%82%A6%E3%82%A3%E3%83%B3%E3%83%89%E3%82%A6%E3%82%92%E5%88%87%E3%82%8A%E6%9B%BF%E3%81%88%E3%82%8B>

が、優先度は低いかなと思って放置中。


## キーバインドの調整 {#キーバインドの調整}

とりあえず一旦は CapsLock が a の左隣にあるのが使いにくいしまあそもそもそんなに使わない機能なので

```text
$ setxkbmap -option "ctrl:nocaps"
```

で CapsLock を Ctrl に置き換えている。

そして [それを永続化するために .xprofile に書いてる](https://github.com/mugijiru/dotfiles/blob/ead701cd58dc2596d8fb883641a1d793ccddd3ed/.xprofile#L6-L7) ので起動直後からもう CapsLock は Ctrl になってる。とりあえず最低限これがないとつらい。


### やれてないこと {#やれてないこと}

本当にやりたいのは

1.  ESC をバッククォートにして、 Shift 押しながらだとチルダが入力される
2.  CapsLock を Ctrl にする
3.  左 Ctrl を ESC にする

だったりする。

これは今愛用しているキーボード [BAROCCO MD650L](https://archisite.co.jp/products/mistel/md650l-barocco/) のキーの都合で通常のキーボードだとファンクションキーが並ぶ行がまるっとなくてチルダがある部分、つまり1の左隣が Esc になっているのが理由で
Windows, Mac だとそれに合わせてキー配置を上述のようにしているので
Linux でもそういう配置にしたいなという願望。

ESC が左 Ctrl の位置だと押したい時にあまりホームポジションから離れずに左手親指で押せるのも便利。


## キーリピートの調整 {#キーリピートの調整}

デフォルトだとキーを押しっぱなしにした時にリピート入力が始まるのとそのリピート速度が遅くてイラッとしたので

```text
$ xset r rate 200 25
```

としている

r は repeat, rate は比率設定で
200 は delay つまりリピート入力が始まるのがどのぐらい押してからかで最後の 25 は1秒間に何回入力されるかの設定。

なので、キーを押して 0.2 秒経過したら毎秒25回の速度でそのキーが連続入力されるってわけ。

正直毎秒25回は多過ぎるかもとかは思ってるがまあ使ってる間に調整していけばいいかなと思っている。

これもまた [永続化のために .xprofile に書いている](https://github.com/mugijiru/dotfiles/blob/ead701cd58dc2596d8fb883641a1d793ccddd3ed/.xprofile#L9-L10)


## タッチパッドでのカーソル移動の調整 {#タッチパッドでのカーソル移動の調整}

デフォルトだとどうもカーソルの移動が遅くて、触りたいところに移動するのが大変だったのでこれも速度を上げている。

```text
$ xinput set-prop <device_id> <prop_id> 0.5
```

的な感じなのだが永続化はしてなさそう……。あとでやらなきゃ。

ちなみに今は Accel Speed は 0.7 にしている。細かい所にカーソル合わせるのが大変なので難しい。


## 解像度の固定 {#解像度の固定}

ひとまずマシン単体で使う場合に最適化しつつ最低限、いつも使う外部ディスプレイに繋いでる場合にも配置だけはいい感じになるように
[.xprofile に以下のように書いている](https://github.com/mugijiru/dotfiles/blob/ead701cd58dc2596d8fb883641a1d793ccddd3ed/.xprofile#L12-L13)

```text
$ xrandr --output eDP-1 --mode 2880x1620 --right-of DP-2
```

eDP-1 がノートPC自体のディスプレイなのでそいつの解像度を指定しつつ
DP-2 という外部ディスプレイの右側に配置されるように指定している。

DP-2 なのは単にコネクタ的にたまたまそうなってるだけの可能性も高いが。

そうそう。
P14s Gen 2 は USB Type-C のポートが複数あってそこに HDMI ポートもある USB Type-C の USB ハブを繋いでも使えるので仕事用の Macbook Pro で使ってるハブがそのまま使えて便利。


### autorandor {#autorandor}

[autorandr](https://github.com/phillipberndt/autorandr) というのもあってなんか解像度設定を保存したりできるっぽいやつがある。

<https://tech.buty4649.net/entry/2018/12/19/224128> の記事を見るとまだつらそうだけどちゃんと試してみる価値はありそう。


## 指紋認証でログイン、ロック解除、sudo に対応できるようにした {#指紋認証でログイン-ロック解除-sudo-に対応できるようにした}

i3lock でのロック解除画面は「本当にちゃんとキー入力できてるんだっけ？」って不安になったりするし、セットアップ中は sudo を多用するけど、その度にパスワード入れるのもだるいし、っていうかパスワード入力なんて今時じゃないな〜ってことで指紋認証でそのあたりができるようにした。

P14s Gen 2 で搭載されている指紋認証機器は fprintd のサポートに含まれているのでまあ普通にやればできたという感じ。

普通にというのは Arch Wiki に書かれてるのを見ながら設定したらできた。
<https://wiki.archlinux.jp/index.php/Fprint>


### 指紋認証機器が使えるかの確認 {#指紋認証機器が使えるかの確認}

まず

```text
$ lsusb
```

で指紋認証のやつを探して、
<https://fprint.freedesktop.org/supported-devices.html>
の中の USB ID と一致する機器があるか調べたら運良く含まれていて
Synaptics Sensors という Driver が使われるようだった


### fprintd のインストール {#fprintd-のインストール}

というわけでほぼ間違いなく使えるなということで

```text
$ sudo pacman -S fprintd
```

ここで `fprintd` ではなくグループとして定義されている `fprint` を入れようとすると一瞬入りそうな気配になるんだけどこのグループには互いに競合するパッケージが含まれてるから無理やでみたいになるので注意。


### 認証エージェントを動かす。 {#認証エージェントを動かす}

fprintd が入れられたので「じゃあ指紋登録しましょっか」ってことで

```text
$ fprintd-enroll
```

するとなんか知らんが

```text
EnrollStart failed: GDBus.Error:net.reactivated.Fprint.Error.PermissionDenied: Not Authorized: net.reactivated.fprint.device.enroll
```

と怒られる。

調べてるとなんか認証エージェントを動かす必要があるとか書いていた認証エージェントとはなんぞやとなりつつも
<https://wiki.archlinux.jp/index.php/Polkit#.E8.AA.8D.E8.A8.BC.E3.82.A8.E3.83.BC.E3.82.B8.E3.82.A7.E3.83.B3.E3.83.88>
とかを見て「ははーん polkit とかいうのが名前に入ってるやつが必要なんだな」と察して

```text
$ pacman -Ss polkit
```

した時に polkit-gnome がインストール済になっていた。ついでにいうと polkit-qt5 もインストール済だったけど、とりあえず gnome の方でいいやということでそれについてくる agent を自動起動するように i3 の config で

```text
exec --no-startup-id /usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &
```

とか書いた。


### 指紋登録 {#指紋登録}

認証エージェントも動いたので再度 `fprintd-enroll` をすると

```text
Using device /net/reactivated/Fprint/Device/0
Enrolling right-index-finger finger.
```

と Terminal に表示された上でポップアッププロンプトが出て来てパスワード入力が求められるので素直にパスワードを入れると。

すると今度はポップアッププロンプトは消えて

```text
Enroll result: enroll-stage-passed
```

とか表示されて、何やら待ってる雰囲気になるので指紋センサーの上に指を乗せたら、同じ行がまた表示されたのでそんじゃあまた指を乗せてってのを数度繰り返すと

```text
Enroll result: enroll-completed
```

と表示されて、指紋登録が完了する。


### 指紋認証の動作確認 {#指紋認証の動作確認}

指紋の登録は完了したので fprint-verify で登録したやつが使えるよねって確認を取る。スキャンした指を乗せるとちゃんと認証できたみたいな雰囲気になる。

```text
$ fprintd-verify mugijiru
Using device /net/reactivated/Fprint/Device/0
Listing enrolled fingers:
- #0: right-index-finger
Verify result: verify-match (done)
```


### pam.d の設定変更 {#pam-dot-d-の設定変更}

その後は [Arch Wiki のログイン設定の説明](https://wiki.archlinux.jp/index.php/Fprint#.E3.83.AD.E3.82.B0.E3.82.A4.E3.83.B3.E8.A8.AD.E5.AE.9A) に従い
`/etc/pam.d/system-local-login` の auth とか書かれている行の上に

```text
auth      sufficient pam_fprintd.so
```

を追加したり同じく `/etc/pam.d/` にある i3lock, lightdm, sudo の各ファイルに同じものを仕込んだ。

以上で sudo の時もロック画面の解除の時もログインする時も指紋センサーに指を置けば OK となった。便利。

ログイン画面は認証 OK になったらパスワード入力欄が非表示になるので、その状態で Enter を叩くか「ログイン」ボタンをクリックする必要があるけどね。まあ起動する WM を選択可能だからそういう手順が必要なんだろう。


## ssh 鍵の生成及び GitHub への登録 {#ssh-鍵の生成及び-github-への登録}

各設定ファイルを GitHub に登録しているのでそこにアクセスしたいよねということで
Manjaro Linux で鍵を生成して GitHub に登録しておいた。

ここらあたりはまあわざわざ細かく書かなくても、再セットアップの時にそんなに僕は困らないだろうなということで、記述はこんなもんにしておく。書いておいたら忘れないよね、ぐらいな感覚。

ファイル名を指定して鍵を作ったのでそれが次の hub コマンド周りに影響がある気はする。


## hub コマンドの設定 {#hub-コマンドの設定}

これも pacman でインスコできたのでそれを使ってる。今時は gh コマンドって気もするが hub コマンドの方が慣れていて……すみません。

で、 hub コマンドを入れて適当なところで `hub clone dotfiles` とかするとユーザー名とパスワードを聞かれて、それを入力すると自動でパーソナルアクセストークンを生成してきてそれを使うように設定されるはずなんだけど、なぜか Not Found しか言わなくて困った。

最終的には PAT を自分で GitHub に作ってさらに自分で `~/.config/hub` という設定ファイルを生成して

```text
---
github.com:
- protocol: https
  user: mugijiru
  oauth_token: MY_PERSONAL_ACCESS_TOKEN
```

みたいな感じで MY\_PERSONAL\_ACCESS\_TOKEN のところに先程生成した PAT を突っ込んでおいたら
`hub clone` が成功するようになった。

というわけで設定ファイルを拾ってきたりできるようになってハッピーになった。


## ssh-agent の自動起動 {#ssh-agent-の自動起動}

設定ファイルを GitHub に上げたりできるようになったのはいいんだけど鍵を作る時にパスフレーズも入れたので
push したり pull したりする度にそれを入力するのがだるいんですわ。

ってことで ssh-agent を使うことにした。

で、どうやら systemd さんに `--user` オプションを渡すことでユーザー権限でそういうものを動かすことができるらしい。便利な世の中じゃ。

というわけで
<https://wiki.archlinux.jp/index.php/SSH%5F%E9%8D%B5#systemd%5F.E3.83.A6.E3.83.BC.E3.82.B6.E3.83.BC.E3.81.A7%5Fssh-agent%5F.E3.82.92.E8.B5.B7.E5.8B.95>
に従い `~/.config/systemd/user/ssh-agent.service` に

```text
[Unit]
Description=SSH key agent

[Service]
Type=forking
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
ExecStart=/usr/bin/ssh-agent -a $SSH_AUTH_SOCK

[Install]
WantedBy=default.target
```

という内容のファイルを置いて

```text
$ systemctl --user enable ssh-agent
```

とかすることでログイン時に ssh-agent が動くようにした。

さらに
<https://github.com/mugijiru/dotfiles/blob/ead701cd58dc2596d8fb883641a1d793ccddd3ed/.zshrc#L157-L160>
に書いてるように .zshrc に末尾に

```text
# use ssh agent with systemd
if [ -e "$XDG_RUNTIME_DIR/ssh-agent.socket" ]; then
  export SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-agent.socket"
fi
```

と書くことで自動起動した ssh-agent に繋がるようになった。
<https://github.com/mugijiru/dotfiles/pull/11>

その状態で

```text
$ ssh-add hoge_key
```

みたいにすることで鍵を登録できるのでそのセッションでは ssh 鍵のパスフレーズを入力しないで済むようになって便利になった。


## zsh の設定 {#zsh-の設定}

mugijiru/dotfiles を clone してきてあるので

```text
$ ln -s /path/to/dotfiles/.zshrc ~/.zshrc
```

するだけで OK


## fzf の設定 {#fzf-の設定}

最近 fzf を使いだすようになったので
Manjaro でもそれを使いたいということで

```text
$ sudo pacman -S fzf
```

でインストール。

さらに dotfiles に `.fzf.zsh` も置いてるので

```text
$ ln -s /path/to/dotfiles/.fzf.zsh ~/.fzf.zsh
```

したら OK

まあ本当はセットアップの時はその記述がちょっと環境と合ってなくて
`/usr/share` にファイルがある場合はそれを読むように変更している。
<https://github.com/mugijiru/dotfiles/pull/8>

まあもうこの調整を push してあるので次回はハマらないはずだ。


## その他 {#その他}

-   w3m はあると便利なのでとりあえず入れている
-   tmux も入れておいた
    -   tmuxinator は入れてないし tmux の設定ファイルも反映してない


## 対応できてないこと {#対応できてないこと}

と、それなりに時間をかけていくつか設定してきたけどまだまだやれてないことはある。

以下はぱっと思い付く、やりたいけどやれてないこと。

-   ログイン画面の解像度変更はうまくいってない
    -   解像度が高過ぎて文字が米粒みたいでちょっと困ってる。
-   音が出ないっぽい
    -   まあ音なんかそんなに必要じゃないので後回しにしている。
    -   Xfce に戻したら鳴りそうな気もしている
-   Emacs 周りの調整
    -   大変そうなので後回し中
-   起動する Terminal 差し替え
    -   今はまだ xfce4-terminal を動かしているが urxvt にした方が軽くて良さそうだなと思っている

ここにリストアップした以外にも文章中にもそういうのが書かれてるしぱっとは思い付いてない何かもありそうなのでまだまだ設定には時間がかかりそう。楽しいねえ。
