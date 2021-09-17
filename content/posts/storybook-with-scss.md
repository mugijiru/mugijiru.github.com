+++
title = "Storybook の SCSS 対応でハマった"
date = 2021-09-17T12:07:00+09:00
categories = ["Frontend"]
draft = false
+++

この記事は、2021年9月16日に起きたことなので、多分賞味期限は凄く短かい内容だけど、自分用にメモとして書いています。

それは置いといて、汎用的なコンポーネントライブラリみたいなのを作ってみたくなったので趣味で Storybook を動かしてみている。

なのでそこで流行りっぽい Storybook で作ってみているのだが素の CSS だと面倒なので SCSS を採用することにした。

SCSS なのは Brace は欲しかったのと、ライブラリの更新が頻繁なので、更新間隔が空きがちな Stylus より良さそうと思ったから。

というわけで `@storybook/html` の `6.3.8` を入れてるわけだけどそこに scss が使えるようにということで

```text
npm install --save-dev css-loader sass-loader style-loader
```

とかしたら

```text
npm ERR! code ERESOLVE
npm ERR! ERESOLVE could not resolve
```

などと言われてしまう。

適当に `--legacy-peer-deps` で突っ込んでもで突っ込んでも、結局は起動時にエラーになってしまう。

で、面倒なら真面目に調べると、
Storybook の内部で使われてる Webpack が 4 系で、最新の sass-loader, css-loader, style-loader だと Webpack 5 でしか動かないのでそこで依存関係が解決できないってことがわかった。
npm の依存関係のエラー読みにくいでござる。

というわけで Webpack4 のサポートをしていたバージョンである

-   style-loader@2
-   css-loader@5
-   sass-loader@10

を入れることで解決した。

Storybook って Issue が 1300 件ぐらいそのままになってるし、
Webpack5 対応にもなってなさそうなので、本当に使って大丈夫なのかは心配……。
