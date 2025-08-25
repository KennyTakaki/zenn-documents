---
title: "はじめてのモノレポ構築記録 – pnpm × Turborepo × AWS で少しずつ整えてみた"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","Monorepo","pnpm","Turborepo","GitHubActions"]
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
キャッシュや依存解決の仕組みを活用することで、実際の開発効率が大きく変わりました。

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
自分自身のビルドだけでなく、依存しているパッケージのビルドも実行される

- lint / test
キャッシュ可能だが成果物を残さないので outputs は空

- dev
ローカル開発用なのでキャッシュを無効化

この最小構成だけでも「どのタスクがどのパッケージに依存しているか」を明示でき、全体を見渡しやすくなりました。

## 並列実行と依存解決

Turborepo はタスクを並列で実行してくれます。
例えば turbo run build を実行すると、依存関係を解決した上で、独立しているパッケージのビルドは同時に走ります。

これによって「待ち時間が長いビルド」も並列化され、従来の直列実行よりかなり早くなりました。
小さな改善に見えて、日々の開発体験に大きく効いてきます。

## キャッシュ戦略（ローカル・リモート）

Turborepo の最大の魅力は キャッシュ機能 です。

### ローカルキャッシュ
直前に実行したタスクの結果を保存し、同じ入力なら再実行をスキップします。
例: 前回ビルドからコードが変わっていなければ、次回は「キャッシュヒット」で瞬時に完了します。

### リモートキャッシュ
（未導入ですが）将来的には CI との連携で活用予定です。開発者のローカルと CI でキャッシュを共有できるようになれば、CI のビルド時間が劇的に短縮されるはずです。

キャッシュ戦略を導入しただけで、特にビルドの繰り返しにかかる時間が大幅に短縮されました。


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
現時点では「環境ごとに S3 バケットを作成し、アプリの成果物を配置できるようにする」という最小限の仕組みにとどまっています。  

---

## 環境ごとの S3 バケット作成
S3 バケットは **開発環境（dev）** と **本番環境（prod）** に分けて用意しました。  
ステージ名を明示的に付与することで、リソースの混在を避けるようにしています。

例:  
- `my-project-dev-bucket`  
- `my-project-prod-bucket`  

まだ単純な構成ですが、これだけでも「環境を分けて安全に動かす」第一歩になります。

---

## ステージ分離の考え方
当初はすべての環境を一つのバケットにまとめることも考えましたが、以下の理由から dev と prod を分けました。  

- 開発中のアプリを誤って本番に上書きしないようにする  
- バケットポリシーやアクセス権限を環境ごとに明確にできる  
- 将来的に CI/CD から自動デプロイする際に分けておいた方が運用しやすい  

「シンプルでも安全に」という方針で、まずは dev/prod の2環境を基本としました。

---

## 今後の拡張余地
現状は S3 バケットの作成にとどまっていますが、将来的には以下のような拡張を考えています。  

- **AWS CDK による IaC 化**  
  → S3 バケットの定義をコード化し、他のリソースも統合的に管理できるようにする  
- **Route 53 によるカスタムドメイン設定**  
  → 静的サイトを独自ドメインで公開する  
- **API Gateway + Lambda**  
  → API サーバーをサーバーレスで構築し、S3 でホストするフロントエンドと連携する  
- **環境ごとの明示的なステージ名付与**  
  → 将来は S3 以外のリソース（DynamoDB, Cognito など）にも同じ方針を適用する  

---

## 振り返り
初心者としては「まずは S3 でファイルを置けるようにする」ことから始めました。  
まだ IaC も導入途中ですが、環境を分けて S3 バケットを持つだけでも **「開発環境と本番環境を意識した運用」**を体験できます。  

この小さな一歩が、将来的に CDK や他のサービスを組み合わせて拡張する基盤になると考えています。


## CI/CD パイプライン

モノレポを進める中で「動いたコードをどう安全に環境へ反映するか」という課題に直面しました。  
ローカルでビルドできても、それを本番に手作業でデプロイしてしまうとヒューマンエラーの温床になります。  
そこで GitHub Actions を使って、**自動でテスト・ビルドし、環境に応じて安全にデプロイするパイプライン**を組みました。

---

### 前提条件チェック（prereq ジョブ）

まず最初に走るのが **前提条件のチェック**です。  
「デプロイに必要な変数やシークレットが設定されているか？」を最初に検査し、足りないものがあれば後続の処理をスキップします。

- 共通で必要なもの: `AWS_REGION`, `PROJECT`, `SERVICE`
- dev 用: `AWS_DEV_OIDC_ROLE_ARN`
- prod 用: `AWS_PROD_OIDC_ROLE_ARN`

不足しているものは `GITHUB_STEP_SUMMARY` に一覧で出力されるので、ワークフロー実行画面からすぐに確認できます。  
このおかげで「設定漏れに気づかないままデプロイが失敗する」といった無駄を防げました。

---

### ステージ判定（set-stage-variable ジョブ）

次に「今回の実行が dev / prod / preview のどれなのか」を判定しています。  
GitHub のイベントは push と PR マージで参照できる変数が違うため、**イベントごとに判定方法を切り替える工夫**をしています。

- PR マージ時 → `base_ref`（マージ先ブランチ）を利用
- push 時 → `ref` を利用

これによって、マージや push のタイミングで誤って環境を判定してしまう事故を防いでいます。  
判定結果は `stage` という出力にまとめられ、後続ジョブが共通で利用できます。

---

### ビルド（build ジョブ）

デプロイの前には必ず全体をビルドするようにしました。  
ここで重要なのは **「一度だけビルドする」** という点です。  

以前はデプロイのたびに再ビルドしていたのですが、時間もかかるし、ビルド結果が環境によってズレる可能性もありました。  
現在は共通の `build:all` スクリプトでモノレポ全体をビルドし、その成果物を dev/prod デプロイで使い回す形にしています。  
これによって、**「同じものが dev でも prod でも動く」** という安心感が得られました。

---

### デプロイ（deploy-dev / deploy-prod ジョブ）

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

---

### タグ付け（tag-on-main ジョブ）

最後に、本番 (`main`) にマージされたときだけ自動で Git タグを打つ処理を追加しました。  
タグは SemVer のパッチ番号を自動で上げるようになっており、これで「どのコミットがどのデプロイに対応しているか」を追いやすくなります。  

最初はタグを手動で打っていましたが、忘れたり作法がバラバラになったりしていたので、自動化してからは履歴がとても綺麗になりました。

---

### コードスニペット

AWS側のロールやOIDCの設定は別途実施する必要がありますが、おおむねGitHub Actionsのコードは以下です。この開発をしている時に、下記を調整するのが一番楽しかったです。

```
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

