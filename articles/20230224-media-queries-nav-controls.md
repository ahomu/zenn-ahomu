---
title: "Web の仕様を眺めるシリーズ Media Queries Level 5 nav-controls | Offers Tech Blog"
emoji: "◀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["navcontrols", "MediaQueries", "CSS", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2023-02-24 09:00
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。

本記事は [Chrome Platform Status](https://chromestatus.com/features#browsers.chrome.status%3A%22Proposed%22) からなんとなく Proposed なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [Scroll Timeline for Web Animations](https://zenn.dev/offers/articles/20230209-scroll-timeline-for-web-animatinos) でした。

# Media Queries Level 5 `nav-controls`

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [Navigation-controls media query](https://chromestatus.com/feature/5717537549451264) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

https://drafts.csswg.org/mediaqueries-5/#nav-controls

## 二重の「戻る」ボタン問題

この仕様についての議論は遡ると 2018 年頃に始まっています。当時の PWA 文脈において実行環境となるユーザーエージェントが「戻る」ボタンを提供しているか判別できず、常に自前の「戻る」ボタンを提供しなければならない点が課題提起されています。

https://github.com/w3c/manifest/issues/693

もちろんユーザーエージェントが「戻る」ボタンを提供していることもあるので、そこで二重に「戻る」ボタンが提供されてしまうことが指摘されています。

実際に仕様に落とし込まれた際のやり取りはこのあたりでしょうか。

https://github.com/w3c/csswg-drafts/issues/4187

## UA が提供するナビゲーションコントロールの有無を判定

問題の解決は単純で、Media Queries の仕様に明らかに発見可能なナビゲーションコントールの有無を判定する機能が加わりました。

明らかに発見可能 (*obviously discoverable*) とは、ユーザーエージェントを利用するユーザーにとって認識・識別できることが当然であると期待されることを指します。
一般的には視覚的に表現されたコントロールを想定しますが、聴覚や触覚を通したインターフェースを通した場合は異なった意味合いになる可能性があります。

いまの時点では「戻る」ボタンのみが想定されていますが、将来的には他のナビゲーションコントロールの有無も判定できるような拡張性を備えています。（後述のサンプルコード）

# 本稿執筆時点で提案されている API

以下のサンプルコードは [7.4. Detecting UA-supplied navigation controls: the nav-controls feature](https://drafts.csswg.org/mediaqueries-5/#nav-controls) を元にした引用（一部改変）です。

```css
@media (nav-controls: back) {
  #back-button {
    display: none;
  }
}
```

`nav-controls: back` がユーザーエージェントが「戻る」ボタンを提供している状態の判定です。このとき、アプリケーションは自前の `#back-button` を `display:none` しています。

現状は「戻る」ボタンしかないので次のサンプルコードも、`nav-controls: back` と同じように動くと書いてありますが賢明な諸氏ならお分かりのとおり将来の拡張可能性を考えると厳密に等価ではありません。

```css
@media (nav-controls) {
  #back-button {
    display: none;
  }
}
```

と言いつつも

>However, given that navigation back is arguably the most fundamental navigation operation, the CSS Working Group does not anticipate user interfaces with explicit navigation controls but no back button, so this problem is not expected to occur in practice.

「戻る」ボタンが不在のナビゲーションコントロールは実質存在しえないだろうからまぁ大丈夫っしょ、というコメントも書かれています。

# まとめ

簡単ですが Media Queries Level 5 `nav-controls` の紹介でした。PWA らしい PWA を開発するお仕事が世間に溢れているわけではないのでお世話になる機会は限られますが、頭の片隅に置いておくとよいかもしれませんね。:)

# 関連記事

https://zenn.dev/offers/articles/20230112-control-ui-customization
https://zenn.dev/offers/articles/20230209-scroll-timeline-for-web-animatinos

