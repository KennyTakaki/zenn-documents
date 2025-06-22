---
title: "cme-auth-setup"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---
# AWS Connected Mobility Solution の認証基盤を理解する

## はじめに

AWS Connected Mobility Solution (CMS) on AWS は、コネクテッドカーやモビリティサービスを構築するためのソリューションです。その中でも「Auth Setup」モジュールは認証基盤として重要な役割を果たしています。この記事では、このモジュール
の役割と仕組みについて、OAuth 2.0の基礎知識から解説します。

## Auth Setup モジュールとは

Auth Setup モジュールは、CMS の認証基盤として以下の機能を提供します：

• CMS 認証モジュールのトークン検証と認可コード交換のための IdP 設定
• ユーザーが Backstage モジュールにログインするための IdP 設定
• CMS モジュール間のサービス間認証のための設定
• オプションで Cognito インフラストラクチャのデプロイと設定

これらの機能は主に3つの AWS Secrets Manager シークレットを通じて実現されています：

1. IdP Config
2. Service Client Config
3. User Client Config

## 認証・認可の基礎知識

### 認証と認可の違い

• **認証 (Authentication)**: ユーザーが「誰であるか」を確認するプロセス
• **認可 (Authorization)**: 認証されたユーザーが「何をする権限があるか」を決めるプロセス

### OAuth 2.0 とは

OAuth 2.0 は、サードパーティアプリケーションに制限付きのアクセス権を付与するための業界標準プロトコルです。

主要な概念：
• **リソースオーナー**: ユーザー自身（データの所有者）
• **クライアント**: ユーザーのデータにアクセスしたいアプリケーション
• **認可サーバー**: トークンを発行するサーバー
• **リソースサーバー**: ユーザーのデータを持つサーバー
• **アクセストークン**: リソースへのアクセス権を表す文字列

主要なフロー：
• **認可コードフロー**: ウェブアプリケーション向け
• **クライアントクレデンシャルフロー**: サービス間通信向け

## Auth Setup モジュールの役割

### ユーザー認証のシナリオ

例えば、エンジニアが Backstage ポータルにログインして車両データを確認する場合：

1. エンジニアが Backstage にアクセス
2. Backstage は認証が必要なため、IdP のログイン画面にリダイレクト
3. エンジニアがログイン情報を入力
4. IdP が認証を行い、アクセストークンと ID トークンを発行
5. Backstage はこのトークンを CMS の他の API に渡して車両データを取得
6. 各 API は Auth Setup モジュールが提供する設定を使ってトークンを検証

### サービス間認証のシナリオ

例えば、車両データ処理サービスが分析サービスの API を呼び出す場合：

1. 車両データ処理サービスが API アクセスのためのトークンが必要
2. Auth Setup モジュールが提供するサービスクライアント設定を使用
3. クライアントクレデンシャルフローで IdP からトークンを取得
4. 取得したトークンを使って分析サービスの API を呼び出し
5. 分析サービスは Auth Setup モジュールの設定を使ってトークンを検証

## シークレットの構造と内容

### IdP Config シークレット

json
{
  "issuer": "https://your-idp-domain.com",
  "token_endpoint": "https://your-idp-domain.com/oauth2/token",
  "authorization_endpoint": "https://your-idp-domain.com/oauth2/authorize",
  "alternate_aud_key": "client_id",
  "auds": ["client-id-1", "client-id-2"],
  "scopes": ["openid", "email", "profile"]
}


### User Client Config シークレット

json
{
  "client_id": "your-user-client-id",
  "client_secret": "your-user-client-secret"
}


### Service Client Config シークレット

json
{
  "client_id": "your-service-client-id",
  "client_secret": "your-service-client-secret"
}


## デプロイメントオプション

Auth Setup モジュールには3つのデプロイメントパスがあります：

1. Cognito デプロイ: モジュールが提供する Cognito 基盤を使用
2. 空の設定デプロイ: 独自の IdP を使用し、後で設定値を入力
3. 既存の設定デプロイ: 以前のデプロイメントから既存の設定を再利用

## クライアント ID の管理

CMS では、主に2種類のクライアントを区別しています：

1. ユーザークライアント: ユーザーが直接操作するアプリケーション用
2. サービスクライアント: バックグラウンドで動作するサービス用

これらのクライアント情報は別々のシークレットとして管理されており、リスト形式ではなく、クライアントタイプごとに分離されています。これにより：

• 明確な責任分離
• セキュリティの向上
• 設定の簡素化

が実現されています。

## まとめ

Auth Setup モジュールは、CMS の様々なコンポーネントが一貫した方法で認証・認可を行えるようにする「共通基盤」です。OAuth 2.0 の複雑な設定を抽象化し、開発者が認証の詳細を気にせずにビジネスロジックに集中できるようにします。

AWS Cognito を使いたい場合は自動セットアップを提供し、既存の IdP を使いたい場合はその設定を保存する場所を提供するという柔軟性も持っています。これにより、セキュリティを確保しながら、開発効率と拡張性を高めることができます。

## 参考リンク

• Connected Mobility Solution on AWS https://aws.amazon.com/solutions/implementations/connected-mobility-solution-on-aws/
• Auth Setup モジュール GitHub https://github.com/aws-solutions/connected-mobility-solution-on-aws/tree/main/source/modules/auth_setup
• OAuth 2.0 仕様 (RFC 6749) https://tools.ietf.org/html/rfc6749