---
title: "root-access-management"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","IAM"]
published: false
---

# 概要
AWSのメンバーアカウントに対するルートアクセスの一元化に関して、以前から気になっていたので触ってみた。

# ルートアクセス一元化の手順
## IAMでの一元化設定の有効化
ルートアクセスの一元化はIAMの設定画面から行う。この画面はOrganizationsの管理アカウントで設定している。
![alt text](/images/articles/root-access-management/iam-enable.png)

Cabpabilities to enableで有効にする機能を選択する。
有効化できる機能は以下2つで、実際にやりたいのはルート認証情報の管理。

- ルート認証情報の管理
  **メンバーアカウントのルート認証情報を削除したり、監査したりできます。** また、特定のメンバーアカウントに対してパスワードリカバリーを許可することも可能です。

- メンバーアカウントでの特権ルート操作
  Amazon SQS の誤設定されたポリシーや Amazon S3 の誤設定を削除するなど、メンバーアカウントで特定のルート操作を実行できます。
![alt text](/images/articles/root-access-management/iam-setting.png)

ちなみに、このルートアクセスの一元管理も委任できるようだが、どのアカウントに委任するのがいいんだろうか？recommendedになっている。

## 一元化後の操作
設定が完了するとIAMのRoot access managementからOrganizationsのヒエラルキーが確認できるようになる。
![alt text](/images/articles/root-access-management/iam-after-setting.png)

# メリット