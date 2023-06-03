+++
title = "少し構成が複雑なアプリの Code Coverage を Code Climate で表示する"
date = 2023-06-03T11:47:00+09:00
categories = ["Rails", "Frontend"]
draft = false
+++

<https://github.com/mugijiru/ember-rails-todo-app> のカバレッジを取るためにやったことです。

書いたコードは
<https://github.com/mugijiru/ember-rails-todo-app/blob/712e6ca7e2b1135ba11c56c8c3e9ca870f020ec2/.github/workflows/ci.yml#L58-L98>
にあるので、それだけ見れば十分かとは思うけど、それで済ませるのも寂しいので少しテキストも書いておきます。


## アプリの構成 {#アプリの構成}

単一の Rails アプリケーション上で複数の Ember.js アプリケーションを動かせるようにするために
Ember.js のアプリケーションは ember/todo-app という位置に作成しています。

```text
RAILS_ROOT
├── ember/todo-app
│    ├── package.json
│    └── yarn.lock
└── spec
```

そのためテストを実行するディレクトリが異なるという状況です。

rspec の方は

```text
$ rspec spec
```

で済むけど Ember.js のテストは

```text
$ cd ember/todo-app && yarn test
```

しないといけないみたいな。


## 複数テストスイートのカバレッジを取る {#複数テストスイートのカバレッジを取る}

今回は上記のように若干構成が複雑なので手を入れていますが、どちらも同じ階層でテストを実行できるのであれば
<https://docs.codeclimate.com/docs/configuring-test-coverage#multiple-test-suites>
に書いていることをやれば問題ないはずです。

なのですが今回は若干構成が複雑なため、そのままではダメでした。いい感じに統合してくれなかった。

じゃあどうしたかというと

```yaml
- name: Format ember test result
  run: cd ember/todo-app && ../../cc-test-reporter format-coverage -o ../../tmp/cc.ember.json --add-prefix ember/todo-app
```

という感じで、深い階層にある Ember.js 側のカバレッジのフォーマットを取得する時にちょっと調整している。

まず `cd ember/todo-app` で Ember.js アプリケーションの階層に移動して、
`cc-test-reporter format-coverage` を実行する際に
`-o` オプションを使って後で扱いやすい出力先を指定して
`--add-prefix` で移動先である `ember/todo-app` を指定している。

この `--add-prefix` は Code Climate の公式ドキュメントにある [List of subcommands](https://docs.codeclimate.com/docs/configuring-test-coverage#list-of-subcommands) という項目には載ってないんですよね。。。あと `--debug` もそこには載ってないけど使えるみたい
<https://github.com/codeclimate/test-reporter/blob/master/man/cc-test-reporter.1.md>

それはともかく、そんな感じで format 時の出力を調整したら後段の sum-coverage の際に良い感じにまとめてくれるようになる。
