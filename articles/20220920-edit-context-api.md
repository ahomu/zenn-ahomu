---
title: "Web の仕様を眺めるシリーズ EditContext API | Offers Tech Blog"
emoji: "🎹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["EditContext","API", "contenteditable", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2022-09-20 09:30
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。


本記事は [Chrome Platform Status](https://chromestatus.com/features#browsers.chrome.status%3A%22Proposed%22) からなんとなく Proposed なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [prefers-reduced-data](https://zenn.dev/offers/articles/20220901-prefers-reduced-data) でした。

# Web におけるテキスト入力をシンプルに統合する API

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [EditContext API](https://chromestatus.com/feature/5041440373604352) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

https://github.com/w3c/editing/blob/d4a128b77c15fb4e0f3fae1a81a87c61d1d06ffd/docs/EditContext/explainer.md

>大昔お世話になった contenteditable 周辺の事情にも革命が起こらないものだろうかと思う今日この頃です。

[CSS Custom Highlight API](https://zenn.dev/offers/articles/20220818-css-custom-highlight-api) を取り上げたときこのように書きましたが、Chrome Platform Status を眺めていたらそれっぽい API があったので取り上げてみます。


## Microsoft Edge チームからの提案

すでに [w3c/editing](https://github.com/w3c/editing) に移っていますが、以前は [MicrosoftEdge/MSEdgeExplainers](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/EditContext/explainer.md) にドキュメントがあったようで、Chrome Platform Status における Owner にも Microsoft 社のメールアドレスが名を連ねています。

おそらくは Microsoft Edge チームからのものと考えてよいでしょう。.NET に [EditContext](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.forms.editcontext?view=aspnetcore-6.0) というクラスがあることから、そこから着想か命名を得ているかも？

## Web におけるテキスト編集 UI はトリックに満ちている

Web プラットフォームにおけるテキスト入力の手段として `<input>` `<textarea>` `contenteditable` がありますが、カスタマイズされた任意の入力 UI を実現し、付随する操作体験を十分なものにするためにはこれらの要素を直接使えることは稀です。

`contentedibale` が手に余るシロモノなのは経験者ならご存じのとおりですが、何にせよ表示の制御とテキスト入力の制御が相互に依存するため、自由度にも限りがあります。入力操作を受け取る編集用の editable な要素と表示用の要素を分離するような実装例もありますが、これも [アクセシビリティ](https://github.com/w3c/editing/blob/d4a128b77c15fb4e0f3fae1a81a87c61d1d06ffd/docs/EditContext/explainer.md#accessibility-issues-in-the-monaco-editor) の問題を誘因します。

[Real-world Examples of Text Input Issues in Top Sites and Frameworks](https://github.com/w3c/editing/blob/d4a128b77c15fb4e0f3fae1a81a87c61d1d06ffd/docs/EditContext/explainer.md#real-world-examples-of-text-input-issues-in-top-sites-and-frameworks) の項を見ると分かりますがリアルタイムコラボレーションや IME との兼ね合いなど数々の課題があります。

## テキスト入力の受け口を DOM の外に持つ

先に挙げた `<input>` `<textarea>` `contenteditable` はすべて DOM 更新と直結した受け口です。テキスト入力を受け取る内部的なテキストバッファに更新があると、ブラウザや OS で定義されたデフォルト挙動に基づいてユーザー向けプレビューと機能する DOM を更新します。

![contentedibaleとEditContextの違いを表す図](/images/20220920-edit-context-api/contentEditable_vs_editContext.png)
*[w3c/editing/explainer.md より引用](https://github.com/w3c/editing/blob/d4a128b77c15fb4e0f3fae1a81a87c61d1d06ffd/docs/EditContext/explainer.md#difference-between-contenteditable-element-and-the-editcontext-element)*

EditContext はテキストバッファを外側に持ち、テキスト入力に関するイベントを DOM と切り離します。代わりにイベントを通じてユーザー向けプレビューである DOM　をどのように更新すべきかを教えてくれます。EditContext が切り離されることでテキストの入力状況と DOM 更新の密結合が疎になり、カスタムされた入力 UI の実現と高度な操作体験が副作用を持つトリック無しで無理のなく実装できるようになります。

# 本稿執筆時点で提案されている API

下記はすべて [w3c/editing/native_selection_demo.html](https://github.com/w3c/editing/blob/d4a128b77c15fb4e0f3fae1a81a87c61d1d06ffd/docs/EditContext/native_selection_demo.html) のサンプルコードからの引用（一部改変）です。

## サンプルコード

次のサンプルコードでは EditContext を初期化して、div に関連づけています。最終的にユーザー向けプレビューを生成するためにも DOM との結びつけが必要です。

```js
var editView = document.getElementById("editView");
var editContext = new EditContext();
editView.editContext = editContext;
editContext.text = editView.textContent;
```

次のコードでは、テキストの更新があったときに `textupdate` イベントが発生し `e.updateText` を編集位置に差し込んで更新しています。元のサンプルコードにはテキスト更新にあたって composition が発生しているときのパターンもあるのでご参考までに。

```js
editContext.addEventListener("textupdate", e => {
    let s = document.getSelection();
    let textNode = s.anchorNode;
    let string = textNode.textContent;
    // update the text Node
    textNode.textContent = string.substring(0, e.updateRangeStart) + e.updateText + string.substring(e.updateRangeEnd);
    // update the caret
    s.collapse(s.anchorNode, e.newSelectionStart);
    console.log(`After textupdate: textContent[${s.anchorNode.textContent}]`);
});
```

非常に簡単な一部抜粋のみのご紹介としていますが、次項のように実行できるサンプルコードがあるので興味がある方は手元で動かしてみるのが早いです。

## Chromium と `#enable-experimental-web-platform-features` フラグ

EditContext は Chromium で `about:flags` から `#enable-experimental-web-platform-features` フラグを有効にすると手元でも実行できます。
前述したサンプルコードの元である [w3c/editing/native_selection_demo.html](https://github.com/w3c/editing/blob/d4a128b77c15fb4e0f3fae1a81a87c61d1d06ffd/docs/EditContext/native_selection_demo.html) を開くと `😂😃😄一二三 hello world すし美味しいです` という謎のメッセージが目に入ります。

composition 周りの挙動チェックの文脈でしょうか...うーん、すしは美味しい。わかる。

# まとめ

ということで壮大な仕様の割りに、だからこそ、あっさり紹介せざるをえなかった EditContext さんでした。

イベント駆動で整然と書けそうな感覚がある一方で、実際にテキスト入力のカスタム UI を作ってみないと真のありがたみを実感できないような気もしつつ、そういう UI を作りたいかと言われると...元気なときにやりたいですね。

# 関連記事

https://zenn.dev/offers/articles/20220901-prefers-reduced-data
https://zenn.dev/offers/articles/20220815-document-pip-proposal
https://zenn.dev/offers/articles/20220711-develop-issues-part2

