+++
title = "プロジェクト固有の Hydra を使う"
date = 2024-06-16T11:55:00+09:00
categories = ["Emacs"]
draft = false
+++

普段 Rails のプロジェクトを触っているのでそれ用に projectile-rails のコマンドを叩けるようにするために
[projectile-rails 用の Hydra を定義](https://mugijiru.github.io/.emacs.d/programming/rails/#%E3%82%AD%E3%83%BC%E3%83%90%E3%82%A4%E3%83%B3%E3%83%89)している

のですが Rails 以外のプロジェクトもあるし
Rails のプロジェクトでもここで定義しているのとは違う形のプロジェクトを触ることもあるのでプロジェクト毎に固有の Hydra が欲しいなって思ってちょっと対応してみた。


## プロジェクトの .dir-locals.el {#プロジェクトの-dot-dir-locals-dot-el}

Emacs ではフォルダ毎の固有設定を `.dir-locals.el` に設定できる。その仕組みを使うことでプロジェクト固有の Hydra を定義することができる。

具体的な設定方法を書くと、前提として `foo-project` という Ruby のプロジェクトが
`/path/to/foo-project` という PATH に入っているとする。

そこで `/path/to/foo-project/.dir-locals.el` に以下のような感じにそのプロジェクトの記述を入れておく。

※ 私は [pretty-hydra](https://github.com/jerrypnz/major-mode-hydra.el#pretty-hydra) が好きなのでそれを使って定義しているけど、それを使っていない場合は `defhydra` を使って同様に定義できる

```emacs-lisp
((nil . ((eval . (pretty-hydra-define foo-project-hydra (:separator "-" :color blue :foreign-keys warn :title "foo-project" :quit-key "q")
                   ("Find"
                    (("l" (my/projectile-find-file-in-dir "lib") "lib")
                     ("s" (my/projectile-find-file-in-dir "spec/lib") "spec"))
                    "Jump"
                    (("g" my/projectile-goto-gemfile "Gemfile")
                     ("d" my/projectile-goto-dockerfile "Dockerfile")
                     ("c" my/projectile-goto-circleci-config "CircleCI")
                     ("r" my/projectile-goto-rubocop-config "Rubocop"))))))))
```

こうするとプロジェクト固有の Hydra を用意できる。プロジェクト毎に同様に定義していけば、それぞれのプロジェクト毎の特別な事情を入れ込んだ Hydra を用意できるので便利。


## プロジェクト固有の Hydra を呼び出すコマンドの定義 {#プロジェクト固有の-hydra-を呼び出すコマンドの定義}

しかしこれだけだと手で `M-x foo-project-hydra/body` を叩くハメになって面倒。定義した Hydra の名前も覚えておく必要があるし。

というわけでプロジェクト固有の Hydra を呼び出すためのコマンドを用意する。

```emacs-lisp
(defun my/project-hydra ()
  "Call the project specific hydra."
  (interactive)
  (let* ((project-path (directory-file-name (projectile-acquire-root)))
         (project-dir-name (file-name-base project-path))
         (hydra-body (intern (concat project-dir-name "-hydra/body"))))
    (cond
     ((fboundp hydra-body)
      (funcall hydra-body))
     (t
      (user-error "project hydra is not defined on %s." project-dir-name)))))
```

ここではそのプロジェクトのフォルダ名から Hydra を呼び出すコマンド名を組み立ててそのコマンドが定義されていたら実行する、という挙動をする。

なので `foo-project` のファイルを編集している時にこれを叩けば `foo-project-hydra/body` が呼び出されるし
`bar-project` のファイルを編集している時にこれを叩けば `bar-project-hydra/body` が呼び出される。どのプロジェクトにいる時も `(my/project-hydra)` を実行すればそのプロジェクト用の Hydra が動くようになる。


## 他にやりたいこと {#他にやりたいこと}

今の状態だといくつか問題がある。

プロジェクト固有の Hydra にはそのプロジェクトに必要なものを全て入れておきたいが
Ruby のプロジェクトなら共通する項目もあるので、それらも各プロジェクト固有の Hydra に定義するのは面倒。

かといって共通で使う Hydra とプロジェクト固有の Hydra とで分離されているのも面倒。

というわけで、共通で使う Hydra をベースに、それを継承するとかマージするとかで新しい Hydra を定義できるようにしたい。

そうするとプロジェクト固有の Hydra を定義する手間が省けるし
1つのプロジェクトを使うのに複数の Hydra を使うという非効率も解消できる。

ま、このあたりは宿題ということで。
