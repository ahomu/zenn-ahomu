---
title: "Web の仕様を眺めるシリーズ Soft Navigations Performance Entry | Offers Tech Blog"
emoji: "⏱️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["softnavigations", "performance", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2024-01-30 09:00
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。

本記事は [Chrome Platform Status](https://chromestatus.com/features) からなんとなく Proposed/In Development etc なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [~~form-sizing~~ field-sizing CSS property](https://zenn.dev/overflow_offers/articles/20231003-form-sizing-normal) でした。

# SPA 的な画面遷移を性能計測したい

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [Soft Navigations Performance Entry](https://chromestatus.com/feature/5144837209194496) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

SPA 的な画面遷移とはつまり `History.pushState` や `History.replaceState` を用いた JavaScript 駆動なナビゲーションを指します。このような遷移を指してここでは Soft Navigations と呼称しています。

https://github.com/WICG/soft-navigations/blob/main/README.md

Soft Navigations の仕様は [Draft Community Group Report](https://wicg.github.io/soft-navigations/) として公開されています。

## SPA と Performance RUM

SPA (_Single Page Application_) における RUM (_Real User Monitoring_) は下記のような課題を抱えています。

- ランディング URL の指標しか計測されない
- SPA の特性上ランディングは不利になりやすい
- SPA の優れたユーザー体験が計測上スポイルされる
- SPA より MPA のほうが Web Vitals 的に有利になるのか

これらの課題が検索ランキング要因を強く意識している前提として Soft Navigations は Google の提案です。モチベーションとして LCP や CLS といった Web Vitals に紐付く Google 提唱の速度指標を測定可能にするための仕様と言っても過言ではないでしょう。

2020 年頃の記事ですが [How SPA architectures affect Core Web Vitals](https://web.dev/articles/vitals-spa-faq) ではそのあたりの事を含めて触れられています。

## `SoftNavigationEntry` が流れてくる

>A Soft Navigation is a same document navigation which satisfies the following conditions:
>- Its navigating task is a descendent of a user interaction task.
>- There exists a DOM node append operation whose task is a descendent of the same user interaction task.
>
>via. [wicg.github.io/soft-navigations/#sec-algos](https://wicg.github.io/soft-navigations/#sec-algos)

- ナビゲーションがユーザーインタラクション由来であること
- 同じユーザーインタラクション由来で DOM の追加操作があること

これらを条件に Soft Navigations が発生したと判定して Performance Timeline に `SoftNavigationEntry` が流れてくるようになります。

# 本稿執筆時点で提案されている API

例によって `chrome://flags/#enable-experimental-web-platform-features` を有効にすると実験的に Soft Navigations Performance Entry を試せるようになっています。

以下のサンプルコードは [Experimenting with measuring soft navigations](https://developer.chrome.com/docs/web-platform/soft-navigations-experiment) からの引用です。

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('Layout Shift time:', entry);
  }
}).observe({type: 'layout-shift', buffered: true, includeSoftNavigationObservations: true});
```

既存の `PerformanceObserver` を利用する際に `includeSoftNavigationObservations` オプションを有効にすると Soft Navigations 相当の計測が可能になります。

多くの場合は `PerformanceObserver` を直接使う代わりに [GoogleChrome/web-vitals](https://github.com/GoogleChrome/web-vitals) を使うことになるでしょう。次のコードは [Web Performance Calendar » Beyond soft-navigations: tracking your SPA’s TTFB](https://calendar.perfplanet.com/2023/soft-navigations-spa-ttfb/) からの引用サンプルコードです。

```js
(function() {
  var script = document.createElement('script');
  script.src = 'https://unpkg.com/web-vitals@soft-navs/dist/web-vitals.attribution.iife.js';
  script.onload = function() {
    webVitals.onTTFB(console.log, {reportSoftNavs: true});
    webVitals.onLCP(console.log, {reportSoftNavs: true});
    webVitals.onCLS(console.log, {reportSoftNavs: true});
    webVitals.onINP(console.log, {reportSoftNavs: true});
  }
  document.head.appendChild(script);
}());
```

`reportSoftNavs` オプションが設けられています。当該ライブラリにおける各指標の計測実装の実態は [GoogleChrome/web-vitals の soft-navs ブランチ](https://github.com/GoogleChrome/web-vitals/tree/soft-navs) を参照すると分かります。本稿執筆時点では soft-navs ブランチを参照しないと利用できません。

# まとめ

ということで例によって簡単ながら Soft Navigations Performance Entry をご紹介しました。

SEO 抜きでも本来のユーザー体験を計測可能にしたい気持ちは分かりますし、LCP、CLS、INP(FID) みたいな指標は User Timings では代替できないので然もありなんと言ったところでしょうか。

## 参考リソース

- [Web Performance Calendar » Beyond soft-navigations: tracking your SPA’s TTFB](https://calendar.perfplanet.com/2023/soft-navigations-spa-ttfb/)
- [A Guide To Soft Navigations And Core Web Vitals Reporting | DebugBear](https://www.debugbear.com/blog/soft-navigations)
- [Experimenting with measuring soft navigations  |  Web Platform  |  Chrome for Developers](https://developer.chrome.com/docs/web-platform/soft-navigations-experiment)

# 関連記事

https://zenn.dev/overflow_offers/articles/20231003-form-sizing-normal
https://zenn.dev/overflow_offers/articles/20230224-media-queries-nav-controls
https://zenn.dev/overflow_offers/articles/20230209-scroll-timeline-for-web-animatinos