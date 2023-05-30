---
title: "Web の仕様を眺めるシリーズ CSS Anchored Positioning｜Offers Tech Blog"
emoji: "🗻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CSS", "UI", "CSSAnchoredPositioning", "positionset", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2022-06-27 09:30
---

<!-- textlint-disable overflow-techblog/sentence-length -->

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。最近書いた JavaScript は Google Apps Script です。UI 書きたいけどマネージャーという名の雑用忙しいウオオ。

# 最上層に浮かせて表示したい UI の配置を制御する仕様

なんとなく [Chrome Platform Status](https://chromestatus.com/features#browsers.chrome.status%3A%22Proposed%22) から Proposed なステータスのフィーチャーを取り上げて、そのプロポーザルな仕様を眺めてみるシリーズです。本当にシリーズにするかは謎です。

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [CSS Anchored Positioning](https://chromestatus.com/feature/5124922471874560) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/6e77d2f7d7a6a82f9fe386dbb896fe1a266375f4/CSSAnchoredPositioning/explainer.md

## Microsoft Edge チームからの提案

[MicrosoftEdge/MSEdgeExplainers](https://github.com/MicrosoftEdge/MSEdgeExplainers) にあることから Microsoft 社の Edge ないし Chromium 開発チームですかね。[^1]

同じく MSEdgeExplainers で [Enabling Popups](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/Popup/explainer.md) という現在は Archived なスタータスのドキュメントで `<popup>` の提案をした/しようとしていた形跡がありますし、同じ [Open UI Community Group](https://www.w3.org/community/open-ui/) の系譜のようです。

[^1]: Microsoft Edge のレンダリングエンジンは 2020 年 1 月以降、[EdgeHTML](https://ja.wikipedia.org/wiki/EdgeHTML) の代わりに Google Chrome 等で使われている [Chromium](https://ja.wikipedia.org/wiki/Chromium) で提供されています。周知のことかと存じますが念のため...。

## バブル、ポップアップ、ツールチップ等の UI 配置がラクに

CSS Anchored Positioning は、バブル、ポップアップ、ツールチップといったビジュアル的に画面の上層に浮き出てくる系の UI の表示位置について、従来のめんどうくさい処理に代わって宣言的に制御できるようにします。

![左: 上のほうに配置されたアンカー要素を基準として Viewport Boundary までのスペースがある下方向にポップアップメニューが出ている。右: その逆で下のほうに配置されたアンカー要素を基準としてスペースのある上方向にメニューが出ている。](/images/20220627-css-anchored-positioning/use-cases.png)
*[MicrosoftEdge/MSEdgeExplainers](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/CSSAnchoredPositioning/explainer.md) より引用*

前述の Explainer からの引用ですが、こちらの画像を見ると分かりやすいでしょう。セレクトボックスのような UI を表示する際、画面の下方向に十分なスペースがあればそのまま下に配置できますが、なければ上方向に配置するためには従来は JavaScript による制御が必要でした。これが不要になります。

`position: relative` のくびきから逃れるために `</body>` の直前にすべての表示要素に勝るグローバルなポップアップ用の `<div>` 要素等を置いたことがある人も少なくないでしょう。あの工夫も `position: fixed` との併用によって不要になります。[^2]

[^2]: 現在も `<dialog>` が利用可能になってきましたが、カスタム UI を作ろうと思うと力不足です。モーダルな UI を想定している作りなのでツールチップやセレクトボックスにはなりません。

# 本稿執筆時点で提案されている API

ではこれをどのように表現するかというと、執筆時点のプロポーザルによれば次のような使い方が想定されています。

## `anchor` HTML 属性と `position-set` CSS プロパティ

HTML はポップアップのような浮かせたい側の要素に `anchor` 属性で、どの要素を基準位置とするか対象要素の `id` 属性値を指定しています。CSS はポップアップ要素側に `position-set` プロパティで表示位置のパターンを指定しています。

*別のプロポーザルにある `<popup>` 要素が例で使われています。`button#myButton` に指定されている `popup="myPopup"` は「このボタンを押すとこの ID の `<popup>` 要素を開くぞ」という宣言と思われます。あまり本筋に関係ないので架空の Custom Element くらいに考えておきましょう。*

```html:MicrosoftEdge/MSEdgeExplainersより引用（一部割愛して後述）
<button id="myButton" popup="myPopup">Anchor</button>
<popup id="myPopup" anchor="myButton" role="menu">…</popup>
<style>
    #myPopup {
        position: fixed;
        position-set: buttonMenuPos;
        overflow: auto;

        /* The popup is at least as wide as the button */
        min-width: calc(anchor(right) - anchor(left));

        /* The popup is at least as tall as 2 menu items */
        min-height: 6em;
    }

    /* This example intentionally left verbose */
    @position-set buttonMenuPos {
        1 {
            top: anchor(bottom);
            left: anchor(left);
            max-width: calc(100vw - anchor(left));
            max-height: calc(100vh - anchor(bottom));
        }
        2 {
          /* 省略（後述） */
        }
        3 {
          /* 省略（後述） */
        }
        4 {
          /* 省略（後述） */
        }
    }
</style>
```

いったんこれがスタンダードなユースケースと言えるでしょう。仕様を読むと他にも色々書いてありますが、今回は眺めるだけなので深追いしません！

## `@position-set` ルールセットは番号順に試行される

ユーザーエージェント（Web ブラウザ）は例でいう `@position-set buttonMenuPos {...}` における 1 番の指定を試行してビューポートの境界からオーバーフローするようであれば、2 番の指定を試行します。全部失敗したら 1 番を使います。

`anchor()` CSS 関数は、例えば `anchor(bottom)` であれば、ビューポートの上端から HTML 側でアンカーとして指定された要素の下端までの距離を返します。よって `top: anchor(bottom)` はアンカー要素の下端に接することを示し、`left: anchor(left)` はアンカー要素の左端に接することを示します。例では `max-width` や `max-height` も `anchor()` を使って調整しています。

```css:MicrosoftEdge/MSEdgeExplainersより引用（前述から割愛した部分）
/* This example intentionally left verbose */
@position-set buttonMenuPos {
    /* First try to align the top, left edge of the popup
    with the bottom, left edge of the button. */
    1 {
        top: anchor(bottom);
        left: anchor(left);

        /* The popup is no wider than the distance from the 
        left button edge to the right edge of the viewport */
        max-width: calc(100vw - anchor(left));

        /* clamp the height of the popup to be no taller than
        the distance between the bottom of the button and the 
        bottom of the layout viewport. */
        max-height: calc(100vh - anchor(bottom));
    }

    /* Next try to align the bottom, left edge of the popup
    with the top, left edge of the button. */
    2 {
        bottom: anchor(top);
        left: anchor(left);

        max-width: calc(100vw - anchor(left));

        /* clamp the height of the popup to be no taller than
        the distance between the top of the button and the 
        bottom of the layout viewport. */
        max-height: anchor(top);
    }

    /* Next try to align the top, right edge of the popup
    with the bottom, right edge of the button. */
    3 {
        top: anchor(bottom);
        right: anchor(right);
        
        /* The popup is no wider than the distance from the 
        right button edge to left edge of the viewport. */
        max-width: anchor(right);

        max-height: calc(100vh - anchor(bottom));
    }

    /* Finally, try to align the bottom, right edge of the popup
    with the top, right edge of the button. Other positions are possible,
    but this is the final option the author would like the rendering 
    engine to try. */
    4 {
        bottom: anchor(top);
        right: anchor(right);
        
        /* The popup is no wider than the distance from the 
        right button edge to left edge of the viewport. */
        max-width: anchor(right);

        max-height: anchor(top);
    }
}
```

テキストにすると次のような表示イメージです。

1. アンカーの左下とポップアップの左上が接し、アンカー右下方向に拡がるポップアップ
2. アンカーの左上とポップアップの左下が接し、アンカー右上方向に拡がるポップアップ
3. アンカーの右下とポップアップの右上が接し、アンカー左下方向に拡がるポップアップ
4. アンカーの右上とポップアップの右下が接し、アンカー左上方向に拡がるポップアップ

...絶望的に分かりづらいので図も作りました。こんな感じでビューポートの境界に触れない良い感じの位置を探し求めるわけですね。

![直前の箇条書きを図示した画像](/images/20220627-css-anchored-positioning/position-set.png)

ざっくりとした解説でしたが、プロポーザルの意図や目的をなんとなく掴めたのではないでしょうか。Auto Layout with Constraits みがありますね。

# プロポーザルは星の数ほどに

星の数は言いすぎですが生まれては消え、なんなら Chromium には実装されても他のベンダーからはシグナルのない仕様もあるわけで、肩の力を抜いて眺めてみるのもよろしいのではないでしょうか。ではでは。

# 関連記事

https://zenn.dev/offers/articles/20220613-component-props-design
https://zenn.dev/offers/articles/20220523-component-design-best-practice

