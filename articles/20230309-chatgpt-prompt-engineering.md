---
title: "ChatGPT API と遊びながら学んだ Prompt Engineering のメモ | Offers Tech Blog"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["chatgpt", "prompt", "tips"]
publication_name: "overflow_offers"
published: true
published_at: 2023-03-09 09:00
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。

# 猫も杓子も ChatGPT

先日、話題の ChatGPT が API で公開されたことによって、様々な既存処理との統合が容易になりました。プログラミングを嗜める層からすると Web UI と比べると一気に "遊びやすく" なったのではないでしょうか。

https://openai.com/blog/introducing-chatgpt-and-whisper-apis

多聞に漏れずわたくしも HTTP で叩けるならばとウキウキで ChatGPT API を叩いて遊んでおりました。 `gpt-3.5-turbo` まじでレスポンス早いので良いですね。

## API Example

[OpenAI に登録後 API Key](https://platform.openai.com/account/api-keys) を取得して Top-Level Await をサポートする JavaScript 実行環境で下記を叩けばすぐ試せます。手っ取り早いのは Web ブラウザの DevTools の Console にコピペして実行することですが API Key を含むため信頼できる環境でお試しください。

```js
const OPENAI_API_KEY = '（・ｘ・）';
const response = await fetch('https://api.openai.com/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${OPENAI_API_KEY}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    model: 'gpt-3.5-turbo',
    messages: [
        {'role': 'system', 'content': 'You are a helpful assistant.'},
        {'role': 'user', 'content': 'Who won the world series in 2020?'},
    ],
    /**
     * サンプリング操作、0.8のような高い値は出力をよりランダムにし、0.2のような低い値は出力を収束させる
     */
    temperature: .9,
    /**
     * 0.1 は上位 10% の確率の塊からなるトークンのみを考慮することを意味する
     * `temperature` によるサンプリングの代替となるが併用は推奨されない
     */
    top_p: 1,
    /**
     * 各入力メッセージに対して生成するチャット補完の選択肢の数
     */
    n: 1,
    /**
     * presence penalty は少なくとも1回サンプリングされた全てのトークンに適用される1回限り加算する寄与
     * frequency penalty は特定のトークンが既にサンプリングされた頻度に比例する寄与
     */
    presence_penalty: 0,
    frequency_penalty: 0
  })
});
console.log(await response.json());
```

# Prompt Engineering について学んだ TIPS

ChatGPT の Web UI がリリースされてから関連情報を目にする中で知るに至りましたが、最近の Generative AI から良い感じの答え（出力）を引き出すための質問文（入力）をアレコレすることを指して Prompt Engineering と呼称されています。

GPT シリーズほか Generative AI が生まれたのはここ 1-2 年の話ではなく、昨今のブレークスルーに至るまでに体系化されてきた知識を再編するにすぎませんが、簡単に紹介します。最後に参考文献的な URL も掲載しています。

## Role propmpting ーー 役割の明示

プロンプトにおいて役割や性格、演じて欲しい設定を示すことで出力に顕著な変化が見られます。

```js
[
  {"role": "user", "content": '101 * 108 / 32 + 23'}
],

/*
101 * 108 / 32 + 23 という式の答えは、366.0625 で、小数点以下4桁で四捨五入しています。
*/
```

適当な計算式を読み込ませると、それっぽい答えを返してきます。ちなみに計算結果は誤っています。ここで役割を与えてみましょう。

```js
[
  {"role": "system", "content": 'あなたは優秀な数学者であり教師です'},
  {"role": "user", "content": '101 * 108 / 32 + 23'}
],

/*
まず、乗算101 * 108を解きます。
101 * 108 = 10,908

次に、除算10,908 / 32を解きます。
10,908 / 32 ≈ 340.875

最後に加算340.875 + 23を解きます。
340.875 + 23 = 363.875

したがって、101 * 108 / 32 + 23の答えは363.875になります。
*/
```

計算式に対して数学者という設定はオーバーキルな気もしますが...、ともかく役割を与えると回答内容が顕著に変化します。[^1] 教導的に計算ステップを踏むことで、計算結果も正しくなっています。
これは何かしらの作文や要約のタスクを遂行させるときにも有効に働くテクニックです。

[^1]: 確率論的に回答は変化するので、何も役割をセットしなくても教導的な応答をしてくれることも割とあります。あくまで一例ということで。

## Zero/One/Few-Shot learning ーー 期待の例示

対話的にまさしくチャットしながら情報を引き出すのではなく、特定の質問文を元にタスクを遂行させるためのプロンプトに含まれる例示の量で呼び分けされています。

まんま [Best practices for prompt engineering with OpenAI API](https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-openai-api) から引用してしまうのですが下記が例です。

### Zero-shot

タスクに関する説明のみで回答の内容について例示なしで推論させます。シンプルで応答パフォーマンスも高いことが期待されますが、望む回答に近いかどうかの精度・確率は低くなるでしょう。

```
Extract keywords from the below text.

Text: {text}

Keywords:
```

### One/Few-shot

1〜n の例を与えることで推論の精度が向上します。典型的には Zero-shot でうまく動作するかを検証しながら、必要に応じて One/Few-shot で例示による精度の補強が必要かどうかを判断しつつ足していく作業になります。

```
Extract keywords from the corresponding texts below.

Text 1: Stripe provides APIs that web developers can use to integrate payment processing into their websites and mobile applications.
Keywords 1: Stripe, payment processing, APIs, web developers, websites, mobile applications
--
Text 2: OpenAI has trained cutting-edge language models that are very good at understanding and generating text. Our API provides access to these models and can be used to solve virtually any task that involves processing language.
Keywords 2: OpenAI, language models, text processing, API.
--
Text 3: {text}
Keywords 3:
```

## Chain of thought prompting ーー 思考ステップの教示

下記の例は [Prompt Engineering: The Ultimate Guide 2023](https://businessolution.org/prompt-engineering/) からの引用です。
例えばこのような質問文に対する回答を正確に回答できないことがあります。

```
どちらが早いか。

イ: 1000分のバスに乗り、その後30分の電車に乗り、最後に自転車で10分。
ロ: 800分のバスに乗り、その後1時間の電車に乗り、最後に自転車で30分。
```

いわゆる算数の文章問題みたいなやつですね。

```
珍回答1:
比較すると、イの合計時間は1,000/60 + 0.5 + 0.17 = 17.5時間であり、
ロの合計時間は800/60 + 1 + 0.5 = 14.33時間です。したがって、ロがイよりも早いです。
```

```
珍回答2:
イの方が早い。イの移動時間は1,030分(1,000分のバス+30分の電車+10分の自転車)、
一方ロの移動時間は830分(800分のバス+1時間の電車+30分の自転車)である。
```

これに思考ステップを指定すると回答精度が改善します。回答精度の変化は実際に試してみるとよく分かるでしょう。

```
どちらが早いか。

イ: 1,000分のバスに乗り、その後30分の電車に乗り、最後に自転車で10分。
ロ: 800分のバスに乗り、その後1時間の電車に乗り、最後に自転車で30分。

はじめに、それぞれの所要時間を分に変換して合計しましょう。
そのあとイとロを比較して早いほうを判断してください。
```

また [参考記事](https://businessolution.org/prompt-engineering/) によれば「let's think step by step」を Zero-shot の prompt に加えるだけでも精度が高まります。ChatGPT API で試した限りは日本語で「ステップバイステップで考えてください」という指示を足しても同様の効果が得られました。

# まとめ

簡単ですが既存の Tips の紹介でした。総じて「役割」と「期待」と「思考ステップ」をそれぞれ与えていくと順に回答精度を高められるというのが基本的な考え方と言えそうです。
その他の TIPS やプロンプトの考え方のヒントは下記の参考 URL を参照してください。

https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-openai-api
https://businessolution.org/prompt-engineering/
https://learn.microsoft.com/en-us/legal/cognitive-services/openai/transparency-note

# 関連記事

https://zenn.dev/offers/articles/20230306-chatgpt_and_deepl
https://zenn.dev/offers/articles/20221215-how-to-use-chat-gpt

