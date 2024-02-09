---
title: "Web フロントエンドのテストと持続可能な方針の組み立て | Offers Tech Blog"
emoji: "⚖️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["test", "vitest", "msw", "react", "frontend"]
publication_name: "overflow_offers"
published: true
published_at: 2024-02-09 12:00
---

[Offers](https://offers.jp/) を運営している株式会社 [overflow](https://overflow.co.jp/) の [あほむ](https://twitter.com/ahomu) でございます。

今回はプロジェクトで Web フロントエンド領域のテストを書くにあたって方針を決めた際の ADR をブログ向けに再整理したものをお届けします。

# テストコードを書くべきか書かざるべきか

逃げ切りが確約された作り捨ての納品プロジェクトでもなければ、継続的なメンテナンスを前提に**テストコードは書くべき**が現代のソフトウェアエンジニアにおける共通了解でしょう。

急がば廻れ、ほとんどの場合においてテストコードを書かないことによるメリットがデメリットを上回るものと捉えられています。ここでは書かなくても良いケースをあえて論じることをしませんが、個別具体でテストが不要と断定できるときはそうすればよいでしょう。

## テストを整える工数をどう捉える

TDD (_Test Driven Development テスト駆動開発_) に代表される、**テストコードを通して要件を洞察し設計を堅牢たらしめる開発プロセス**は工数をかける価値のある取り組みです。

習熟した開発者は息をするように当然のものとしてテストコードを書きます。アクセシビリティ巧者が当然のように操作性に配慮した UI を実装できるのと同様です。

一方でテストを書いて実行する環境が整っていない、テストが構造的にやりづらい、コーディング的に面倒さが残るなどの不便は「時間が無い」という理由を生み出し、テストを書け vs 時と場合によるが勃発します。[^1]

[^1]: 「テストを書こう」は正義であり「時と場合による」も正論なのでどうしようもない。

## 書くことそのものより持続可能な計画

言葉遊びのようですがテストにおいてコードを書くことではなく計画のほうが重要です。テストコードを書かなくても相応の効果が得られるならば良く、それは開発者の負担を減らしソフトウェアの品質担保における持続可能性に寄与するものです。AI でテストシナリオを作成し自動実行するサービス等が分かりやすい例と言えます。[^2]

[^2]: お金と時間に余裕があるならば手動でテストを請け負ってくれる会社に業務を委託するのもよいでしょう。

今のところ開発者が自力で書くべきテストコードは少なくありませんが、それらも最小で最大の効果を得られるように計画することが重要です。最善を尽くそうと E2E をセッティングしたものの全く使わなかったり、あっという間に廃墟になったりしたプロジェクトは誰にでも思い当たる節があるでしょう。

# 何を対象にどのようなテストをするべきか

全くテストしないは論外ですが、あらゆる角度からテストしすぎて同じものをテストしている、何をテストしているのか分からない、テストコードが複雑になるなどの不利益も持続可能な計画のためには避けるべきです。

## 先人の知恵と方針づくり

Web フロントエンドないし JavaScript の文脈において古くは [Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html) や近年だと [Testing Trophy](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications) がよく言及されています。先人の知恵を参考にしつつも現実のテスト戦略（または方針）はクリティカルなユースケースの有無や、立ち上げフェーズまたは運用フェーズか、チームの規模、メンバーの習熟度、開発リードの趣味などで異なります。

今回のプロジェクトは、典型的な Web メディア + ログインユーザー向けの操作 (但し売上に関わるため絶対落とせないものを含む) が少々のアプリケーションを私を含む 1-2 名による新規開発です。以後は技術領域を限定しない複数名の Web エンジニアでメンテナンスされるイメージです。身も、蓋もないことを言えば、多少の表示崩れの可能性に目を瞑ってログインユーザー向けのユースケースだけケアすれば致命的な事態は避けられそうです。

## テスト方針としての DO/DON'T とカバレッジ

ADR の皮算用時点では下記のような方針になりました。プロジェクトでは React と Next.js を利用しています。VRT や E2E といった維持コストの高い仕組みは優先度を下げています。

| テスト種別 | 優先 | Do | Don't |
|---|---|---|---|
| Integration Test | ✓ | ユーザー操作を伴う機能の正常系をテストする（必要に応じて異常系、例外も） | スタイル等のビジュアル的なテストおよび機能を伴わないコンポーネントのテストはしない |
| Unit Test | ✓ | 独立した Hooks や utils の入出力を通してロジックをテストする | private 相当のテストや、Integration Test で間接的にカバーされる特定のコンポーネントに依存した Hooks のテストはしない |
| Visual Regression Test | | ビジュアル的な変更を検知してテストする | 画像比較でカバーできないテストすべて |
| E2E Test | | クリティカルなユーザーストーリーの正常系をテストする | テキストの有無のような単純な出力結果、クリティカルでないストーリーのテストはしない |
| Static Test | | TypeScript の型チェック、eslint、stylelint でカバーできるコード規約チェックをテストする | 静的検査でカバーできないテストすべて |

Integration Test について補足すると、今回はページ全体を対象にこそしないものの複数コンポーネントを組み合わせたユースケースを担う部分（これもコンポーネント）に対して実施するイメージです。小さい粒度のコンポーネントはテストを必要とせず、ロジックを含むときは Hooks などに抽出して単純な Unit Test に変換します。

| テスト観点 | Integration | Unit | Visual Regression | E2E | Static |
|---|---|---|---|---|---|
| ユーザー向けの振る舞い | ✓ |  |  | ✓ |  |
| ビジネスロジック、その他ロジック |  | ✓ |  |  |  |
| ビジュアル上の整合性 |  |  | ✓ |  |  |
| コードベース上の整合性 |  |  |  |  | ✓ |

観点に対するカバレッジを大まかに整理するとこのようになります。E2E がサーバーサイドを含めた外形実行であり、Integration はモックによってフロントエンドの実装内で完結しうる前提確認ということでダブルチェックが成り立っています。

# テストを実行する技術スタック

ここではテストごとの技術スタックを簡単に紹介していきます。テスト環境を構築するための詳細はインターネット上の他の記事を参照ください。

## Integration Test —— クリティカルを抑える要

[vitest](https://vitest.dev/) に [happy-dom](https://github.com/capricorn86/happy-dom) を添えて、[@testing-library/react](https://testing-library.com/) を使用するオーソドックスな構成です。モックには [msw](https://mswjs.io/) を差し込んでいます。

jsdom は [Dialog Element のサポートが暗礁に乗り上げていた](https://github.com/jsdom/jsdom/pull/3403#issuecomment-1565259525) ため、少なからず実装のある happy-dom のほうを採用しています。

```ts
// vitest.config.ts
export default defineConfig({
  plugins: [viteReact()],
  test: {
    globals: true,
    environment: 'happy-dom',
    setupFiles: './src/vitest-setup.tsx',
    css: {
      include: /.+\.module\.css/,
    },
    env: {
      NEXT_PUBLIC_GRAPHQL_ENDPOINT: 'https://example.com/api/graphql',
      NEXT_PUBLIC_BASE_URL: 'https://example.com',
    },
  }
});
```

モック周りだけ簡単なユーティリティを提供していますが、基本は testing-library の標準的な API に沿ってコンポーネントの振る舞いをテストしています。

```tsx
it('ダイアログを invoke ボタンで開閉できる', () => {
  mockServer.use(mockLoggedIn(true));
  mockServer.use(mockRecommendContents());

  const { dialog, invokeButton } = renderDialog();
  expect(dialog).not.toBeVisible();
  fireEvent.click(invokeButton);
  expect(dialog).toBeVisible();
  fireEvent.click(invokeButton);
  expect(dialog).not.toBeVisible();
});

it('invoke ボタンに適切な属性が付与されている', () => {
  const { invokeButton } = renderDialog();

  expect(invokeButton).toHaveAttribute('aria-haspopup', 'dialog');
  expect(invokeButton).toHaveAttribute('aria-expanded', 'false');
  expect(invokeButton).toHaveAttribute('type', 'button');

  fireEvent.click(invokeButton);
  expect(invokeButton).toHaveAttribute('aria-expanded', 'true');
  fireEvent.click(invokeButton);
  expect(invokeButton).toHaveAttribute('aria-expanded', 'false');
});
```

一度環境さえ作ってしまえば効果的なテストを記述できるため要所では積極的に適用しています。単なる表示の整合性などは Integration Test の関心から外していて、ロジックは単純な関数に追い出していくことで Unit Test の比率を上げています。

## Unit Test —— 低コストやはり本命

Integration Test と同様に単に vitest を使用しています。単純な関数や Hooks を [`renderHook`](https://testing-library.com/docs/react-testing-library/api/#renderhook) でテストする程度なのでやはり面倒がありません。

```ts
test('formatWorkingPlace', () => {
  expect(formatWorkingPlace('REMOTE')).toBe('リモート');
  expect(formatWorkingPlace('OFFICE')).toBe('オフィス');
  expect(formatWorkingPlace('FLEXIBLE')).toBe('相談の上決定する');
});
```

ユニットテストは特に GitHub Copilot の恩恵を感じられる箇所です。私が使用しているのは RubyMine ですが[^3]、テスト対象の関連ファイルや既存のテストケースが含まれるファイルをタブで開いておけば、`it(...)` などでテストケースの説明を正しく書けば高い精度でテストコードが生成されます。

[^3]: 全体としては Ruby on Rails プロジェクトなので WebStorm のままでは居られなくなっている (´ω`)

## Visual Regression Test —— 必要なときだけローカル実行

VRT は CI で都度回すには鈍重な仕組みであり、周辺環境の整備コストもかかるので本当に必要なときだけローカル上で実行しています。スタックとしては `storybook` と `storycap`、`reg-cli` の単純な組み合わせです。

```json
{
  "scripts": {
    "dev": "storybook dev -p 6006",
    "shot-a": "storycap http://localhost:6006 -o __shots__/a",
    "shot-b": "storycap http://localhost:6006 -o __shots__/b",
    "shot-diff": "reg-cli __shots__/a __shots__/b __shots__/diff -R __shots__/report.html -M 0.0001 -T 0.0001"
  }
}
```

例えば global.css のように広域に影響する CSS の変更や、複数箇所で使われている UI コンポーネントの表示パターンの変更などが意図した影響範囲におさまっているかを確認します。

Storybook そのものはシンプルに利用しており、さらに生々しく書くことによって [CSF](https://storybook.js.org/docs/api/csf) (_Component Story Format_) のアップデートに対する耐性を上げています。上がっていると良いなぁ。

```tsx
import * as Icon from './index';
export default {
  title: 'ui/Icon',
};
export const _Icon = () => {
  return (
    <div className="container">
      <h1>Icon</h1>
      <p>
        {Object.entries(Icon).map(([key, Component]) => {
          return (
            <Component
              name={key}
              style={{ margin: '4px', width: '48px', height: '48px' }}
              key={key}
            />
          );
        })}
      </p>
    </div>
  );
};
```

## E2E Test —— 転ばぬ先の杖？

Playwright を以前から運用しているので適宜シナリオを追加します。ほか特筆すべきがないので割愛。テスト対象に Flaky 化を恐れるほどの複雑性はありませんが、現状において転ばぬ先の杖として積極的に運用するほどではないという所感です。

## Static Test —— 現代の嗜み

現代の嗜みとして Static Test 類はプロジェクト共有の設定を読み込んで eslint、stylelint、type-check を行っています。TypeScript の設定は strict 路線です。

```json
{
  "devDependencies": {
    "@own-project/eslint-config-offers": "workspace:*",
    "@own-project/prettier": "workspace:*",
    "@own-project/stylelint-config-offers": "workspace:*",
    "@own-project/tsconfig": "workspace:*",
  }
}
```

あわせて [evilmartians/lefthook](https://github.com/evilmartians/lefthook) などの導入も考えるところですが、CI できちんと検知できている限りローカル開発での強制はお節介であると考えて未導入です。

# 理想は必要十分で軽やかなテスト環境

テスト環境の整備もクライアントサイドバンドルのファイルサイズ影響こそ無いものの依存を肥大化させ、メンテナンスコストが高騰する温床です。また堅牢すぎるテスト環境は CI/CD の速度およびデリバリーサイクルに負の影響を与えてしまいます。

>- 依存パッケージのアップデート
>- アップデートするためのアップデート
>- アップデートするためのアップデートのためのアップデート
>- バージョン整合性パズルがしばしば開催される
>
>via. [控えめな App Router と持続可能な開発 - PWA Night vol.59](https://speakerdeck.com/ahomu/kong-emena-app-router-tochi-sok-ke-neng-nakai-fa?slide=32)

テスト環境のメンテナンスコストにどれくらい時間を割けるかは人やチームによって異なりますが、便利そうなテスト環境のための依存関係を無闇に増やすよりは、愚直に書いてシンプルに運用できるほうが持続可能観点で望ましいと考えています。Storybook はアドオンと合わせて色々できてしまうので特に怖い。

ではでは。

# 関連記事

https://havelog.aho.mu/misc/e792.html
https://speakerdeck.com/ahomu/kong-emena-app-router-tochi-sok-ke-neng-nakai-fa
https://zenn.dev/overflow_offers/articles/20231215-directory-structure
