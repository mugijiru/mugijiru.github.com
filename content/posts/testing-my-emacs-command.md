+++
title = "ivy を使った自作コマンドなどをテストするようにした"
date = 2021-06-13T15:18:00+09:00
categories = ["Emacs"]
draft = false
+++

これまで Emacs Lisp のテストコードを書いてなかった。

というのも、自分は特にパッケージを作ってるわけでもなく、設定ファイルを弄ってるだけなので動かなくなっても困るのは自分だけ、という状況なのでまあテストしなくてもいいかな、みたいな。

とは思っているものの、やっぱりテストコード書いてみたいよねってことで設定ファイルに書いている自作の関数やコマンドをテストすることにした。


## 自作関数のテスト {#自作関数のテスト}

自作コマンドの内部で使ってる関数のテストを
<https://github.com/mugijiru/.emacs.d/pull/233>
で書いてみた。

なぜそんなものをテストしているかというと、リファクタリングしたかったので、ならテスト書いた方がいいよね〜みたいな。

テストコードは以下のような感じ。前提条件として `org-todo-keywords` を用意して、関数を実行した時の期待する結果と、実際の結果を
should マクロを使って合っていることを確認している。

```emacs-lisp
(ert-deftest test:my/org-todo-keyword-strings ()
  "Test of `my/org-todo-keyword-strings'."
  (setq org-todo-keywords
        '((sequence "TODO" "DOING(!)" "WAIT" "|" "DONE(!)" "SOMEDAY(s)")))
  (should (equal '("TODO" "DOING" "WAIT" "DONE" "SOMEDAY")
                 (my/org-todo-keyword-strings))))
```

ま、とても簡単な例だと思う。

ちなみにこの記事や出した PR では setq を使ってるけど、最新のコードでは let を使うようにテストを書き直してある。


## ivy を使った自作コマンドのテスト {#ivy-を使った自作コマンドのテスト}

上に示した PR では
ivy を使った処理はテストができない(しづらい)と判断しコマンド内部で使ってる関数だけテストしている。

だけどやっぱり ivy を使ってるコマンド自体もテストしてみたいよねということで以下の PR を作った

<https://github.com/mugijiru/.emacs.d/pull/235>


### with-simulated-input の導入 {#with-simulated-input-の導入}

ivy を使ってるコマンドのテストで難しそうだなと思っていたのが
ivy の操作部分のシミュレーション。

なのだけど
<https://github.com/DarwinAwardWinner/with-simulated-input>
を見つけて解決した。

こいつはユーザーの入力を文字列で表現してその通りに操作をしてくれるようなライブラリ。

第一引数に `"hello SPC world RET"` みたいに入れたら
"hello world" と入力して Enter を叩く、みたいな入力をしてくれる。わかりやすい。

入力と入力の間に wait を持たせたかったら
`wsi-simulate-idle-time` という関数でミリ秒単位で wait も入れられるっぽい。その機能はまだ使ったことないけど便利そう。


### cl-letf による関数の差し替え {#cl-letf-による関数の差し替え}

今回テスト対象にしているコードは内部で org-mode の `org-todo` 関数を呼んでいる。だけど別に `org-todo` の機能をテストしたいわけではなく
ivy を使った絞り込みで適切に値が選択されることを検証したいのである。

というわけで cl-letf を使って呼んだ先で使われている org-todo を以下のように一時的に差し替える。

```emacs-lisp
(cl-letf (((symbol-function 'org-todo)
           (lambda (keyword)
             (setq result keyword))))
  ;; ここにテストコードを書く
  )
```

ちなみにこの着想に到ったのは
<https://g000001.cddddr.org/3690354344>
の記事を読んだから。

そう。私もまた壊れた flet が欲しかったのである。

そして世の中にいい記事があってありがたいね。


### 実際のテストコード {#実際のテストコード}

以上で説明したようなやつを使って、以下のようなテストコードになった。

```emacs-lisp
(ert-deftest test:my/org-todo ()
  "Test of `my/org-todo'."
  (let ((org-todo-keywords '((sequence "TODO" "DOING(!)" "WAIT" "|" "DONE(!)" "SOMEDAY(s)")))
        (result))
    ;; org-mode を読まずに済むように org-todo を差し替えてテストしている
    (cl-letf (((symbol-function 'org-todo)
               (lambda (keyword)
                 (setq result keyword))))
      (with-simulated-input "DOI RET" (my/org-todo))
      (should (equal "DOING" result)))))
```

`my/org-todo` を実行したら org-todo のキーワードが ivy に表示されてそこから絞り込んで選択する、ということを期待したテストである。

今回は "DOI" まで入力して候補が DOING だけになりそこで Enter を押すとそれが選択されることをテストしている。

選択された結果は org-todo を差し替えた関数で
result という変数に束縛しているので、それが "DOING" という文字列と一致しているかを検証しているだけ。


## 最後に {#最後に}

ところで、
buffer の変更のテストってどうしたらいいんでしょうかね。

何か適当なバッファを用意して関数を実行した後に
`(buffer-string)` と期待値がマッチするかを確認したらいいのかな
