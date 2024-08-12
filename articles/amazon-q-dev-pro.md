---
title: "とりあえずAmazon Q Developer Pro を Subscription する"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,AmazonQ]
published: true
---

# 概要
無料版で体験がよかったのでAmazon Q DeveloperのProを利用しようと思う。育休期間中で、息子のおしめ替え、あやし、ミルク、などで時間がブツ切れになる。強力なメンターや調べものをショートカットできるなら月19USDは安いなと感じた。生成AI界隈が流行っているし、ちょっとしたお遊びだ。

# 手順
[Amazon Q DeveloperのProの認証はIAM Identity Centerで行われる](https://aws.amazon.com/q/developer/pricing/?nc1=h_ls)。Freeプランの場合は Amazon Builder ID に紐づけたので少し意外に感じた。

私の管理しているAWSアカウントではus-east-1でIAM Identity Centerを管理しているので、東京リージョンでQのサービスページを開くとRegionをSwitchするようなボタンが現れる。
![alt text](/images/articles/amazon-q-dev-pro/tokyo-q.png)

IAM Identity Centerの管理リージョンに切り替えると、Amazon Q と連携させることが可能。
![alt text](/images/articles/amazon-q-dev-pro/us-east1-q.png)

Subscriptionのボタンから進んでいって、IAM Identity Centerのユーザを指定するとヒットしない。どうやら IAM Identity Center の FirstNameを指定することで検索にヒットするようだ。
IAM Identity Centerの画面
![alt text](/images/articles/amazon-q-dev-pro/IdentityCenter.png)

Amazon Q のSubscription画面
![alt text](/images/articles/amazon-q-dev-pro/subscription.png)

今回はユーザでSubscriptionを実施したが、グループでも購読可能。ただしグループで登録すると利用開始までに最悪24時間程度の遅延が発生する可能性がある。
>You have successfully created a Amazon Q Developer Pro subscription for 1 users.
If you created subscriptions using groups, there may be a delay of up to 24 hours before your users can successfully access the subscription. Visit Q Developer console to learn more about features.

登録後は管理画面でユーザが確認可能。
![alt text](/images/articles/amazon-q-dev-pro/user.png)


エディタでAmazon Qを利用するために先ほど登録したユーザでログインする。(VS Codeにエクステンションは導入済み)
![alt text](/images/articles/amazon-q-dev-pro/vscode-login.png)

ブラウザが開くので、通常のようにログインを実施する。

::: message
WSLなどを使っているとブラウザが開かないことがある。その場合は環境変数にWindows側のブラウザへのパスを定義してあげればよい。
下記は一例で、Chromeに対してのパスを定義している。
export BROWSER="/mnt/c/Program Files/Google/Chrome/Application/chrome.exe"
:::

ログインが完了するとAmazon Qに対してのチャットが左のペインに表示される。これでPro版のQが利用できるようになったはずだ。
![alt text](/images/articles/amazon-q-dev-pro/complete.png)


次回以降に実際に機能のレビューをしようと思う。