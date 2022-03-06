+++
title = "el-get-lock の情報を使ってパッケージの更新確認ができるようにした"
date = 2022-03-06T23:36:00+09:00
categories = ["Emacs"]
draft = false
+++

変なこだわりがあったり時代の波に乗れてなかったりで
use-package を使わずに el-get を使い続けている麦汁です。

このところは [el-get-lock](https://github.com/tarao/el-get-lock) で今入れているバージョンがわかるようにしつつ定期的に el-get-update-all 的なことをしていました。そこもまあ非同期でまとめて更新処理がかかると変なことになるので適当なスクリプト組んでたけども。

で、なんでそんなことをしているかというと今入れているバージョンとアップデートしたバージョンの差分をチェックしたいから。

まあ Emacs 界隈では基本的にそんな差分チェックとかしてないで入れてしまう世界っぽいのですが、個人的な妙なこだわりで差分チェックをしたくって、この運用を続けていた。

なのだけどまとめて更新すると
<https://github.com/mugijiru/.emacs.d/pull/396/files>
こんな感じになって差分を調べるのがマジ大変。そしてちょっとサボるとめっちゃ溜まるのでめっちゃ大変。

というわけでそれがしんどくなったので
el-get-lock の lock ファイルの情報から
[更新があるパッケージの情報がわかる更新チェックスクリプト](https://github.com/mugijiru/.emacs.d/blob/a447ee75a0159af6361fe654462626e08e19c679/el-get-lock-update-check.el)を書いてみた。

[自前の el-get の初期化ファイル](https://github.com/mugijiru/.emacs.d/blob/a447ee75a0159af6361fe654462626e08e19c679/el-get.lock) に依存しているので動かすにはこちらも必要だし当然自分の el-get.lock ファイルも必要なのだけど自分用に書いたスクリプトなのでそのあたりはそんなに気にしてない。


## 使い方 {#使い方}


### 更新のあるパッケージの数が知りたい場合 {#更新のあるパッケージの数が知りたい場合}

```text
$ emacs --batch -Q -l /home/mugijiru/.emacs.d//el-get-lock-update-check.el --eval=(el-get-lock-update-check-execute t)
```

と叩けばその数を出力してくれる。
`el-get-lock-update-check-execute` の第一引数が t だと更新が必要なパッケージの数だけ表示するようになっている。

まあ作りが雑なので標準エラー出力にもごちゃごちゃ吐かれるから

```text
$ emacs --batch -Q -l /home/mugijiru/.emacs.d//el-get-lock-update-check.el --eval=(el-get-lock-update-check-execute t) 2> /dev/null
```

としてそのあたりの情報は捨てた方がいい


### 更新のあるパッケージがどれなのか知りたい場合 {#更新のあるパッケージがどれなのか知りたい場合}

```text
$ emacs --batch -Q -l /home/mugijiru/.emacs.d//el-get-lock-update-check.el --eval=(el-get-lock-update-check-execute)
```

と叩けばどのパッケージが更新可能なのか吐いてくれる。どのパッケージが更新可能か分かれば後は `(el-get-update hoge-package)` とか実行してあげれば良い。
<https://github.com/mugijiru/.emacs.d/pull/467> などはそうやって更新して
el-get.lock の checksum の差分から GitHub の compare を見たりしている。

このパターンでは他にもいくつかの情報を吐くけど後で説明する。


## 仕組み {#仕組み}

el-get.lock では例えば Git 管理のパッケージの場合だと
checkout した revision の hash 値を el-get.lock に記録するので
remote の最新コミットと比較して違う値だったら孤雲があると判定するだけ


## 制限 {#制限}


### Git/GitHub 以外のパッケージの更新チェックはできない {#git-github-以外のパッケージの更新チェックはできない}

EmacsWiki とか CSV とかのパッケージの更新チェックはできません。
el-get.lock でもそのあたりは checksum は記録するけどロックはできないのでしてなかったりするしまあ今の世の中大体は GitHub かそうじゃなくても Git で管理されてるのでこのスクリプトでは一旦そこだけサポートするようにした。自分用だしテキトーで良い。

ちなみに EmacsWiki に入ってるようなやつは大体
<https://github.com/emacsmirror> とか <https://github.com/emacsorphanage?type=source> あたりにあるのでそっちを使うように recipe を書けば解決する。
<https://github.com/mugijiru/.emacs.d/pull/476> みたいな感じで。


### (el-get-bundle XXX/YYY) 形式で入れているパッケージは更新チェックできない {#el-get-bundle-xxx-yyy--形式で入れているパッケージは更新チェックできない}

レシピファイルの情報をチェックするので
`(el-get-bundle XXX/YYY)` で直接 GitHub のリポジトリを指定していてレシピファイルが存在しない場合はこれもまた更新チェックができません。

まあレシピファイルさえ書けば解決するのでそれでいいかってなってる


### main がデフォルトブランチの場合はレシピファイルでそれを指定する必要がある {#main-がデフォルトブランチの場合はレシピファイルでそれを指定する必要がある}

どちらかというと el-get-lock 側の制限な気もするけど
el-get-lock では branch の指定がなければ
master ブランチの情報で固定するようになっているのでその場合もまたレシピファイルを追加して回避した方がいい。
ox-hugo が master から main に切り替えられてるっぽいのだけど手元は古い状態で止まってしまってた……。

そのため main ブランチがあるかどうかを調べられる関数も仕込んでいる。
`el-get-lock-update-check-use-main-p` という関数がその役割で

```emacs-lisp
(dolist (version (cdr el-get-lock-package-versions))
  (let ((package (replace-regexp-in-string "\\\\\\\." "\\\." (symbol-name (car version)))))
    (el-get-lock-update-check-use-main-p package)))
```

みたいに叩くとインストール済のやつでかつ main ブランチを使ってるやつがわかるようになっている。雑な作りだけどね。


## i3blocks との連携 {#i3blocks-との連携}

i3blocks の設定ファイルに

```text
[emacs-update]
command=emacs --batch -Q -l ~/.emacs.d//el-get-lock-update-check.el --eval='(el-get-lock-update-check-execute t)' 2> /dev/null
label=Emacs:
interval=1800
```

とか書いておけば30分毎に更新チェックのスクリプトが実行されるので更新情報がすぐ分かって便利かもしれない。


## 更新があるパッケージ以外の情報 {#更新があるパッケージ以外の情報}

更新のあるパッケージのリスト表示の時は他の情報も出力すると上に書いていたけどじゃあ何を出すかというと「EmacsWiki から入れているパッケージはこいつらだよ〜」とか、「hash 値が取れないパッケージはこれこれだよ〜」とか表示するようにしている。更新チェックができないやつも分かるようにしたかったので。


## 最後に {#最後に}

とりあえず自分のやりたい更新差分チェックが
1パッケージ毎にチェックできるようになったので確認が楽になりました。

今回のスクリプトは[2週間ぐらい前から作っていて](https://github.com/mugijiru/.emacs.d/pull/405)
その時点のやつでも最低限のチェックはできたので
2週間前から1つずつ更新しつつスクリプトを直したりしました。

けど更新をサボっていたから長かった……。今日でやっと更新も全部対応できた……。

今後は更新をサボらないようにしたいです……!
まあ今1つ更新があるのは無視して明日対応するけどな。
