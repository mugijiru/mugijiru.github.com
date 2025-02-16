+++
title = "2024 年の Emacs 設定変更ふりかえり"
date = 2024-12-22T14:44:00+09:00
categories = ["Emacs"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">&#30446;&#27425;</div>

- [はじめに](#はじめに)
- [Moderinize](#moderinize)
    - [setq → custom-set-variables](#setq-custom-set-variables)
    - [custom-set-variables → setopt](#custom-set-variables-setopt)
    - [global-set-key → keymap-global-set](#global-set-key-keymap-global-set)
- [tree-sitter](#tree-sitter)
    - [treesit-auto](#treesit-auto)
- [Copilot](#copilot)
    - [copilot.el](#copilot-dot-el)
        - [各モードでの有効化](#各モードでの有効化)
        - [warning を黙らせる対応](#warning-を黙らせる対応)
        - [copilot 関係のキーバインド](#copilot-関係のキーバインド)
    - [copilot-chat](#copilot-chat)
- [Ruby 関係](#ruby-関係)
    - [major-mode](#major-mode)
    - [YARD 周り](#yard-周り)
    - [erb](#erb)
    - [リファレンス検索](#リファレンス検索)
    - [Ruby 関係のキーバインド調整](#ruby-関係のキーバインド調整)
    - [Ruby 周りのその他の調整](#ruby-周りのその他の調整)
- [projectile](#projectile)
    - [コマンドの追加など](#コマンドの追加など)
    - [projectile-switch-project の調整](#projectile-switch-project-の調整)
    - [projectile 関係のキーバインド調整](#projectile-関係のキーバインド調整)
- [lsp-mode](#lsp-mode)
    - [eslint の自動フォーマット](#eslint-の自動フォーマット)
    - [TypeScript 周りの調整](#typescript-周りの調整)
    - [graphql-lsp](#graphql-lsp)
    - [lsp-tailwindcss](#lsp-tailwindcss)
    - [lsp-mode 関係のキーバインド調整](#lsp-mode-関係のキーバインド調整)
    - [起動処理の修正](#起動処理の修正)
- [コード検索](#コード検索)
    - [color-moccur &amp; moccur-edit](#color-moccur-and-moccur-edit)
    - [rg.el](#rg-dot-el)
    - [キーバインド調整](#キーバインド調整)
- [org-mode 関係](#org-mode-関係)
    - [org-mode 本体周り](#org-mode-本体周り)
        - [バージョン更新騒動](#バージョン更新騒動)
        - [tangle の調整](#tangle-の調整)
        - [org-mode 周りのその他の変更](#org-mode-周りのその他の変更)
    - [org-agenda](#org-agenda)
        - [設定の記述分割](#設定の記述分割)
        - [custom command の調整](#custom-command-の調整)
    - [org-journal](#org-journal)
        - [Optimize](#optimize)
        - [refile の調整](#refile-の調整)
        - [journal ファイルを org-clock-report の対象になるように調整](#journal-ファイルを-org-clock-report-の対象になるように調整)
        - [org-roam 連携](#org-roam-連携)
    - [org-capture](#org-capture)
        - [capture template の更新](#capture-template-の更新)
        - [org-protocol-capture-html](#org-protocol-capture-html)
    - [org-gcal](#org-gcal)
    - [org-download](#org-download)
    - [org-mode 関係のキーバインド調整](#org-mode-関係のキーバインド調整)
- [el-get](#el-get)
    - [レシピ修正](#レシピ修正)
        - [依存関係の調整](#依存関係の調整)
        - [ブランチ指定漏れ対応](#ブランチ指定漏れ対応)
        - [その他のレシピ修正](#その他のレシピ修正)
    - [el-get 関係のキーバインド調整](#el-get-関係のキーバインド調整)
    - [パッケージ更新](#パッケージ更新)
        - [手動パッケージ更新](#手動パッケージ更新)
        - [自動更新処理の修正](#自動更新処理の修正)
        - [Revert](#revert)
        - [その他](#その他)
- [CI](#ci)
    - [CI 上の Emacs の指定変更](#ci-上の-emacs-の指定変更)
    - [CI 周りのその他の調整](#ci-周りのその他の調整)
- [init.org の調整](#init-dot-org-の調整)
    - [コードとドキュメントの同期](#コードとドキュメントの同期)
- [その他のパッケージ関係](#その他のパッケージ関係)
    - [tsx 関係の修正](#tsx-関係の修正)
    - [text-lint](#text-lint)
    - [pdf-tools](#pdf-tools)
    - [git-modes](#git-modes)
    - [CalibreDB](#calibredb)
    - [blamer.el](#blamer-dot-el)
    - [emojify](#emojify)
    - [smartparens-mode](#smartparens-mode)
    - [minimap](#minimap)
    - [company-mode](#company-mode)
    - [dmacro.el](#dmacro-dot-el)
    - [docker.el](#docker-dot-el)
    - [which-function-mode](#which-function-mode)
    - [plantuml-mode](#plantuml-mode)
    - [nerd-icon-dired](#nerd-icon-dired)
    - [Eask](#eask)
    - [sqlformat](#sqlformat)
    - [chatgtp-shell](#chatgtp-shell)
- [その他のキーバインド調整](#その他のキーバインド調整)
    - [Hydra](#hydra)
    - [key-chord](#key-chord)
- [不要な設定の削除](#不要な設定の削除)
- [レビュー対象 PR 抽出用スクリプトの変更](#レビュー対象-pr-抽出用スクリプトの変更)
- [その他の設定](#その他の設定)
- [ふりかえり](#ふりかえり)
- [来年トライしたいこと](#来年トライしたいこと)

</div>
<!--endtoc-->


## はじめに {#はじめに}

今年のふりかえりってやつをしたいのでとりあえず今年入れた Emacs の設定更新をざっと舐めていく。

とりあえずチェック用のクエリは以下。
<https://github.com/mugijiru/.emacs.d/pulls?q=is%3Apr+is%3Amerged+author%3Amugijiru+merged%3A2024-01-01..2025-01-01+sort%3Acreated-asc>

なんか161個も PR がある。多いな……。

なお当初は月毎に並べてふりかえろうとしてたけど数があまりにも多くてカテゴリ毎にふりかえる方が良さそうなのでそうする。


## Moderinize {#moderinize}

古い記述を新しい記述に置き換える作業。地味に大事。あんまりやってないけど。


### setq → custom-set-variables {#setq-custom-set-variables}

-   <https://github.com/mugijiru/.emacs.d/pull/3916>
    -   カスタマイズ変数を setq で設定していたのを custom-set-variables を使うように修正している


### custom-set-variables → setopt {#custom-set-variables-setopt}

-   <https://github.com/mugijiru/.emacs.d/pull/4281>
    -   Emacs 29.1 から生えた setopt を使うようにしている
    -   custom-set-variables より書きやすくて好み


### global-set-key → keymap-global-set {#global-set-key-keymap-global-set}

global-set-key から keymap-global-set に置き換え

-   <https://github.com/mugijiru/.emacs.d/pull/6329>
-   <https://github.com/mugijiru/.emacs.d/pull/6330>


## tree-sitter {#tree-sitter}


### treesit-auto {#treesit-auto}

今年になってから `*-ts-mode` デビューした。
[treesit-auto](https://github.com/renzmann/treesit-auto) を導入すると `*-ts-mode` が確認されているやつは自動的にインストールしてそれを使うようにしてくれて便利ってことでその対応をしたり

-   <https://github.com/mugijiru/.emacs.d/pull/3688>
    -   ここで treesit-auto を導入した。
-   <https://github.com/mugijiru/.emacs.d/pull/4188>
    -   treesit-auto を入れた影響で docker-ts-mode が動くようになったので調整している
-   <https://github.com/mugijiru/.emacs.d/pull/4189>
    -   同様に yaml-ts-mode でも同じことをしている
    -   とはいえインデントが気に入らなかったので yaml-mode を使うように戻していたりする
        -   <https://github.com/mugijiru/.emacs.d/pull/6283>
        -   そのついでに yaml-pro とかいうのも入れたけど、これってどんなやつだっけ


## Copilot {#copilot}


### copilot.el {#copilot-dot-el}

昨年から [copilot.el](https://github.com/copilot-emacs/copilot.el) を導入しているのでその調整


#### 各モードでの有効化 {#各モードでの有効化}

-   <https://github.com/mugijiru/.emacs.d/pull/4461>
    -   よく使う major-mode では copilot-mode が有効になるようにしている


#### warning を黙らせる対応 {#warning-を黙らせる対応}

-   <https://github.com/mugijiru/.emacs.d/pull/3906>
    -   デカいファイルを開いた時に warning が popup してうざいので黙らせるようにしたやつ
    -   いくつかデカいファイルがあるからね……
-   <https://github.com/mugijiru/.emacs.d/pull/6116>
    -   indent offset の設定がない時の warning を黙らせた
    -   なんかやたら出て来て邪魔だったのよね


#### copilot 関係のキーバインド {#copilot-関係のキーバインド}

-   <https://github.com/mugijiru/.emacs.d/pull/5917>
    -   単に tab で補完する以外のこともできるのでキーバインドを追加した。
    -   しかしさっぱりキーバインドを覚えてないので活用できてない。困った
-   <https://github.com/mugijiru/.emacs.d/pull/4744>
    -   copilot-mode を Hydra で toggle できるようにした


### copilot-chat {#copilot-chat}

-   <https://github.com/mugijiru/.emacs.d/pull/6117>
    -   導入してみた。最近よく使ってる。便利。
-   <https://github.com/mugijiru/.emacs.d/pull/6241>
    -   設定を弄って使いやすくした
    -   もうちょっと弄りたいけど
-   <https://github.com/mugijiru/.emacs.d/pull/6264>
    -   設定読み込み順の問題に対応しただけ


## Ruby 関係 {#ruby-関係}


### major-mode {#major-mode}

-   <https://github.com/mugijiru/.emacs.d/pull/5791>
    -   ruby-ts-mode を使えるようにするため enh-ruby-mode に使ってるのと同じ hook を追加している
    -   のだけどインデントの挙動の問題で結局 enh-ruby-mode を使っている


### YARD 周り {#yard-周り}

-   <https://github.com/mugijiru/.emacs.d/pull/4847>
    -   YARD の method comment 用に snippet を追加した。地味に便利。
-   <https://github.com/mugijiru/.emacs.d/pull/5849>
    -   yard-mode を入れて YARD コメントのハイライト表示などに対応


### erb {#erb}

-   <https://github.com/mugijiru/.emacs.d/pull/6327>
    -   erb を弄るのに使っている web-mode の設定変更
    -   いくつかの minor-mode を有効にしている


### リファレンス検索 {#リファレンス検索}

-   <https://github.com/mugijiru/.emacs.d/pull/5876>
    -   Emacs 内でるりまサーチとかを検索できるように engine-mode の設定を追加している
-   <https://github.com/mugijiru/.emacs.d/pull/5918>
    -   rurema 用の w3m-filter の調整


### Ruby 関係のキーバインド調整 {#ruby-関係のキーバインド調整}

-   <https://github.com/mugijiru/.emacs.d/pull/5796>
    -   projectile-rails 用の Hydra の更新
    -   rails console などを開けるようにしている
-   <https://github.com/mugijiru/.emacs.d/pull/5885>
    -   projectile-rails の finder を充実させた
-   <https://github.com/mugijiru/.emacs.d/pull/5880>
    -   ruby 用の Hydra から LSP 関係を抜いた
    -   lsp-mode 用の Hydra は別途定義しているからね〜
-   <https://github.com/mugijiru/.emacs.d/pull/5914>
    -   rubocop の設定ファイルを projectile-rails-hydra から開きやすくした程度
    -   ruby 用の Hydra に入れても良かった気がする


### Ruby 周りのその他の調整 {#ruby-周りのその他の調整}

-   <https://github.com/mugijiru/.emacs.d/pull/5916>
    -   ruby/rspec 関係の設定を色々更新
    -   秋フェスに向けての準備をしていた時期だった
-   <https://github.com/mugijiru/.emacs.d/pull/5881>
    -   rails-i18n というのを入れたけど cache でバグるので使えてない。無念。
-   <https://github.com/mugijiru/.emacs.d/pull/5886>
    -   bundler.el を入れた。
    -   便利そうなので入れたけど使ってないな。
    -   Hydra に組込むと良いのかも
-   <https://github.com/mugijiru/.emacs.d/pull/5944>
    -   とりあえず ruby-refactor を入れてみた
    -   入れたことも忘れてたし keybind もわからんので使ってない
    -   Hydra も弄ってるからそれ見て思い出してね自分


## projectile {#projectile}


### コマンドの追加など {#コマンドの追加など}

-   <https://github.com/mugijiru/.emacs.d/pull/4750>
    -   projectile と連携したコマンドをいくつか用意している
    -   プロジェクト固有の Hydra も定義できるようにしているけど使ってね〜


### projectile-switch-project の調整 {#projectile-switch-project-の調整}

-   <https://github.com/mugijiru/.emacs.d/pull/4237>
    -   projectile-switch-project で magit が開くようにしたやつ
    -   元々は neotree が動くようになってたけど neotree 活用してないんだよね……


### projectile 関係のキーバインド調整 {#projectile-関係のキーバインド調整}

-   <https://github.com/mugijiru/.emacs.d/pull/5884>
    -   projectile 用の Hydra に counsel-projectile のコマンドを色々追加
    -   ついでに counsel-projectile-org-capture のテンプレートも設定しているのでプロジェクト毎の capture が取れるようになった
-   <https://github.com/mugijiru/.emacs.d/pull/5915>
    -   projectile-hydra から Dockerfile とかを開きやすくしている。使ってないけど


## lsp-mode {#lsp-mode}


### eslint の自動フォーマット {#eslint-の自動フォーマット}

eslint のフォーマットに lsp-format-buffer を使うようにしたり lsp-eslint-fix-all を使うようにしたりしている。うまく動いてくれないって悩んでた。最終的には `lsp-eslint-apply-all-fix` を使うようにしたみたい

-   <https://github.com/mugijiru/.emacs.d/pull/3679>
-   <https://github.com/mugijiru/.emacs.d/pull/3689>
-   <https://github.com/mugijiru/.emacs.d/pull/3845>
    -   ここではついでに tsx-ts-mode で smartparens-strict-mode も有効化している


### TypeScript 周りの調整 {#typescript-周りの調整}

-   <https://github.com/mugijiru/.emacs.d/pull/3844>
    -   監視対象ファイルが多いぞいって怒られるので無視するディレクトリの指定をしたりインデント設定の調整をしたり
        -   監視対象が多いってのは大体 eslint で怒られてた気がする
    -   tsx-ts-mode を入れたばかりで調整が必要な時期だったっぽい
-   <https://github.com/mugijiru/.emacs.d/pull/4187>
    -   ここでも無視するディレクトリを調整している


### graphql-lsp {#graphql-lsp}

-   <https://github.com/mugijiru/.emacs.d/pull/6328>
    -   graphql-lsp を入れたはいいけどなんか重いので tsx ファイルなんかでは動かないようにした


### lsp-tailwindcss {#lsp-tailwindcss}

TailwindCSS を使うことになったので導入した

-   <https://github.com/mugijiru/.emacs.d/pull/6215>


### lsp-mode 関係のキーバインド調整 {#lsp-mode-関係のキーバインド調整}

-   <https://github.com/mugijiru/.emacs.d/pull/5798>
    -   lsp-mode 用の Hydra で lsp-workspace-restart を実行できるようにしている
    -   shutdown とかも入れると便利そうだけど lsp-mode 用の Hydra はもう混雑しているのよね
-   <https://github.com/mugijiru/.emacs.d/pull/5883>
    -   lsp-mode 用の Hydra に色々コマンドを追加しておいたやつ
    -   lsp-ui 系のやつとか色々入れてるけど特に使ってないな〜


### 起動処理の修正 {#起動処理の修正}

-   <https://github.com/mugijiru/.emacs.d/pull/5797>
    -   不必要に引数を追加していたので修正


## コード検索 {#コード検索}


### color-moccur &amp; moccur-edit {#color-moccur-and-moccur-edit}

-   <https://github.com/mugijiru/.emacs.d/pull/4232>
    -   プロジェクト内の検索に便利かなって color-moccur と moccur-edit を導入している。
    -   けど最近はもっぱら rg.el を使っているな……


### rg.el {#rg-dot-el}

-   <https://github.com/mugijiru/.emacs.d/pull/4236>
    -   ここで [rg.el](https://github.com/dajva/rg.el) と [wgrep](https://github.com/mhayashi1120/Emacs-wgrep) を導入している
-   <https://github.com/mugijiru/.emacs.d/pull/6284>
    -   上の PR でrg.el と wgrep 入れたはいいけど連携できてなかった
    -   最近はもっぱらこの組み合わせで検索してて便利。


### キーバインド調整 {#キーバインド調整}

-   <https://github.com/mugijiru/.emacs.d/pull/5913>
    -   rg.el の menu を開く方法を覚えられないのでそれだけのために Hydra を用意した


## org-mode 関係 {#org-mode-関係}


### org-mode 本体周り {#org-mode-本体周り}


#### バージョン更新騒動 {#バージョン更新騒動}

バージョンを 9.7 に上げようとして諦めたという流れがありました。ただ ox-hugo や org-gcal が動かなくなるので今は 9.6 に固定しています

-   <https://github.com/mugijiru/.emacs.d/pull/5404>
-   <https://github.com/mugijiru/.emacs.d/pull/5411>

そして orgit が org-mode 9.7 に依存するようになっていたけど
orgit をあまり使っていないので、じゃあこいつにいなくなってもらおうという判断で消えてもらってたり

-   <https://github.com/mugijiru/.emacs.d/pull/5420>


#### tangle の調整 {#tangle-の調整}

-   <https://github.com/mugijiru/.emacs.d/pull/3919>
    -   tangle した時の indent 周りの調整をしてる


#### org-mode 周りのその他の変更 {#org-mode-周りのその他の変更}

-   <https://github.com/mugijiru/.emacs.d/pull/3871>
    -   org-mode のファイルを開いた時に見出しが折り畳まれてるように設定した。
    -   いつからかデフォルト値が変わったようでずっと開いたままになっていて邪魔だったのをようやく直した感じ
-   <https://github.com/mugijiru/.emacs.d/pull/6311>
    -   subword-mode を有効にしただけ
-   <https://github.com/mugijiru/.emacs.d/pull/3912>
    -   昔は階層に合わせて indent する設定にしていた名残りが残っていてその対応
-   <https://github.com/mugijiru/.emacs.d/pull/3975>
    -   [ob-graphql](https://github.com/jdormit/ob-graphql) が便利と聞いてとりあえず入れたやつ
    -   活用してないな……


### org-agenda {#org-agenda}


#### 設定の記述分割 {#設定の記述分割}

init.org で1つのコードブロックに全部書かれていたけどそれだとドキュメント的に微妙だったので分割して書けるようにしていた

-   <https://github.com/mugijiru/.emacs.d/pull/3917>
-   <https://github.com/mugijiru/.emacs.d/pull/3957>
-   <https://github.com/mugijiru/.emacs.d/pull/4471>
-   <https://github.com/mugijiru/.emacs.d/pull/4742>
-   <https://github.com/mugijiru/.emacs.d/pull/4745>


#### custom command の調整 {#custom-command-の調整}

agenda の表示はずっと課題なので度々修正している

-   <https://github.com/mugijiru/.emacs.d/pull/3767>
    -   休日に見るための custom command で優先度が低いやつは表示しないようにしている。
    -   agenda が長くなるの未だに課題なのでなんとかしたい
-   <https://github.com/mugijiru/.emacs.d/pull/3768>
    -   こちらは平日に見る agenda にスケジュールを設定してないタスクを表示するようにしている
    -   スケジュール入れてないタスクの存在が忘れ去られないようにするための対策だね
-   <https://github.com/mugijiru/.emacs.d/pull/3769>
    -   これは平日用 agenda の見た目の調整。
    -   とはいえあまりうまくいってないので、現状は子タスクを作らない運用になっている
-   <https://github.com/mugijiru/.emacs.d/pull/3807>
    -   休日用 agenda に Private という agenda-group に放り込んでるタスクを表示するようにした
    -   ちなみに Private に分類されているものは「炊飯器の予約時間を変更する」みたいな分類が面倒だったようなやつが入りがち
        -   家事に分類しても良かったなこれ。まあいいけど
-   <https://github.com/mugijiru/.emacs.d/pull/4817>
    -   agenda に予定日とか締切日も表示するようになった。便利。
-   <https://github.com/mugijiru/.emacs.d/pull/4858>
    -   平日見る用 agenda の調整。見てないようなやつは表示しなくて良いよねと。
-   <https://github.com/mugijiru/.emacs.d/pull/4869>
    -   業務タスクを引っ張り出すための agenda を追加している。完全に忘れてた。
-   <https://github.com/mugijiru/.emacs.d/pull/4870>
    -   打刻とかの業務開始直後のタスクを入れてる agenda の調整
    -   朝以外にやっても構わないやつを取り除いている
-   <https://github.com/mugijiru/.emacs.d/pull/5790>
    -   日中見るための agenda に朝やるべきやつも表示するようにした
    -   忙しくてやり忘れてることとかあるからね


### org-journal {#org-journal}

昨年から org-journal を導入しているのでちょこちょこと調整をしている


#### Optimize {#optimize}

journal ファイルが増えて来て遅くなったのでそれを解消したりしていた

-   <https://github.com/mugijiru/.emacs.d/pull/4274>
    -   org-journal のファイルが増えて来て org-agenda のバッファ構築が重くなって来たから org-journal の全ファイルを舐めないように調整している
-   <https://github.com/mugijiru/.emacs.d/pull/4280>
    -   そして org-journal の保存も重くなってるので org-agenda-targets が自動更新される仕組みをオフにしている


#### refile の調整 {#refile-の調整}

org-journal ファイルを新しく作った時にそのファイルが refile のターゲットになるようにしている。その日やる予定のタスクなんかは journal ファイルに移動する運用なので、これがないと厳しい。

本来は org-journal の機能で自動的に良い感じにしてくれた気がするけど、
Optimize でそれを無効化しているので自前で適当に対応している。

-   <https://github.com/mugijiru/.emacs.d/pull/3846>
-   <https://github.com/mugijiru/.emacs.d/pull/4860>


#### journal ファイルを org-clock-report の対象になるように調整 {#journal-ファイルを-org-clock-report-の対象になるように調整}

これも Optimize の影響で良い感じにならなくなったやつの対応かな
org-clock-report の対象にその日の journal ファイルも入れたいので org-agenda-files を良い感じに調整するコマンドを追加した。

コマンドではなく何かの hook で動くとより便利な気がするけどまだそこまでは設定してない

-   <https://github.com/mugijiru/.emacs.d/pull/6289>
-   <https://github.com/mugijiru/.emacs.d/pull/6326>


#### org-roam 連携 {#org-roam-連携}

-   <https://github.com/mugijiru/.emacs.d/pull/4859>
    -   出力先を org-roam の下に移動することで id を付与したら検索対象となるようにしている
    -   id 付与してないけどな……


### org-capture {#org-capture}

ぱっとメモを取るのに org-capture は便利でよく使ってるけど使っていると不満が出て来るのでちょこちょこ更新している


#### capture template の更新 {#capture-template-の更新}

出力内容を増やしたり減らしたりしている。未だに最適な出力が分からない〜

-   <https://github.com/mugijiru/.emacs.d/pull/4288>
    -   org-capture-template で日付も入れるようにしている。
    -   いつ登録したタスクなのかが分かると捨てる判断材料になりそうなので入れている
-   <https://github.com/mugijiru/.emacs.d/pull/4376>
    -   capture template を更新。使ってないのを消したり、逆に今後使いたい読書メモ用の capture を用意したり
    -   読書メモとか取ってないけどな……!
-   <https://github.com/mugijiru/.emacs.d/pull/6097>
    -   不要な capture template の削除
-   <https://github.com/mugijiru/.emacs.d/pull/6288>
    -   counsel-projectile-org-capture-template でバグとかを登録できるようにした
    -   まだ活用できてね〜


#### org-protocol-capture-html {#org-protocol-capture-html}

[org-protocol-capture-html](https://github.com/alphapapa/org-protocol-capture-html) ってのを[フォーク](https://github.com/mugijiru/org-protocol-capture-html)した上で導入している。

Web ページを org-mode の記述に変換して capture できるやつ。なんだけど活用できてない。ま、うまく変換できないこともあるしね

-   <https://github.com/mugijiru/.emacs.d/pull/4385>
-   <https://github.com/mugijiru/.emacs.d/pull/4470>


### org-gcal {#org-gcal}

-   <https://github.com/mugijiru/.emacs.d/pull/4748>
    -   org-gcal で使う OAuth token を plstore で暗号化して保存するようにしたやつ。便利。


### org-download {#org-download}

-   <https://github.com/mugijiru/.emacs.d/pull/5879>
    -   キャプチャとか貼るのに便利なので入れたけどあんまり使ってないな


### org-mode 関係のキーバインド調整 {#org-mode-関係のキーバインド調整}

-   <https://github.com/mugijiru/.emacs.d/pull/3754>
    -   こちらは org-mode の見出し検索が counsel でできるのでそのコマンドを Hydra から呼び出せるようにしている。
    -   ほぼ使ってないけど便利そう……。
-   <https://github.com/mugijiru/.emacs.d/pull/4469>
    -   org-agenda-mode 用の Hydra を更新している


## el-get {#el-get}


### レシピ修正 {#レシピ修正}

自前でレシピを適当に置いているけど、まあまあミスってるので度々修正している。これにミスがあるとパッケージの更新 PR 作成ワークフローがコケてしまったりする


#### 依存関係の調整 {#依存関係の調整}

時々依存関係に変更があるのでそれに追従している

-   <https://github.com/mugijiru/.emacs.d/pull/3744>
    -   prescient.el が corfu に依存していたのでレシピを修正している。
-   <https://github.com/mugijiru/.emacs.d/pull/3750/files>
    -   こちらは copilot.el のレシピ修正。
    -   json-rpc に依存している思ったら jsonrpc に依存していたというややこしさ
-   <https://github.com/mugijiru/.emacs.d/pull/4170>
    -   Ember.js 関係のパッケージのレシピを修正している。
-   <https://github.com/mugijiru/.emacs.d/pull/5166>
    -   emacsql の依存関係に pg が増えたので対応している
-   <https://github.com/mugijiru/.emacs.d/pull/5421>
    -   pg が peg に依存していることに気付いたので


#### ブランチ指定漏れ対応 {#ブランチ指定漏れ対応}

自前のパッケージ自動更新機構の都合上
main ブランチで開発されている場合はそれを指定しないといけなくなっているので、度々その修正をしている

-   <https://github.com/mugijiru/.emacs.d/pull/5981>
    -   pg は main ブランチの指定が必要だった
-   <https://github.com/mugijiru/.emacs.d/pull/6225>
    -   chatogpt-shell と shell-maker でもブランチ指定が必要だった


#### その他のレシピ修正 {#その他のレシピ修正}

-   <https://github.com/mugijiru/.emacs.d/pull/3775>
    -   これは copilot.el のリポジトリが移動されたのでそれに追従しているだけ
-   <https://github.com/mugijiru/.emacs.d/pull/6263>
    -   lsp-tailwindcss のレシピでミスってたので修正


### el-get 関係のキーバインド調整 {#el-get-関係のキーバインド調整}

-   <https://github.com/mugijiru/.emacs.d/pull/3751>
    -   EmacsWiki とか ELPA とかのレシピを build するためのコマンドを Hydra から呼び出せるようにしている。
    -   まあほぼ使ってないんだけど


### パッケージ更新 {#パッケージ更新}

自動で毎日 PR が作られるようにしているけど時々うまく動かなかったり、更新したら壊れたりするのでそういう時は手で対応している。


#### 手動パッケージ更新 {#手動パッケージ更新}

magit 系のパッケージが texi ファイルで差分を生むようになっていて
el-get-update ができなくなっていたのでローカルでごにゃごにゃ対応してるやつ

-   <https://github.com/mugijiru/.emacs.d/pull/4746>
-   <https://github.com/mugijiru/.emacs.d/pull/5203>
-   <https://github.com/mugijiru/.emacs.d/pull/5204>
-   <https://github.com/mugijiru/.emacs.d/pull/5244>
-   <https://github.com/mugijiru/.emacs.d/pull/5664>
-   <https://github.com/mugijiru/.emacs.d/pull/5731>

それ以外にもレシピがおかしくて更新できないパターンもある

-   <https://github.com/mugijiru/.emacs.d/pull/6046>
    -   peg 依存の対応が必要だったかなこれは


#### 自動更新処理の修正 {#自動更新処理の修正}

手動更新なんてしたくないので問題が起きた時は修正している。

<!--list-separator-->

-  texi ファイルの上書き対応

    texi ファイルを上書きするせいでパッケージ更新ができなくなる問題への対応。

    -   <https://github.com/mugijiru/.emacs.d/pull/4753>
        -   ここではドキュメント生成処理をスルーしている
    -   <https://github.com/mugijiru/.emacs.d/pull/6019>
        -   ここでは texi ファイルの変更を戻すという荒技を使ってる
        -   前のやり方の方が平和なのでは!?

<!--list-separator-->

-  古い更新 PR の自動削除

    パッケージ更新の PR を毎日出しているけど前日分の PR が残り続ける問題への対応をした。単に古い PR は閉じちゃうという仕組み。

    -   <https://github.com/mugijiru/.emacs.d/pull/4775>
    -   <https://github.com/mugijiru/.emacs.d/pull/4776>

<!--list-separator-->

-  無駄な努力の形跡

    これは別の原因でコケてるのに気付かずに努力した形跡。結局はレシピを間違えていたやつでした

    -   <https://github.com/mugijiru/.emacs.d/pull/6133>
        -   \*.elc が自動生成されるせいでパッケージ更新ができない問題の対応
        -   結局これで解決しなかったけどね
    -   <https://github.com/mugijiru/.emacs.d/pull/6216>
        -   \*.elc の生成の対応がうまくいかなかったので対応第二弾
        -   これもだめでした
    -   <https://github.com/mugijiru/.emacs.d/pull/6219>
        -   うまくいかないので debug のためのコードを入れたり
    -   <https://github.com/mugijiru/.emacs.d/pull/6220>
        -   そしてそれもうまくいかなかったり
    -   <https://github.com/mugijiru/.emacs.d/pull/6221>
        -   .gitignore に書き込むことで \*.elc を無視させようとしたり


#### Revert {#revert}

-   <https://github.com/mugijiru/.emacs.d/pull/3918>
    -   なんか forge が動かなくなってたみたい
    -   el-get.lock でバージョン管理しているので戻すこともできるのは大変良い


#### その他 {#その他}

-   <https://github.com/mugijiru/.emacs.d/pull/4773>
    -   自動パッケージ更新 PR の codeberg の URL 出力を調整したつもりだったけどうまくいってないねハハハ


## CI {#ci}


### CI 上の Emacs の指定変更 {#ci-上の-emacs-の指定変更}

-   <https://github.com/mugijiru/.emacs.d/pull/4774>
    -   CI で使ってる Emacs のマイナーバージョンを調整
-   <https://github.com/mugijiru/.emacs.d/pull/5878>
    -   テストする Emacs のバージョンを削減
-   <https://github.com/mugijiru/.emacs.d/pull/6331>
    -   Emacs 29.1 以降でしか動かないコードやパッケージがあるのでテスト対象を狭めた


### CI 周りのその他の調整 {#ci-周りのその他の調整}

-   <https://github.com/mugijiru/.emacs.d/pull/3638>
    -   パッケージ更新 PR に貼るラベルを `dependencies` にしたらしい。まあ趣味の範囲か。
-   <https://github.com/mugijiru/.emacs.d/pull/4462>
    -   CI で動かす Hugo のバージョンを上げてる。割とよく放置するのよね。


## init.org の調整 {#init-dot-org-の調整}


### コードとドキュメントの同期 {#コードとドキュメントの同期}

今年の頭まではドキュメントとコードにズレが生じていたのでそのズレを解消するための PR が度々立てられていた。

-   <https://github.com/mugijiru/.emacs.d/pull/3756>
-   <https://github.com/mugijiru/.emacs.d/pull/3757>
-   <https://github.com/mugijiru/.emacs.d/pull/3900>
-   <https://github.com/mugijiru/.emacs.d/pull/3908>
-   <https://github.com/mugijiru/.emacs.d/pull/3909>
-   <https://github.com/mugijiru/.emacs.d/pull/3910>
-   <https://github.com/mugijiru/.emacs.d/pull/3911>
-   <https://github.com/mugijiru/.emacs.d/pull/3913>

なんだけど
<https://github.com/mugijiru/.emacs.d/pull/3915>
で差分を検知するための workflow を追加している。というわけでここからはズレが生じないはず。


## その他のパッケージ関係 {#その他のパッケージ関係}


### tsx 関係の修正 {#tsx-関係の修正}

-   <https://github.com/mugijiru/.emacs.d/pull/3755>
    -   context-skk-mode を有効にしているだけ


### text-lint {#text-lint}

-   <https://github.com/mugijiru/.emacs.d/pull/3899>
    -   magit-commit-create のタイミングで動くようにしたいけどうまくいってないやつ
    -   今でもうまくいってないんじゃないかなこれ


### pdf-tools {#pdf-tools}

-   <https://github.com/mugijiru/.emacs.d/pull/3907>
    -   PDF を Emacs で読むために設定していた形跡がある。実際ほぼ使ってない……


### git-modes {#git-modes}

-   <https://github.com/mugijiru/.emacs.d/pull/4213>
    -   便利そうなので追加したみたやつ
    -   Git の設定ファイルってあんまり触らないから実際便利なのかは覚えてない


### CalibreDB {#calibredb}

-   <https://github.com/mugijiru/.emacs.d/pull/4218>
    -   蔵書管理を [calibredb.el](https://github.com/chenyanming/calibredb.el) でやろうとしていた時期のやつ
    -   ちゃんとメンテしてないんですよね蔵書周り。


### blamer.el {#blamer-dot-el}

-   <https://github.com/mugijiru/.emacs.d/pull/4282>
    -   カーソル位置のコミット情報を見られるようにするやつが便利そうなので入れてみている
    -   とはいえ実際なんか vc-annotate の方をよく使う気がする……


### emojify {#emojify}

-   <https://github.com/mugijiru/.emacs.d/pull/4283>
    -   コード中でも容赦なく変換されると邪魔なので global-emojify-mode をやめたようだ


### smartparens-mode {#smartparens-mode}

-   <https://github.com/mugijiru/.emacs.d/pull/4428>
    -   なんか動かなくなったので issue にあるモンキーパッチを当ててる
-   <https://github.com/mugijiru/.emacs.d/pull/4456>
    -   そして本体で直されたのでモンキーパッチの PR を revert


### minimap {#minimap}

-   <https://github.com/mugijiru/.emacs.d/pull/4463>
    -   あると便利かなって使えるように調整している
    -   けど実際使ってないのよな


### company-mode {#company-mode}

-   <https://github.com/mugijiru/.emacs.d/pull/5919>
    -   scss での backends を調整したり


### dmacro.el {#dmacro-dot-el}

dmacro.el は秋フェスで聞いて便利そうだから入れた。けどイマイチ活用できてない。無念。

-   <https://github.com/mugijiru/.emacs.d/pull/5945>
-   <https://github.com/mugijiru/.emacs.d/pull/5946>


### docker.el {#docker-dot-el}

-   <https://github.com/mugijiru/.emacs.d/pull/5964>
    -   docker.el を導入してた。すっかり忘れてたので使ってないですね


### which-function-mode {#which-function-mode}

デフォルトで入ってる which-function-mode って便利そうだなってことで設定してみている。最初はただの minor-mode だと思ってたけど実はこいつ global-minor-mode だったということに気付いて設定を修正したり。

-   <https://github.com/mugijiru/.emacs.d/pull/4272>
-   <https://github.com/mugijiru/.emacs.d/pull/5882>


### plantuml-mode {#plantuml-mode}

-   <https://github.com/mugijiru/.emacs.d/pull/5973>
    -   plantuml を書く時のインデント幅がデフォだと8文字だったけどそんなに幅取らないで欲しいので調整した


### nerd-icon-dired {#nerd-icon-dired}

-   <https://github.com/mugijiru/.emacs.d/pull/5877>
    -   dired にアイコン表示をしたくなったので nerd-icon-dired を追加
    -   でも dired でファイル追加した時とかのアイコン表示位置がおかしいんだよな〜


### Eask {#eask}

-   <https://github.com/mugijiru/.emacs.d/pull/6127>
    -   eask-mode と flycheck-eask を導入。
    -   まあ flycheck-eask の方はうまく動いていないんだけど


### sqlformat {#sqlformat}

-   <https://github.com/mugijiru/.emacs.d/pull/5972>
    -   SQL を書く仕事をしている時に整形したいなあってことで導入した。
    -   また SQL を書く時に存在を思い出せると良いな
    -   major-mode-hydra でも定義する?


### chatgtp-shell {#chatgtp-shell}

-   <https://github.com/mugijiru/.emacs.d/pull/6310>
    -   copilot-chat の中で使われてるのでこちらもついでにセットアップして Gemini と話せるようにしたやつ


## その他のキーバインド調整 {#その他のキーバインド調整}


### Hydra {#hydra}

-   <https://github.com/mugijiru/.emacs.d/pull/4273>
    -   機能を Toggle するための Hydra の1列に物が増えて来たので分割して調整している
-   <https://github.com/mugijiru/.emacs.d/pull/5911>
    -   pretty-hydra 用の snippet を少し分かりやすくした


### key-chord {#key-chord}

-   <https://github.com/mugijiru/.emacs.d/pull/4493>
    -   オリジナル版を使うようにした


## 不要な設定の削除 {#不要な設定の削除}

-   <https://github.com/mugijiru/.emacs.d/pull/3914>
    -   mocha.js は使わなくなったので設定ファイルを消した
    -   けど、また使うこともあるかもしれないってことでまだ init.org には残しているみたい
-   <https://github.com/mugijiru/.emacs.d/pull/6004>
    -   beorg 用の同期は Dropbox に移行したので WebDAV の設定は不要になりました


## レビュー対象 PR 抽出用スクリプトの変更 {#レビュー対象-pr-抽出用スクリプトの変更}

-   <https://github.com/mugijiru/.emacs.d/pull/3727>
    -   自前で適当に作ってるレビュー対象の Pull requests を取得するやつの改修。
    -   なんか適当に定義していた関数を使ってないだろうって消してたけど実は使われていたのでそれを戻しているだけ。
-   <https://github.com/mugijiru/.emacs.d/pull/4190>
    -   エラーハンドリングに問題があったみたい


## その他の設定 {#その他の設定}

-   <https://github.com/mugijiru/.emacs.d/pull/4743>
    -   .gitignore に自動生成ファイルを登録した
-   <https://github.com/mugijiru/.emacs.d/pull/5912>
    -   Emacs から GitHub のコード検索を開けるようにしたやつ。
    -   ただし開くのは Firefox
    -   地味によく使っている


## ふりかえり {#ふりかえり}

-   全体
    -   1年分まとめてふりかえるの大変だったから来年は小まめにやりたいね……!
-   新しく使い始めたもの
    -   copilot-chat が良い感じ
        -   chatgtp-shell も気になるけどあまり触れてないな〜
    -   rg.el + wgrep
        -   ここ最近よく使ってる。便利。
    -   tree-sitter
        -   TypeScript 系はこれで良さげ。
        -   Ruby, YAML はインデントが微妙なので戻したけど
            -   tree-sitter がそのあたりも担当するはずなので頑張ればなんとかなるのか……?
-   Ruby 関係
    -   色々更新して良くなった部分もある
    -   けど、使ってないのも沢山あるなあ……
        -   bunlder.el, ruby-refactor, etc...
        -   リファレンス検索も今のところあまり使ってないし
-   projectile
    -   Ruby 関係を弄ったついでか結構弄った。良き
    -   finder を色々追加しているけど projectile-find-file で間に合う場合そっちを使いがち
        -   Dockerfile を開くとかね……
        -   counsel-projectile-org-capture は良さそう
-   lsp-mode 周り
    -   面倒なのであまり設定弄りたくないけど度々重くなるので困ってる様子
    -   自動フォーマットもちょくちょく動かなくなるので悩ましい
        -   重くて LSP サーバが反応しないとかもあって切り分けも面倒。
-   org-mode 関係
    -   agenda 周りで困ってることがよくわかる。未だに困ってるもんね。
    -   org-journal 周りも結局 org-agenda-files への登録系で困ってるからね
    -   org-roam はイマイチ活用できてないかも。とはいえ時々メモを入れてるけどね
        -   org-capture を調整すると便利なのかも
-   el-get
    -   自動パッケージ更新 PR 作成周りがちょくちょくバグるけど、十分助かってる感じ
-   その他
    -   ちょっとだけ Modernize しているの良き
    -   ドキュメントとコードがズレない仕組みを入れてるの素晴らしい
    -   sqlformat は出番は少ないけど地味に便利。次 SQL 書く時まで忘れないであげてほしい
    -   textlint は割と無視しているので要らないかもしれない
    -   選択範囲から GitHub のコード検索に飛べるようにしているのは地味に便利。
        -   emacs lisp のコードとかよく調べている
    -   blamer.el あんまり要らないかも。vc-annotate 便利。
    -   minimap もそう
    -   nerd-icons-dired も挙動変なところあるし


## 来年トライしたいこと {#来年トライしたいこと}

-   tab-bar-mode
    -   便利らしいので気になってきた
-   leaf.el
    -   そろそろ設定ファイルの書き方も Modernize したいなって
-   脱 el-get
    -   straight.el とかの方が使い方的には合ってそうな気がして
-   読書環境の調整
    -   pdf-tools とか入れてるけど活用できてない
    -   ちゃんと本を読んでメモる習慣を身に付けたい
        -   なお昨今は iPhone の読み上げに頼ってるので腰を据えて読書する方針とは相性が悪い模様
    -   CalibreDB もちゃんと更新しようね僕……
-   smartparens → puni 移行を検討したい
    -   なんなら今年やりたかった
