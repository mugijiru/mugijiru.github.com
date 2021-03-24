+++
title = "Ember.js の共通コンポーネントの NPM への分離"
date = 2021-03-24T13:04:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

相変わらず Rails における Ember.js 関連で遊んでいます。

今回の記事で書くのは「Ember.js Addon を別リポジトリに分離して NPM パッケージにして利用する」なんだけど、後追いで出す「分離したリポジトリを Gem としても使えるようにし ember-rails 環境で動かす」という感じに記事の前振りです。

本当は1つの記事にしたかったけど記述量が増えたから分割……。


## 目的 {#目的}

なんでこんなことをしているかというと「単一ソースで ember-cli にも ember-rails にも対応したい」「じゃあ NPM と Gem で公開してインスコしたらいいんじゃね」という考えから。

というのも、この一連の記事は ember-rails から ember-cli-rails に徐々に移行していく手段を確立することが目的なので、
1つの Rails の中に複数の Ember.js アプリがあって
ember-rails と ember-cli-rails が混在している状況も有り得るかなあと。

そういう時に、共通コンポーネントは単一ソースで使いたいよね〜と思って両対応ができるようにしてみている。

まあ今回は ember-cli-rails だけの対応なんだけども。


## 実践 {#実践}

ember-cli-rails に移行した時に元々 `RAILS_ROOT/app/assets/javascripts/ember-libs` というところに共通コンポーネントとして置いていたファイル群を
`RAILS_ROOT/ember/my-components` というところに
Ember.js のアドオンという形で設置していました。

正直そのままの方が、同一リポジトリなので改修とかしやすいんだけど「他のプロジェクトでも使いたい」といった時には分離も必要になるかなと。まあ今回の目的は別のところにあるけども。


### 従来の実装を ember-components に移植 {#従来の実装を-ember-components-に移植}

<https://github.com/mugijiru/ember-components/commit/847981e9732385d08db4f5f703813196622b80d2>

でやっていること。

基本的には、元々のソースを addons 以下に置いているだけ。なんとなく、コンポーネントの prefix を my- から mg- に変えてるけど。

あとは ember-cli-htmlbars を dependencies にも移動する必要あり。

<https://github.com/mugijiru/ember-components/commit/922d1f7ed5f6b3372b1d1551792f4e9739f5b1e3>

他にも [Docker で動かせるようにしたり](https://github.com/mugijiru/ember-components/commit/30b3257227dab623c86dedfab032b85f32414e42)
[GitHub Actions でテストできるようにしたり](https://github.com/mugijiru/ember-components/commit/34e81e2905e32dd2878b95fb9d5c7eb3b3a0b463)
ちょっと細かい修正をしたりしている。

ここまでの差分は
<https://github.com/mugijiru/ember-components/compare/bbaf38aa0f6c99ebbc7e0cb7ee5ac2c201706bc6...34e81e2905e32dd2878b95fb9d5c7eb3b3a0b463>
で確認可能。


### GitHub Packages の NPM Package の公開 {#github-packages-の-npm-package-の公開}

まず [パッケージを公開する](https://docs.github.com/ja/packages/guides/configuring-npm-for-use-with-github-packages#publishing-a-package) に従って以下の変更をしている。

パッケージ名を `@mugijiru/ember-components` にしたり、

```json
"name": "@mugijiru/ember-components",
```

publishConfig の registry に GitHub Packages の URL を入れることでそこで公開できるようにしている。

```json
"publishConfig": {
  "access": "restricted",
  "registry": "https://npm.pkg.github.com"
},
```

access は GitHub 側の記載は何もないが
<https://tech.plaid.co.jp/npm-private-registry-to-github-packages-registry/>
を参考にして restricted にすることで、許可された人だけが使えるようにしている。

今は公開リポジトリにしているから public でもいい気もするけど、実装当時はより業務でやりそうな雰囲気にしたかったので、非公開リポジトリかつ限定的な公開で進めていたので、このようになっている。

さらに、今後 GitHub Packages に複数パッケージ公開するかもしれないので
[同じリポジトリへの複数パッケージの公開](https://docs.github.com/ja/packages/guides/configuring-npm-for-use-with-github-packages#publishing-multiple-packages-to-the-same-repository) に従って registory を指定したりしている

```json
"repository": "git://github.com/mugijiru/ember-components.git",
```

その上で
<https://github.com/mugijiru/ember-components/blob/main/.github/workflows/release.yml>
のようなワークフローを用意すると
Tag を打って push して
GitHub 上でそのタグを使って Release を作成すると
NPM Package として公開されるようになっている。

上にも出した <https://tech.plaid.co.jp/npm-private-registry-to-github-packages-registry/> を真似するともっとスマートな感じになりそうだけど、一旦これでいいやってなってる。


### 公開したパッケージを利用する {#公開したパッケージを利用する}

<https://github.com/mugijiru/ember-rails-todo-app/pull/48> の PR でやったこと。

元々は `RAILS_ROOT/ember/my-components` に置いていたやつを NPM Package にしているので
my-components 関連のやつをさっくり消してあげている。

具体的には `ember/my-components` は全部消して
package.json の devDependencies に入れていた
`"my-components": "link:../my-components"` を削除している。

今思ったけどこれ devDependencies だと多分 production 環境だと動かなかったな。まあ 2.18 なので公開する気がゼロだったからすっかり気付かなかったんだけど。

まあそれは置いといて公開したパッケージを入れるため dependencies に以下のように記述する。

```json
"dependencies": {
  "@mugijiru/ember-components": "^0.0.1"
},
```

あとはプライベートなパッケージを入れられるように
`RAILS_ROOT/ember/todo-app/.npmrc` に以下のような設定を入れている。

```text
@mugijiru:registry=https://npm.pkg.github.com
```

この設定は [パッケージをインストールする](https://docs.github.com/ja/packages/guides/configuring-npm-for-use-with-github-packages#installing-a-package) の通りだとなんかうまく動かなかったので
[他のOrganizationからのパッケージのインストール](https://docs.github.com/ja/packages/guides/configuring-npm-for-use-with-github-packages#installing-packages-from-other-organizations) のやり方を採用している。あとでまた検証した方がいいかもなあ。。。

それと [GitHub Packages への認証を行う](https://docs.github.com/ja/packages/guides/configuring-npm-for-use-with-github-packages) に従って

```text
//npm.pkg.github.com/:_authToken=${NPM_TOKEN}
```

としている。
NPM\_TOKEN には GitHub のパーソナルアクセストークンが入るので環境変数にしている。

なので GitHub Actions で CI を回す際のパッケージのインストール時に

```yaml
env:
  NPM_TOKEN: ${{ secrets.NPM_AUTH_TOKEN }}
```

みたいに環境変数に PAT を入れてあげる必要あり。

他には、これまた公開したパッケージを使う上で本質的ではないんだけど、移植した際に `my-button` から `mg-button` みたいに全部
`my-` prefix だったのを `mg-` prefix にしているので利用箇所でそれらの修正の必要あり。命名を適当にやってたのでここでそれが仇になってる。つらい。

以上で GitHub Packages に NPM として公開した Ember.js の Addon を
ember-cli-rails で使えるようになりますよっと。正直 NPM とかに慣れてる人ならさっくりできそうな内容。。。

まあ Ember.js の Addon も実際は NPM Package なので普通に NPM Package として公開するだけで使えたりするってだけですね。
.ember-cli-build.js を活用したらまたちょっと話は違うはずだけど今回のはそこまでのやつじゃないし……。
