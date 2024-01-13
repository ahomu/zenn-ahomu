---
title: "Next.js App Router と控えめにお付き合いして普通の Web アプリを配信する | Offers Tech Blog"
emoji: "🪔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "react", "frontend", "approuter", "architecture"]
publication_name: "overflow_offers"
published: true
published_at: 2024-01-12 09:30
---

# まずは長いものに巻かれたいときもある

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。

先日 [コードベースのディレクトリ構成にフォーカスした記事](https://zenn.dev/overflow_offers/articles/20231215-directory-structure) を公開した関連記事として、Next.js App Router をどのように取り扱っているかについてご紹介します。

:::message
表題通り Next.js App Router をめっちゃ使いこなすぜイエー、という記事ではありません :)
:::

【AD?】今回の記事の内容を含んだり含まなかったりすると思いますが、来週 2024/01/17(水) 19:20 〜 オンライン開催の PWA Night vol.59 で発表予定なので興味のある方はぜひ。

https://pwanight.connpass.com/event/306410/

## 今回の前提

[前回記事](https://zenn.dev/overflow_offers/articles/20231215-directory-structure#%E4%BB%8A%E5%9B%9E%E3%81%AE%E5%89%8D%E6%8F%90) の引用ですが今回も同様です。

>- 最終目標は単体事業でありつつ実質マルチプロダクトな画面群のリプレース
>- クライアントサイドでヘビーなビジネスロジックを持つ必要がない
>- アプリケーション特性としては SaaS というよりは古典的 Web メディアに近い
>- いわゆる Web フロントエンドエンジニアだけが触るわけではない
>- 既存の巨大 Ruby on Rails モノリスからクライアントサイドを引っぺがす過程

# Next.js App Router の採用

[過去に紹介した技術課題](https://zenn.dev/overflow_offers/articles/20220711-develop-issues-part2) の解消を背景に、既存の Rails スタックを GraphQL サーバーとして活かしつつ、表示側を現代的な Web フロントエンド技術スタックに置き換えることで中長期の開発効率とユーザー体験の改善を図るプロジェクトを進行しています。

現代的な技術スタックとは言ったものの社内ナレッジが Ruby on Rails に寄っているため、私[^1]が手離れした後も Next.js なら「世間一般の情報」を頼りに問題解決しやすいであろう期待が意思決定を誘引した節はあります。

[^1]: 現在の職務上は組織マネジメントが主ですが、一時的にプレイングにカムバック中

## 他の Web フレームワーク等の比較

当時の技術選定を記した ADR を見直すと Next.js、Qwik City、Remix (v1.x)、SvelteKit、Vite ベースの手作り構成...を候補としていくつかの観点で比較していました。

- 社内エンジニアへの要求知識の増分 (特に React 以外)
- スターター構成 (FW のベース部分) の見通しの良さ、簡潔さ
- 成果物のファイルサイズ
  - クライアントサイドバンドルのサイズを絞り込む機構の有無を含む
- Serverless ソリューション向けの成果物から Lambda での実行が簡潔に行えるか

AWS Lmabda 上での SSR を想定して成果物のファイルサイズやビルド・デプロイ・リリースの取り回しの良さも注目しています。結果として [AWS Lambda Web Adapter](https://github.com/awslabs/aws-lambda-web-adapter) を挟めばいずれも実行は問題なく、[AWS のブログ](https://aws.amazon.com/jp/blogs/news/implementing-ssr-streaming-on-nextjs-with-aws-lambda-response-streaming/) のとおり Next.js の Streaming SSR も対応できます。

Next.js がなんだかんだ手堅く整備されていたことや `exports: standalone` による成果物を [fujiwara/lambroll](https://github.com/fujiwara/lambroll) で Lambda にアップロードするだけの爆速デプロイを簡潔に実現できた等の背景から Next.js に落ち着いています。[^2]

[^2]: 気持ち的には Qwik 味見したかった...。

## App Router への逡巡と割り切り

当時は Next.js v13 がリリースされた頃の周辺だった記憶ですが、あらためて調査するほど App Router のエコシステム影響 ([msw](https://github.com/mswjs/msw/issues/1644)、[storybook](https://github.com/storybookjs/storybook/issues/21540)) が明らかになりました。

一方の Pages Router は Next.js を普段からキャッチアップしているであろう [識](https://x.com/matamatanot/status/1697245171510055309?s=20)[者](https://x.com/koichik/status/1697253521115258931?s=20) の話を聞くと、確かではないがいずれ deprecated 相当になってもおかしくない旨の見解を見聞きしました。

App Router の評判がよほど悪ければ Pages Router の寿命が伸びる可能性もありますが、今回は主題でもある「控えめなお付き合い」を前提に App Router の採用をしています。

# 控えめなお付き合いのポイント4点

土台として考えていたことは「Pages Router 時代の延長線上で使いたい = 普通の Web アプリ」ということです。古典的なメディア系 Web のアプローチとして、完成品の HTML を CDN でキャッシュし、クライアントサイドバンドルで UI 制御をします。その他、App Router + React (canary) に備わった機構[^3]の多くを意図的にスポイルすることで必要知識を減らしています。

[^3]: 〇〇は React の機能だから Next.js 固有ではない等の区分は現状あまり意味を成していないため、まとめて Next.js とのお付き合いとしています

## ① サーバーサイドの各種キャッシュ挙動を封じる

App Router で賛否両論の元になりがちなのがパワフルすぎる [キャッシュ](https://nextjs.org/docs/app/building-your-application/caching) ですね。`revalidateTag` で `Surrogate-Key` ヘッダを思い出しますがさておき、今回はキャッシュをアプリケーション層に持たないので下記のように `app/layout.tsx` で [route segment config](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config) を指定します。

```ts
export const fetchCache = 'default-no-store';
export const dynamic = 'force-dynamic';
```

`fetchCache` で例の fetch API のキャッシュを封じて、`dynamic` で static rendering を抑制して、オリジンには常に動的な生成を強制します。`dynamic = 'force-dynamic'` あたりは Pages Router からの移行パスとしても紹介されているくらいなので、今回のようなユースケースでは一応アリと思われます。

App Router が Cache-Control を任意で指定させてくれない点は、こちらは下記の記事のとおり middleware で解決しています。

https://havelog.aho.mu/develop/javascript/e791-nextjs_app_router_apply_cache_control_by_middleware.html

## ② `<Link>` を使わずソフトナビゲーションを封じる

ナビゲーションは **Web ブラウザ (と CDN) を信じろ** を是として `<Link>` を使わないことで SPA 的な [Soft Navigations](https://github.com/WICG/soft-navigations) の代わりに MPA 的な従来的な動作を優先しています。画面遷移するたびにオンメモリな状態 etc を破棄できるのはある意味メリットでしょう。

[react/forbid-elements](https://github.com/jsx-eslint/eslint-plugin-react/blob/master/docs/rules/forbid-elements.md) で `import Link from 'next/link'` を想定した Component 名の安易な利用を抑制します。特別な事情で使う必要があるときは `Link as NecessaryLink` とでもします。

```json
{
  "react/forbid-elements": [true, {
    "forbid": [
      {
        "element": "Link",
        "message": "use <a> element instead"
      }
    ]
  }]
}
```

## ③ `page.tsx` のみ明示的 Server Components として扱う

今回は Ruby on Rails に根ざした GraphQL エンドポイントおよびその裏側の既存資産の流用が前提です。各所で宣言した GraphQL の Fragment[^4] を Server Components たる `page.tsx` に集約してクエリを送る Fragment Colocation パターンを採用しています。結果 `page.tsx` のみが Server Components として振る舞っています。

React Server Components を活用して各所が自律的に fetch する構成も魅力的ではありましたが、現状の Storybook[^5] 等のエコシステムにおいては `page.tsx` 以下のコンポーネントを従来的な React Component として取り扱えるのも副次的なメリットです。

[^4]: GraphQL Code Generator client-preset の [Fragment Masking](https://the-guild.dev/graphql/codegen/plugins/presets/preset-client#fragment-masking) が有効になっていますが使用感よいです
[^5]: Storybook v8 で `experimentalNextRSC` サポートは出たようですが、どのみち GraphQL Fragment で事足りているしなぁ

## ④ RSC + Suspence を使わない

CloudFront を前提とした配信上の HTML キャッシュおよび HTML 生成の概念をシンプルに保つため React Server Components + Suspense は使用していません。普通の...従来的な Web アプリらしくサーバーは 1 枚の完成品 HTML を返却します。

実行環境上の検証は済んでいるので HTML をキャッシュ出来ないログインユーザー向けのアプリケーションに着手する際は段階的に取り入れてみる見込みです。

# 個人的な思想とポイント

そこまでやるなら Next.js じゃなくても良いのでは...？と思う方もいるかもしれませんが、目立った機能を除いてもルーティングやエラーハンドリングなど基本的な部分がよく作り込まれているので十分にメリットを享受できています。

弊社にとっては段階的に Next.js を習得するという過程の序盤であると捉えれば、将来的にパワフルな App Router[^6] を使う選択肢を得られたことの意義もあるでしょう。

[^6]: その頃にはブラッシュアップ＆安定化されているといいんですが😇

## 依存のダメージコントロール

依存パッケージの増加は本当に必要なものだけに極力留めたいことも根底にあります。例えば storybook や msw などの依存パッケージを Next.js と密に結合する形で導入することはメンテナンスの複雑性を加速度的に悪化させます。

https://twitter.com/ahomu/status/1728038273250160778

当初の App Router が各種の依存パッケージとのかみ合わせが悪かったことは、逆にパッケージ間の依存関係 (距離) を程良い加減に留める抑止力となっていました。

## 使わなくて良いパーツの見極め

パワフルなキャッシュ機構をはじめとして、開発時だけでなくデバッグ時にも知識と経験を要する機能は本稿にここまで書いたとおり意図的に封印しています。もちろんフレームワーク固有の Way に乗らないことで生じるリスクもあるので見極めは必要です。

今回は開発者が最もよく触れる React Components で実装と結果を予測するため新たなメンタルモデルを要する要素を主に排除しました。おかげさまで試しに Remix へコードベースを移行テストしたとき[^7]もそれほど苦労せずに載せ換え出来てしまいました。

[^7]: 今回のユースケースにおいて特に性能面で Next.js に満足しているわけでは決してないのです...

# トラディショナルな Web アプリもまだ必要？

出所を忘れましたが Pages Router の既存ニーズも App Router である程度は満たすつもりがありそうなメンテナのコミュニケーションも見かけたので、App Router で普通もとい従来的な Web アプリを構築するパスも整備されていくのではないかと思います。ひいては今回のアプローチも決して邪道というわけではないと願う次第です。

この記事を書くにあたって改めて類似事例を見直しましたが、特に [一休](https://user-first.ikyu.co.jp/entry/2023/12/15/093427) や [サイボウズ](https://speakerdeck.com/mugi_uno/next-dot-js-app-router-deno-mpa-hurontoendoshua-xin) の方とは話題を共有できそうな気がしましたとさ。

# 関連記事

https://zenn.dev/overflow_offers/articles/20231215-directory-structure
https://havelog.aho.mu/develop/javascript/e791-nextjs_app_router_apply_cache_control_by_middleware.html
https://havelog.aho.mu/develop/javascript/e790-app_router_client_components.html