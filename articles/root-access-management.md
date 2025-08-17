---
title: "AWSルートアクセスの一元管理でセキュリティを強化する"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","IAM"]
published: true
---

# 概要
AWS Organizations のメンバーアカウントに対する **ルートアクセスの一元化** を実際に触ってみたのでまとめる。

# 手順

## IAM での一元化設定の有効化
ルートアクセスの一元化は、**Organizations の管理アカウント**から IAM の設定画面で有効化する。  
![IAM有効化画面](/images/articles/root-access-management/iam-enable.png)

「Capabilities to enable」から有効にする機能を選択する。選べる機能は2種類：

- **ルート認証情報の管理**  
  メンバーアカウントのルート認証情報を削除・監査できる。また、特定アカウントにパスワードリカバリーを許可可能。

- **特権ルート操作**  
  例: 誤設定された Amazon SQS/S3 のポリシー削除など、特定のルート操作を実行可能。

![IAM設定画面](/images/articles/root-access-management/iam-setting.png)

なお、このルートアクセス管理自体も **委任可能** になっている。推奨（recommended）とあるが、どのアカウントに委任するのがベストかは検討の余地がある。

## 一元化後の操作
設定完了後は **IAM > Root access management** から Organizations のヒエラルキーを確認できる。  
![設定後の画面](/images/articles/root-access-management/iam-after-setting.png)

対象アカウントを選択し **「Take Privileged action」** を押すとルート関連操作を実行できる。  
![特権アクション](/images/articles/root-access-management/take-privileged-action.png)

削除対象を確認すると、**ルート資格情報（アクセスキー、パスワード、署名証明書、MFA）がすべて削除** されると案内される。  
結果としてそのアカウントでは **ルートサインイン不可** となり、通常のパスワードリセットも無効になる。  
（もし必要になった場合、どう復帰させるのかは気になるところ…）

![削除確認](/images/articles/root-access-management/delete-root-user-credentials.png)

実際に対象アカウントへルートでログイン（MFA有効）したところ、問題なく利用可能だった。  
（削除前のログイン後画面）  
![ログイン前](/images/articles/root-access-management/login-before.png)

管理アカウントから削除を実行すると「Access Denied」が表示される。  
![Access Denied](/images/articles/root-access-management/Denied.png)

その後、対象アカウントでもログインは失敗し、期待通りの動作を確認。  
![ログイン失敗](/images/articles/root-access-management/FailedLogin.png)

削除直後は完全に「取りつく島なし」状態だが、しばらくすると **パスワードリカバリーが可能** になる。  
![パスワードリカバリー](/images/articles/root-access-management/AllowPasswordRecovery.png)

# 感想
- メンバーアカウントのルート操作やアクセスを完全に禁止できるのは便利。  
- 特に **SecurityHub の IAM.6** を無効化できる点はありがたい。  
- アカウント作成時のルート管理タスクや物理MFA管理の負担がなくなるのも良い。  

一方で「新規アカウント作成時にメールアドレス不要」にはならないのは少し残念。  
ただ通知系統を考えると、AWS からの案内を受け取るためにメールが必須なのは当然とも言える。

やってみたあとに気が付いたが[こちら](https://dev.classmethod.jp/articles/root-access-management/)がよくまとまっている。