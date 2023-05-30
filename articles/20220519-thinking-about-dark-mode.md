---
title: "なんとなく好ましいだけではないダークモード、我々は何のために対応するのか？｜Offers Tech Blog"
emoji: "🌃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ダークモード", "デザイン", "アクセシビリティ", "ユーザビリティ"]
publication_name: "overflow_offers"
published: true
---

<!-- textlint-disable overflow-techblog/sentence-length -->

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。今回は個人的に気になって調べてみた系のネタを散漫に書いたブログです。

# ダークモードに対する疑問の発端

美観やバッテリーパフォーマンス[^1]を理由とする話を念頭に置きつつ OS レイヤーにおける UI のスイッチングは自動で行われるとして、モバイルアプリや Web の個々の提供者が向き合う必然性はどこにあるのでしょうか？

Web で実装するときは `prefers-color-scheme` を使って CSS Custom Properties で宣言されたカラーパレットを切り替えることになります。モダンブラウザのサポートが充実しているので実装上の難関はありません。よって本記事では何のためにダークモード[^2]を提供するのかという動機付けの理解を目的とします。

[^1]: アプリケーションのパフォーマンス観点において全くバカにできない重要な要素のひとつではありますがさておき。

[^2]: 元々がダークなサイトはライトモードに対応する、という文脈があっても良さそうなものですがあまり聞かないので本稿では便宜上ダークモードに限定します。

# ダークモードの存在を肯定しうる理由たち

よく目にする主要な論拠についてそれぞれ調べてみました。以下で紹介している記事は色々見た中で「もっともらしそうな情報」を中心にピックアップして言及しています。

## 美観、感性によるもの

「なんか黒いのカッコいい」わかります、わかりますよ。コマンドラインにせよ、エディタにせよ、我々ソフトウェアエンジニアは特に黒地と白文字に親しんできました。

https://medium.com/dev-channel/let-there-be-darkness-maybe-9facd9c3023d

>Let’s get this out of the way: the data is definitely biased. The survey was only shared on Twitter and the audience was more on the tech side. 

本文中にある通り技術屋バイアスが大きくかかっている中でダークモードに対して非常に肯定的な態度を示すサーベイの結果です（記事の論旨自体はかなり中立的な見解）。割合はともかくとしてダークモードを好む理由を知る参考になるかもしれません。

## 視覚への負荷軽減によるもの

こちらの動画はダークモードの視覚への影響について検眼医協会の会長（どれくらいの権威なのかは謎である）を引き合いに出して解説しています。

https://www.youtube.com/watch?v=bCaFRN3aaP8

ダークモードは焦点を合わせるのにより大きな努力を必要とし、真っ黒な背景+白い文字のような場合に輪郭がぼやけて見えるなどの事象について述べています。少なくともダークモードだから目に優しいという論拠はないように判断できます。

ほか、手元のメモだとロービジョンの人が暗い背景に明るいテキスト、つまりダークモードを好むという傾向にある、みたいな論文を参照した形跡があったのですが出所を見失ってしまいました。ご存じの方がいればぜひ...。
なんにせよ昔から OS の視覚サポート機能として色反転モードは存在しているので、そういう需要があること自体は想像に難くありません。

_※ 追記: 視力、視覚障害とダークモード vs ライトモードについて [タレコミ (thanks for @hiloki-san)](https://twitter.com/hiloki/status/1527114415916474368?s=20) いただきました。下記リンク先に詳しい解説があるので興味のある方は是非。_

https://www.nngroup.com/articles/dark-mode/

## バッテリーパフォーマンスによるもの

ディスプレイの光量を減らすことでモバイルデバイスのバッテリーを節約できるのは皆さまご存じのとおりです。[^3]

https://wired.jp/2019/10/05/dark-mode-chrome-android-ios-science/

>**検証3）ダークモードはバッテリー寿命を延ばすか？**
>（中略）有機EL（OLED）ディスプレイでは、ダークモードにすればバッテリーの節約につながる。有機ELではピクセルが個別に発光する仕組みのため、画面を黒にすれば発光がオフになるからだ。一方、バックライトを発光させる液晶ディスプレイでは、画面を黒にしても発光は止まらないため、このようなメリットはない。

据え置きディスプレイの類はともかく、スマートフォンやノートパソコンのディスプレイは有機 EL への置き換えも進んでいるので、バッテリーパフォーマンスは肯定的な材料のひとつになりそうです。

[^3]: ところで Web は一般的には常駐しない/できない世界観だからか、モバイルアプリと比べてバッテリーパフォーマンスが話題になること少ない気がしますね。

# Android と iOS のダークモードの便益説明

OS レベルでダークモードのサポートを備えた主要なスマートフォン OS のベンダーによる説明も参照してみました。

## Android は手堅い理屈推し

https://developer.android.com/guide/topics/ui/look-and-feel/darktheme

>- Can reduce power usage by a significant amount (depending on the device’s screen technology).
>- Improves visibility for users with low vision and those who are sensitive to bright light.
>- Makes it easier for anyone to use a device in a low-light environment.

バッテリーについては前述の通り、ディスプレイ技術に依存するものの一定の効果があるということでわかりますし、ロービジョンや光過敏の人に価値があるというのも（おそらく何らかの研究が背景にあるという前提で）有益そうです。
また明るくない（暗い）環境下で視認性の確保が容易になるというのは、ディスプレイの輝度を MAX にすればどうにでもなる気はしますが、それをしなくとも十分な視認性を確保できるアプローチであると理解できます。

## iOS はルック＆フィール推し

https://support.apple.com/en-us/HT208976

>Dark Mode is a dramatic new look that's easy on your eyes and helps you focus on your work.

Android の説明と比べるとさすが俺たちの Apple さん！というルック＆フィールに全振りの主張ですが、ここまで調べた内容を踏まえると根拠の薄弱さが否めない印象です。「こういうのが好きな人は好きでしょう」が適当ではないか。

なお [Let there be darkness! 🌚 Maybe…](https://medium.com/dev-channel/let-there-be-darkness-maybe-9facd9c3023d) という記事の中で [Supporting Dark Mode in Your Interface](https://developer.apple.com/documentation/uikit/appearance_customization/supporting_dark_mode_in_your_interface) に "The choice of whether to enable a light or dark appearance is an aesthetic one for most users, and might not relate to ambient lighting conditions." と書いてあるとのことですが、本記事を書いている時点ではその記述が見つけられませんでした。美的センスに依るものと論ずること自体はいっそ潔い見解のひとつです。

# まとめ

ダークモードは OS 側が主導して各種のサービスやアプリに踏み込んで色覚的な体験を促している取り組みです。とはいえ色彩がブランドに含まれることは当然ありますし、それを踏まえて Dark なカラーパレットを再構築するのも簡単な仕事ではありません。だからといって [Auto Dark Theme](https://developer.chrome.com/blog/auto-dark-theme/) のようなアプローチは手法としてちょっと乱暴な気もします。[^4]

[^4]: 結局 Origin Trials で試験されたあと [このあたりから追える情報](https://developer.chrome.com/origintrials/#/view_trial/1626925365387591681) を見るにあまり芳しい進展はないように見受けられます。

一方の実利においてダークモードにすればエンゲージメントが改善するというのは短絡的な誤りですが、[それを好む環境にダークモードを提供することが有益](https://web.dev/terra-dark-mode/) であることは納得できます。また環境条件によってはダークモードのほうが機能的にも優れている状況が存在することも認められます。[クライアントサイドの多様な利用環境](https://speakerdeck.com/ahomu/pahuomansuhanazezhong-yao-nafalseka-mu-xian-false-ux-nitorawarenaikuraiantosaidokai-fa-falseben-zhi?slide=24) に適応する選択肢のひとつとして対応する妥当性があるのではないでしょうか。

総じてダークモード対応は提供者として必須ではないにせよ、ユーザーの期待により柔軟に応えるための選択肢としては挙がってもよいだろうと考える立場に至りました。他の総論としては下記の記事も参考になりましたので最後に紹介しておきます。

- [prefers-color-scheme: Hello darkness, my old friend](https://web.dev/prefers-color-scheme/)（特に技術的な TIPS を含む）
- [Wait, is dark mode actually bad for productivity?](https://zapier.com/blog/dark-mode-bad-productivity/)
- [What Is Dark Mode – And Should You Be Using It? – Forbes Advisor UK](https://www.forbes.com/uk/advisor/mobile-phones/what-is-dark-mode-and-should-you-be-using-it/)

ではではノシ

# 関連記事

https://zenn.dev/offers/articles/20220411-open-api-schema
https://zenn.dev/offers/articles/20220418-what-is-bff-architecture
https://zenn.dev/offers/articles/20220425-universal-attitude