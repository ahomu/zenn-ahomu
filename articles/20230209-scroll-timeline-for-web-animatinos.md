---
title: "Web の仕様を眺めるシリーズ Scroll Timeline for Web Animations | Offers Tech Blog"
emoji: "🎞️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ScrollTimeline", "WebAnimations", "CSS", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2023-02-09 09:00
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。

本記事は [Chrome Platform Status](https://chromestatus.com/features#browsers.chrome.status%3A%22Proposed%22) からなんとなく Proposed なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [Control UI Customization](https://zenn.dev/offers/articles/20230112-control-ui-customization) でした。

# Scroll Timeline for Web Animations

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [Scroll Timeline for Web Animations](https://chromestatus.com/feature/6752840701706240) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。W3C の Editor's Draft に倣うと Scroll-driven Animations です。

https://drafts.csswg.org/scroll-animations-1/

## パララックス表現のネイティブサポート

もの凄く端的に言うと、スクロールに連動して何らかアニメーションするパララックス表現がネイティブサポートされるような仕様です。

従来 Web Animatinos は経過時間に依存したタイムラインで実行されるのに対して、Scroll Timeline は経過時間ではなく特定のコンテナにおけるスクロール位置と連動したタイムラインでアニメーションを実行できるようにします。

[Scroll-driven Animations](https://drafts.csswg.org/scroll-animations-1/) 内には *View Progress Timeline* (`ViewTimeline`) という特定の要素が画面内に表示されている割合に応じたタイムラインについても言及があります。こちらも大体は同じような取り回しになると思われるので今回は `ScrollTimeline` を中心に紹介します。

## CSS Animation Worklet API の Scroll Timeline

>ScrollTimeline is mostly implemented for Animation Worklet, but there are still spec issues to work out and implementation work to be done to integrate it into Web Animations

[Chromium の当該チケット](https://bugs.chromium.org/p/chromium/issues/detail?id=916117) によれば、最近話題としては落ち着いた気もする[^1] CSS Houdini 的な [Animation Worklet API](https://drafts.css-houdini.org/css-animation-worklet/) では先行して Scroll Timeline が組み込まれているようです。これを JavaScript と CSS 双方の Web Animations への統合が意図されています。

[^1]: さすがに市井の WebDev が一喜一憂するには低レイヤー API すぎるということで、さもありなん

## 実験的機能の有効化とデモサイト

本稿執筆時点では少なくとも Chrome Canary (M112) で `chrome://flags` から **Experimental Web Platform features** を Enabled にすることで下記のデモが [Polyfill](https://github.com/flackr/scroll-timeline) なしで動作することを確認しています。

https://codepen.io/collection/qOEWgO
https://codepen.io/collection/jbwpJB

# 本稿執筆時点で提案されている API

以下のサンプルコードは [ScrollTimeline Examples](https://scrolltimeline-playground.glitch.me/) と [scroll-timeline - CSS: Cascading Style Sheets | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/scroll-timeline) を元にした引用（一部改変）です。

## JavaScript での例

Web Animations API をご存じない/思い出したい方は [ミツエーリンクスさんのこちらの記事](https://www.mitsue.co.jp/knowledge/blog/frontend/202006/17_1757.html) を参照すると良さそうです。[^2]

下記は Animation クラス等を直接取り扱う例です。ポイントは `new Animation(keyframe, scrollTimeline)` で引数に `ScrollTimeline` インスタンスを渡しているところでしょう。

```javascript
const scroller = document.querySelector('div.scroll');
const parallax = document.querySelector('div.parallax');
  
const scrollRange = scroller.scrollHeight - scroller.clientHeight;
const parallaxRange = scrollRange * 0.2;

const keyframe = new KeyframeEffect(parallax, [{ 'transform': 'translateY(0)' }, { 'transform': 'translateY(' + -parallaxRange + 'px)' }], 1000);
const scrollTimeline = new ScrollTimeline({ source: scroller, orientation: 'block' });

const parallaxAnimation = new Animation(keyframe, scrollTimeline);
const parallaxAnimation.play();
```

現状は経過時間に従う `DocumentTimeline` [^3] (→ `document.timeline` ) しか渡せなかったところに `ScrollTimeline` が仲間入りしそう、という形です。`Element#animate()` で扱う場合は `el.animate(keyframes, {timeline: new ScrollTimeline(...)})` で同様です。

[^2]: 私も思い出させてもらいました...
[^3]: `AnimationTimeline` は各種タイムライン系インターフェースの親にあたります

## CSS での例

こちらは MDN の記載からの引用です。HTML 構造はこちら。

```html
<div id="container">
  <div id="square"></div>
  <div id="stretcher"></div>
</div>
```

スクロールの基準となる要素で `scroll-timeline-name` で名前を付けて `scroll-timeline-axis` で方向を定義します。`scroll-timeline` プロパティもショートハンドとして使えます。定義されたものをアニメーション対象要素で `animation-timeline` プロパティを通して呼び出します。

```css
#container {
  height: 300px;
  overflow-y: scroll;
  position: relative;

  /* scroll-timeline: squareTimeline block; と同義 */
  scroll-timeline-name: squareTimeline;
  scroll-timeline-axis: block;
}

#stretcher {
  height: 600px;
}

#square {
  background-color: deeppink;
  width: 100px;
  height: 100px;
  margin-top: 100px;
  animation-name: rotateAnimation;
  animation-duration: 3s;
  animation-direction: alternate;
  animation-timeline: squareTimeline;

  position: absolute;
  bottom: 0;
}

@keyframes rotateAnimation {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
```

これは名前をつけるパターンですが [`scroll()`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline/scroll) 関数で匿名指定もできます。シンプルな実用においてはこちらのほうが使われそうに見えますね。

```css
#square {
  /* 略 */
  animation-timeline: scroll(block nearest);
}
```

CSS における記法は関連ドキュメントを追いかけるとそれなりの変遷を経ている様子が見受けられるので上記の記法が正式に採用されるかは謎です。

# まとめ

簡単ですが Scroll Timeline の紹介でした。記事を書いている終盤に見つけた下記の動画が豊富なデモ URL と共に安心と信頼の Google Chrome Developers の発信なので一定の信頼ができる解説かと存じます。🫠

https://www.youtube.com/watch?v=Qf5wdXOxW3E

一昔前の悪いパララックス表現が再流行しないことを望みつつも、そもそも一定のパフォーマンス改善効果（≒当時ほど酷い体験にはなりづらい？）は期待できますし、どちらかというとカルーセル UI のアニメーションのほうが捗るかもしれませんね。ではでは 👋

# 関連記事

https://zenn.dev/offers/articles/20230112-control-ui-customization
https://zenn.dev/offers/articles/20220920-edit-context-api

