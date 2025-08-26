---
title: "はじめてのモノレポ構築記録 – pnpm × Turborepo × AWS で少しずつ整えてみた"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","Monorepo","Turborepo","zennfes2025free"]
published: false
---

# **はじめに**
## 本記事の目的
本記事では、私がこれまでに構築してきたモノレポ環境の仕様と設計について整理し、技術的な背景や採用したツール群を体系的にまとめます。  
この環境は、複数のアプリケーションと共通パッケージを効率的に管理することを目的としており、CI/CD・インフラ・コード品質・開発体験（DX）までを含めた統合的な開発基盤を目指しています。  

単なる備忘録に留めず、同じようにモノレポを導入しようとしているエンジニアやチームにとって、参考になる実践的な知見を共有することを狙いとしています。

## 対象読者
- モノレポ環境の導入を検討している個人開発者や小規模チーム  
- AWS・GitHub Actions・pnpm・Turborepo といったモダンな開発スタックに関心のあるエンジニア  
- セキュリティチェックや CI/CD を組み込んだ統合的な開発基盤を設計したい人  
- DevContainer や VS Code を活用して、ローカル開発と CI 環境を統一したい人  

## モノレポを採用した背景
従来の複数リポジトリ管理では以下の課題がありました。  

- 各アプリケーション／ライブラリごとに依存関係やビルド環境を個別管理する必要がある  
- 共通コードやユーティリティの再利用が煩雑になり、更新やバージョン管理が分散してしまう  
- CI/CD の設定がリポジトリごとに分散し、運用コストが高くなる  

これらを解決するためにモノレポを採用しました。  
モノレポ化することで以下の利点を得ています。  

- **依存関係の一元管理**：pnpm workspace によってライブラリやアプリ間の依存を効率的に解決  
- **ビルド・タスクの高速化**：Turborepo によるキャッシュと並列実行で開発効率を向上  
- **統合的なCI/CD**：GitHub Actions を中心に、dev / prod 環境に応じたデプロイを自動化  
- **コード品質とセキュリティ担保**：ESLint, Prettier, Husky, Gitleaks などによる開発段階での品質チェック  
- **統一された開発体験**：DevContainer と VS Code により、ローカルとCI環境を一致させ再現性を確保  

これらの背景を踏まえ、本記事では「仕様」と「設計」という2つの観点からモノレポ環境を整理していきます。

# **モノレポの全体仕様**
## 管理するアプリケーションとパッケージの構成
最初は「アプリケーションと共通ライブラリを一緒に管理したい」というシンプルな動機からスタートしました。  
個別リポジトリで管理していたときは、ちょっとした変更でも複数のリポジトリを行き来する必要があり、効率が悪いと感じていました。  

そこで、モノレポでは以下のような構成を採用しました。  

- `apps/` : 実際に動作するアプリケーションを配置  
  - 例: Web フロントエンド, API サーバー  
- `packages/` : 複数のアプリケーションから利用する共通パッケージやライブラリを配置  
  - 例: infra（インフラ定義）, ui（共通UIコンポーネント）, utils（共通ユーティリティ関数）  

この「apps」と「packages」の2階層で整理することで、最初から過度に複雑にせず、必要に応じて拡張していけるようにしました。

---

## ワークスペースの基本方針
ワークスペース管理には **pnpm workspace** を採用しました。  

理由は次の通りです。  
- node_modules を賢く共有してくれるため、ディスク容量とインストール時間を節約できる  
- アプリとライブラリ間の依存関係を `pnpm-workspace.yaml` でシンプルに定義できる  
- 最初の小さな構成から大規模な拡張まで対応できる柔軟性がある  

「まずは動くものを作りたい」という気持ちで選びましたが、結果的に後から追加するライブラリやアプリもスムーズに統合できています。

---

## 開発環境の前提条件
開発環境を揃えるのは、重要なポイントでした。  

- **エディタ**は VS Code を採用し、全員が同じ開発体験を得られるようにしました  
- **DevContainer** を用意して、ローカル環境がどこでも再現できるようにしました  
- Node.js のバージョンや AWS CLI などのツール群もコンテナで統一し、環境依存のトラブルを減らしました  

このおかげで「環境構築でつまずく」ことがほぼなくなり、開発に入れるようになっています。  

---

## 振り返り
こうして全体仕様を整理すると、当初は「とにかく一つのリポジトリにまとめる」ことから始めたのですが、結果的に  
- **apps と packages の役割分担**  
- **pnpm workspace による依存関係管理**  
- **DevContainer による再現性の高い開発環境**  

といった要素が自然と揃っていきました。  

大きな設計をいきなり目指すよりも、**「小さく始めて後から拡張する」** という流れが自分には合っていたと感じています。

# 使用しているツール群の一覧

モノレポを構築するにあたり、さまざまなツールを試しました。  
最初からすべてを揃えていたわけではなく、「これが必要かも」と思ったタイミングで追加していきました。  
ここでは現在利用しているツール群と、それぞれを導入した理由・役割をまとめます。

---

## 開発環境
- **VS Code**  
  エディタは VS Code を標準としました。拡張機能が豊富で、TypeScript や AWS 開発との相性も良いためです。  
- **DevContainer**  
  開発環境をチームや CI と揃えるために導入しました。Node.js や AWS CLI などのツールをコンテナにまとめ、環境差異によるトラブルを避けています。  

---

## 生成AI
- **Amazon Q Developer CLI**  
  開発中の試行錯誤を補助するために活用しています。タスクやコード生成のサポートを受けながら、モノレポの整備を進めやすくしています。  

- **Chat GPT Plus**
  セカンドオピニオン的に利用しています。こちらはローカルのエージェントとして利用しているのでなく、方針などを壁打ちするために利用しました。スマホアプリで移動中などに頭の整理をできるのが最大の利点だと感じています。

---

## パッケージ管理
- **pnpm**  
  workspace 機能が強力で、モノレポとの相性が非常に良いです。  
  npm や yarn よりインストール速度が速く、ディスク容量も節約できます。  

---

## タスクランナー
- **Turborepo**  
  モノレポ内のタスクを効率的に管理するために導入しました。  
  キャッシュ機能のおかげで、変更がない部分のビルドをスキップでき、開発効率が大幅に改善しました。  

---

## インフラ管理
- **AWS CDK**  
  現状はまだ S3 バケットの構築にとどまっていますが、将来的に IaC (Infrastructure as Code) を本格導入するために選択しました。  
  TypeScript で書けるため、フロントや API と同じ言語で統一できる点も魅力です。  

---

## CI/CD
- **GitHub Actions**  
  プルリクエストやブランチへの push をトリガーに、Lint やテスト、デプロイを自動化しています。  
  OIDC を使った AWS 認証を組み込み、シークレット管理をシンプルにしました。  

---

## コード品質
- **ESLint**  
  TypeScript/JavaScript の静的解析を行い、バグを防ぎます。  
- **Prettier**  
  コードフォーマットを統一し、レビュー時の無駄な差分を減らしました。  
- **Markdownlint**  
  ドキュメント（README やブログ記事）の整形ルールを統一するために利用しています。  

---

## セキュリティ
- **Gitleaks**  
  誤って API Key やパスワードをコミットしないよう、シークレットスキャンを行います。  
  Husky の pre-commit フックと連携して、自動的にチェックされるようにしました。  

---

## テスト
- **Jest**  
  基本的なユニットテストのフレームワークとして採用しました。  
- **Vitest**  
  フロントエンド側のテストでは Jest より軽量な Vitest も試しています。  
- **React Testing Library**  
  React コンポーネントの挙動をユーザー目線で検証するために導入しました。  

---

## Git Hooks
- **Husky**  
  Git フックを簡単に設定できるツールです。  
  `pre-commit` で Lint や Gitleaks を実行し、問題があればコミットできないようにしています。  

---

## 振り返り
こうして見ると、モノレポの仕様を支えているのは「一度に導入したツール」ではなく、必要に応じて少しずつ追加してきたものばかりです。  
最初は **pnpm + Turborepo** の最低限のセットから始め、後から **ESLint, Prettier, Husky, Gitleaks** を足していきました。  

初心者としては「いきなり完璧なスタックを組む」のは難しいですが、段階的に導入していくことで結果的に今のような構成に育ちました。

# ディレクトリ構成とワークスペース設計

## apps/ と packages/ の役割
モノレポを始めたときに一番迷ったのが「どのようにディレクトリを分けるか」でした。  
最初は全部を一つのフォルダに入れていましたが、だんだん管理が大変になってきたので、以下のような構成に落ち着きました。

- **apps/**  
  実際に動くアプリケーションを置く場所です。  
  例:  
  - `web/` (フロントエンドSPA)  
  - `api/` (APIサーバー)  

- **packages/**  
  複数のアプリで使う共通モジュールやライブラリを置く場所です。  
  例:  
  - `infra/` (CDKによるインフラ定義)  
  - `ui/` (共通Reactコンポーネント)  
  - `utils/` (共通ユーティリティ関数)  

この分け方はシンプルですが、**「アプリ」と「ライブラリ」を物理的に分ける」**ことで頭の整理がつきやすくなりました。  

---

## pnpm-workspace.yaml の定義
pnpm では `pnpm-workspace.yaml` を使って、どのフォルダをワークスペースに含めるかを指定します。  
私の設定はシンプルに以下のようになっています。

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

こうすることで、apps/ と packages/ 配下のプロジェクトが自動的にワークスペースとして認識されます。
これによって、例えば apps/web が packages/ui に依存する場合でも、ローカル開発時には直接リンクされるので、変更がすぐ反映されるのが便利です。


## モジュール間依存関係の方針

初心者のうちは「依存関係がぐちゃぐちゃになるのでは？」という不安がありました。
そこで、自分なりに以下のルールを置きました。
apps → packages への依存はOK
アプリはライブラリを使う想定なので依存してよい
packages → apps への依存はNG

共通ライブラリがアプリに依存すると循環参照になりやすいため禁止
packages 同士の依存は最小限に
どうしても必要な場合だけ依存する（utils など汎用ライブラリは例外）
こうしたルールを決めたことで、依存関係が複雑になりすぎず、管理がしやすくなりました。


## 振り返り

最初は「モノレポって難しそう」と思っていましたが、実際にやってみるとapps と packages を分けるだけでも頭が整理できる、pnpm-workspace.yaml を書くだけで依存管理が楽になる、と、意外とシンプルに始められることがわかりました。
このあたりから「モノレポって実は初心者にも優しいのでは？」と感じ始めました。



# タスク実行とキャッシュ設計

モノレポを導入したときに次に悩んだのが、**「複数パッケージのビルドやテストをどう効率よく回すか」**でした。  
単純に `pnpm -r build` のように全パッケージを毎回実行すると、変更がないパッケージもビルドされてしまい、時間が無駄になります。  

そこで導入したのが **Turborepo** です。  
Turborepo の便利な点のひとつに、**Git のヒストリを使って差分を検出し、必要なパッケージだけをビルド対象にする**仕組みがあります。  

通常、`pnpm run build` のように全体をビルドすると、変更がないパッケージまで毎回処理されてしまいます。しかし Turborepo は Git のコミット履歴を参照し、「今回の変更で影響を受けるパッケージ」を特定します。  

また、ビルドした結果をキャッシュすることもでき、依存解決の仕組みを活用することで、実際の開発効率が変わることが期待できます。

---

## Turborepo によるタスク定義

Turborepo では、ルートに `turbo.json` を配置してタスクを定義します。  
私の最初の設定はシンプルに以下のようなものでした。

```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", "build/**"]
    },
    "lint": {
      "outputs": []
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "dev": {
      "cache": false
    }
  }
}
```
ここでのポイントは以下です。

- build タスク
自分自身のビルドだけでなく、依存しているパッケージのビルドも実行される。Turborepoのデフォルトの挙動では依存関係を考慮せずに並列にビルドが行われるため、dependsOnによる制御が必要となります。

- lint / test
キャッシュ可能だが成果物を残さないので outputs は空

- dev
ローカル開発用なのでキャッシュを無効化

この最小構成だけでも「どのタスクがどのパッケージに依存しているか」を明示でき、全体を見渡しやすくなりました。

## 並列実行と依存解決

Turborepo はタスクを並列で実行してくれます。
例えば turbo run build を実行すると、依存関係を解決した上で、独立しているパッケージのビルドは同時に走ります。

これによって「待ち時間が長いビルド」も並列化され、従来の直列実行より時間短縮が改善が見込めます。小さな改善に見えて、日々の開発体験が向上しました。

## キャッシュ戦略（ローカル・リモート）

Turborepo の最大の魅力は キャッシュ機能 です。

### ローカルキャッシュ
直前に実行したタスクの結果を保存し、同じ入力なら再実行をスキップします。
前回ビルドからコードが変わっていなければ、次回は「キャッシュヒット」で瞬時に完了します。

### リモートキャッシュ
（未導入ですが）将来的には CI との連携で活用予定です。開発者のローカルと CI でキャッシュを共有できるようになれば、CI のビルド時間が劇的に短縮されるはずです。

キャッシュ戦略を導入しただけで、特にビルドの繰り返しにかかる時間が短縮されました。


## 実際のコマンド例
-すべてのパッケージでビルド

```
pnpm run build:all
```
→ turbo.json の設定に従って、依存解決＋並列実行が走ります。

- 特定のパッケージだけ実行

```
pnpm --filter @frommiddle/web build
```

## 振り返り
最初は「タスクランナーってなくてもいいのでは？」と思っていました。
しかし、モノレポのように複数アプリ・複数パッケージを管理すると、**「どのタスクをいつ実行すべきか」**を明確に定義しないとすぐに破綻します。

Turborepo を入れてみた結果、依存関係を意識せずタスクを実行できる安心感

キャッシュによる大幅な時間短縮ローカルと CI/CD の一貫性といったメリットを体感できました。

「まずは build と lint と test を定義する」程度でも十分に効果があるので、初心者が最初導入するツールとしてもおすすめできます。

# インフラ設計（現状）

モノレポの中で扱うインフラは、まだ大規模なものではありません。  
現時点では「環境ごとに S3 バケットを作成し、アプリの成果物を配置できるようにする」という最小限の仕組みにとどまっています。リソースを展開する際にCI/CDを前提として、リソース名を適切にコントロールする仕組みづくりにトライしてみました。  

---


## 名前付けの方針
名前付けの方針として **開発環境（dev）** と **本番環境（prod）** を分けられるようにする、ということを重要視しました。またAWSアカウントはステージごとに用意されているので、それらの切り替えを意識してリソースの中にはAWSアカウントIDも付与するように設計しています。ステージ名を明示的に付与することで、リソースの混在を避けるようにしています。
具体的な書式としては以下です。
`{固有prefix}-{stage}-{accountid}-{project名}-{service名}-{リソース種別}`

例:  
- `assets-dev-123456789012-myproj-web-bucket`  
- `assets-prod-210987654321-myproj-web-bucket`  

## 実装上の工夫1：Meta 情報を「props」で受け渡す設計

実装で考慮したのは、**ステージ（dev/prod）やアカウントID**のような「横断的なメタ情報（Meta）」を、  **どこで定義して、どうやって各スタック／コンストラクトに伝えるか**という点でした。

最初はスタックやコンストラクトの内部で `process.env` や `cdk.Context` を直接読む実装にしていましたが、  だんだん「どこで値が決まっているのか分かりづらい」「テストしづらい」という問題が出てきました。

そこで方針を改めて、**Meta を型で表現し、スタック／コンストラクトには `props` 経由で明示的に渡す**設計に統一しました。

## 実装上の工夫2:Meta 情報の解決順序（コンテキスト > 環境変数）

もうひとつの実装上の工夫として、Meta 情報を取得するときに **「CDK の context」 と 「環境変数」** を両方考慮し、  **context > 環境変数の順で優先する**ようにしました。

### なぜこの順序か？
- **context を最優先**  
  - `cdk deploy --context stage=prod` のように実行時に明示的に指定した値を尊重する  
  - CI/CD やローカルで意図的に上書きしたいケースに柔軟に対応できる  
- **環境変数を fallback に**  
  - `process.env.STAGE=dev` のように普段の開発環境で自動的に設定される値を使う  
  - context が指定されていない場合にデフォルトのように効く  

これらを考慮して簡単な関数を作成し、命名関数に渡すことでCDK実行時の環境情報を統一的に入力できるようにしました。この関数はエントリポイントのコードでのみ呼び出す方針として、後段のコンストラクトにはpropsでその値を伝えるルールを設けました。

## リソース命名のコードスニペット

```ts
import type { Construct } from 'constructs';

export interface Meta {
  stage: string;
  project: string;
  service: string;
}

/** stage / project / service を ①context ②環境変数 ③デフォルト の順に取得 */
export function getMeta(scope: Construct): Meta {
  const node = scope.node; // ← ここを Stack.of(scope).node から変更

  const stage = (node.tryGetContext('stage') as string | undefined) ?? process.env.STAGE ?? 'dev';

  const project =
    (node.tryGetContext('project') as string | undefined) ?? process.env.PROJECT ?? 'myproject';

  const service =
    (node.tryGetContext('service') as string | undefined) ?? process.env.SERVICE ?? 'web';

  return { stage, project, service };
}

```

上記のgetMetaで生成したメタな情報を部分的に受け取る前提として作った名前作成のコードが以下です。AWSはリソースによって名前の上限調があるので、そういった値も受け取れるように実装しました。

```ts
import { Environment } from 'aws-cdk-lib';
import { Meta } from './meta-util';

export interface NameBuilderOptions {
  prefix?: string; // 例: bucketNamePrefix
  maxLen?: number; // 例: S3 は 63
  requireStartEndAlnum?: boolean; // 先頭末尾を英数字にする（S3向け）
  includeAccount?: boolean; // account を入れるか
  includeRegion?: boolean; // region を入れるか
}

/**
 * メタデータとenv情報を受け取ってリソース名を生成する関数
 */
export function nameBuilder(
  meta: Meta,
  env: Environment,
  resourceType: string,
  opts: NameBuilderOptions = {},
): string {
  const {
    prefix,
    maxLen = 63,
    requireStartEndAlnum = true,
    includeAccount = true,
    includeRegion = false,
  } = opts;

  const { stage, project, service } = meta;

  // セグメントを構築
  const segments = [
    prefix,
    stage,
    includeAccount ? env.account : undefined,
    project,
    service,
    resourceType,
    includeRegion ? env.region : undefined,
  ].filter((v): v is string => !!v && v.length > 0);

  // 名前を生成
  let name = segments
    .join('-')
    .toLowerCase()
    .replace(/[^a-z0-9-]/g, '-') // 非許可文字→-
    .replace(/-{2,}/g, '-'); // 連続ハイフン畳む

  // 先頭末尾のハイフンを除去（必要に応じて）
  if (requireStartEndAlnum) {
    name = name.replace(/^-+/, '').replace(/-+$/, '');
  }

  // 最大長で切り詰め
  name = name.slice(0, maxLen);

  // 再度先頭末尾のハイフンを除去し、空文字ガード
  if (requireStartEndAlnum) {
    name = name.replace(/^-+/, '').replace(/-+$/, '');
    if (!name) name = 'x0'; // 全滅ガード
  }

  return name;
}

```
## CDK ユーティリティの独立とバージョン管理の工夫

インフラ部分で共通的に使う **命名関数や Meta 解決関数** は、せっかくモノレポでワークスペースを利用しているので、  
`packages/cdk-utils` のように **独立したパッケージ** として切り出しました。  
これによりスタックやコンストラクトのコードがシンプルになり、再利用性も高まります。

---

### バージョンずれの課題

ただしこのユーティリティ内では `aws-cdk-lib` をインポートしているため、  
他のインフラパッケージ（例: `packages/infra`）と **CDK のライブラリバージョンがずれる可能性** がありました。  

モノレポ内で CDK のバージョンが不揃いになると、`cdk synth` や `cdk deploy` が不安定になったり、型定義が一致せずエラーになることもあります。

---

### peerDependencies で整合性を担保

そこで `packages/cdk-utils/package.json` では `aws-cdk-lib` と `constructs` を **peerDependencies** として定義し、  
利用側（`packages/infra`）に依存関係を委ねるようにしました。  

```json
// packages/cdk-utils/package.json
{
  "name": "@myscope/cdk-utils",
  "version": "1.0.0",
  "peerDependencies": {
    "aws-cdk-lib": "2.x",
    "constructs": "^10.4.0"
  }
}
```
こうすることで、インフラ側のパッケージとライブラリのバージョンが自動的に揃い、ずれを防止できます。

### pnpm overrides での上書き

さらにルートの package.json に pnpm の overrides を設定し、
リポジトリ全体で利用する CDK のバージョンを強制的に揃えるようにしました。
```json
// package.json (root)
{
  "pnpm": {
    "overrides": {
      "aws-cdk-lib": "2.206.0",
      "constructs": "^10.4.2"
    }
  }
}
```

これによって、仮に下位パッケージで異なるバージョンが依存として解決されたとしても、
ルートで一元的に上書きされ、常に統一されたバージョンで CDK を利用できるようになっています。


## 振り返り

インフラ部分については、まだ S3 バケットを dev / prod に分けて構築するところからのスタートでした。  最初は単純に `my-project-dev-bucket` / `my-project-prod-bucket` のような名前を手で書くだけでも十分だと思っていましたが、CI/CDのことを念頭においたり、一貫性のことを想像すると開発の余地が生まれました。

そこで取り入れた工夫が次の点です。

- **命名規則の共通化**  
  - CDK の context や環境変数から取得した情報を基に、`nameBuilder` ユーティリティで統一的にリソース名を生成  
  - AWS リソースごとの制約（63文字制限や英数字で始まり終わる必要があるなど）も考慮し、自動的に調整できるようにした  

- **Meta 情報の props 化**  
  - `stage`, `project`, `service` といった横断的なメタ情報を `Meta` 型として定義  
  - スタックやコンストラクトには `props.meta` として明示的に渡す方針に変更  
  - これにより「どの値がどこから来ているか」が可視化され、テストやレビューがしやすくなった  

- **context > 環境変数 の優先順**  
  - `cdk deploy --context stage=prod` のように明示的に指定した値を最優先  
  - 指定がなければ環境変数を fallback として利用  
  - これにより CI/CD では context を使って制御、ローカル開発では環境変数だけで自然に動作、という柔軟な運用が可能になった  

- **CDK 用 util パッケージの整備**  
  - 命名関数や Meta 解決関数を `packages/cdk-utils` にまとめました。
  - すべてのスタックやコンストラクトから共通利用できるようにし、運用ポリシー変更にも強い構成にした  

---

こうして見ると、最初は「とりあえずバケットを作る」程度の小さな一歩でしたが、  
命名ルール → Meta props 化 → context/環境変数の優先順 → util パッケージ化 と改善を積み重ねた結果、  
**将来リソースが増えても安心して拡張できる基盤**ができてきたと感じています。  

インフラはまだシンプルですが、この設計の積み重ねが後の拡張（Route 53, API Gateway, Lambda, DynamoDB など）にも効いてくるはずです。

# CI/CD パイプライン

モノレポを進める中で「動いたコードをどう安全に環境へ反映するか」という課題に直面しました。  
ローカルでビルドできても、それを本番に手作業でデプロイしてしまうとヒューマンエラーの温床になります。  
そこで GitHub Actions を使って、**自動でテスト・ビルドし、環境に応じて安全にデプロイするパイプライン**を組みました。

また、AWS との接続方法については **OIDC (OpenID Connect)** をベースに設計しています。  
従来のように長期利用のアクセスキーを Secrets に置かず、GitHub Actions 実行時に OIDC トークンを発行 → AWS 側で一時的に信頼関係を結ぶことで、  
よりセキュアかつシンプルに認証・認可が行えるようになっています。  

## AWSでの事前準備

GitHub Actions から AWS にリソースをデプロイする方法は、大きく分けて 2 種類あります。  
ひとつは **永続キー（アクセスキー／シークレットキー）** を利用する方法、もうひとつは **OIDC (OpenID Connect)** を利用する方法です。

従来よく使われてきたのは、IAM ユーザーに発行された **アクセスキーとシークレットキーを Secrets に保存し、GitHub Actions 内で環境変数として展開して利用する方式**です。  
シンプルで分かりやすい一方で、次のような課題があります。

- 長期利用のキーがリポジトリの Secrets に保存されるため、漏洩した場合のリスクが非常に大きい  
- キーのローテーション（定期更新）を手動で管理する必要があり、運用コストが高い  
- Public リポジトリでは特に、外部コントリビューション経由で漏洩リスクが高まる  

これに対して、現在推奨されているのが **OIDC を利用する方法**です。  
GitHub Actions 実行時に GitHub が OIDC トークンを発行し、それを AWS 側で信頼して一時的な認証情報を払い出す仕組みです。これには、次のようなメリットが得られます。

- **Secrets にキーを保存する必要がなくなる** → 万が一リポジトリが流出しても長期キーが漏れない  
- **キーのローテーションが不要** → 毎回一時的な認証情報が払い出されるため、管理コストが大幅に減る  
- **IAM ロールと信頼ポリシーでアクセス範囲を最小化できる** → どのリポジトリ／どのブランチからの実行を許可するかを細かく制御可能  

現在のベストプラクティスは、永続キーではなく **OIDC による認証を利用すること**です。  
これにより、セキュリティと運用性の両立を実現できます。


さてｍOIDC を利用する場合は **AWS 側で事前設定を行う必要**があります。  
セキュリティ上重要になるのが、IAM ロールにおける **GitHub Actions に対する信頼関係の設定**です。  

IAM ロールの信頼ポリシーでは、OIDC プロバイダを `arn:aws:iam::{accountid}:oidc-provider/token.actions.githubusercontent.com` として指定し、  
`sts:AssumeRoleWithWebIdentity` を許可します。  

さらに Condition 句で制限をかけることで、**「どのリポジトリから」「どのイベントで」発行されたトークンを受け入れるか**を細かくコントロールできます。  

特に **StringLike 句** がセキュリティの肝になります。  
以下のように指定することで、  
- `repo:{username}/{reponame}:ref:refs/heads/dev` → `dev` ブランチの実行だけ許可  
- `repo:{username}/{reponame}:pull_request` → PR 実行だけ許可  

という形で、想定外のブランチやリポジトリからのトークン利用を防げます。  

開発ステージごとに AWS 環境を切り替える場合は、IAM ロールの信頼ポリシーに記述する Condition 句 もステージごとに調整する必要があります。具体的には、dev 環境であれば refs/heads/dev、prod 環境であれば refs/heads/main のように、ブランチ名と環境を対応付けて StringLike 句を設定することで、各ステージごとのデプロイを正しく制御できます。

下記はdevというブランチが運用上存在していて、ついとなるAWSアカウントにロールを設定する場合の信頼関係のドキュメントです。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::{accountid}:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": [
                        "repo:{username}/{reponame}:ref:refs/heads/dev",
                        "repo:{username}/{reponame}:pull_request"
                    ]
                }
            }
        }
    ]
}
```

今回 IAM ロールに対して設定したポリシーは、以下のような内容になっています。Assumeする先のリソースcdk-hnb659fds-... という名前は、CDK のブートストラップ時に自動で作成されるリソースです。具体的には CDK が cdk bootstrap を実行した際に、CloudFormation スタックを通じてデプロイようのIAM ロールを生成しており、それを利用します。

``` json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Resource": [
                "arn:aws:iam::{accountid}:role/cdk-hnb659fds-deploy-role-{accountid}-ap-northeast-1"
            ]
        }
    ]
}
```

---

## GitHubにおける環境変数と値の設定
やっとGitHub Actions側の記述に戻ってこれました。しかし、GitHub Actionsのコードに映る前に、まだ設定の必要があります。

### GitHub リポジトリにおける環境変数とシークレットの管理

CI/CD パイプラインを安全に運用するうえで欠かせないのが、**環境変数（Variables）** と **シークレット（Secrets）** の適切な管理です。  

- **環境変数 (Repository Variables)**  
  - プロジェクト名やサービス名、ステージ名（dev/prod など）のように、比較的機微ではない設定値を保存する場所。  
  - `vars.PROJECT`, `vars.SERVICE`, `vars.AWS_REGION` など、リポジトリの CI/CD 内で参照できる。  

- **シークレット (Repository Secrets)**  
  - AWS IAM ロール ARN、API トークン、外部サービスの認証キーなど、外部に漏れてはいけない機密情報を保存する場所。  
  - 例: `secrets.AWS_DEV_OIDC_ROLE_ARN`, `secrets.AWS_PROD_OIDC_ROLE_ARN`  
  - GitHub Actions のログやワークフロー内で参照するときも **マスク処理**されて表示されないようになっている。  

---

### Public リポジトリでの注意点

GitHub Actions は **Public リポジトリの場合、実行ログが誰でも閲覧可能** という仕様になっています。そのため、とくにPublicリポジトリでは**Secrets や Variables が誤って公開されないようにする**ことに細心の注意を払う必要があります。  

- **ログ出力に注意**  
  - `echo $AWS_DEV_OIDC_ROLE_ARN` のようにシークレットを直接ログに出すと、マスクされるとはいえ意図しない漏洩リスクになる。  
  - デバッグ時でも秘密情報を出力しない運用が望ましい。  

- **環境変数とシークレットの分離**  
  - 機微でない情報（サービス名や環境名）は Variables に置き、Secrets 側には絶対に置かない。  
  - 逆に Secrets 側には **「漏れて困るものだけ」** を限定して保存する。  

## 前提条件チェック（prereq ジョブ）
ここからがGithub Actionsで定義した処理の内容説明になります。

まず最初に走るのが **前提条件のチェック**です。  
「デプロイに必要な変数やシークレットが設定されているか？」を最初に検査し、足りないものがあれば後続の処理をスキップします。

- 共通で必要なもの: `AWS_REGION`, `PROJECT`, `SERVICE`
- dev 用: `AWS_DEV_OIDC_ROLE_ARN`
- prod 用: `AWS_PROD_OIDC_ROLE_ARN`

不足しているものは `GITHUB_STEP_SUMMARY` に一覧で出力するようにしました。ワークフロー実行画面からすぐに確認できます。  
このおかげで「設定漏れに気づかないままデプロイが失敗する」といった無駄を防げます。

---

## ステージ判定（set-stage-variable ジョブ）

次に「今回の実行が dev / prod / preview のどれなのか」を判定しています。  
GitHub のイベントは push と PR マージで参照できる変数が違うため、**イベントごとに判定方法を切り替える工夫**をしています。

- PR マージ時 → `base_ref`（マージ先ブランチ）を利用
- push 時 → `ref` を利用

これによって、マージや push のタイミングで誤って環境を判定してしまう事故を防いでいます。  
判定結果は `stage` という出力にまとめられ、後続ジョブが共通で利用できます。

---

## ビルド（build ジョブ）

デプロイの前には必ず全体をビルドするようにしました。  
ここで実行しているのはシンプルに **`pnpm run build:all`** で、Turborepo がワークスペース全体のビルドを調整してくれます。  

各パッケージごとに定義された `build` タスクがこのコマンドを通じて呼ばれる仕組みになっており、  
例えば `packages/infra` にある CDK パッケージでは `build` タスクに `cdk synth` を紐づけています。  

つまり、`build` ジョブそのものは「全体ビルドを一括で起動するだけ」ですが、  
個々のパッケージが内部で何をビルドするか（TypeScript コンパイルや CDK synth など）は各パッケージ側に責務を委ねています。  


---

## デプロイ（deploy-dev / deploy-prod ジョブ）

デプロイは PR がマージされたタイミングだけ動くようにしています。  
しかも **前提条件が満たされている場合のみ実行**されるので、設定不足のまま間違って本番にデプロイすることはありません。

- dev 環境
  - `dev` ブランチにマージされた PR のみ
  - `AWS_DEV_OIDC_ROLE_ARN` が設定されている場合のみ
- prod 環境
  - `main` ブランチにマージされた PR のみ
  - `AWS_PROD_OIDC_ROLE_ARN` が設定されている場合のみ

AWS への認証には **OIDC** を使っており、長期のアクセスキーを Secrets に置かなくてもよくなりました。  
CDK を実行するときには `--context stage/project/service` を渡しており、環境やサービス名ごとに命名規則を自動で適用できるようにしています。

### 学びと工夫

実際に動作させてみて分かったのは、**GitHub Actions ではジョブ間で仮想マシンの状態が引き継がれない**という点です。そのため、ビルド済みの成果物をデプロイジョブでそのまま使うことはできず、  
**デプロイ直前に `pnpm -w run build:all` を再実行する必要がある**という学びがありました。  

さらに、CDK のスタックは `packages/infra` ディレクトリから実行されるため、  
例えばバケット名を生成する命名関数のように **別パッケージにあるユーティリティに依存**するケースがあります。  
この依存関係を正しく解決するためには、**単一パッケージのビルドではなく `-w` オプションを付けてワークスペース全体をビルド**する必要がありました。（buildステップではrootでコマンド実行していたので`-w`オプションを付ける必要がなく、この仕様に気が付くまでに時間がかかりました。）  

結果として、  
- **毎回ジョブごとにビルドをやり直す**（ジョブ間状態は共有されない）  
- **依存解決のために全体をビルドする**（`pnpm -w run build:all`）  

という手順が CI/CD パイプラインに組み込まれ、安定したデプロイを実現できるようになりました。

---

## タグ付け（tag-on-main ジョブ）

最後に、本番 (`main`) にマージされたときだけ自動で Git タグを打つ処理を追加しました。  
タグは SemVer のパッチ番号を自動で上げるようになっており、これで「どのコミットがどのデプロイに対応しているか」を追いやすくなります。  

最初はタグを手動で打っていましたが、忘れたり作法がバラバラになったりしていたので、自動化してからは履歴が部分的にきれいにできたと思っています。

---

## コードスニペット

AWS側のロールやOIDCの設定は別途実施する必要がありますが、おおむねGitHub Actionsのコードは以下です。この開発をしている時に、下記を調整するのが一番楽しかったです。

```yaml
name: CI Workflow

on:
  push:
    branches:
      - 'feature/**'
  pull_request:
    types: [closed]
    branches: [main, dev]

jobs:
  prereq:
    runs-on: ubuntu-latest
    outputs:
      dev_configured: ${{ steps.check.outputs.dev_configured }}
      prod_configured: ${{ steps.check.outputs.prod_configured }}
      missing_dev: ${{ steps.check.outputs.missing_dev }}
      missing_prod: ${{ steps.check.outputs.missing_prod }}
    steps:
      - id: check
        run: |
          missing_common=()
          [[ -z "${{ vars.AWS_REGION }}" ]] && missing_common+=(AWS_REGION)
          [[ -z "${{ vars.PROJECT }}"   ]] && missing_common+=(PROJECT)
          [[ -z "${{ vars.SERVICE }}"   ]] && missing_common+=(SERVICE)

          missing_dev=("${missing_common[@]}")
          [[ -z "${{ secrets.AWS_DEV_OIDC_ROLE_ARN }}" ]] && missing_dev+=(AWS_DEV_OIDC_ROLE_ARN)
          if (( ${#missing_dev[@]} )); then
            echo "dev_configured=false" >> $GITHUB_OUTPUT
            printf "missing_dev=%s\n" "${missing_dev[*]}" >> $GITHUB_OUTPUT
          else
            echo "dev_configured=true" >> $GITHUB_OUTPUT
            echo "missing_dev=" >> $GITHUB_OUTPUT
          fi

          missing_prod=("${missing_common[@]}")
          [[ -z "${{ secrets.AWS_PROD_OIDC_ROLE_ARN }}" ]] && missing_prod+=(AWS_PROD_OIDC_ROLE_ARN)
          if (( ${#missing_prod[@]} )); then
            echo "prod_configured=false" >> $GITHUB_OUTPUT
            printf "missing_prod=%s\n" "${missing_prod[*]}" >> $GITHUB_OUTPUT
          else
            echo "prod_configured=true" >> $GITHUB_OUTPUT
            echo "missing_prod=" >> $GITHUB_OUTPUT
          fi

  set-stage-variable:
    needs: prereq
    if: >
      github.event_name == 'push' ||
      (github.event_name == 'pull_request' && github.event.pull_request.merged == true)
    runs-on: ubuntu-latest
    outputs:
      stage: ${{ steps.stage.outputs.stage }}
    steps:
      - uses: actions/checkout@v4
      - id: stage
        run: |
          if [[ "${{ github.event_name }}" == "pull_request" && "${{ github.pull_request.merged }}" == "true" ]]; then
            if [[ "${{ github.base_ref }}" == "main" ]]; then stage="prod"
            elif [[ "${{ github.base_ref }}" == "dev" ]]; then stage="dev"
            else stage="preview"; fi
          else
            if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then stage="prod"
            elif [[ "${{ github.ref }}" == "refs/heads/dev" ]]; then stage="dev"
            else stage="preview"; fi
          fi
          echo "STAGE=$stage" >> $GITHUB_ENV
          echo "stage=$stage" >> $GITHUB_OUTPUT

  build:
    needs: set-stage-variable
    runs-on: ubuntu-latest
    env:
      STAGE: ${{ needs.set-stage-variable.outputs.stage }}
      PROJECT: ${{ vars.PROJECT }}
      SERVICE: ${{ vars.SERVICE }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: npm i -g pnpm
      - run: pnpm install
      - run: pnpm run build:all

  deploy-dev:
    needs: [prereq, set-stage-variable, build]
    if: >
      needs.set-stage-variable.outputs.stage == 'dev' &&
      github.event_name == 'pull_request' &&
      github.event.pull_request.merged == true &&
      needs.prereq.outputs.dev_configured == 'true'
    runs-on: ubuntu-latest
    permissions: { id-token: write, contents: read }
    env:
      AWS_REGION: ${{ vars.AWS_REGION }}
      STAGE: dev
      PROJECT: ${{ vars.PROJECT }}
      SERVICE: ${{ vars.SERVICE }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: npm i -g pnpm
      - run: pnpm install
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_DEV_OIDC_ROLE_ARN }} # arn:aws:iam::<ACCOUNT_ID>:role/<ROLE_NAME>
          aws-region: ${{ env.AWS_REGION }}
      - name: CDK deploy (dev)
        working-directory: packages/infra
        run: |
          pnpm install
          pnpm -w run build:all
          pnpm cdk deploy --require-approval never \
            --context stage=${{ env.STAGE }} \
            --context project=${{ env.PROJECT }} \
            --context service=${{ env.SERVICE }}

  deploy-prod:
    needs: [prereq, set-stage-variable, build]
    if: >
      needs.set-stage-variable.outputs.stage == 'prod' &&
      github.event_name == 'pull_request' &&
      github.event.pull_request.merged == true &&
      needs.prereq.outputs.prod_configured == 'true'
    runs-on: ubuntu-latest
    permissions: { id-token: write, contents: read }
    env:
      AWS_REGION: ${{ vars.AWS_REGION }}
      STAGE: prod
      PROJECT: ${{ vars.PROJECT }}
      SERVICE: ${{ vars.SERVICE }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: npm i -g pnpm
      - run: pnpm install
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_PROD_OIDC_ROLE_ARN }} # arn:aws:iam::<ACCOUNT_ID>:role/<ROLE_NAME>
          aws-region: ${{ env.AWS_REGION }}
      - name: CDK deploy (prod)
        working-directory: packages/infra
        run: |
          pnpm install
          pnpm -w run build:all
          pnpm cdk deploy --require-approval never \
            --context stage=${{ env.STAGE }} \
            --context project=${{ env.PROJECT }} \
            --context service=${{ env.SERVICE }}

  tag-on-main:
    needs: deploy-prod
    if: github.base_ref == 'main' && github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    permissions: { contents: write }
    steps:
      - uses: actions/checkout@v4
      - id: get_tag
        run: |
          git fetch --tags
          latest=$(git tag --sort=-v:refname | head -n 1)
          echo "latest_tag=$latest" >> $GITHUB_OUTPUT
      - name: Create new tag
        run: |
          latest=${{ steps.get_tag.outputs.latest_tag }}
          if [[ -z "$latest" ]]; then new_tag="v0.1.0"
          else IFS='.' read -r M m p <<< "${latest#v}"; p=$((p+1)); new_tag="v$M.$m.$p"; fi
          git config user.name "github-actions"
          git config user.email "github-actions@github.com"
          git tag "$new_tag"
          git push origin "$new_tag"

```

### 振り返り

こうして CI/CD を整えてみて実感したのは、**ガードレールを先に置いておく大切さ**です。  

- 前提条件が揃っていなければそもそもデプロイしない  
- ステージ判定を厳密にして誤デプロイを防ぐ  
- ビルドを一度にまとめて一貫性を担保する  
- OIDC で認証を安全にする  
- タグ付けで履歴をきちんと残す  

最初は「ただビルドしてデプロイできればいい」と思っていましたが、こうして工夫を積み重ねることで、安心して開発できる土台ができてきました。  
今後は Turborepo のリモートキャッシュや、より高度なデプロイ戦略を組み合わせて、さらに効率的なパイプラインにしていく予定です。


# コード品質とセキュリティ設計

モノレポを進めるうえで「コードをどうきれいに保つか」「秘密情報をどう守るか」という課題に直面しました。  
最初は動けばよいと考えていましたが、規模が少しずつ大きくなるにつれて **「コードレビューで毎回同じ指摘が出る」「誤ってキーをコミットしそうになる」** という不安が強くなってきました。  

そこで、**コード品質を自動で担保する仕組み**と**セキュリティを強化する仕組み**をモノレポに組み込みました。

---

## Lint とフォーマットの導入

まず着手したのは **Lint** と **フォーマット**です。  
レビュー時にスタイルの違いで議論するのは時間の無駄だと感じていたので、自動で揃えてしまうことにしました。

- **ESLint**  
  - JavaScript/TypeScript の静的解析を行い、潜在的なバグを防止。  
  - TypeScript サポートを有効にして型安全性を高めました。  

- **Prettier**  
  - コードフォーマッタ。セミコロンやインデントなどを自動整形。  
  - 「誰が書いても同じ形」になることで、レビュー時の不要な差分が激減しました。  

- **Markdownlint**  
  - ドキュメントもコードと同じように品質を維持するために導入。  
  - 記事や README を書くときのフォーマットを統一できるので、後から読みやすい状態を保てます。  

結果として、**Lint とフォーマットを CI や Git Hook に組み込む**ことで、コミット時点で問題を自動検出できるようになりました。

---

## Git Hooks と Husky

自動化の流れをさらに強化するために **Husky** を導入しました。  

- **pre-commit フック**  
  - `eslint` / `prettier --check` / `markdownlint` を実行  
  - 問題があるとコミット自体がブロックされる仕組み  

- **pre-push フック**  
  - `pnpm test` を実行し、最低限のテストが通らなければリモートに push できないようにしました。  

最初は「フックに時間がかかって面倒かな？」と思いましたが、慣れると **「壊れたコードがリポジトリに入らない安心感」** の方が大きくなりました。

---

## シークレット検出（Gitleaks）

もう一つ重要だったのが **シークレットの誤コミット防止**です。  
AWS のアクセスキーや API トークンをうっかりコミットしてしまうと、公開リポジトリなら即アウトですし、プライベートでも漏洩リスクになります。  

そこで導入したのが **Gitleaks** です。  

- **pre-commit フックに組み込み**  
  → コミット前に自動スキャンし、もしキーが含まれていれば即ブロック。  
- **CI でも実行**  
  → 万が一ローカルフックを bypass しても、PR 時に検出できる二重チェック体制。  

最初に設定したときは「誤検知（false positive）」に悩まされましたが、`.gitleaks.toml` を調整して不要なパターンを除外することで落ち着きました。  

---

## CI との統合

これらのチェックはローカルだけではなく、GitHub Actions の CI パイプラインにも統合しました。  

- `lint:all` → ESLint, Prettier, Markdownlint を一括実行  
- `gitleaks` → シークレットスキャンを CI 上でも実施  

こうしておくことで **「ローカルで逃しても CI で必ず止まる」** 仕組みになり、安心して開発を進められるようになりました。

---

## 振り返り

最初は **「コードスタイルなんて後で直せばいい」「秘密情報は気をつければ大丈夫」** と考えていました。  
しかし実際にプロジェクトが進むと、スタイルの不統一やシークレット漏洩リスクは大きな負担になってきます。  

- Lint とフォーマッタで「人間が指摘しなくても済む環境」を整える  
- Husky で「壊れたコードや漏洩リスクをリポジトリに入れさせない」  
- Gitleaks で「セキュリティの最後の砦を用意する」  

こうした仕組みを入れることで、コード品質とセキュリティを自動で担保できるようになりました。  
結果としてレビューは「設計やアーキテクチャの本質的な議論」に集中でき、開発体験が格段に良くなったと感じています。


# 開発体験（DX）設計

モノレポを始めてしばらくすると、今度は **「どうやったらもっと快適に開発できるか」** という課題が出てきました。  
せっかく CI/CD や品質チェックを整えても、日々の開発がストレスフルだと続きません。  
そこで意識したのが **DX (Developer Experience)** ― 開発者体験を向上させる仕組みづくりです。

---

## DevContainer による環境統一

最初に取り組んだのは **環境の統一**でした。  
ローカル環境に直接ツールをインストールすると、OS やバージョンの違いで動作が変わる問題が起きやすくなります。  

- **DevContainer** を導入し、Node.js / AWS CLI / pnpm / CDK などをコンテナ内にまとめました。  
- VS Code の Remote Container 機能を使えば、プロジェクトを開いた瞬間に同じ環境で作業開始できます。  
- CI 環境でもほぼ同じ Docker イメージを利用できるため、**「ローカルでは動いたのに CI で落ちる」**問題を大幅に減らせました。  

これによって「環境構築のつまずき」がなくなり、誰でもすぐに開発に参加できるようになりました。

---

## ローカル開発と CI の一貫性

DX を考える上で意識したのは **「ローカルと CI の体験を揃える」**ことです。  

- タスク実行はすべて **pnpm scripts** に寄せる  
- CI でも `pnpm run lint:all`, `pnpm run test`, `pnpm run build:all` をそのまま利用  
- Turborepo のパイプライン設計を使って、依存関係やキャッシュも統一  

こうすることで、開発者がローカルで叩くコマンドと、CI が実行するコマンドが一致し、**「CI 専用のよくわからないコマンド」**を覚える必要がなくなりました。

---

## テスト戦略（Jest / Vitest / React Testing Library）

テストは最初ほとんど書けていませんでした。  
しかし規模が大きくなると「安心してリファクタリングできない」状態が辛くなり、徐々にテストを書き足していきました。

- **Jest**: サーバーサイドやユーティリティ関数のテストに利用  
- **Vitest**: フロントエンドの軽量なユニットテストに利用  
- **React Testing Library**: コンポーネントのレンダリング確認をユーザー目線で記述  

これらをモノレポ全体に組み込み、`pnpm test` でまとめて実行できるようにしました。  
特に React Testing Library は「ユーザーがどう画面を操作するか」に近い視点で書けるので、バグの再現や回避に役立っています。

---

## デバッグと監視の工夫

開発中に「問題が起きたときに素早く気づける」仕組みも DX には欠かせません。  

- ローカル開発では `pnpm dev` でフロントと API を並列起動し、エラーログをまとめて確認できるようにしました。  
- Turborepo のログを活用して、どのパッケージで失敗したのか一目でわかるようにしました。  
- CI では失敗したステップを GitHub Actions の UI で確認できるので、問題の切り分けが容易になりました。  

まだ本格的なモニタリングまではできていませんが、**「失敗に早く気づける仕組み」**を持つだけで心理的な安心感が違います。

---

## 振り返り

最初は **「とりあえずコードが動けばよい」** という状態から始まりましたが、DX を整備していくうちに次のような効果が出ました。  

- **DevContainer** で環境差異をなくし、新規参加者もすぐ開発できる  
- **ローカルと CI の一貫性**で、覚えるべきコマンドが減り、トラブルも少なくなった  
- **テスト導入**により安心してリファクタリングできるようになった  
- **デバッグ・監視の工夫**で失敗にすぐ気づけるようになった  

DX を意識することで「開発者として気持ちよくコードを書ける環境」が整い、モノレポを継続的に運用する土台が強くなったと感じています。


# まとめと今後の展望

## 現状の到達点
このモノレポ環境は、最初は「複数のアプリをひとつにまとめたい」という小さな動機から始まりました。  
試行錯誤を繰り返す中で、少しずつ次のような仕組みを整えてきました。

- **ディレクトリ構成とワークスペース管理**  
  → `apps/` と `packages/` に分けて整理、pnpm workspace で依存関係を一元管理  

- **タスク実行とキャッシュ**  
  → Turborepo によりビルドやテストを効率化、キャッシュで開発体験を改善  

- **インフラ設計（現状）**  
  → dev/prod 用に S3 バケットを分離し、安全にデプロイできる基盤を整備  

- **CI/CD パイプライン**  
  → GitHub Actions で前提チェック → ステージ判定 → ビルド → デプロイ → タグ付けの流れを自動化  

- **コード品質とセキュリティ**  
  → ESLint / Prettier / Markdownlint / Husky / Gitleaks を組み込み、安心してコードをコミットできる環境を実現  

- **開発体験（DX）**  
  → DevContainer による環境統一、テスト戦略の導入、ローカルと CI の一貫性を意識した開発基盤を構築  

「最初は難しそう」と思っていたモノレポですが、少しずつ機能を積み上げることで、今では **安心して開発できる環境** が手に入ったと感じています。

---

## 今後追加を検討している機能
現状はまだ “最小限の基盤” です。これからは以下のような拡張を検討しています。

- **インフラの本格的な IaC 化**  
  - AWS CDK で S3 以外のリソース（API Gateway, Lambda, Route 53, DynamoDB など）を統合的に管理  
  - ステージ名付与ポリシーを徹底し、リソース衝突を防止  

- **リモートキャッシュの導入**  
  - Turborepo のリモートキャッシュを活用し、CI/CD のビルド時間をさらに短縮  

- **セキュリティチェックの高度化**  
  - Gitleaks に加えて Trivy や Dependabot を導入し、依存関係やコンテナの脆弱性にも対応  

- **自動リリースフロー**  
  - Git タグ付けを起点に、自動でリリースノートを生成する仕組みを追加  
  - バージョニングと変更履歴をより明確に管理  

- **開発者向けのドキュメント充実**  
  - 新しく参加した人が迷わないよう、README や設計ドキュメントを整備  

---

## 改善サイクルの回し方
モノレポの良いところは「小さな改善をすぐ全体に反映できる」点です。  
たとえば、命名ライブラリを共通化したように、小さな工夫が後から効いてきます。  

今後も **「小さく試す → 共通化する → 自動化する」** というサイクルを回しながら、  
少しずつ開発体験と運用基盤を磨き上げていく予定です。

---

## 最後に
このモノレポはまだ発展途上ですが、初心者が一歩ずつ組み上げていった記録そのものです。  
同じように「モノレポを始めてみたいけれど難しそう」と感じている方に、この記事が少しでも参考になれば幸いです。  
そしてこれからも、より実践的で快適なモノレポ運用を目指して改善を続けていきます。

