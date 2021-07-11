+++
title = "key-chord を使って Sticky Shift を一部実現した"
date = 2021-07-11T13:42:00+09:00
categories = ["Emacs"]
draft = false
+++

Emacs 使っていると左手小指を酷使しますよね。

まあそれはみんな分かり切ってるので、それぞれが色々な工夫で、左手小指を必要以上に使わないような対応をされていると思います。

私もそんな中の一人で、最近もうシフトキーを押したくないなあという気持ちになって来ました。

というわけで key-chord を使って Sticky Shift を一部実現して少しだけシフトキーを押さなくても済むような状態を作ってみました。

とりあえず以下の文章を読むのが面倒だったら
<https://github.com/mugijiru/.emacs.d/pull/324/files#diff-9fab5d4bac36f504628abcc3cea5b2a1092d96f7bd2286944d5b822c6821bb3a>
にコードがあるのでそっちを見てもらえればと。


## 実現したこと {#実現したこと}


### セミコロン + アルファベットで対応する大文字を出力 {#セミコロン-plus-アルファベットで対応する大文字を出力}

プログラムを書いているとやれ camelCase だの PascalCase だのでちょくちょくと大文字を入力する機会多いですよね。

というわけで、例えば `;a` と素早く入力したら `A` が出力されるようにしてみたというのが以下のコードになります。

```emacs-lisp
(mapc (lambda (key)
        (key-chord-define-global (concat ";" (char-to-string key)) (char-to-string (- key 32))))
      (number-sequence ?a ?z))
```

`number=sequence` で a から z までのシーケンスを作成して
`;` との組み合わせを key-chord に食わせて対応するアルファベットの大文字を渡しているというシンプルな構成。

`key-chord-define-global` などは第二引数に文字列を与えるとそれをそのまま `define-key` の第三引数に渡すのであたかもそれが入力されたかのような振舞をするのでそれを利用している。

といいつつ Hydra とか magit の上でそれをやってもうまくいかないのでキーボード入力とは少し違う様子。


### セミコロン2回でシフトキーが押された状態にする {#セミコロン2回でシフトキーが押された状態にする}

上に書いたように Hydra とか magit とかでいい感じに動かすためにはまだ足りてない。

が、それをいい感じに解決する方法をまだ知らないので以下の実装により `;;` と入力することで、シフトキーが押されてる状態を実現している。

```emacs-lisp
(key-chord-define-global ";;"
                         'event-apply-shift-modifier)

(key-chord-define key-translation-map
                  ";;"
                  'event-apply-shift-modifier)
```

多分本当はこんな感じで2回似たような事を書かなくてもいいやり方があるんだろうけど今のところこれで動くから良しとしている。

ここで使っている `event-apply-shift-modifier` という関数がシフトを入力している状態にしてくれるやつで、類似品に `event-apply-control-modifier` など、修飾キー系は全部用意されている。

これらの `event-apply-*-modifier` 系の関数は `C-x @ S` などと叩くと呼ばれたりするので
iTerm とかの中で Emacs を動かしている勢が無理やり修飾キー入力を実現したりするために何やら設定するのにも使っているが今回のように sticky 的な機能を実現するのに便利なやつだったりします。
[sticky-control](https://github.com/martialboniou/emacs-revival/blob/master/sticky-control.el) でも使われているしね。


## 今後対応していきたいこと {#今後対応していきたいこと}


### セミコロン + 数字キーや記号キーの対応 {#セミコロン-plus-数字キーや記号キーの対応}

今回はとりあえずアルファベットだけ Sticky Shift に対応しているがやはり記号系のキーとか数字系のキーも対応できてないと
Shift を押す機会を減らせないので今後はそのあたりも対応していきたい。

なお rubikitch 氏の [sticky.el](https://www.emacswiki.org/emacs/sticky.el) では実現できていそうなので似たようなことをすれば良さそう。


### セミコロン + アルファベットでその文字を入力したことにしたい {#セミコロン-plus-アルファベットでその文字を入力したことにしたい}

今回の実装では「セミコロン+アルファベット」で \*\*大文字をバッファに出力\*\* できるように対応しているが
\*\*大文字を入力したこと\*\* にはできてない。

この2つの何が違うかというと前者しか実現できてない現状だと例えば magit を操作している時に `P` の代わりに `;p` を入力した場合には
`Quit transient!` と表示されるだけで Pull の操作ができない状態である。

`;;` で `event-apply-shift-modifier` しているので、
`;;p` とすれば Pull の操作もできるのだがやはりできる限り簡単なキー入力でやりたいことを実現できるようになりたい。
