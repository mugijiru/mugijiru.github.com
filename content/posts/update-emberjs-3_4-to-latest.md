+++
title = "Ember.js@3.4 から最新の 3.26 に上げた"
date = 2021-05-09T14:35:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

いつも Ember.js ネタを書く時に使ってる
<https://github.com/mugijiru/ember-rails-todo-app>
のリポジトリですが、ゴールデンウィークで Ember.js の最新版への対応を完了させました。

そこへの対応のために <https://github.com/mugijiru/ember-components> の addon の方も
2.18 から最新化することになりました。


## 対応の方針 {#対応の方針}

どう対応させていったかというと、
[ember-cli-rails の Ember.js を 2.18 から 3.4 にアップデート]({{< relref "update-emberjs-2.18to3.4" >}})
の記事でも書いた

> 3系で LTS であったバージョンを順番に適用していく方針

を実際にやってみたって感じ。


## 実際の対応 {#実際の対応}


### eslint 対応 {#eslint-対応}

3.4 に上げた後に、eslint で怒られてるのに対応できそうだなとなったので
3.8 に上げる前に修正をした

<https://github.com/mugijiru/ember-rails-todo-app/pull/66>

大きな変更点は
jQuery を使って要素を取得していたところを純粋な JS に書き換えたところぐらい。


### 3.4 → 3.8 {#3-dot-4-3-dot-8}

<https://github.com/mugijiru/ember-rails-todo-app/pull/67>

1.  `ember-cli-update --to 3.8`
2.  `yarn`
3.  `ember-cli-update --codemods`

を叩いたぐらい。codemods では特がないので難しいところは何もない。


### 3.8 → 3.12 {#3-dot-8-3-dot-12}

<https://github.com/mugijiru/ember-rails-todo-app/pull/68>

3.8 に上げるよりはちょっと面倒だった。

が、基本的には

1.  `ember-cli-update --to 3.12`
2.  `yarn`
3.  `ember-cli-update --codemods`

を叩いてるだけである。

とりあえず [codemods で結構変更があった](https://github.com/mugijiru/ember-rails-todo-app/pull/68/commits/cd7e96493dc9f52c67ee499801144c25b54a4d36) ので、それを軽く話すと

-   handlebars で AngleBraket を使うようになった
    -   3.4 からサポートされ始めてるけど 3.10 でちゃんと使えるようになったっぽい
-   handlebars でコンポーネントに値を渡す時に `@hoge={{value}}` みたいに `@` をつけるようになった
    -   これで component の変数か、単なる HTML の属性値かの区別がつくようになったっぽい
-   model が `ember-data` ではなく `@ember-data/model` を import するようになった
    -   この頃から個別機能を import させる方針になり始めてるっぽい

という感じ。モダンっぽいし、こっちの方が好みの書き方ですね。

あとは Observer を使ってるところが eslint で怒られていたけど直すのが大変そうだったので eslint の方を無視するようにしちゃった。まあ、これよりずっとあとの手順で直してるんだけども。


### 3.12 → 3.16 {#3-dot-12-3-dot-16}

Octane が入って来てるからか、
ember-cli-update での差分も大きく、動くようにするまでにいくつかやることがあった。

まずはいつものように

1.  `ember-cli-update --to 3.16`
2.  `yarn`
3.  `ember-cli-update --codemods`

を実行。で、[ember-cli-update したところで](https://github.com/mugijiru/ember-rails-todo-app/pull/69/commits/da37e4342c120a4c73a88f7b72ee7344d07eb3e6)
app.js が書き換えられて自分の設定が消えたのを直すハメになったり同じく config/environment.js が書き換えられてそれも直すハメになったりしてた。新しい文法にするのはいいけど設定を吹っ飛ばすのはやめてほしい。

あと config/optional-features.json で jquery-integration が false にされたのもいただけなかった。
[ember-cli-rails-addon-csrf](https://github.com/rondale-sc/ember-cli-rails-addon/blob/master/app/initializers/ember-cli-rails-addon-csrf.js) 的に必要なんだよぉ。

あとは [Ember.js Octane vs Classic Cheat Sheet](https://ember-learn.github.io/ember-octane-vs-classic-cheat-sheet/) を参考に書き換えた。

-   [hbs の書き方がさらに変わったので修正](https://github.com/mugijiru/ember-rails-todo-app/pull/69/commits/5a2d971f18fa3da9d3f45c344666e7c134b2bf4a)
    -   親から貰った受け取ったプロパティは @ を prefix とするように変更
    -   自身の持つプロパティは this. を prefix とするように変更
    -   `@click=` で定義していた click イベントは on を使うように変更
-   [jQuery に頼ってた部分を素の JS に書き換え](https://github.com/mugijiru/ember-rails-todo-app/pull/69/commits/ed3e7e3c1fdac5a88277fafa6c5ce42a603ff9cb)
-   [Component の action の書き方などの変更](https://github.com/mugijiru/ember-rails-todo-app/pull/69/commits/0f4113e6a918dbfd5f75500370fd2e00486d1cec)
    -   actions で囲むのではなく `@action` というデコレータをメソッドにつける方法になった
    -   form の button を叩いた時に submit されるようになってしまったので preventDefault で送信されないようにした
-   [Controller の action などの書き方などを変更](https://github.com/mugijiru/ember-rails-todo-app/pull/69/commits/3362321970ed616212ce1250fec3555eba434bd4)
    -   actions で囲むのではなく `@action` というデコレータをメソッドにつける方法になった
    -   こっちでも jQuery に頼ってたのを直した

ということをやっている。

設定が変わってるのをちゃんと戻す作業とか
jquery-integration の問題とか書き方が色々変わったりしているので、なかなか苦労した。
hbs は結構変わってるしね……。

まあでも大体そんな感じのことをしたらなんとかなる。


### 3.16 → 3.20 {#3-dot-16-3-dot-20}

<https://github.com/mugijiru/ember-rails-todo-app/pull/71>

これは

1.  `ember-cli-update --to 3.16`
2.  `yarn`
3.  `ember-cli-update --codemods`

だけで済んでるのでちょー楽だった


### 2.18 → 3.4 for @mugijiru/ember-components {#2-dot-18-3-dot-4-for-mugijiru-ember-components}

ここで突然別の流れをぶち込むハメに。

というのも ember-todo-rails-app の Ember.js を 3.24 に上げようとしたらその中で使ってるコンポーネントである @mugijiru/ember-components の方を更新しないと上げられない状態になってしまったから。

多分 Classic な書き方がダメなんだろうなという推測で、こっちも Octane 対応をしないといけないな、という判断になった。

で、こっちも段階的に上げていくわけですが、ひとまず 3.4 にするにあたり
ember-rails サポートも切っておく方が楽なので

-   <https://github.com/mugijiru/ember-components/pull/5>
-   <https://github.com/mugijiru/ember-components/pull/6>

で 2.18 のままだけど Gem のサポートをやめて module を使う仕組みに書き換えている。
ember-rails でなければ古い書き方をする必要はないのだ。

そしてさらに <https://github.com/mugijiru/ember-components/pull/7> で 3.4 に上げている。

コード変更で面倒だったのは

-   [codemods 適用](https://github.com/mugijiru/ember-components/pull/7/commits/7bf7f7c7905af7a671d84b25991b9ceb68800048)
    -   テストの書き方が多少変わってるのでそっちを覚えないといけない
-   [test 用に読み込むパッケージ名の修正](https://github.com/mugijiru/ember-components/pull/7/commits/b70537441d63aa6f9efafa03c407ef30973461c5)
    -   package.json でパッケージ名が修正されたのに伴う変更。まあこうあるべきって感じ。
-   [テストでのレンダリングの仕組みが変わったようなので対応](https://github.com/mugijiru/ember-components/pull/7/commits/a6505609e2ccf2c8d17ee703cc1aa7cff2847aea)
    -   どうもレンダリングで Wrapper になる div が入るようになったっぽいので雑に div で取ってたのが動かなくなった

あたりかな。パッケージ名の修正と div のやつはどっちも気付くのに時間がかかってしまったやつ。つらかった。


### 3.4 → 3.8 for @mugijiru/ember-components {#3-dot-4-3-dot-8-for-mugijiru-ember-components}

<https://github.com/mugijiru/ember-components/pull/8>

まあ難しいことをしなくても普通に上がったやつですね。はい。なので詳細はいいや。


### 3.8 → 3.12 for @mugijiru/ember-components {#3-dot-8-3-dot-12-for-mugijiru-ember-components}

<https://github.com/mugijiru/ember-components/pull/9>

これもあっさり上がったので特筆することなし


### 3.12 → 3.16 for @mugijiru/ember-components {#3-dot-12-3-dot-16-for-mugijiru-ember-components}

-   <https://github.com/mugijiru/ember-components/pull/10>
    -   3.16 に上げたのはこっち
-   <https://github.com/mugijiru/ember-components/pull/12>
    -   3.16 に上げただけだと修正が足らなかったので追加修正したやつ

いつもの手順はもういいとして、特別にやったことは

-   [hbs の書き方を新しい形式に合わせた](https://github.com/mugijiru/ember-components/pull/10/commits/8bc5e01b97e1740601a4710496d28d6840ce6e4d)
    -   this をつけただけだけど
-   Component の書き方修正([mg-button](https://github.com/mugijiru/ember-components/pull/10/commits/654619191256c8a778440007fab2c9f16185f247), [mg-checkbox](https://github.com/mugijiru/ember-components/pull/10/commits/9b4beeec63023fecb43b5208a1f324030916e4b0), [mg-toggle-switch](https://github.com/mugijiru/ember-components/pull/10/commits/7c2b92a8a1634b0a1d4fa1d8dc3fd02732cf9525))
    -   addon/templates/components/\*.hbs から addon/components/\*.hbs に移動
        -   どうもいつの間にか templates/components に置かなくて良くなったっぽい
            -   さらに新しい [pods layout](https://cli.emberjs.com/release/advanced-use/project-layouts/#podslayout) というのもあるけど addon の場合は互換性維持のために classic 推奨。
    -   import 元を `@glimmer/component` にして Native Class を用いた記述に変更
        -   これまで `export default Component.extend` していたのが `export default class Hoge extends Components` という書き方になった
        -   Native Class になったのでプロパティも普通に `hoge = 'fuga'`" みたいに書くようになった
    -   className, classNameBindings は使えなくなったので調整
    -   親から渡って来るパラメータの初期値調整のための記法の使用
        -   `get hoge () { return this.args.hoge ?? '' }` のようにしてデフォルト空文字列にするなど
            -   親から渡って来るパラメータは `this.args` のように隔離された場所に入るようになってる。便利。
-   [Click が実行されない問題の修正](https://github.com/mugijiru/ember-components/pull/12)
    -   onClick というパラメータで渡って来たやつをクリック時に実行するように調整している


### 新しい @mugijiru/ember-components に更新 {#新しい-mugijiru-ember-components-に更新}

<https://github.com/mugijiru/ember-rails-todo-app/pull/72>

MgButton とかに `{{on "click" this.save}}` とかでアクションを渡せていたのがここからは `@onClick={{this.save}}` のように渡さないといけない。なぜなら @mugijiru/ember-components を Octane 対応する際にそういう風に仕様が変わったからだ。

<https://guides.emberjs.com/release/in-depth-topics/patterns-for-actions/>
を見た感じ、そうするのが正しいっぽい。ただの button には `{{on "click" this.save}}` というように書くことになるので混乱がありそうでだるいけど。


### 3.20 → 3.24 {#3-dot-20-3-dot-24}

@mugijiru/ember-components の Ember.js を 3.16 にして Octane 対応を済ませることで
ember-rails-todo-app の Ember.js を 3.24 に上げられるようになった。

<https://github.com/mugijiru/ember-rails-todo-app/pull/73>

ここではあんまり対したことはしてないけど
[active\_model\_serializer を使うことを明示的に書かないといけなくなった](https://github.com/mugijiru/ember-rails-todo-app/pull/73/commits/9268c5414c2c3052c2537fa1e2dacc0b4ee55886)
というのが一番だるいポイントかな。これも原因判明までに時間食ったやつ……。


### 3.24 → 3.26(最新) {#3-dot-24-3-dot-26--最新}

-   [ember-bootstrap を 4系に上げた](https://github.com/mugijiru/ember-rails-todo-app/pull/74)
-   [Ember.js を最新化し、文法も最新に合わせて書き直した](https://github.com/mugijiru/ember-rails-todo-app/pull/76)

あたりで、最新化できた。
ember-bootstrap は上げられるから上げておいただけだけども。

-   native class としてクラス定義するように変更
-   @tracked, @sevice, @action, @attr などのデコレータを使うように変更
-   computed property は tracked を使う書き方に置き換え
-   set, get を使った書き方も古いので `this.get('hoge')` などを `this.hoge` みたいに書き換え

あたりのことをしている。結構変更は多いけど、まあ、基本的に lint のいうことに従って対応しただけである。

ひとつ initializer で current-user を無理やり inject している処理が
Native Class に置き換えできなかったので、ここはどこかでなんとかしないとまずそう。多分 Ember.js@4 に追従できなくなる
<https://github.com/mugijiru/ember-rails-todo-app/pull/76/commits/f6b6ed65863a9e94ed1b41154387e87e4f0da1bf>

User class を用意して instanceInitializer でそこに突っ込んでやるとか、
Service class を用意したりしたら、なんとかなりそうな気はしている。


### 3.16 → 3.20 for @mugijiru/ember-components {#3-dot-16-3-dot-20-for-mugijiru-ember-components}

アプリ本体だけでなく addon も最新化対象である。
<https://github.com/mugijiru/ember-components/pull/13>

3.20 に上げるのは何も問題なし。
3.16 に上げる時に苦労したからねえ。


### 3.20 → 3.24 for @mugijiru/ember-components {#3-dot-20-3-dot-24-for-mugijiru-ember-components}

<https://github.com/mugijiru/ember-components/pull/15>

3.24 に上げるのも特に苦労なし。


### 3.24 → 3.26(最新) for @mugijiru/ember-components {#3-dot-24-3-dot-26--最新--for-mugijiru-ember-components}

<https://github.com/mugijiru/ember-components/pull/16>

基本的には難しいことなし。ただし Interactive な要素じゃないのに click させるなって eslint に怒られてたから
MgChecobox や MgToggleSwitch を div から button に変更した。
<https://github.com/mugijiru/ember-components/pull/16/commits/d9f14499f92584479fe7154879003bfd5f237f03>

`input type="checkbox"` が良い気もするけど、ま、一旦いいや。


### 最新の @mugijiru/ember-components 取り込み {#最新の-mugijiru-ember-components-取り込み}

<https://github.com/mugijiru/ember-rails-todo-app/pull/77>

最新化する際に、
div から button に書き換えた影響で style 崩れがあるので調整している。

@mugijiru/ember-components 用の css は addon 側に置いても良いよなあと思ってるけど今回はそこは放置している。めんどくさくて。


## 最後に {#最後に}

ゴールデンウィークをこれで結構消費してしまったのでちょっと残念な気持ちになっている。

けどまあ、どこで引っ掛かりやすいかは結構洗い出せたので良い。

3.8 → 3.12
: codemods で結構変更される

3.20 → 3.24
: addon が 2.18 で使えるような古い書き方だと使えなくなる

Octane 対応
: 書き方が色々変わる。古い書き方も一部まだサポートされてそうだけど。

とりあえず Octane が鬼門かねえ。
