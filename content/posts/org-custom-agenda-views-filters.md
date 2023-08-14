+++
title = "org-mode の Custom Agenda Views の抽出条件を少し調整した"
date = 2023-08-15T23:03:00+09:00
categories = ["Emacs", "org"]
draft = false
+++

しばらく使っていて、ここなんとかしたいな〜と思っていたところの一部をなんとかしたので記事に残しておく。


## Heading Level でのフィルタリング {#heading-level-でのフィルタリング}

TODO タスクの中に更に TODO でサブタスクを生やしていると
agenda 表示をした時にそのサブタスクも全部表示されてうんざりしていたのが最近ようやく解決できた。

```emacs-lisp
(tags-todo "LEVEL=2" ...)
```

みたいに `LEVEL=?` みたいに記述したらレベル2の TODO だけが抽出できる。

具体的には
<https://github.com/mugijiru/.emacs.d/pull/2618>
とかでやっている。

`tags-todo` だけじゃなく `tags` でも絞り込みはできる。
<https://github.com/mugijiru/.emacs.d/pull/2657> でやってるから間違いないはず。


## 優先度でのフィルタリング {#優先度でのフィルタリング}

org-mode の TODO は優先度が付与できる。
<https://orgmode.org/manual/Priorities.html>

なので、優先的にやりたいタスクだけとりあえず抽出したかった。

まあ上の Heading Level でのフィルタリングと大体一緒で

```emacs-lisp
(tags-todo "PRIORITY=\"A\"" ...)
```

とか指定したら OK

具体的には
<https://github.com/mugijiru/.emacs.d/pull/2631>
でその設定を突っ込んでみている


## 直近1週間以内の TODO の抽出 {#直近1週間以内の-todo-の抽出}

これは [org-super-agenda](https://github.com/alphapapa/org-super-agenda) を使って対応している。

各タスクに schedule を設定しておいて
`org-super-agenda-groups` で「7日後より前」「今日より後」の2つの条件を and で繋ぐと良い。

以下のコードは自分の設定ファイルを見ながら書いたやつだけど、多分次の感じで動く

```emacs-lisp
(all-todo ""
          (org-super-agenda-groups
           '((:name
              "今後1週間の作業"
              :and
              (:scheduled (before
                           ,(format-time-string
                             "%Y-%m-%d"
                             (time-add (current-time)
                                       (days-to-time 7))))
                          :scheduled (after
                                      ,(format-time-string
                                        "%Y-%m-%d"
                                        (current-time))))))))
```

具体的に動くようにしているコードは以下
<https://github.com/mugijiru/.emacs.d/pull/2617>
