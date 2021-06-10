+++
title = "org-todo を ivy で設定できるようにするパート2"
date = 2021-06-10T20:53:00+09:00
categories = ["Emacs"]
draft = false
+++

[org-todo を ivy で設定できるようにした]({{< relref "set-org-todo-from-ivy" >}}) という記事で
org-todo を ivy で設定できるようにしたつもりだったんですけど、ダメでした。

何がダメだったかというと
`org-todo-keywords-for-agenda` という変数を使ってるのがダメだった。

この変数、何かよくわからんタイミングで設定されたりするっぽくてほとんどの場合で空の値になっていた。

というわけで、ちょっと例の関数だと使いたい時にその値が空になっていてばかりで正直使い物にならない関数になっていた。死蔵していた。

まあ俺はそんな半端な状態で放置するような男ではない。嘘です。1年ぐらい放置していました。だけど逆にいうと1年越しでなんとか対応しました。というのが以下の PR になります。

<https://github.com/mugijiru/.emacs.d/pull/231>

PR の description にも書いている通り愚直に org-todo-keywords を加工するように変更している。

私の設定している org-todo-keywords は現在は

```emacs-lisp
((sequence "TODO" "DOING(!)" "WAIT" "|" "DONE(!)" "SOMEDAY(s)"))
```

という感じ。

これを

```emacs-lisp
(mapcar (lambda (element)
          (replace-regexp-in-string "\(.+\)" "" element))
        (--remove (string= "|" it) (cdar org-todo-keywords)))
```

のような処理で

```emacs-lisp
'("TODO" "DOING" "WAIT" "DONE" "SOMEDAY")
```

みたいな感じで、キー指定の `(s)` や、未完了or完了状態を区切る `|` とかを取り除いた文字列のリストにしている。

org-todo-keywords は本当は

```emacs-lisp
(setq org-todo-keywords
      '((sequence "TODO" "|" "DONE")
        (sequence "REPORT" "BUG" "KNOWNCAUSE" "|" "FIXED")
        (sequence "|" "CANCELED")))
```

みたいに複数のシーケンスを持つことができるけどそういう使い方はしてないので、そういうケースは無視している。

とりあえずこれでようやく ivy で org-todo を設定できるようになった。ちょっと便利になった。
