+++
title = "ember-cli-rails の Ember.js を 2.18 から 3.4 にアップデート"
date = 2021-04-04T15:38:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

前回までで ember-rails と ember-cli-rails の共存周りを一通り済ませてそのあたりは大体満足したので次の段階である Ember.js アプリの最新化を進めていくぞい。

で、どう進めていくかというと
2.18 は最新版からはかなり遠いので
3系で LTS であったバージョンを順番に適用していく方針。

それ以外のところだと基本的に
<https://cli.emberjs.com/release/basic-use/upgrading/>
に従って対応をしていく。

というのを実践した PR がこちらになります。
<https://github.com/mugijiru/ember-rails-todo-app/pull/61>


## ember-cli の更新 {#ember-cli-の更新}

<https://github.com/mugijiru/ember-rails-todo-app/pull/61/commits/0b71b86330ab46ec8df46cdcb308daf3ed766681>
のコミットでやっていることですね。

まずは Dockerfile で入れている ember-cli を 2.18 から 3.4 にする。
3.4 系の最終バージョンは 3.4.4 なのでそれを指定している。

```Dockerfile
# install ember-cli
RUN yarn global add ember-cli@3.4.4
```


## ember-cli-update の導入 {#ember-cli-update-の導入}

<https://github.com/mugijiru/ember-rails-todo-app/pull/61/commits/ded293ff2f686081549d0019e500facb5c2aaa3d>
のコミットでやってることですね。

Ember.js をアップデートする際には ember-cli-update を使うのが王道っぽいのでそれも Dockerfile でインストールしておく。

```Dockerfile
# install ember-cli-update
RUN yarn global add ember-cli-update
```

また、こいつは今後も 3 系で更新していくにあたり必要と思われるので
ember-cli よりも先に入れておくことにする。


## bundle && yarn {#bundle-and-and-yarn}

Dockerfile を更新したので bundle install と yarn install を実行しておく。

```text
$ docker-compose run rails bundle
```

```text
$ docker-compose run rails bash -c "export NPM_TOKEN=XXXXXXXXXX && cd ember/todo-app && yarn"
```

NPM\_TOKEN という環境変数を使ってるのは
ember-components という自作の NPM パッケージを使うために
[.npmrc](https://github.com/mugijiru/ember-rails-todo-app/blob/7916518d766145fc0b8d9978efbfb08d6937f813/ember/todo-app/.npmrc) で GitHub Packages を使うような設定をしているため。


## ember-cli-update bootstrap の実行 {#ember-cli-update-bootstrap-の実行}

<https://github.com/ember-cli/ember-cli-update/wiki/Getting-Started> に書かれているように

```text
$ docker-compose run rails bash -c "export NPM_TOKEN=XXXXXXXXXX && cd ember/todo-app && ember-cli-update bootstrap"
```

を実行することで config/ember-cli-update.json が生成される。
ember-cli-update は実行時にこのファイルを見て色々処理をする様子。

雰囲気的には Addon もこれを使って更新できそうだが、ちょっとまだ調べてない。

とりあえずこの実行結果をコミットしたのが以下。
<https://github.com/mugijiru/ember-rails-todo-app/pull/61/commits/35177e82eac6a9d490c49348ec8e50b31828bb10>


## ember-cli-update で 3.4.4 に更新 {#ember-cli-update-で-3-dot-4-dot-4-に更新}

いよいよアップデート作業である。とりあえず 3.4.4 に上げたいので以下のコマンドを叩く。

```text
% docker-compose run rails bash -c "export NPM_TOKEN=XXXXXXXXXX && cd ember/todo-app && ember-cli-update --to 3.4.4"
```

すると

```text
? Blueprint updates have been found. Which one would you like to update?
> app, current: 2.18.2, latest: 3.25.3
```

というように質問される。恐らく Addon も ember-cli-update で管理できるようにしていたら他の選択肢も出て来るんだろうが、とりあえず今回は app を更新したいだけなので何も考えずに Enter を叩く。

すると、以下のように何やらファイルが生成されたようなログが出て来る。しかも2回も生成されてる雰囲気。

```text
installing app
  create .editorconfig
  create .ember-cli
  create .eslintrc.js
  create .travis.yml
  create .watchmanconfig
  create README.md
  create app/app.js
  create app/components/.gitkeep
  create app/controllers/.gitkeep
  create app/helpers/.gitkeep
  create app/index.html
  create app/models/.gitkeep
  create app/resolver.js
  create app/router.js
  create app/routes/.gitkeep
  create app/styles/app.css
  create app/templates/application.hbs
  create app/templates/components/.gitkeep
  create config/environment.js
  create config/targets.js
  create ember-cli-build.js
  create .gitignore
  create package.json
  create public/robots.txt
  create testem.js
  create tests/helpers/destroy-app.js
  create tests/helpers/module-for-acceptance.js
  create tests/helpers/start-app.js
  create tests/index.html
  create tests/integration/.gitkeep
  create tests/test-helper.js
  create tests/unit/.gitkeep
  create vendor/.gitkeep
WARNING:
WARNING: The 'package.json' file for the addon at /usr/local/share/.config/yarn/global/node_modules/ember-cli/lib/tasks/server/middleware/tests-server
WARNING:   specifies a missing dependency 'exists-sync'
WARNING: Node v14.16.0 is not tested against Ember CLI on your platform. We recommend that you use the most-recent "Active LTS" version of Node.js. See https://git.io/v7S5n for details.
installing app
  create .editorconfig
  create .ember-cli
  create .eslintignore
  create .eslintrc.js
  create .template-lintrc.js
  create .travis.yml
  create .watchmanconfig
  create README.md
  create app/app.js
  create app/components/.gitkeep
  create app/controllers/.gitkeep
  create app/helpers/.gitkeep
  create app/index.html
  create app/models/.gitkeep
  create app/resolver.js
  create app/router.js
  create app/routes/.gitkeep
  create app/styles/app.css
  create app/templates/application.hbs
  create app/templates/components/.gitkeep
  create config/environment.js
  create config/optional-features.json
  create config/targets.js
  create ember-cli-build.js
  create .gitignore
  create package.json
  create public/robots.txt
  create testem.js
  create tests/helpers/.gitkeep
  create tests/index.html
  create tests/integration/.gitkeep
  create tests/test-helper.js
  create tests/unit/.gitkeep
  create vendor/.gitkeep
```

で、実行後に Ember アプリのディレクトリで `git status` を叩くと以下のような感じ。

```text
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   .eslintignore
        modified:   .eslintrc.js
        modified:   .gitignore
        new file:   .template-lintrc.js
        modified:   README.md
        modified:   config/ember-cli-update.json
        new file:   config/optional-features.json
        modified:   config/targets.js
        modified:   package.json
        modified:   testem.js
        new file:   tests/helpers/.gitkeep
        deleted:    tests/helpers/destroy-app.js
        deleted:    tests/helpers/module-for-acceptance.js
        deleted:    tests/helpers/start-app.js

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add/rm <file>..." as appropriate to mark resolution)
        deleted by us:   .travis.yml
```

.travis.yml は「どうせ使わねーだろ」ってことで自分で消してあるので改めて `git rm .travis.yml` すれば良い。

それ以外の変更点も、ざっと眺めた感じは、きっといい感じに 3 系に対応してくれてそうなので気にせずコミットする。適当である。

この実行結果のコミットは
<https://github.com/mugijiru/ember-rails-todo-app/pull/61/commits/c78deca904a43aa5587cf1489cabd90461469c31>
ですね。


## 古いコードを自動置き換え {#古いコードを自動置き換え}

<https://cli.emberjs.com/release/basic-use/upgrading/#updatingyourcodeautomatically>

に書かれてるように `ember-cli-update` では
`--run-codemods` オプションで実行することで古い記述を自動的に新しい書き方に直してくれるという便利機能があるので、それを実行する。後で理由を記載するが、ここでもまた NPM\_TOKEN が必要になる。

```text
$ docker-compose run rails bash -c "export NPM_TOKEN=XXXXXXXXXX && cd ember/todo-app && ember-cli-update --run-codemods"
```

するとまた

```text
? Which blueprint would you like to run codemods for?
> ember-cli
```

というように1つしかない選択肢を出される。これもまた Addon を ember-cli-update で更新管理できるようにしていたら選択肢が増えるんだろう、という推測をしてそのまま Enter を叩く。

すると、今度は以下のようにいくつかの選択肢が出て来る。

```text
? These codemods apply to your project. Select which ones to run. (Press <space> to select, <a> to toggle all, <i> to invert selection)
❯◯ ember-modules-codemod
 ◯ ember-qunit-codemod
 ◯ ember-test-helpers-codemod
 ◯ es5-getter-ember-codemod
 ◯ notify-property-change
 ◯ qunit-dom-codemod
```

それぞれ何をしてくれるかというと、多分大体以下の感じ。

[ember-modules-codemod](https://github.com/ember-codemods/ember-modules-codemod)
: `import Ember from 'ember'` という古い記述を `import Component from '@ember/component'` とかに修正するやつ

[es5-getter-ember-codemod](https://github.com/ember-codemods/es5-getter-ember-codemod)
: `obj.get('foo')` みたいな古い記述を `obj.foo` みたいな記述方式に変更するやつ

[ember-test-helpers-codemod](https://github.com/ember-codemods/ember-test-helpers-codemod)
: テスト用の記述を新しい書き方に変更するやつ

[ember-qunit-codemod](https://github.com/ember-codemods/ember-qunit-codemod)
: ember-qunit の moduleFor とかの書き方を新しい方式に変更するやつ

[notify-property-change](https://github.com/ember-codemods/ember-3x-codemods/tree/master/transforms/notify-property-change)
: notifyPropertyChange の書き方が変わる。使ったことないからよくわからん。

[qunit-dom-codemod](https://github.com/simplabs/qunit-dom-codemod)
: DOM 選択の記述を jQuery 依存じゃないようにするっぽい

例えばここで
ember-modules-codemod だけ選択して Enter すると何かよくわからないが NPM Package を Fetch しにいき、それが終わるとコードの自動補正が実行される。

```text
Running codemod ember-modules-codemod
Running command 1 of 1
Skipping path addon which does not exist.
Skipping path addon-test-support which does not exist.
Skipping path test-support which does not exist.
Skipping path lib which does not exist.
Processing 11 files...
Spawning 7 workers...
Sending 2 files to free worker...
Sending 2 files to free worker...
Sending 2 files to free worker...
Sending 2 files to free worker...
Sending 2 files to free worker...
Sending 1 files to free worker...
All done.
Results:
0 errors
6 unmodified
0 skipped
5 ok
Time elapsed: 1.785seconds

Done! All uses of the Ember global have been updated.
Finished running command 1 of 1
Finished running codemod ember-modules-codemod
```

これで何が変更されているかというと
`git diff --cached` の一部を表示するとこんな感じ。

```text
diff --git a/ember/todo-app/app/components/todo-item.js b/ember/todo-app/app/components/todo-item.js
index bc28c83..a803fc5 100644
--- a/ember/todo-app/app/components/todo-item.js
+++ b/ember/todo-app/app/components/todo-item.js
@@ -1,12 +1,14 @@
-import Ember from 'ember';
+import { later } from '@ember/runloop';
+import { computed } from '@ember/object';
+import Component from '@ember/component';

-export default Ember.Component.extend({
+export default Component.extend({
   tagName: 'li',
   classNames: ['p-todo-item'],
   classNameBindings: ['isCompleted:p-todo-item__completed'],

   item: null,
-  isCompleted: Ember.computed('item.isCompleted', function () {
+  isCompleted: computed('item.isCompleted', function () {
     return this.get('item.isCompleted');
   }),
```

`import Ember from 'ember'` という記述はもう古いので個別に `import Component from '@ember/component'` とするような記述に変更されている感じ。

いい感じにコードを変更してくれることがわかったので、同じ調子で ember-qunit-codemods なども適用していく。

すると `ember-test-helpers-codemod` でエラーになったりするけどそもそもこのプロジェクトでは Ember.js に対する qunit でのテストをまだ書いてないので多分それが原因で単にファイルがないだけとかなので軽く無視する。

という感じで実際に適用して変更があったのが

-   <https://github.com/mugijiru/ember-rails-todo-app/pull/61/commits/815ccd8c6d79f6e6bc6171f214f8cd375e1a6537>
-   <https://github.com/mugijiru/ember-rails-todo-app/pull/61/commits/d21b563c6352718c702f4ccd883cd780973d9982>

の2つだけ。ま、複雑なことしてないしね。


## packages の更新 {#packages-の更新}

[ember-cli-update で 3.4.4 に更新](#ember-cli-update-で-3-dot-4-dot-4-に更新) の方でやっておけば良かったんだけど、ここまでの作業で package.json は更新されてるけど実際にインストールされてるライブラリの更新はされてなかったorz

というわけで Ember.js アプリのディレクトリで `yarn` を叩いたら色々新しくインストールされて
<https://github.com/mugijiru/ember-rails-todo-app/pull/61/commits/196d9f7b389c5e2c8d690cd1e608251c69a377d2>
みたいな感じで yarn.lock も更新されると。


## テストの実施 {#テストの実施}

まあ後はちゃんと動くよねということを確認するためにテストを実行して問題なければ OK ですと。

このプロジェクトだと system spec を書いているのでそれを実行した上で、念の為手でも動作確認して問題なかった、という感じ。

もっと複雑なケースだと色々問題あるんだろうな〜。


## 問題があった場合 {#問題があった場合}

もし問題があったら、エラー内容などを確認しつつ

-   [Ember Deprecations](https://deprecations.emberjs.com/)
-   [Ember release blog post](https://blog.emberjs.com/tag/releases/)

と睨めっこしたら良いんだと思う。今回問題がなかったから、そのあたりの知見は得られなかったけど……。


## その他 {#その他}

実は eslint で怒られてるのはまだ無視しています。そこまで修正入れると面倒なのと、差分が大きくなるなと思って。それは別の機会に直しておきます。


## 最後に {#最後に}

とりあえず3系にするだけならそんなに難しくなさそうな所感を得た。

eslint で怒られてるの直す必要があるな〜というのと、
ember-components@0.0.3 がいつから使えなくなるか気になるのと
ember-bootstrap あたりの Addon を ember-cli-update で管理できるようにしたいなという気持ちは残ったけど。

ま、そこもおいおい試していく
