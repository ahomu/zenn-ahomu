---
title: "Web の仕様を眺めるシリーズ Document Picture-in-Picture (PiP)｜Offers Tech Blog"
emoji: "🍳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PiP", "requestPictureInPictureWindow", "Web仕様眺め👀"]
publication_name: "overflow_offers"
published: true
published_at: 2022-08-15 09:30
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。


本記事は [Chrome Platform Status](https://chromestatus.com/features#browsers.chrome.status%3A%22Proposed%22) からなんとなく Proposed なステータスのフィーチャーを取り上げて、そのプロポーザルを眺めてみるシリーズです。前回は [CSS Anchored Positioning](https://zenn.dev/offers/articles/20220627-css-anchored-positioning) でした。

# HTMLVideoElement 以外も Picture-in-Picture したい

:::message
取り上げる「Web の仕様」は Web ブラウザベンダーや WebDev コミュニティにおいて必ずしも十分なコンセンサスがとられた標準仕様とは限らず、各 Web ブラウザのベンダーが現在から将来に渡って ship することを執筆時点において一切保証しません :)
:::

今回は [Document Picture-in-Picture](https://chromestatus.com/feature/5755179560337408) を眺めてみます。本当に眺めるだけで深入りしないので概要のみのライトな記事とご認識ください。

https://github.com/steimelchrome/document-pip-explainer/blob/e4c811a12bb21a7d9e21c4bc1fbd2e0c181ec873/explainer.md

## 専用の Window を丸ごと PiP すればいいじゃない！

Picture in Picture (以下 PiP) は動画コンテンツ等を画面隅などに置かれる小さな表示領域で独立して再生させる一種のマルチウィンドウ UI です。現在の PiP は `HTMLVideoElement.requestPictureInPicture()` ということで `<video>` 要素に紐付く形で実装されています。

Document Picture-in-Picture は `window.requestPictureInPictureWindow()` として動画だけでなく、Window 単位でまるまる PiP できるようにすればいいじゃない！という提案です。既存の `window.open()` と同じような同一生成元ウィンドウを想定しつつも、セキュリティ上の理由から幾つかの制約を伴うデザインとなっています。詳細は後述。

なお `window.open()` に近しいのであれば `alwaysOnTop` フラグを追加すればよいのでは？という問いについては、新しい API を新設することで feature Detection によるフォールバックをしやすくする意図があると [説明されています](https://github.com/steimelchrome/document-pip-explainer/blob/main/explainer.md#since-this-is-pretty-close-to-windowopen-why-not-just-add-an-alwaysontop-flag-to-windowopen)。

## カスタムされたメディアコントロールや字幕などを表示する用途

https://discourse.wicg.io/t/proposal-document-picture-in-picture/5736

議論の中でユースケースとしては、カスタムされたメディアコントロール（再生・停止など）や字幕コンテンツ（ニコニコ動画的なのとか）が挙げられています。スクロール可能なプレイリストなども挙げられていますが、専用の `window.document` をまるまる生成するので動画以外のユースケースでも基本的には何でもありでしょう。

従来の `<video>` 要素に紐付く PiP ではブラウザネイティブのコントロールや VTT しか扱えないので、前述したようなカスタム UI は利用できません。一方で `Element.requestFullscreen()` は特定の要素を親として丸ごとフルスクリーン表示に転送できるのでカスタム UI が使い放題です。PiP も同じように `Element.requestPictureInPicture()` でよいのでは？と思うフシもありますが explainer では次のように書かれています。

>Why not extend the HTMLVideoElement.requestPictureInPicture() idea to allow it to be called on any HTMLElement?
>
>Any API where the UA is taking elements out of the page and then reinserting them ends up with tricky questions on what to show in the current document when those elements are gone (do elements shift around? Is there a placeholder? What magic needs to happen when things resize? etc). By leaving it up to websites to move their own elements, the API contract between the UA and website is much clearer and simpler to understand.

大雑把にいうと、ブラウザ側で親ウィンドウの一部を子ウィンドウに貸し与えるモデルだと子に転送された要素群を親でどのように扱うか考えなければならなくなるので、いっそ子にコンテンツを転送するなり作成するなりは Web サイト側のコードで明示的にコントロールさせるほうがシンプルである、的な趣旨です。

過去にも [w3c/picture-in-picture](https://github.com/w3c/picture-in-picture/issues/113) で議論があったようですが、こちらは `Element.requestPictureInPicture()` のようなイメージだったようです。

# 本稿執筆時点で提案されている API

仕様を見た限り結構大胆なアイディアのようにも思えますが、ここからは動作上の制約やコードサンプルを追います。

## PiP Window の制約

PiP Window は `interface DocumentPictureInPictureWindow` として定義されます。あくまで PiP であることから親ウインドウよりも長く存続することがない等、いくつかの制約が挙げられています。

>- The PiP window will never outlive the opening window.
>- The PiP window can only be opened as a response to a user gesture.
>- The website cannot set the position of the PiP window.
>- The PiP window cannot be navigated (any window.history or window.location calls that change to a new document will close the PiP window).
>- The PiP window cannot open more windows.
>- The PiP window must be populated via JS (i.e., cannot be loaded via URL).
>- The UA can restrict the size of the PiP window.
>- The UA can restrict input on the window.
>
>[steimelchrome/document-pip-explainer](https://github.com/steimelchrome/document-pip-explainer/blob/main/explainer.md) より引用

実装イメージを掴む上で重要なのは*PiP window must be populated via JS (i.e., cannot be loaded via URL)* とある通り URL を指定してロードする代わりに、JavaScript を介して手動でコンテンツを挿入する必要があることでしょう。

## `window.requestPictureInPictureWindow()` メソッド

前述の制約を踏まえるとサンプルコードは素直に読み取れると思うのでざっくり [サンプルコードを引用](https://github.com/steimelchrome/document-pip-explainer/blob/e4c811a12bb21a7d9e21c4bc1fbd2e0c181ec873/explainer.md#example-code) して終えます。

```html
<body>
  <div id="player-container">
    <div id="player">
      <video id="video" src="foo.webm"></video>
      <!-- More player elements here. -->
    </div>
  </div>
  <input type="button" onclick="enterPiP();" value="Enter PiP" />

  <script>
  let pipWindow = null;

  function enterPiP() {
    const player = document.querySelector('#player');

    const pipOptions = {
      initialAspectRatio: player.clientWidth / player.clientHeight,
      lockAspectRatio: true,
    };

    window.requestPictureInPictureWindow(pipOptions).then((_pipWin) => {
      pipWindow = _pipWin;

      // Style remaining container to imply the player is in PiP.
      playerContainer.classList.add('pip-mode');

      // Add styles to the PiP window.
      const styleLink = document.createElement('link');
      styleLink.href = 'pip.css';
      styleLink.rel = 'stylesheet';
      const pipBody = pipWindow.document.body;
      pipBody.append(styleLink);

      // Add player to the PiP window.
      pipBody.append(player);

      // Listen for the PiP closing event to put the video back.
      window.addEventListener('leavepictureinpicture', onLeavePiP, { once: true });
    });
  }

  // Called when the PiP window has closed.
  function onLeavePiP(event) {
    // Remove PiP styling from the container.
    const playerContainer = document.querySelector('#player-container');
    playerContainer.classList.remove('pip-mode');

    // Add the player back to the main window.
    const player = event.pictureInPictureWindow.document.querySelector('#player');
    playerContainer.append(player);

    pipWindow = null;
  }
  </script>
</body>
```

1. `window.requestPictureInPictureWindow()` で `pipWindow` を取得する
2. `pipWindow.document.body` に手動で必要な要素を `append()` する
3. `leavepictureinpicture` イベントに PiP が閉じたときの後処理を記述する

おおよそこのような流れで、単純なユースケースにおいてはシンプルな使い勝手の API と思えます。[キーシナリオ](https://github.com/steimelchrome/document-pip-explainer/blob/main/explainer.md#key-scenarios) もステップバイステップで想定される使用方法と挙動の参考になります。

# ユーザーに有益な使い道を考えるのはおもしろいかも

`Element.requestFullscreen()` 等と同じように、ユーザーインタラクション由来によってのみ実行できる API であることを踏まえつつ、どういうユースケースがあるか考えるのは面白いかもしれません。ユーザーの滞在時間の長い SaaS 等もなにかしら使い道の発明があるかもしれませんね。（本当に実装されるかはさておき）

# 関連記事

https://zenn.dev/offers/articles/20220627-css-anchored-positioning
https://zenn.dev/offers/articles/20220711-develop-issues-part2

