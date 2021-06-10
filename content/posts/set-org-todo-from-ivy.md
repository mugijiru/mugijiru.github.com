+++
title = "org-todo を ivy で設定できるようにした"
date = 2020-05-31T01:32:00+09:00
categories = ["Emacs"]
draft = false
+++

posframe っていいですよね。いきなりタイトルと関係なさそうな単語出したけど。まあ ivy-posframe を使ってるので ivy を使うと posframe が使えて便利って話なんだけど。

ところで話は若干変わって、
org-todo って実行するとウインドウが分割されてバッファが表示されてそこから選ぶ形になるじゃないですか。もしかしたら設定がちゃんとしてたりしたらならないのかもしれませんけど、とりあえず私の環境だとなるんですよ。

で、それだと何が問題かというとウインドウ分割される時に元々見ていたバッファがガチャガチャと移動しちゃってつらいんですよ。
posframe を使えるとそれが起きなくて便利なんですよ。

というわけで、org-todo でキーワード選ぶ時にも
posframe が使えるといいなって思ったんですよ。

で、色々調べた結果、自分にはそういうのを提供してくれる設定とか拡張とか見つけられなかったんですよ。

じゃあ作るしかないじゃん?
というわけで、そういう関数作った

```emacs-lisp
(defun my/org-todo ()
  (interactive)
  (ivy-read "Org todo: "
            org-todo-keywords-for-agenda
            :require-match t
            :sort nil
            :action (lambda (keyword)
                      (org-todo keyword))))
```

org-todo の代わりにこの関数を呼ぶと
ivy で TODO のキーワードが設定できる。
ivy は ivy-posframe を使ってるから、画面がガチャガチャ動かなくなる。便利。

この変更に関する Emacs の設定ファイルへの Pull request は
<https://github.com/mugijiru/.emacs.d/pull/74>
に置いてるので興味があれば見てもらえると。

ところで ivy でこういう選択するインターフェース書いたの初めて。とりあえず書いてみたらできたので、また別のやつも ivy を使って書いてみたい。
