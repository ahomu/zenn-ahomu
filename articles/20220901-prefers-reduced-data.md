---
title: "Web の仕様を眺めるシリーズ prefers-reduced-data | Offers Tech Blog"
emoji: "💾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CSS","mediaqueries", "prefersreduceddata", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2022-09-01 09:30
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。


本記事は [Chrome Platform Status](https://chromestatus.com/features#browsers.chrome.status%3A%22Proposed%22) からなんとなく Proposed なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [CSS Custom Highlight API](https://zenn.dev/offers/articles/20220818-css-custom-highlight-api) でした。

# `prefers-reduced-data` データーセーバー設定を検出するメディアクエリ

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [Media Queries Level 5 `prefers-reduced-data`](https://chromestatus.com/feature/5922192355229696) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

https://drafts.csswg.org/mediaqueries-5

Editor's Draft ですが [Working Draft](https://www.w3.org/TR/mediaqueries-5/) にもあがっています。Chrome Platform Status で Proposed ですが仕様がリリースされてからはそれなりの年月が経っています。その上で、いずれかのレンダリングエンジンにおける実装が先行しまくっている状況ではないようです。

## CSS におけるデータセーバーの検出

プラットフォームのシステム環境におけるいわゆるデータセーバー的な設定が有効になっていることを検出して、それに応じた処理を CSS で記述できるようにするメディアクエリの機能です。

既に各レンダリングエンジンからリリース済みでお馴染みの `prefers-reduced-motion` は OS のアクセシビリティ機能でエフェクトやモーションの類をなくす設定が有効かどうかを検知しますが、それのデータセーバー版です。

## なぜ ship されていないのか

[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-data) によれば正確には Chromium であれば `#enable-experimental-web-platform-features` フラグを有効にすることで利用できます。ユースケースは限られるものの見切り発車で ship してしまっても問題なさそうな親切機能のようにも思えます。ではなぜ、Chromium でさえもフラグなしで ship せず、他のベンダーでも実装が進んでいないのでしょうか。

### Finger Printing に使われる懸念

[1.4. Prefers-* Media Features Security and Privacy](https://drafts.csswg.org/mediaqueries-5/#mq-prefers-security) として下記のように書かれています。

>Information about a user can be used as an active fingerprinting vector. Analysis of impact pending, more information to be provided before spec is published.
>User agents and developers implementing this specification need to be aware of this vector and take it into consideration when deciding whether to use the feature. Specifically `prefers-reduced-motion`, `prefers-color-scheme` and `prefers-reduced-data` are currently of concern for exploitation.

フィンガープリントのベクタとして使えてしまう懸念が表明されており、仕様を公開する前に何らか分析のうえ議論を着地させる必要がありそうなステータスです。こうなってくると、Mozilla や Apple といったベンダの普段からの態度としてネガティブな立場の可能性もありえます。

実際選考して実装している [Chromium の当該チケット](https://bugs.chromium.org/p/chromium/issues/detail?id=1051189) においても CSSWG の動き待ちで長期間ペンディングしているような様子が伺えます。

### HTTP でも公開してるやん...とはいえ

Chrome 等では既に Network Client Hints のひとつとして [`Save-Data`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Save-Data) HTTP リクエストヘッダに乗せて送信されるようサポートされています。

また同じく Chrome 等では [`navigator.connection.saveData`](https://developer.mozilla.org/en-US/docs/Web/API/NetworkInformation/saveData) として JavaScript からもアクセスできます。

Chrome の周辺世界においてはとっくに公開している情報だし便利そうだから ship しちゃおうぜという意志決定もありえそうですが、昨今のプライバシー/セキュリティーに関する動向を踏まえると CSSWG で結論が出されていないものを独断専行するには憚られるのかもしれません。

ちなみに同じく Chromium を使っている Brave では `navigator.connection` が無効になっています。おそらく `Save-Data` HTTP ヘッダーも同様でしょう。これはレンダリングエンジンとして可能だが、ブラウザのポリシーとしてやらない、といった事情が垣間見えます。

# 本稿執筆時点で提案されている API

メディアクエリのフィーチャーが増えただけなので、使い方、記述方法そのものに新奇性はありません。

## `@media (prefers-reduced-data: reduce)`

[仕様書の当該箇所](https://drafts.csswg.org/mediaqueries-5/#prefers-reduced-data) のサンプルコードをそのまま紹介します。

```css
.image {
  background-image: url("images/heavy.jpg");
}

@media (prefers-reduced-data: reduce) {
  /* Save-Data: On */
  .image {
    background-image: url("images/light.jpg");
  }
}
```

`prefers-reduced-data: reduce` つまりデータ通信量を減らす設定にしているときは背景画像に `images/light.jpg` を読み込むようにするという記述です。

値の指定バリエーションは `no-preference | reduce` となっていて、`no-preference` はユーザーがシステムに対してなにも設定を明示していないことを示します。

# まとめ

以上、簡単ですが `prefers-reduced-data` のご紹介でした。一見してあったら便利そうで無害に見える機能も先鋭化したユーザートラッキング技術に絡むと色々ありそうですねという事例でした。

日本国内含めて近年の各国におけるインターネット上のプライバシーに関する議論を鑑みるに、我々開発者も自分たちがどんなツールを使っていて、どのような情報を収集し、何と連携しているのか把握する必要が一定あるように感じます。

# 関連記事

https://zenn.dev/offers/articles/20220815-document-pip-proposal
https://zenn.dev/offers/articles/20220627-css-anchored-positioning
https://zenn.dev/offers/articles/20220711-develop-issues-part2

