+++
title = "Emacs のロックファイルと Webpack dev server の設定"
date = 2021-09-16T10:22:00+09:00
categories = ["Emacs"]
draft = false
+++

また同じ罠を踏んだ時に同じ対応ができるようにということでメモ。

プライベートで vue-cli とかを使って
`npm run serve` とかしている時にファイルを変更する度に

```text
[Error: ENOENT: no such file or directory, stat '/path/to/src/components/.#HelloWorld.vue']
```

とか怒られて辛かった。
Emacs のロックファイルである。

ロックファイルを生成しないというのもアリかもしれないが正常に保存されれば自動で消えるファイルだしそこに手を入れるよりは、監視対象ファイルを制限した方が良さそう、と判断して調べた。

`npm run serve` では裏側で webpack-dev-server が動いているようなのでそこで watch 対象からロックファイルを外すべく
`vue.config.js` を以下のように設定した。

```js
module.exports = {
  configureWebpack: {
    devServer: {
      watchOptions: {
        ignored: ['node_modules', 'public', '**/.#*'],
      }
    }
  }
}
```

肝は `'**/.#*'` である。
Emacs のロックファイルは `.#` の後にオリジナルのファイル名がくっついてくる形なのでそいつを全部無視する、というだけ。

これでファイルを変更しても怒られなくなってハッピー。

ちなみに同じ問題が React の開発の時にも発生するので多分同じような方法で直せる。まだ試してないけど。
