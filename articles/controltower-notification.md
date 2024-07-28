---
title: "Control Towerの通知系を整理してカスタマイズしたい"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,ControlTower,SNS,EventBridge]
published: true
---

# 概要
[前回のSecurity Hubとの関係性に続いて](https://zenn.dev/frommiddle1/articles/controltower-securityhub-spec)、AWSの組織構造にセキュリティ上の検出機構を適用させたかったので Control Tower を調べた。私にとっては複雑に感じたので記事を作成する形で整理する。今回は特に Control Tower の通知系統を整理する。

# Control Towerの通知の不満点
触ってみて直感的に以下の感想が思い浮かんだ。これらを改善したい。
1. 通知が多すぎる
2. 通知の内容が生のイベントでツライ
3. 通知先がSecurity Tooling (Audit)以外のメールアドレスに来るので修正したい

上記に対してどのリソースに対して、どのような修正を入れたらいいのかいまいちわからない点が課題。
基本的にはControlTowerのリソースはSCPで保護されており削除、修正をすることが

# ControlTowerのセキュリティの検出、通知系のリソース概要
ひとまず概要レベルでControlTowerが作成するリソースの関係性を取りまとめてみた。通知にフォーカスしたリソースのみ記述しており、すべてのリソースを記述してはいない。青色で名前を記述しているサービスはリージョンごとにリソースが作成される。
![alt text](/images/articles/controltower-notification/arc.png)

リソースに関しての簡易な説明は下記
### 全アカウント共通
- AWS CloudTrail
各アカウントのホームリージョンに証跡が設定される。証跡はLog Archive アカウントのS3バケットで保存されるとともに、管理アカウントのCloud Watch Logsに配信される。

- AWS Config
レコーダーが設定される。デフォルトの記録頻度は連続で、リソースに変化があるたびに記録される。[リソースタイプごとに制御したい場合はソリューションが公開されているよう](https://aws.amazon.com/jp/blogs/news/customize-aws-config-resource-tracking-in-aws-control-tower-environment/)だが、詳細は調べていない。

- Amazon EventBridge(aws-controltower-ConfigComplianceChangeEventRule)
Configに対して下記のイベントをキャッチしてSNSに一致したイベントを送付する。通知内容を加工するためにトランスフォーマなどを入れたくなるが、ControlTowerのSCPで保護されており変更はできない。
```
{
  "detail-type": ["Config Rules Compliance Change"],
  "source": ["aws.config"]
}

```
- Amazon SNS①(aws-controltower-SecurityNotifications)
ConfigのComplianceChangeを検知して後段のLambdaに

- AWS Lambda(aws-controltower-NotificationForwarder)
Audit アカウントの集約用SNSにConfigのComplianceChangeを送付する。

### 管理アカウント
- Amazon CloudWatch(aws-controltower/CloudTrailLogs)
ロググループが作成される。すべてのアカウントからCloudTrailの証跡が配信されている。

### Log Archive アカウント
- Amazon S3(aws-controltower-logs-{accountid}-{region})
各アカウントのAWS CloudTrailの証跡とAWS ConfigのConfiguration Itemsが保存される。

### Audit アカウント
- Amazon SNS②(aws-controltower-AllConfigNotifications)
Configからのリソース変更に関する通知と、CloudTrailからの配信の設定がなされている。CloudTrailの配信設定は証跡のファイルがS3に配置されたことを通知するもので、SNSを通じてイベントの中身を知ることはできない。デフォルトではサブスクリプションの通知は設定されていない。

- AWS Config
Aggregator が設定されており、メンバーアカウントとLog Archiveアカウントのリソースが確認可能。また、数種のConfig Ruleが設定されている。

- Amazon SNS③(aws-controltower-AggregateSecurityNotifications)
Configのコンプライアンス通知をメンバーアカウントやLog Archiveアカウントから収集する。デフォルトではAuditアカウント作成時のメールアドレスに対して、サブスクリプションが設定されている。

# 通知先制御に利用する仕様確認
通知先の制御に利用するリソースは下記のページに記述がある。

[こちらにあるように](https://docs.aws.amazon.com/controltower/latest/userguide/getting-started-guidance.html)基本的にはControlTowerが作成したリソースに手を入れるべきではない。
> Do not modify or delete any resources created by AWS Control Tower, including resources in the management account, in the shared accounts, and in member accounts. If you modify these resources, you may be required to update your landing zone or re-register an OU, and modification can result in inaccurate compliance reporting. 

[通知に関しての受信はSNS①～③を用途に分けてサブスクライブせよとある]。(https://docs.aws.amazon.com/controltower/latest/controlreference/receive-notifications.html)

:::message
補足。Control Towerに限らずだが、ドキュメントは英語で確認したほうがよい。
アクセスできる情報の量が倍以上になる。
英語版
![alt text](/images/articles/controltower-notification/doctop-eng.png)
日本語版
![alt text](/images/articles/controltower-notification/doctop-jpn.png)
:::

# 結局？
仕様上はリソースを変更してはだめだし、ControlTowerで生えるSNSのサブスクリプションでの制御しかできないことになる。EventBridgeのトランスフォーマーなんかで設定したかったが、フィルタリングや通知内容を整形したい場合はLambda関数を独自に作成せよとのことであった。
> Administrators who wish to filter out specific types of notifications from an SNS topic can create an AWS Lambda function and subscribe it to the SNS topic. Alternatively, you can set up an EventBridge rule to filter notifications, as described in this support article, How can I be notified when an AWS resource is non-compliant using AWS Config?

SNS③を購読するLambda関数をつくって、そこから再度SNSを経由して通知を流すのがよさそうだ。