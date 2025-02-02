+++
title = "OfflineIMAP + mu4e の設定をしている"
draft = true
+++

<div class="ox-hugo-toc toc">

<div class="heading">&#30446;&#27425;</div>

- [やったこと概要](#やったこと概要)
- [やってないこと](#やってないこと)
- [OfflineIMAP](#offlineimap)
    - [インストール](#インストール)
    - [設定](#設定)
        - [general](#general)
        - [Main](#main)
        - [Nue](#nue)
        - [Prov](#prov)
    - [functions.py](#functions-dot-py)
    - [メール取り込み](#メール取り込み)
    - [おまけ: nametrans を設定せずに同期しちゃった場合](#おまけ-nametrans-を設定せずに同期しちゃった場合)
- [mu](#mu)
    - [インストール](#インストール)
    - [初期化](#初期化)
    - [インデックス構築](#インデックス構築)
- [mu4e](#mu4e)
    - [require](#require)
    - [ユーザー名/パスワード読み込み関数の定義](#ユーザー名-パスワード読み込み関数の定義)
    - [基本的な設定](#基本的な設定)
    - [mu4e-contexts](#mu4e-contexts)
- [その他](#その他)

</div>
<!--endtoc-->

ついったーで「久々に Emacs でメール見れるようにしたいけど Mew と Wanderlust どっちが良いんだっけ」とかついーとしていたら「OfflineIMAP + mu4e 快適」との情報を得たので、その環境を用意している。


## やったこと概要 {#やったこと概要}

-   複数アカウントのメール受信
-   Emacs でメール表示


## やってないこと {#やってないこと}

-   メールの表示調整
    -   HTML メールの表示など
-   メールの送信周りの設定
-   通知
    -   [mu4e-alert](https://github.com/iqbalansari/mu4e-alert) を入れたら良さそうだけど後回し
-   org-mode 連携
    -   なんか mu に mu4e-org とかいうのが付随しているけどこれも後回し


## OfflineIMAP {#offlineimap}

[OfflineIMAP](https://www.offlineimap.org/) はなんかローカルに IMAP でメールを同期して保存するための Python ユーティリティらしい。


### インストール {#インストール}

Manjaro Linux だと

```text
$ yay -S offlineimap
```

でインストールできた。


### 設定 {#設定}

複数のメールアドレスをチェックしたかったので最初から複数アカウント対応という若干面倒な道を歩んでいる

ざっと `~/.config/offlineimap/config` に書いている設定を抜粋しておく。


#### general {#general}

まずは全体的な設定。アカウントを現時点では3つほど同期していてそれぞれ Prov, Main, Nue という名前を付けている。

また、パスワードとかを秘匿するために便利関数を用意してそれ経由でそういう情報は取得しているのでその便利関数を格納するファイルを設定ファイルと同じ階層に `functions.py` という名前で保存している。中身については後で書くか。

```text
[general]
metadata = ~/.offlineimap
accounts = Prov, Main, Nue
pythonfile = ~/.config/offlineimap/functions.py
```


#### Main {#main}

メインで使ってる Gmail アカウントの設定。

Account の方はほぼ何も設定してないから解説は不要か。

RemoteMain はリモートつまり GMail からデータを取って来る部分の設定。

まず Gmail のアカウントなので type は当然 `Gmail` となる。そして Gmail に接続する時に CA 証明書を指定しないと接続エラーになったのでそれを設定している。

また ユーザー名やパスワードはセンシティブ情報ということで外部の暗号化されたファイルから取得するために
`remoteusereval`, `remotepasseval` を使い `functions.py` に定義した関数を呼び出すようにしている。

それから Gmail からデータを取得すると日本語のフォルダ等が Modified UTF-7 で作られてしまうので
`function.py` で UTF-8 に変換するための関数 `utf7_imap_to_utf8_by_iconv` を用意して `nametrans` で利用している。名前の通り、実装的には `iconv` を使って変換している。詳細は後述する、多分。

LocalMain はローカルの保存先フォルダの設定かな。
type は `Maildir` を指定している。
OfflineIMAP は他に `GmailMaildir` と `IMAP` をサポートしてるみたいだけど、
mu4e(mu) が期待しているのは Maildir 形式なのでこれを指定するしかない感じ。

で、出力先として `~/Maildir/main` を指定している。アカウントが1つだと `~/Maildir` の指定で良いんだけど、複数設定する前提なのでこうしている。

あとは nametrans で UTF-8 から Modified UTF-7 に変換するようにしている。
RemoteMain の時と逆変換だね。なんかドキュメントに「逆変換できるようにしておかないと困るからそうしな!」って書いてた。

```text
[Account Main]
localrepository = LocalMain
remoterepository = RemoteMain

[Repository RemoteMain]
type = Gmail
sslcacertfile = /etc/ssl/certs/ca-certificates.crt
remoteusereval = get_username("offlineimap-main")
remotepasseval = get_password("offlineimap-main")
nametrans = lambda foldername: utf7_imap_to_utf8_by_iconv(foldername)

[Repository LocalMain]
type = Maildir
localfolders = ~/Maildir/main
nametrans = lambda foldername: utf8_to_utf7_imap_by_iconv(foldername)
```


#### Nue {#nue}

こっちも Gmail のアカウント。なので設定は Main とほぼ一緒。

```text
[Account Nue]

localrepository = LocalNue
remoterepository = RemoteNue

[Repository RemoteNue]
type = Gmail
sslcacertfile = /etc/ssl/certs/ca-certificates.crt
remoteusereval = get_username("offlineimap-nue")
remotepasseval = get_password("offlineimap-nue")
nametrans = lambda foldername: utf7_imap_to_utf8_by_iconv(foldername)

[Repository LocalNue]

type = Maildir
localfolders = ~/Maildir/nue
nametrans = lambda foldername: utf8_to_utf7_imap_by_iconv(foldername)
```


#### Prov {#prov}

こっちは Provider のアカウント。なので Remote の方の type は IMAP にしている。

remotehost を指定していて nametrans をしていないこと以外は Gmail と一緒。

なんか日本語フォルダを作っていたら nametrans が必要かもしれないけどほぼ使ってないアカウントで、そんなもん用意してないので一旦このまま。

```text
[Account Prov]

localrepository = LocalProv
remoterepository = RemoteProv

[Repository RemoteProv]

type = IMAP
remotehost = mail.biglobe.ne.jp
ssl = yes
sslcacertfile = /etc/ssl/certs/ca-certificates.crt
remoteusereval = get_username("offlineimap-prov")
remotepasseval = get_password("offlineimap-prov")

[Repository LocalProv]

type = Maildir
localfolders = ~/Maildir/prov
```


### functions.py {#functions-dot-py}

<https://kenbeese.hatenablog.com/entry/20121129/1354442823> に書かれているものをベースに作っている

`get_output` はそのまんま。

`get_password_emacs` の実装はほぼ同じで emacs-client を使って Emacs からパスワードを取るようにしている。名前は `get_password` にしたり、得られた output を `decode('ascii')` したりとちょっと改変はしている。

```python
def get_password(host):
  cmd = "emacsclient --eval '(offlineimap-get-password \"%s\")'" % (host)
  return get_output(cmd).decode('ascii').strip().lstrip('"').rstrip('"')
```

そしてほぼ同じ実装で `get_username` という関数も用意した。アカウント名がメアドだったりするので、それも隠したく。

```python
def get_username(host):
  cmd = "emacsclient --eval '(offlineimap-get-username \"%s\")'" % (host)
  return get_output(cmd).decode('ascii').strip().lstrip('"').rstrip('"')
```

`utf7_imap_to_utf8_by_iconv` は IMAP で使われている Modified UTF-7 から UTF-8 に変換するための関数。手元の環境に `iconv` コマンドがあるので、それを使えばいいやって感じで外部コマンドにぶん投げている。

```python
def utf7_imap_to_utf8_by_iconv(str):
  cmd = "echo -n '%s' | iconv -f UTF-7-IMAP -t UTF-8" % (str)
  return get_output(cmd).decode('UTF-8')
```

逆変換のために `utf8_to_utf7_imap_by_iconv` というコマンドも用意している。

```python
def utf8_to_utf7_imap_by_iconv(str):
  cmd = "echo -n '%s' | iconv -f UTF-8 -t UTF-7-IMAP" % (str)
  return get_output(cmd).decode('ascii')
```


### メール取り込み {#メール取り込み}

とりあえず上の設定が済んだら

```text
$ offlineimap
```

とか叩くと同期が始まる。
`Ctrl+c` で中断してから再開もできる。

自分の場合は20年ぐらい使っている Gmail の全てを同期したら途中中断しながら断続的にやっていたとはいえ2週間かかったので注意。そういうのを避けるためには Remote の方で folderfilter をかけて必要なフォルダだけ取得すべきみたいだけど自分は一旦全部取得してみている。


### おまけ: nametrans を設定せずに同期しちゃった場合 {#おまけ-nametrans-を設定せずに同期しちゃった場合}

上の設定ではしっかり nametrans とか書いているのですが最初に同期を試した時なんかはそういうのはやっていなかったので Modified UTF-7 でフォルダが作られていた。それだとかなり使い勝手が悪いので、後から UTF-8 に変換したりした。

以下がとりあえず変換に使った bash スクリプト。指定したフォルダ直下にある Modified UTF-7 のフォルダを UTF-8 に一括変換してくれる君。まあ一部フォルダが変換されなかったけどそれは一部だったので個別に iconv 使ってごにゃごにゃした。

```bash
#!/bin/bash

cd $1
dirs=$(ls -1 . | grep '[\x00-\x7F]')

echo $dirs

for dir in $dirs
do
    echo "processing $dir..."
    utf8_dir=$(echo -n $dir | iconv -f UTF-7-IMAP)

    if [[ "$dir"  != "$utf8_dir" ]]; then
        echo "Convert ${dir} to $utf8_dir"
        mv "$dir" "$utf8_dir"
    fi
done
```

後から変換しても同期を再開したらちゃんと続きからになったので、
Modified UTF-7 で作ってしまっていても慌てないで良し。
2週間ぐらい断続的に同期していたのが無駄にならなくてまじ助かった。


## mu {#mu}

mu4e は [mu](https://github.com/djcb/mu) という Maildir 形式のメールをいい感じ使うためのツールの Emacs フロントエンドなのでまず mu の index を初期化する必要がある。


### インストール {#インストール}

Manjaro Linux では

```text
$ yay -S mu
```

でインストールできた。これに mu4e も付属してくるのでこっちを使うのが良さそう。


### 初期化 {#初期化}

```text
$ mu init --my-address='foo@example.com' --my-address='bar@example.com'
```

みたいに、利用するメアドを指定して初期化している。ここでは例として `foo@example.com` とか書いているけど各自自分のメアドを必要な分だけ指定すべし。

この時どのフォルダを使うのかも指定できるんだけど `~/Maildir` が初期値なのでそこは特に手を出さず。


### インデックス構築 {#インデックス構築}

初期化したら index 構築をするため

```text
$ mu index
```

を実行する。メールの件数が多いと少し時間がかかるけど数分程度なので `offlineimap` に比べると一瞬。


## mu4e {#mu4e}

同期ができたら Emacs から見られるように設定する
mu に付属してくるんで、インストールは不要。


### require {#require}

el-get とかで入れていたら `require` とかは自動でやってくれるけど今回は mu の付属でついてきたやつを使うので明示的に require する

```emacs-lisp
(require 'mu4e)
```

なお [mu のインストール時](#インストール)に `/usr/share/emacs/site-lisp/mu4e/` に入れてくれるので load-path は気にする必要はない。


### ユーザー名/パスワード読み込み関数の定義 {#ユーザー名-パスワード読み込み関数の定義}

[functions.py](#functions-dot-py) で Emacs 経由でユーザー名やパスワードを取れるような関数を用意したので
Emacs 側にそれに対応する関数を用意しておいた。

```emacs-lisp
(defun offlineimap-get-username (host)
  (plist-get (nth 0 (auth-source-search :host host)) :user))

(defun offlineimap-get-password (host)
  (funcall (plist-get (nth 0 (auth-source-search :host host :max 1)) :secret)))
```


### 基本的な設定 {#基本的な設定}

使うフォルダを指定したり、メール取得用のコマンドを指定したり。

```emacs-lisp
(setopt mu4e-root-maildir "~/Maildir")
(setopt mu4e-get-mail-command "offlineimap")
```

`~/Maildir` にはそれぞれ Mail とかのフォルダが生えてるけどとりあえずこの指定が良さそう。もしかしたら後述するコンテキストで切り替えもできるかもしれないけど今はこの設定にしている。

メール取得コマンドの方はとりあえず `offlineimap` を叩くだけで十分っぽいのでそうしているけどなんか良い感じのオプションがあるか調べても良さそう


### mu4e-contexts {#mu4e-contexts}

mu4e には context を切り替える機能があるのでそれを適当に設定している。

以下はとりあえずプロバイダメールと Gmail の設定をしている。

`:enter-func` と `:leave-func` はメッセージ出すだけで大したことはしてないのでまあどうでも良いとして。

`:name` は context を切り替える時に使うので大事かな。切り替えには頭文字を使うので小文字にしておく方が便利。

`:match-func` は、その条件にマッチしたメールを開いた時に自動でそのコンテキストになるっぽい。多分、メール送信設定をして返信する時なんかに役立つはず。このあたりはメールの送信設定をした時に確認する

`:vars` はそのコンテキストの時に適用される設定。今のところ `mu4e-maildir-shortcusts` だけ役立っている気がする。

`mu4e-maildir-shortcusts` を設定しておくとチェックしたい条件のメールをぱっと引っ張り出せるので便利げ。一旦よくあるフォルダだけ設定しているけど、よく見る系のフォルダを設定するとか
[Query](https://djcbsoftware.nl/code/mu/mu4e/Queries.html) を駆使して良い感じに取れるようにするとかできそう。

```emacs-lisp
(setopt mu4e-contexts
        (list
         (make-mu4e-context
          :name "prov"
          :enter-func(lambda () (mu4e-message "Switch to the Provider context"))
          :leave-func (lambda () (mu4e-message "Leaving the Provider context"))
          :match-func (lambda (msg)
                        (when msg
                          (string-prefix-p "/prov" (mu4e-message-field msg :maildir))))
          :vars `((user-mail-address . ,(offlineimap-get-username "offlineimap-prov"))
                  (user-full-name . ,(plist-get (nth 0 (auth-source-search :host "offlineimap-prov")) :fullname))
                  (mu4e-sent-folder . "/prov/Sent")
                  (mu4e-drafts-folder . "/prov/Drafts")
                  (mu4e-trash-folder . "/prov/Trash")
                  (mu4e-compose-signature . ,(concat (plist-get (nth 0 (auth-source-search :host "offlineimap-prov")) :fullname) "\n"))
                  (mu4e-maildir-shortcuts . (("/prov/INBOX" . ?i)
                                             ("/prov/Sent" . ?s)
                                             ("/prov/Drafts" . ?d)
                                             ("/prov/Trash" . ?t)))))
         (make-mu4e-context
          :name "main"
          :enter-func(lambda () (mu4e-message "Switch to the Main context"))
          :leave-func (lambda () (mu4e-message "Leaving the Main context"))
          :match-func (lambda (msg)
                        (when msg
                          (string-prefix-p "/main" (mu4e-message-field msg :maildir))))
          :vars `((user-mail-address . ,(offlineimap-get-username "offlineimap-main"))
                  (user-full-name . ,(plist-get (nth 0 (auth-source-search :host "offlineimap-main")) :fullname))
                  (mu4e-sent-folder . "/main/[Gmail].送信済みメール")
                  (mu4e-drafts-folder . "/main/[Gmail].下書き")
                  (mu4e-trash-folder . "/main/[Gmail].ゴミ箱")
                  (mu4e-refile-folder . "/main/[Gmail].すべてのメール")
                  (mu4e-compose-signature . ,(concat (plist-get (nth 0 (auth-source-search :host "offlineimap-main")) :fullname) "\n"))
                  (mu4e-maildir-shortcuts . (("/main/INBOX" . ?i)
                                             ;; ("/main/[Gmail].送信済みメール" . ?s)
                                             ;; ("/main/[Gmail].すべてのメール" . ?a)
                                             ;; ("/main/[Gmail].下書き" . ?d)
                                             ;; ("/main/[Gmail].ゴミ箱" . ?t)
                                             ))))))
```


## その他 {#その他}

って感じでとりあえず受信はできるようになった。

まだまだやれてないことはたくさんあるのでもう少しいい感じに使えるようにしていく。他にも連携したいアカウントあるしね。
