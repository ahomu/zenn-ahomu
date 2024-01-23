---
title: "Web の仕様を眺めるシリーズ form-sizing CSS property | Offers Tech Blog"
emoji: "⬆️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["form-sizing", "CSS", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2023-10-03 09:00
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。

本記事は [Chrome Platform Status](https://chromestatus.com/features) からなんとなく Proposed/In Development なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [Media Queries Level 5 nav-controls](https://zenn.dev/overflow_offers/articles/20230224-media-queries-nav-controls) でした。

# 追記

## 2023.10.10  `form-sizing: fixed | content`

紹介したそばから `normal` → `fixed` 、 `auto` → `content` に変更されていました。([IRCログ](https://github.com/w3c/csswg-drafts/issues/7542#issuecomment-1747803316)) 次項のサンプルコードは `normal | auto` のままですが使い方自体は変更されていないようです。

`form-sizing` ではなく `field-sizing` というプロパティ名への変更も提案されていてポジティブな流れなので、そのうちこれも変更されるかもしれません。テーマ通りいかにも時期尚早な紹介といったオチでした。:)

## 2024.01.23 `field-sizing`

ということで前回の追記直後に `field-sizing` になっていました。観測はしていたのだけど追記忘れてたので今更ですが。whatwg のほうでも議論が継続していますね。

https://github.com/whatwg/html/pull/9903

# 入力量に応じて大きさが変わる textarea を CSS で

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [form-sizing CSS property](https://chromestatus.com/feature/5176596826161152) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

https://github.com/tkent-google/explainers/blob/main/form-sizing.md

もともと [whatwg/html](https://github.com/whatwg/html/issues/6807) で議論されていた話題が [w3c/csswg-drafts](https://github.com/w3c/csswg-drafts/issues/7542) に移り、その具体的な提案として `form-sizing` プロパティの仕様が [Pull Request されている](https://github.com/w3c/csswg-drafts/pull/9251) ようです。

## JavaScript で再発明され続けてきた定番の振る舞い

入力量に応じて高さなどが変わるテキストエリアの振る舞いは、決して珍しいものではありませんが古来より JavaScript で DOM を操作することで実現されてきました。

これは弊社サービスの [Offers](https://offers.jp/lp) で使われている箇所の例です。

![入力量に応じて高さが可変するテキストエリアの例](/images/20231003-form-sizing-normal/textarea-auto-grow.gif)

ついつい泥臭い実装を思い浮かべてしまいがちですが、[`lh` で表現するアイディア?](https://github.com/w3c/csswg-drafts/issues/7542#issuecomment-1295227946) も議論されている issue にコメントがついていてナルホドと思ったりもしました。

## `form-sizing: normal | auto`

新しいプロパティを追加せず `height: max-content` のような既存構文を活かす方向性なども検討されたようですが、最終的には `form-sizing` の追加で意思決定されたようです。[合意がとられた IRC のログ](https://github.com/w3c/csswg-drafts/issues/7542#issuecomment-1542505774) を見てると経緯がなんとなく分かります。

決議のあと `normal` ってなんやねん、という声もあがっているようですが `auto` というキーワードに対するイメージ次第でしょうか。個人的には `normal` のほうが可変性をもつことにそれほど違和感がありません。`auto` は常に既存の振る舞い側にあるという感覚をこれまでの経験で暗黙的に備えているせいかしら。

また、本件と関連する話題として下記の issue も興味深そうなので機会を改めて言及するかもしれません。

https://github.com/w3c/csswg-drafts/issues/7552

# 本稿執筆時点で提案されている API

以下のサンプルコードは [explainers/form-sizing.md at main · tkent-google/explainers](https://github.com/tkent-google/explainers/blob/main/form-sizing.md) からの引用です。

```html
<div style="width:11em; background:yellow; padding:4px;">
  <textarea style="form-sizing:normal;">
  </textarea>

  <br>

  <textarea style="form-sizing:normal;">
  The quick brown fox jumps over the lazy dog.
  </textarea>
</div>
```

`form-sizing: normal` をフォーム要素が一定のサイズを保持する動作を無効にし、コンテンツに基づいてサイズが決定されるようになります。

よって上記の例では 1 つめの `<textarea>` は中身が空なので縦横ともにギュッと縮んだ最小サイズになります。一方、2 つめの `<textarea>` はコンテンツ量に応じたサイズが確保されて表示されます。

## Google Chrome Canary で試せます

既に Google Chrome 119.0.6022.0 では `chrome://flags/#enable-experimental-web-platform-features` を有効にすると実験的に `form-sizing` を試せるようになっています。

https://twitter.com/int32_t/status/1705003314218475884

explainer も Chrome での実装も (おそらく) Google Japan の Kent Tamura さんのようです。
せっかくなので手元でも最新の Canary を開いて Codepen でインスタントに試してみました。

![Codepen に example のコードを張り付けて、期待通りに表示されているスクリーンショット](/images/20231003-form-sizing-normal/form-sizing-canary-codepen.png)

こんな感じですね。前述の 1 つめと 2 つめの `<textarea>` の表示のされ方の違いも分かります。

# まとめ

ということで例によって簡単ながら `form-sizing: normal` の CSS プロパティをご紹介しました。

JavaScript を書けても、UI のこういう生臭い処理を書くのはまた別の経験値が必要であり、実際に社内でもこのテの実装に苦しむメンバーもいたので CSS で可能になるならば歓迎したいところ。`<textarea>` に限らず使い所がありそうです。:)

# 関連記事

https://zenn.dev/overflow_offers/articles/20230224-media-queries-nav-controls
https://zenn.dev/offers/articles/20230112-control-ui-customization
https://zenn.dev/offers/articles/20230209-scroll-timeline-for-web-animatinos
