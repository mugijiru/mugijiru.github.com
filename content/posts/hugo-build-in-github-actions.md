+++
title = "Hugo の build を GitHub Actions でやることにした"
date = 2021-05-29T23:31:00+09:00
categories = ["Hugo", "GitHub"]
draft = false
+++

このサイトを構築するにあたりいつも手元で Hugo を build して push していたけどもまあだるいし、そろそろ GitHub Actions にも慣れて来たし探したら既に Action が提供されていたのでそれを使って build を自動化しました。

「しました」って書いているけど実は今書いてるこの記事自体がその実験用の記事であり、これを書いている時点ではまだ自動化検証中。なので記事に乗せられるものはほとんどない。

一応 <https://github.com/mugijiru/mugijiru.github.com/tree/master/.github/workflows/build.yml>
のあたりにそれ用の workflow が入って来る予定。

使ってるのは [hugo-setup](https://github.com/marketplace/actions/hugo-setup) というやつ。
`run: hugo` するだけで build してくれるようで便利そう。
minify は差分的に微妙な感じするのでとりあえず使わないことにした

成功可否はいつものように Slack 通知するし
push して放置したら良いのは良さそう。

ま、これで build しないで済むようになったらちょっと楽になりそうで楽しみ。
