---
title: "Web の仕様を眺めるシリーズ Control UI Customization | Offers Tech Blog"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CustomControlUI", "ControlUICustomization", "OpenUI", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2023-01-12 09:00
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。

本記事は [Chrome Platform Status](https://chromestatus.com/features#browsers.chrome.status%3A%22Proposed%22) からなんとなく Proposed なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [Capability Delegation](https://zenn.dev/offers/articles/20221201-capability-delegation) でした。

# Control UI Customization

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [Feature: Customizable <select> Element](https://chromestatus.com/feature/5737365999976448) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/c0c0db33240a92173d5cedd916dbd053c4b37660/ControlUICustomization/explainer.md

## Open UI 方面からの提案

以前紹介した CSS Anchored Positioning と同じく [MicrosoftEdge/MSEdgeExplainers](https://github.com/MicrosoftEdge/MSEdgeExplainers) に提案書があります。Author の所属も多くが Microsoft 社であり、半数は CSS Anchored Positioning の Author でもあります。当該ドキュメントを眺める限り、今回も [Open UI Community Group](https://www.w3.org/community/open-ui/) に絡んだ提案でしょう。

https://zenn.dev/offers/articles/20220627-css-anchored-positioning

## ビルトインコントロール UI のカスタマイズ

Chrome Status 上では「Customizable `<select>` Element」という表題の登録ですが、総体としては Control UI Customization です。その中から `<select>` における実装にフォーカスしたチケットであるという理解です。今回はせっかくなので大元の Explainer に沿って紹介します。

`<select>` や `<input type="checkbox">` のブラウザネイティブな Built-in UI Controls の代わりに、`<div>` や CSS、JavaScript を駆使して UI を再実装することは Web UI の開発者であれば誰もが一度は通った道でしょう。

![DevTools Element Pane のスクリーンショット。select要素の中のshadow-rootに配置されたslotが、select要素の子要素であるoption要素を引き取っている](/images/20230112-control-ui-customization/select-shadow-root.png)
*`<select>` 要素内の Shadow Root*

これらの Built-in UI Controls の要素や見た目は、例えば Chrome DevTools で Show user agent shadow DOM を有効にして Element から Inspect すると Shadow DOM でカプセル化されていることが分かります。カプセル化されているためドロップダウンメニューに対してスタイルを柔軟に適用するなどは叶いません。

既存の枠組みにおいてカスタマイズ不能だったこれら UI のスタイルを、いくつかの方法によってカスタマイズ可能にするのが今回紹介する Control UI Customization の主旨です。

## 構造、振る舞い、パーツ命名の標準化

Built-in Control UI の中身はユーザーエージェントに委ねられていて標準化されていなかったので、仮に Pseudo Classes/Elements でアクセス可能にしたとしても構造や命名についてユーザーエージェント間の互換性を保証する土台がない状態でした。

[Open UI](https://open-ui.org/) プロジェクトでは `<select>` ドロップダウンメニューやチェックボックス、ラジオボタン、日付ピッカー、カラーピッカー etc Web における各種の UI Controls の構成要素や状態、振る舞いの標準化を進めています。

https://open-ui.org/components/select

このドキュメントは `<select>` の例ですが、このような Open UI の取り組みによって外側から Built-in UI Control の見た目をカスタマイズするための土台が整いつつあります。

# 本稿執筆時点で提案されている API

以下のサンプルコードはすべて [MSEdgeExplainers/ControlUICustomization/explainer.md](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/c0c0db33240a92173d5cedd916dbd053c4b37660/ControlUICustomization/explainer.md) のからの引用（一部改変）です。

>- Using standardized parts and states to override the native control styles via pseudo-classes and pseudo-elements.
>- Using named <slots> to replace parts of the native control UI with developer content, while optionally leaving some native parts in place.
>- Replacing the entire UI of the native control with an author-supplied shadow root.

パーツや状態ごとのスタイルを疑似要素、疑似クラスでオーバーライド可能にしたり、名前のついた `<slots>` を通して部分的に置き換えたり、独自の実装で Shadow Root を丸ごと置き換えたりすることが想定されています。

## 疑似クラスと疑似要素でスタイリングする

下記のサンプルコードでは `<select>` 要素内の `<button>` パーツ要素[^]のスタイルに `::part` 疑似要素セレクタでアクセスしています。また、`:open` 疑似クラスセレクタで `<select>` 要素が `open` 状態であるときのスタイルにもアクセスしています。

```html
<style>
  .styled-select::part(button) {
    background-color: red
  }
  .styled-select:open::part(button) {
    /* Change the color of the button when the <select> dropdown is open */
    background-color: lightgray;
  }
</style>
<select class="styled-select">
  <option>choice 1</option>
  <option>choice 2</option>
</select>
```

[^1]: いわゆる普通の HTML Elements との区別をするため、本稿では便宜上「パーツ要素」としています。

## 名前付き Slots による要素の置き換え

下記のサンプルコードでは `slot="button"` と `slot="listbox"` によって `button` パーツ要素と `listbox` パーツ要素を Light DOM 上の自前実装で置き換えています。

```html
<style>
  .custom-button { /*...*/ }
  option { /*...*/ }
  .option-text { /*...*/ }
</style>
<select>
  <div slot="button" part="button" class="custom-button">Choose a pet</div>
  <div slot="listbox" part="listbox" class="custom-listbox">
    <option>
      <img src="./cat-icon.jpg"/>
      <div class="option-text">Cat</div>
    </option>
    <option>
      <img src="./dog-icon.jpg"/>
      <div class="option-text">Dog</div>
    </option>
  </div>
</select>
```

Open UI では `<select>` 要素の内部構造は次に示す図のように [定義](https://open-ui.org/components/select#diagram-1) されています。この構造図において前述のサンプルコードは `button-container [slot]` および `listbox-container [slot]` にアクセスしているイメージでしょう。

![select をルートとして、button と listbox があり、button の中には selected-value と marker があり、listbox の中には opt-group > option があるという構造で定義および命名されている図](/images/20230112-control-ui-customization/select-element-diagram.png)
*`<select>` 要素内の構造図*

`part` 属性によって本来あったパーツ要素と、どのカスタムされた要素が対応関係にあるかを示しています。これを目印として Control UI の Controller がネイティブのイベントハンドラを適用するなどのイメージです。自作 Control UI でありがちな各種の振る舞いやアクセシビリティを提供する JavaScript の記述[^2]が不要になります。

[^2]: 既存の UI コンポーネントの類が頑張って再実装しているビヘイビアをネイティブコードに預けられるので、各種のデザインシステムとそれに紐付く UI コンポーネントライブラリの実装にも良い影響が期待されます。

## 任意の Shadow DOM と丸ごと交換する

部分的な置き換えに留まらず、`shadowRoot` ごと丸ごと置き換えるパターンです。次のサンプルコードでは JavaScript で Shadow DOM を置き換えています。サンプルコードでは適当なテキストを `innerHTML` に代入していますが、実際には前項と同じように `part` 属性で `button` および `listbox` パーツ要素の明示が要求されます。

```javascript
let customSelect = document.createElement('select');
customSelect.setAttribute("custom", "");
let selectShadow = customSelect.attachShadow({ mode: 'open' });
selectShadow.innerHTML = `My custom select UI`;
document.body.appendChild(customSelect);
```

現時点では JavaScript による置き換えが例示されていますが、Declarative Shadow DOM ([dom](https://github.com/whatwg/dom/pull/858)/[html](https://github.com/whatwg/html/pull/5465)) によって JavaScript を必要としなくなるはず、とのことです。


# まとめ

簡単ですが Control UI Customization の紹介でした。整合性や実装上の課題等、もろもろ検討＆議論中の仕様ではありますが実現したいことは明確ですし、Web 開発者としても待望と言って良い取り組みといえるのではないでしょうか。

Control UI Customization は見た目のカスタマイズを認めつつも `part` 属性によって標準構造に従ったパーツ要素が正しく存在することを要求します。それによってカスタムされた Control UI におけるアクセシビリティセマンティクスや標準的な振る舞い（ユーザー入力に対するイベントハンドラ）が期待される完全な状態を保つようにデザインされています。

アクセシビリティ周りについては [Ensuring accessibility by default](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/c0c0db33240a92173d5cedd916dbd053c4b37660/ControlUICustomization/explainer.md#ensuring-accessibility-by-default) に書いてあるので、興味がある方は参照してください。

ではでは 👋

# 関連記事

https://zenn.dev/offers/articles/20220920-edit-context-api
https://zenn.dev/offers/articles/20220815-document-pip-proposal
https://zenn.dev/offers/articles/20220711-develop-issues-part2

