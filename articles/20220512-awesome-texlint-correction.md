---
title: "テックブログの校正を支える技術 textlint とルール設定について｜Offers Tech Blog"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["textlint", "techblog"]
publication_name: "overflow_offers"
published: true
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) です。「〜を支える技術」的な技術評論社さまの伝統的パンチラインに感化されてしまう世代です。

# 文書校正の自動化でおなじみの textlint

[textlint](https://github.com/textlint/textlint) は [JSer.info](https://jser.info/) や [JavaScript Primer](https://github.com/asciidwango/js-primer) でおなじみの [azuさん](https://twitter.com/azu_re) が開発されている OSS です。

ESLint に代表される各種 Lint ツールの自然言語バージョンです。プレーンテキストや Markdown テキストを対象にいわゆる文章校正を自動的に走らせて、ルールによっては自動修正も適用できます。

弊社のテックブログは [GitHub リポジトリで zenn のコンテンツを管理](https://zenn.dev/zenn/articles/connect-to-github) しており、そのリポジトリ上で textlint を使ってルールに従った文章校正の自動化を試みています。

# GitHub Actions の設定

設定はシンプルで Pull Request 時に CI チェックとして走らせているだけです。textlint + GitHub Actions の設定について最近見た中だと [Markdown 執筆に textlintの自動校正を取り入れる | DevelopersIO](https://dev.classmethod.jp/articles/markdown-writing-with-textlint-ci/) が丁寧解説＆親切実装だったので諸々を委ねます。

```yml
name: textlint

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  textlint:
    name: textlint
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: 14
          cache: npm
          cache-dependency-path: ./package-lock.json

      - name: prepare
        run: |
          npm install

      - name: run
        run: |
          npx textlint articles/* README.md
```

# ルール設定の解説

継続は力なり！ということで週 2 本のペースで公開し続けることを目標に運営しています。当初は私が簡単な校正...Google Document にテキストを張り付けて自動校正をかけて誤字脱字を確認するなどしていましたが、週 2 本だとまぁまぁの作業量になるので自動化したいというモチベーションです。

## 導入済みパッケージ

当テックブログでは下記のパッケージを導入しています。それぞれのパッケージの説明はリンク先の README か、次項の `.textlintrc` の解説を参照してください。

- [textlint-filter-rule-comments](https://github.com/textlint/textlint-filter-rule-comments)
- [textlint-rule-ja-space-around-link](https://github.com/textlint-ja/textlint-rule-preset-overflow-techblog/tree/master/packages/textlint-rule-ja-space-around-link)
- [textlint-rule-preset-overflow-techblog](https://github.com/textlint-ja/textlint-rule-preset-overflow-techblog)
- [textlint-rule-preset-overflow-techblog](https://github.com/textlint-ja/textlint-rule-preset-overflow-techblog)

## `.textlintrc` の解説

textlint の設定は社内のメンバー数名が実際にいくつかの原稿を書き終えた時点で適用しながらルールを決定しました。

テックブログは厳格な文書であることよりも自分たちの技術やキャラクターを外に発信することでフィードバックを受けることのほうが大切と考えているため、テキストを書くのが億劫になっても良くないので各人の文章のクセを踏まえて自動修正できないルールなどは緩めています。

```json
{
  "filters": {
    // textlint-filter-rule-comments の設定
    // 局所的な脱出ハッチのため <!-- textlint-disable xxx-rule --> を有効化
    "comments": true
  },
  "rules": {
    // 日本語におけるスペースの扱いを定義したプリセットルール群
    "preset-overflow-techblog": {
      // インラインコードの前後はスペースをいれたほうが見た目が安定しやすいので有効化
      "ja-space-around-code": {
        "before": true,
        "after": true
      },
      // アルファベットの前後はスペースをいれたほうが見た目が安定しやすいので有効化
      "ja-space-between-half-and-full-width": {
        "space": "always"
      }
    },
    // 日本語の技術文書を念頭においたプリセットルール群
    "preset-overflow-techblog": {
      // 初期設定の100文字制限だと窮屈そうだったので150文字制限に拡張
      "sentence-length": {
        "max": 150
      },
      // テックブログはお堅い文書ではないので感嘆符（！、？）を許可
      "no-exclamation-question-mark": false,
      // 「かも」とか「思います」のような歯切れの悪い表現を許可
      "ja-no-weak-phrase": false,
      // 「あるある」とか「つよつよ」のようなスラングのために単語の連続を許可
      "ja-no-successive-word": false,
      // zenn の Markdown 方言である :::message および :::details と相性が悪いため無効化
      "ja-no-mixed-period": false
    },
    // ja-space-between-half-and-full-width がリンクには反応しないのでこちらでカバー
    // 日本語で構成されているリンクには不要なのだけどやむなく
    "ja-space-around-link": {
      "before": true,
      "after": true
    },
    // JavaScript とか WordPress などの固有名詞を正しく書くためのフレーズ定義
    "prh": {
      "rulePaths": [
        "./prh.yml"
      ]
    }
  }
}
```

## `overflow-techblog/no-doubled-joshi` の壁

ひとつだけ、メンバー各人が頻繁にひっかかり、しかも自動修正できないにも関わらず有効なままにしているルールがあります。それが `preset-overflow-techblog` に含まれる **[`no-doubled-joshi`](https://github.com/textlint-ja/textlint-rule-no-doubled-joshi)** です。

>1つの文中に同じ助詞が連続して出てくるのをチェックするtextlintルールです。
>文中で同じ助詞が連続すると文章が読みにくくなります。
>
>例) でという助詞が連続している
>>材料不足で代替素材で製品を作った。
>
>で という助詞が1つの文中に連続して書かれているのをチェックすることができます。

[README](https://github.com/textlint-ja/textlint-rule-no-doubled-joshi/blob/master/README.md) からの引用ですが、助詞の重複を避けて読みやすい文章にしやすくなります。

中にはあまり違和感のない助詞の重複もありえますが、助詞の連続によって違和感のある文書になってしまうケースのほうが圧倒的に多いのであえて有効なままです。[^1]そういうものだと思って訓練すれば避けられるようになるはずなので心を鬼にしております。

[^1]: textlint 導入時点の既存文書で引っかかった分は修正が手間すぎたので `<!-- textlint-disable overflow-techblog/no-doubled-joshi -->` で回避しました😇

# ブログどんどん書こう！

**自分のもっているネタを発信して外界からフィードバックを得られる楽しさ、嬉しさがやはり醍醐味**だと考えているので楽しみながら書いていければと思います。最近だと以下の記事がよく読まれて、それぞれの著者が嬉しそうにしていたので良かったです。🤗

https://zenn.dev/offers/articles/20220425-universal-attitude
https://zenn.dev/offers/articles/20220418-what-is-bff-architecture
https://zenn.dev/offers/articles/20220411-open-api-schema

なるべくアウトプットを楽しんでもらいたいので、モチベーションを歪ませる個人単位の PV 目標やインセンティブ[^2]は設けていません。代わりに全体 PV と連動で n 円（実は具体額未定、そろそろ決める）の宴会予算？が積み立てられるようなチームインセンティブを設計しています。

なお副業参画の方がテックブログにご寄稿いただくときは別途執筆フィーを設定しています。興味がある方はジョイナス！（強引な採用オチ）

[^2]: 当方、安易な金銭インセンティブによるモチベーション設計に厳しい派です。

