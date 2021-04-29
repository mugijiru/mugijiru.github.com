+++
title = "org-agenda を活用し始めた"
date = 2021-04-29T19:03:00+09:00
categories = ["Emacs", "org"]
draft = false
+++

昨年から org-mode をもっと活用していこうということで、
org-clock を使い始めたり org-pomodoro を導入したり org-habits を設定してみたりしている麦汁さんです。

org-mode は機能が豊富過ぎてマジで何が出来るのか全貌を把握し切れてないのですが、そんな中で、やっぱり全然把握できてなくて使ってなかった機能の1つが org-agenda ってやつ。

なんかうまく使うと、予定を組んでたり締切を設定していたりするタスクが一目瞭然になってお仕事などが捗るという素敵な機能らしいのだけど、どうもイマイチ使い方がわからなくて放置していました。

ただまあ仕事が捗るなら試してみる価値はあるよな〜ということで、今年の頭ぐらいから使い始めている。

まず、平日朝イチでやっておきたい習慣タスクに対して
`Weekday` と `Start` の2つのタグを振ってるか、
`Daily` というタグを振ってるのでそれを表示できるやつを以下のように仕込んでる。

```emacs-lisp
("hs" "Weekday Start"
 ((tags "Weekday&Start|Daily"
        ((org-super-agenda-groups '((:name "予定が過ぎてる作業" :scheduled past)
                                    (:name "今日の作業" :scheduled today)
                                    (:discard (:anything t))))))))
```

<https://github.com/mugijiru/.emacs.d/blob/a523566f2be993655f74ebf20afc4da444019f5e/inits/60-org.el#L89-L93>

麦汁さんのやりたいことを実現するには、デフォルトの org-agenda だとなんか機能が足りないっぽかったのでそれを補うために [org-super-agenda](https://github.com/alphapapa/org-super-agenda) も使っている。

とりあえず上の例の2行目でタグによる絞り込みをしているがこれは org-mode の標準機能を使っていて
<https://orgmode.org/manual/Storing-searches.html#Storing-searches>
あたりに書いてる方法で絞り込んでいる。

で、その後に org-super-agenda の機能である
[Group Selectors](https://github.com/alphapapa/org-super-agenda#group-selectors) というやつでスケジュール通りのやつと、スケジュールが過ぎてるやつとで表示を切り分けてる。

`(:discard (:anything t))` は、そこまでの条件にマッチしなかったやつを全部無視するような設定。これがないと `Other items` という形で全部並んでしまって邪魔になる。

最初の絞り込みで綺麗に絞り込めると良いかもしれないが、そこまで高度な機能は org-mode には備わってなさそう。それか、そういう高度な機能を見つけて使いこなせる能力を俺が有してないか。

まあそれはともかく、上の感じでタスクを登録していると以下のように表示される。

```text
Headlines with TAGS match: Weekday&Start|Daily

 予定が過ぎてる作業
  next-actions:TODO 排便                                                      :Weekday:Start:

 今日の作業
  next-actions:TODO 体重・体脂肪率計測                                                 :Daily:
```

実際のやつはもっと色々あるというか、そもそもわざわざ排便を org-mode で管理はしてないのであくまでサンプルとして2つ置いてるだけだとご認識ください。

同じノリでその日の締め作業も取れるように設定している。

また、日中使うための設定も用意していて、こっちは結構複雑。

```emacs-lisp
("d" "Today"
 ((agenda "会議など"
          ((org-agenda-span 'day)
           (org-agenda-files my/org-agenda-calendar-files)))
  (tags-todo "-Weekday-Daily-Holiday-Weekly-Weekend"
             ((org-agenda-prefix-format " ")
              (org-agenda-overriding-header "今日の作業")
              (org-habit-show-habits nil)
              (org-agenda-span 'day)
              (org-agenda-todo-keyword-format "-")
              (org-overriding-columns-format "%25ITEM %TODO")
              (org-agenda-files '("~/Documents/org/tasks/next-actions.org"))
              (org-super-agenda-groups '((:name "仕掛かり中" :todo "DOING")
                                         (:name "TODO" :todo "TODO")
                                         (:name "待ち" :todo "WAIT")
                                         (:discard (:anything t))))))
  (alltodo ""
             ((org-agenda-prefix-format " ")
              (org-agenda-overriding-header "予定作業")
              (org-habit-show-habits nil)
              (org-agenda-span 'day)
              (org-agenda-todo-keyword-format "-")
              (org-overriding-columns-format "%25ITEM %TODO")
              (org-agenda-files '("~/Documents/org/tasks/projects.org"))
              (org-super-agenda-groups '((:name "〆切が過ぎてる作業" :deadline past)
                                         (:name "予定が過ぎてる作業" :scheduled past)
                                         (:name "今日〆切の作業" :deadline today)
                                         (:name "今日予定の作業" :scheduled today)
                                         (:discard (:anything t))))))
  (tags-todo "Weekday|Daily|Weekly"
             ((org-agenda-overriding-header "習慣")
              (org-habit-show-habits t)
              (org-agenda-files '("~/Documents/org/tasks/next-actions.org"))
              (org-super-agenda-groups '((:name "予定が過ぎてる作業" :scheduled past)
                                         (:name "今日予定" :scheduled today)
                                         (:discard (:anything t))))))))
```

<https://github.com/mugijiru/.emacs.d/blob/a523566f2be993655f74ebf20afc4da444019f5e/inits/60-org.el#L109-L144>

通常なら Agenda for current week or day ってのが標準で用意されてるのでそれを使えばいいかなって思うんだけど
1 view でいい感じにカテゴライズされていて取得できるってのが欲しかったんですよね。で、それをやろうと思うとやはり org-super-agenda が必要そうだったって感じ。

とりあえず現在は next-actions.org に、直近やるつもりの作業を詰めていて
projecs.org に、直近ではないけどやることリストを並べてるって感じ。スプリントバックログとプロダクトバックログみたいな扱いのつもりですね。あとは別途 org-gcal で同期している Google Calendar から取得した予定用のファイルもあったりする。

そうやっていくつかあるファイルからいい感じになるようにということで設定しているのが先程のコードでそれで org-agenda のバッファを生成すると以下のような雰囲気のやつになる。

```text
 会議など
 11:00-12:00 すごい会議
 15:00-18:00 長い会議
------------------------------
今日の作業
 仕掛かり中
 - 藁人形の作成
 待ち
 - 五寸釘発注の稟議

予定作業
 〆切が過ぎてる作業
 - 五寸釘の発注

習慣
 予定が過ぎてる作業
 - 排便
 今日予定
 - 体重・体脂肪率計測
```

もちろん、内容はサンプル用に適当にでっち上げたやつです。

本当はタグも表示されてしまうけど、それはあまり要らないかなと思ってるので、それはなんとか非表示にしたいなあと願ってる。
`org-overriding-columns-format` を弄っても今のところいい感じにならなくて悲しい。

あと、今の設定と使い方だと「あと数日で着手しないといけないタスク」とかそういうやつはわからないので、それもなんとかしたい。

多分着手予定の日を早めに設定して起きつつ〆切を設定してたら今でもある程度いけるんだろうけど、気付いたら着手予定が同じ日になってていきなりその日に全部やらないといけない雰囲気になるとかありそうでこわい。

数日前から、着手予定のやつがいつ着手予定なのか見れるようにしたらいいんだろうな〜。ま、少しずつ改善を入れていくしかないか……。
