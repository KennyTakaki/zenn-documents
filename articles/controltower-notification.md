---
title: "ControlTowerの通知系を整理してカスタマイズしたい"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,ControlTower,SNS,EventBridge]
published: false
---

# 概要
[前回のSecurity Hubとの関係性に続いて](https://zenn.dev/frommiddle1/articles/controltower-securityhub-spec)、AWSの組織構造にセキュリティ上の検出機構を適用させたかったので Control Tower を調べた。私にとっては複雑に感じたので記事を作成する形で整理する。今回は特に Control Tower の通知系統を整理する。

# Control Towerの通知の不満点
触ってみて直感的に以下の感想が思い浮かんだ。これらを改善したい。
1. 通知が多すぎる
2. 通知の内容が生のイベントでツライ
3. 通知先がSecurity Tooling (Audit) のメールアドレスに来るので修正したい

上記に対してどのリソースに対して、どのような修正を入れたらいいのかいまいちわからない点が課題。
基本的にはControlTowerのリソースはSCPで保護されており削除、修正をすることが



# 通知先制御に利用するリソース
## SNS Topic
通知先の制御に利用するリソースは下記のページに記述がある。
https://docs.aws.amazon.com/controltower/latest/userguide/sns-guidance.html

https://docs.aws.amazon.com/controltower/latest/controlreference/receive-notifications.html

:::message
Control Towerに限らずだが、ドキュメントは英語で確認したほうがよい。
アクセスできる情報の量が倍以上になる。
英語版
![alt text](/images/articles/controltower-notification/doctop-eng.png)
日本語版
![alt text](/images/articles/controltower-notification/doctop-jpn.png)
:::


- aws-controltower-AllConfigNotifications  
コンプライアンス通知や Amazon CloudWatch イベント通知などAWS Configが発生させる全ての通知を受信する
- The aws-controltower-SecurityNotifications
- aws-controltower-AggregateSecurityNotifications



# 証跡
- aws-controltower-BaselineCloudTrail

