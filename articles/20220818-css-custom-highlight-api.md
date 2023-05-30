---
title: "Web の仕様を眺めるシリーズ CSS Custom Highlight API | Offers Tech Blog"
emoji: "🪄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CSS", "CustomHighlightAPI", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2022-08-18 09:30
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。

本記事は [Chrome Platform Status](https://chromestatus.com/features#browsers.chrome.status%3A%22Proposed%22) からなんとなく Proposed なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [Document Picture-in-Picture](https://zenn.dev/offers/articles/20220815-document-pip-proposal) でした。

# ユーザー選択領域 `::selection` 以外を任意にハイライト

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [CSS Custom Highlight API](https://chromestatus.com/feature/5436441440026624) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

https://drafts.csswg.org/css-highlight-api-1/

今回は W3C のサイト内にその所在がある仕様ですが W3C の冠がついていても Editor's Draft なので[^1]合意形成が保証されない文書であることに注意してください。

[^1]: https://www.w3.org/standards/types#ED を参照のこと

## これまでのカスタムハイライト沼

特定の範囲にスタイルをつけてハイライトする表現はよくみられますが、Web 開発者が任意の表現を求めてカスタムコントロールを作り込もうとすると実装上のペインが多い沼になっています。

ひとつはユーザーが選択した領域または [Selection API](https://developer.mozilla.org/ja/docs/Web/API/Selection) を操作して指定した領域に対して [`::selection` 疑似要素](https://developer.mozilla.org/ja/docs/Web/CSS/::selection) でつける方法があります。この選択領域は一意であり、ユーザー自身が選択した領域ともバッティングするため使い勝手としては今ひとつです。

もうひとつの方法は `<span>` や `<div>` 等でハイライト範囲を適宜ラップ or オーバーレイしてを表現する方法です。もちろんフルカスタムが可能ですが実装が煩雑になることは想像に難くありません。該当箇所を要素でラップする方法だと、描画アップデートが高頻度で生じることによるレンダリング負荷が問題になるケースもあるでしょう。

Custom Highlight API は、このような課題背景において開発者が任意のハイライト表現をコントロールできるようにする仕様です。具体的な API については後述します。

## Micsoroft と Apple と Bloomberg の連名ドラフト

仕様文書の Editor's をみると下記の 3 名の名前があがっています。

- Florian Rivoal (On behalf of Bloomberg)
- Sanket Joshi (Microsoft Corporation)
- Megan Gardner (Apple Inc.)

議論の根元まで追っていないので完全な憶測ですが、Boolmberg の方はカスタムハイライトを実装したい Web 開発者としての立場があり、なんらか Microsoft と Apple の開発者も支持するに至っているのでしょうか。このような布陣なので Chromium 側のオーナーは Microsoft 社のメアドが並んでおり、WebKit にも既に Ship されています[^2]。

[^2]: https://developer.apple.com/safari/technology-preview/release-notes/ の Release 99 に "Added support for rendering highlights specified in CSS Highlight API" が確認できます。

# 本稿執筆時点で提案されている API

CSS における `::highlight` 疑似要素と JavaScript における `Highlight` API のあわせ技で使用します。

## `::highlight` 疑似要素と `CSS.highlights` レジストリの操作

[仕様書序盤](https://drafts.csswg.org/css-highlight-api-1/#intro) のサンプルコードが最もシンプルなので引用して紹介します。

```html
<style>
  :root::highlight(example-highlight) {
    background-color: yellow;
    color: blue;
  }
</style>
<body><span>One </span><span>two </span><span>three…</span>
<script>
  let r = new Range();
  r.setStart(document.body, 0);
  r.setEnd(document.body, 2);

  CSS.highlights.set("example-highlight", new Highlight(r));
</script>
```

1. `new Range()` でテキスト範囲を作成
2. `new Highlight()` でハイライト範囲を作成
3. `CSS.highlights.set` で `example-highlight` という名前でハイライト範囲を登録
4. ハイライト範囲に `::highlight(example-highlight)` で定義されたスタイルを反映

`CSS.highlights` は `HighlightRegistry` というインターフェースで [`maplike`](https://webidl.spec.whatwg.org/#idl-maplike) 宣言されているので `Map` 相当の操作をできます。`CSS` の下にレジストリを持つインターフェースは他に見ない珍しい API デザインのように思えますね。

また、`Highlight` のコンストラクタは可変長で複数の `Range` をとれるため複数箇所をまとめてハイライトできます。既存の `Selection` にはできない芸当です。

## Custom Highlight が動作するデモ

https://microsoftedge.github.io/Demos/custom-highlight-api/

手っ取り早く実行できるデモもあります。少なくとも Chrome Canary Version 106.0.5213.0 で動作することを確認しました。ソースコードも短いので実際的な用途のイメージもしやすいでしょう。
Safari は Develop > Experimental Features > Highlight API を有効にすれば動作すると見せかけて、[`StaticRange`](https://developer.mozilla.org/en-US/docs/Web/API/StaticRange) を要する実装となっている等の差分があるようで、このデモそのままは動きません。

# まとめ

簡単ですが CSS Custom Highlight API のご紹介でした。既に実装が ship されている分、前回までに紹介したプロポーザルと比べてデモサイトの紹介だけで一定以上の説明ができてしまっている感。
大昔お世話になった `contenteditable` 周辺の事情にも革命が起こらないものだろうかと思う今日この頃です。

# 関連記事

https://zenn.dev/offers/articles/20220815-document-pip-proposal
https://zenn.dev/offers/articles/20220627-css-anchored-positioning
https://zenn.dev/offers/articles/20220711-develop-issues-part2

