---
title: "Web フロントエンドの新規コードベースと推しディレクトリ構造 | Offers Tech Blog"
emoji: "🌳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["monorepo", "frontend", "nextjs", "turborepo", "architecture"]
publication_name: "overflow_offers"
published: true
published_at: 2023-12-18 17:00
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。暖冬と言われつつもすっかり寒い季節ですね。おかげさまで割と走っているほうの師です。（師走）

# n 年ぶり n 回目の Web フロントエンド

最後にメイン開発者の立場でコードをスクラッチしたのいつだったっけ？と遡ると [2018年ごろのブログ記事](https://havelog.aho.mu/develop/others/e750-scratch_memo_3.html) がでてきました💀 実際には 2017 年から 2018 年にかけての作品ですかね。当時の構想から読み取れる重厚かつ自己表現の感に内心苦笑いしつつ久々の新規建立です。

今回はディレクトリ構造の面から紹介していきます。

## 推しディレクトリの先達たち

推しディレクトリという言葉に乗っかってみたものの、ゴメンそこまでの熱感は持っていないかもしれない🥺 とはいえ先達の記事もご紹介しておきます。

https://zenn.dev/knowledgework/articles/99f8047555f700
https://zenn.dev/sakito/articles/af87061a5016e6

## 今回の前提

本稿において、これらの前提に依存した論はほとんど含まれない認識ですが一応のコンテキストとして記載しておきます。

- 最終目標は単体事業でありつつ実質マルチプロダクトな画面群のリプレース
- クライアントサイドでヘビーなビジネスロジックを持つ必要がない
- アプリケーション特性としては SaaS というよりは古典的 Web メディアに近い
- いわゆる Web フロントエンドエンジニアだけが触る**わけではない**
- 既存の巨大 Ruby on Rails モノリスからクライアントサイドを引っぺがす過程

# 現在のディレクトリ構成（要約版）

まずはこちらが現在のディレクトリ構成を `tree` コマンドで吐き出して記事用に編集したものです。後述のとおり奇をてらうことなく無難な構成をしています。（たぶん）

```
.
├── apps
│   ├── acme-app  # 任意の粒度における1アプリケーション
│   │   ├── public
│   │   └── src
│   │       ├── __generated__  # graphql-codegen の出力先
│   │       ├── app
│   │       │   ├── xxx
│   │       │   │   └── [xxxId]
│   │       │   └── yyy
│   │       │       └── [yyySlug]
│   │       ├── components
│   │       │   ├── Breadcrumbs
│   │       │   ├── Footer
│   │       │   ├── Header
│   │       │   └── ...
│   │       ├── features
│   │       │   └── acme-feature
│   │       │       ├── components
│   │       │       │   └── AcmeComponent
│   │       │       │       ├── AcmeComponent.module.css
│   │       │       │       ├── AcmeComponent.stories.css
│   │       │       │       ├── AcmeComponent.test.tsx
│   │       │       │       ├── AcmeComponent.tsx
│   │       │       │       └── index.ts
│   │       │       ├── hooks
│   │       │       ├── providers
│   │       │       └── utils
│   │       ├── hooks
│   │       ├── mocks
│   │       ├── providers
│   │       └── utils
│   └── storybook
├── docs
│   └── adr   # ADR 置き場
└── packages  # 共通パッケージ置き場
    ├── eslint-config-offers
    ├── md2html
    ├── prettier
    ├── stylelint-config-offers
    ├── test-utils
    ├── tsconfig
    └── ui    # デザインシステム相当の実装置き場
        ├── src
        │   ├── Avatar
        │   ├── Button
        │   ├── Checkbox
        │   ├── Dialog
        │   └── ...
        ├── static
        │   └── icon
        └── tokens
```

## `apps/**`

マルチプロダクト的なコードベース管理を志向して apps 内に独立した複数のアプリケーションを配置されることを想定しています。直近のアプリケーションは Next.js ですが、理屈としては Remix や Qwik など他のフレームワークを利用したコードが配置される可能性もあります。

いったん今回は、直近のあんまり複雑でないアプリケーションの例です。

### Next.js App Router

直近は Next.js App Router アプリケーションなので [file-system based router](https://nextjs.org/docs/app/building-your-application/routing/defining-routes) に紐付く `app` ディレクトリが配置されています。`src` ディレクトリが無いと何となく気持ちが落ち着かなかったので一応挟んであります😇

### Separating features

feature、domain、 concern、context ...命名の悩ましさはありつつ一旦 feature としているのが関心事による縦割り分類です。求人応募、ブックマーク、求人情報表示、のような単位でざっくりと切り分けています。

```
.
└── features
    └── acme-feature
        ├── components
        │   └── AcmeComponent
        │       ├── AcmeComponent.module.css
        │       ├── AcmeComponent.stories.css
        │       ├── AcmeComponent.test.tsx # 操作インタラクションがあれば必須
        │       ├── AcmeComponent.tsx
        │       └── index.ts
        ├── hooks     # 状態ロジックの追い出し先 (コンポーネントテストで済ませがち)
        ├── providers
        └── utils     # 表示ロジックや制御ロジックの追い出し先 (ユニットテスト必須)
```

features 以下は特段のモデルを整備しているわけではなく、Components から適宜 `hooks` や `utils` としてテストしやすい単位を念頭にロジックを切り出してすだけの単純な構成です。表示上のフォーマット処理などは `utils` のユニットテストで担保し、コンポーネントテストはインタラクション（またはビヘイビア）のテストに関心を限定しています。

### Layering components

コンポーネントのレイヤリングは下記のとおり。feature 別のコンポーネントについては 1 ファイル 200 行以内を目安に適宜コンポーネント分割を試みるくらいのゆるふわ運用です。

| コンポーネント種別 | 責務 | fetch | test | 
| ---- | ---- | ---- | ---- | 
| App Router Page | ページ単位レイアウトと全体データ取得<br><br>必要に応じ Suspense や Hydration する | Server Components のみ | しない | 
| Feature in App | アプリ内の関心事ごとの UI と機能の束<br><br>必要に応じ `use client` で境界を宣言  | Client Components のみ | やる | 
| Common in App | アプリ内で広く共有される UI と機能<br>`<Breadcrumbs>`、`<Header>` など<br><br>必要に応じ `use client` で境界を宣言 | Client Components のみ | やる | 
| Design System UI | 基礎ビジュアルと操作性の一貫性を担保<br>`<Button>`、`<Dialog>` など<br><br>`@offers-www/ui` Internal Package<br>今のところ `use client` 宣言なしで運用 | なし | やる |

fetch 周りは根元の Server Components (`page.tsx`) と、末端の Client Components のいずれかに寄せることで全体をテストしやすくしています。

#### ディレクトリツリー上の配置

1. Next.js App Router Page Components
    - `apps/acme-app/src/app/**/page.tsx`
2. Feature Compoents in App
    - `apps/acme-app/src/features/**/components/*.tsx`
3. Common Components in App
    - `apps/acme-app/src/components/*.tsx`
4. Design System UI Compoents
    - `packages/ui/**/*.tsx`

app 内で共通のコンポーネントは `hooks` 等と合わせて `features/shared` にまとめるほうが扱いの違いを階層で暗黙的に示そうとするより自明かもしれない点は検討中です。

## `packages/**`

packages 以下はインターナルパッケージとして pnpm の workspace 機能を介して参照されます。うまく育てていきたい所ですが、影響範囲が広い依存関係を産むので注意が必要です。

例えば eslint のベース設定は下記のように参照し、個別に extends して必要に応じて上書きをします。同様に prettier や stylelint もベース設定を切り出しています。

```json
// package.json
{
  "devDependencies": {
    "@offers-www/eslint-config-offers": "workspace:*"
  }
}
// .eslintrc.json
{
  "extends": ["@offers-www/eslint-config-offers"],
}
```

## `docs/adr/*.md`

個人的に関わりのある Resilire さんで ADR の書き方を~~パクって~~学んできたので、後世に鋭いもので刺されないよう [ADR (Architecture Decision Record)](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) を書き溜めています。

ADR については別の場所でケーススタディを紹介しているので、こちらの記事もよろしければご覧ください。

https://resilire-engineer.hatenablog.com/entry/2023/12/18/163335

# コンセプト「無難」

コンセプトがどうこうではありませんが、万事を無難にまとめようと努めています。Web フロントエンドにおける"現代的構成"が中長期にわたって無難かと問われると何とも言い難いところですが…。

## Turborepo のスターターを踏襲

`pnpm dlx create-turbo@latest`

今回の構成では pnpm workspace を前提として Turborepo のスターターをまるっと踏襲しています。前述の `apps` `packages` の分類も独自に考案したものではなく無難に踏襲しています。

前回も workspaces/packages で分けてパッケージの独立境界は引いていたのでマイナーアップデートの範疇ではあります。

https://github.com/vercel/turbo/tree/main/examples/basic
https://speakerdeck.com/mh4gf/pnpm-workspaceshi-jian-nouhau

## Colocation 志向

コンポーネント内のファイル配置に限らずですが、全体的に Colocation を大事にしています。拡大解釈すれば feature も Colocation に近しいものがありますし無難ですね。

GraphQL も Fragment Colocation を使用しています。規模的にヘビーユースする程でないですが悪くない使用感を得ています。

https://kentcdodds.com/blog/colocation
https://www.mizdra.net/entry/2022/12/11/203940

## 上から順に読めるコードベース

トリッキーな実装の混入や処理の流れが大きくジャンプすることを避け、コードベースを上から順に読んで適宜参照を追えば無難に理解できる状態の維持も優先度が高いテーマです。

コードベースを理解する上での初見殺しを仕込まないのは、副業でスポット稼働してもらいやすくするためにも必要な観点でしょう。いいトシなので `ぼくのかんがえたさいきょう` は自重します。

# リプレース初期段階（絶賛開発中）

今回紹介したコードベースは [Offers](https://offers.jp/) の某所を置き換えるべく段階的にリリースされている最中です。使用感も良くなっていくと思うので利用者の皆さまにおかれましてはご期待ください。

- dependencies を切り詰めて清貧を尊ぶ編
- Next.js を薄く浅く反抗的に使う編
- Design System UI Components 建立編
- 限られたリソースの現実的なテスト戦略編

機会があればこのあたりのトピックもどこかで記事にするかもしれません。ではでは。

# 関連記事

自分のブログにも小ネタを書いていたので、こちらでも紹介しておきます。

https://havelog.aho.mu/develop/javascript/e791-nextjs_app_router_apply_cache_control_by_middleware.html
https://havelog.aho.mu/develop/css/e789-display-contents-utility-component.html
https://havelog.aho.mu/develop/javascript/e790-app_router_client_components.html
