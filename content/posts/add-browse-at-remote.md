+++
title = "browse-at-remote を導入した"
date = 2021-01-23T23:18:00+09:00
categories = ["Emacs"]
draft = false
+++

仕事中、プルリクのレビューをする際に、「ここのコードがこんな感じになってるから〜」みたいな感じで既存コードへのリンクを張ることがちょくちょくある。

そういうことをする時、まずそもそも差し示したいコードを確認するんだけどその時は Emacs の中で探す方が早い。で、探して確認するまではいいんだけど、そこから GitHub 上のコードへのリンクを取得しようとするとちょっと面倒。

これまでは、GitHub のリポジトリのトップからディレクトリを辿って行って当該コードを再度探していました。めんどくさいねっ。

というわけで解決する手段を探していて最近導入したのが [browse-at-remote](https://github.com/rmuslimov/browse-at-remote) というやつ。

こいつを入れてる状態で、GitHub のリポジトリに突っ込んであるコードの上で
`M-x browse-at-remote` を実行すると
GitHub でのリポジトリでのコードの位置でブラウザを開いてくれる。

コードの上でと書いたけど、リージョンを選択していればその範囲が選択された状態で実行すると選択した行がハイライトされた状態で開いて便利。

その状態から GitHub 上で `Copy permalink` をしておいて
PR のコメントにコピーしたリンクを貼り付けるとコードも表示されて便利。

で、結構よく使うコマンドとなったので
Hydra から即呼び出せるようにしてある。

Global に使うコマンドを突っ込んでる Hydra は key-chord で `jk` を叩くと呼べるようにしていてその中で `B` を叩けば browse-at-point が呼ばれるようにしてある。

というわけで、導入と Hydra の設定を追加しているプルリクが以下になります。
<https://github.com/mugijiru/.emacs.d/pull/205>

という使い方をしているけど、実は似た機能を提供している [git-link](https://github.com/sshaw/git-link) で `git-link-use-commit` のフラグを立てておいて他にもいくつか設定を入れたりしたらもっといい感じのことができるのかもしれない。今度試すか……。