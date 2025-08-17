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

アカウントを選択してTake Privileged actionを押すことで、ルートアクセスに関する操作が可能。
![alt text](/images/articles/root-access-management/take-privileged-action.png)

選択すると削除される対象の確認が促される。どうやら削除すると
ルート資格情報（ルートアクセスキー、パスワード、署名証明書）を含む、特定のメンバーアカウントのルート資格情報が削除され、多要素認証（MFA）も削除されるらしい。
結果としてそのアカウントでルートとしてサインインできなくなる。さらに、通常のルートのパスワードリセットなども無効になる。（仮に必要な場合どのように復帰するんだろう？）
![alt text](/images/articles/root-access-management/delete-root-user-credentials.png)

実行すると、ルートでログインできなくなるようなので、削除対象のアカウントに一度ルートでサインインしてみる。MFAも利用しており、問題なくログインできた。
(この画面は削除対象のアカウントの画面)
![alt text](/images/articles/root-access-management/login-before.png)

削除してすぐはAccess Deniedが吐かれていて、取りつく島がない状態だった。
![alt text](/images/articles/root-access-management/Denied.png)

削除対象のアカウントにログインしようとすると、失敗するので期待する動作が実現できている。
![alt text](/images/articles/root-access-management/FailedLogin.png)


しばらく待つと、パスワードリカバリーが可能になった。
![alt text](/images/articles/root-access-management/AllowPasswordRecovery.png)

# 感想
メンバーのルートの操作やアクセスの一切を禁止できるので便利だ。特にSecurityHubのIAM.6を無効にできるのはとてもありがたい。メンバー側でのアカウント作成時の機微なタスクを減らしつつ、物理的なMFAデバイスの管理負担がなくなってとても便利だ。
ただ、新規アカウント登録時にメールアドレスの用意が不要になる、といったものではないので少し残念だった。まぁ、連絡系統が用意できていないと個別のアカウントへのAWSからの案内や通知ができないから当然といえば当然か。