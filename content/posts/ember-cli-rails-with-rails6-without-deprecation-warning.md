+++
title = "ember-cli-rails アプリで Rails6 の Deprecation Warning が出ないようにした"
date = 2021-08-22T00:50:00+09:00
categories = ["Rails", "Ember-js"]
draft = false
+++

ember-cli-rails なアプリを Rails6 対応にしてみた。が、リリースされている Gem をそのまま使うと

> DEPRECATION WARNING: Initialization autoloaded the constants ActionText::ContentHelper, ActionText::TagHelper, and EmberCliRailsAssetsHelper.
>
> Being able to do this is deprecated. Autoloading during initialization is going
> to be an error condition in future versions of Rails.
>
> Reloading does not reboot the application, and therefore code executed during
> initialization does not run again. So, if you reload ActionText::ContentHelper, for example,
> the expected changes won't be reflected in that stale Module object.
>
> These autoloaded constants have been unloaded.
>
> Please, check the "Autoloading and Reloading Constants" guide for solutions.
> (called from <top (required)> at /app/config/environment.rb:5)

という Deprecation Warning が出てつらかった。


## 原因 {#原因}

[ember-cli-rails@0.11.0](https://github.com/thoughtbot/ember-cli-rails/tree/v0.11.0) 及びそれが依存している [ember-cli-rails-assets@0.6.2](https://github.com/seanpdoyle/ember-cli-rails-assets/tree/v0.6.2) ではまだ Rails6 の Deprecation Warning への対応が入っていなかった。


## 解決方法 {#解決方法}

よくよく調べると
ember-cli-rails の方は既にマージされている
<https://github.com/thoughtbot/ember-cli-rails/pull/587>
で対応がされていて、
ember-cli-rails-assets の方は
<https://github.com/seanpdoyle/ember-cli-rails-assets/pull/12>
に対応ブランチが用意されていた。

というわけで、
ember-cli-rails の方は master から
ember-cli-rails-assets は上のプルリクエストのブランチを使うように
Gemfile を書き換えるだけで済んだ。

が、とりあえず master を見るのはいつか知らない間に更新が入りそうで怖いので
ember-cli-rails は現在の最新の ref を取るようにしている。

というわけで以下のような書き方になった。

```ruby
gem 'ember-cli-rails', github: 'thoughtbot/ember-cli-rails', ref: '1e56a03fb8437f52dfd450454c71cffda2981d66'
gem 'ember-cli-rails-assets', github: 'seanpdoyle/ember-cli-rails-assets', branch: 'rely-on-engine-to-load-helper'
```

ということを ember-cli-rails-app というリポジトリの
<https://github.com/mugijiru/ember-rails-todo-app/pull/109>
で対応した。

なお Rails6 には
<https://github.com/mugijiru/ember-rails-todo-app/pull/106/files>
でアップグレードしている。結果的にはこのプルリクエストでは rails app:update して load\_defaults を 6.0 にして関係ない Warning を潰しただけになっていたりする。


## その他 {#その他}

ember-cli-rails は
Ruby 3.0 対応のプルリクエストも含めて結構なプルリクエストや Issue が放置されてるのでこのまま使い続けるのは厳しそうな雰囲気。

いずれ ember-cli-rails も捨てて ember-cli 単体で動かすようにした方が良さそう
