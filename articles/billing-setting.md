---
title: "AWS の IAM で請求関連の操作を可能にする"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS]
published: true
---

# 概要
AWSの請求関連の操作をIAMで実行可能にしたい。簡単に手順をメモしておく。

# 手順
1. root でマネコンにログインする。
2. サービスでBilling and Cost Management で[アカウント](https://us-east-1.console.aws.amazon.com/billing/home#/account)を選択
3. 「IAM ユーザーおよびロールによる請求情報へのアクセス」で編集を選択
![Alt text](/images/articles/billing-setting/billing.png)  
4. IAM アクセスをアクティブ化をチェックして更新
![Alt text](/images/articles/billing-setting/setting.png) 
5. enabledを確認できたらOK
![alt text](/images/articles/billing-setting/enabled.png)