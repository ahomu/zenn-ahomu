---
title: "Web の仕様を眺めるシリーズ Capability Delegation | Offers Tech Blog"
emoji: "🏈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CapabilityDelegation","FullScreenAPI", "PaymentRequestAPI", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2022-12-01 09:00
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。


本記事は [Chrome Platform Status](https://chromestatus.com/features#browsers.chrome.status%3A%22Proposed%22) からなんとなく Proposed なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [EditContext API](https://zenn.dev/offers/articles/20220920-edit-context-api) でした。

# 実行にユーザー操作が必要な API のケイパビリティ委譲

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [Capability Delegation](https://chromestatus.com/feature/5708770829139968) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

https://wicg.github.io/capability-delegation/spec.html

## Capability (ケイパビリティ) という概念

当該文脈において Capability は見慣れない単語ですが平たく?いうと、特定の条件を満たした [Browsing Context](https://developer.mozilla.org/docs/Glossary/Browsing_context) が持っている、一定の制約がある API を実行可能な能力を指します。

代表的な例としては特定の API について条件を満たさないと実行不可にする Features gated by user activation の仕組みがあります。これは `Element.requestFullScreen()` や `PyamentRequest.show()` 、`Window.open()` など Web ページ所有者の JavaScript が好き勝手に呼び出せると悪用可能な API をユーザーの操作に起因していないと呼び出せなくする仕組みです。

https://developer.mozilla.org/en-US/docs/Web/Security/User_activation

この仕組みにおいて、ユーザー操作を通して特定の API を実行可能である瞬間、その Browsing Context は委譲可能なケイパビリティを所有している状態です。

## Capability の委譲（Delegation）

Capability Delegation はそのようなケイパビリティを、既知の信頼できる Browsing Context に委譲する仕組みです。

Capability であって Permission ではないのは、ユーザーの明示的な許可を伴う権限の委譲ではなく、あくまで特定の条件を満たしたときに獲得しうる能力（やや直訳気味だが原文だと ability）を委譲する仕組みだからです。

制約がある API が全てこの仕様に定義される委譲の仕組みに対応する必要はありません。ユーザーアクティベーションのように Capability Delegation の概念がないと別の Browsing Context に代理実行させることが不可能な場合に乗っかるインターフェースとして存在します。

## 呼び出し元のケイパビリティを必要とするサードパーティ

仕様の [提案ドキュメント](https://docs.google.com/document/d/1IYN0mVy7yi4Afnm2Y0uda0JH8L2KwLgaBqsMVLMYXtk/edit#) では、支払い決済や地図検索、チャットアプリなど iframe 等で提供される既知の信頼されたサードパーティに対して各種のケイパビリティを委譲するいくつかのシチュエーションが例示されています。

# 本稿執筆時点で提案されている API

下記はすべて [Draft Community Group Report, 15 June 2022](https://wicg.github.io/capability-delegation/spec.html#examples) のサンプルコードからの引用です。

## ケイパビリティの委譲元と委譲先のコード例

次のコードではユーザーアクティベーション（`onclick`）に基づいて獲得された Payment Request API を呼び出せるケイパビリティを `targetWindow` に委譲しています。

```js
window.onclick = () => {
  targetWindow.postMessage('a_message', {delegate: "payment"});
};
```

HTML の仕様としては [`WindowPostMessageOptions`](https://html.spec.whatwg.org/multipage/nav-history-apis.html#windowpostmessageoptions) に `delegate` というオプションが拡張される想定です。

さらに次のコードでは `postMessage()` を通じてケイパビリティを委譲された側の Browsing Context が `onmessage = () => {}` で `payRequest.show()` をコールしています。

```js
window.onmessage = () => {
  const payRequest = new PaymentRequest(...);
  const payResponse = await payRequest.show();
  ...
}
```

この例はあくまで Payment Request API だとしたら...という設定になっており、Capability Delegation 固有の仕様としては `WindowPostMessageOptions` の `delegate` というインターフェースの定義に留まります。将来的にケイパビリティを委譲可能にするアップデートが個々の API において生じた際、実行時の挙動や詳細は個別に定義される想定です。

# まとめ

簡単ですが Capability Delegation の紹介でした。こういう仕様を読むと、逆に今なにが出来なくてどのようなペインが存在しているのかが分かるかもしれません。

ちなみに Google Chrome (≒ Chromium) では M100 で Capability Delegation に対応しています。 また `Element.requestFullScreen()` での対応を M104 で Ship 済みです。

# 関連記事

https://zenn.dev/offers/articles/20220920-edit-context-api
https://zenn.dev/offers/articles/20220815-document-pip-proposal
https://zenn.dev/offers/articles/20220711-develop-issues-part2

