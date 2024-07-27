---
title: "ControlTowerの仕様調査"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,ControlTower]
published: false
---
# 概要
前回記事で設計したOUを実装していく。ControlTowerを設定して、追加のOUを定義する。
この記事では簡単な手順も記載するが、利用するAWSアカウントは過去に何度かControlTowerやSecurityHubといったセキュリティ系のサービスを設定したことがあり、手作業で作成する必要のあるリソースなどの既存リソースが存在する可能性があるので手順は参考程度のものです。

# ControlTower
[ControlTower](https://docs.aws.amazon.com/controltower/)はUserGuideに加えてセキュリティやコントロールに特化したドキュメントがあるので、上記からたどっていくとよい。

管理者としてのノウハウは[このページで確認可能](https://docs.aws.amazon.com/controltower/latest/userguide/best-practices.html)

# リソース削除/修正に関して
基本的にはControlTowerが作成したリソースに手を加えてはいけない。
https://docs.aws.amazon.com/controltower/latest/userguide/getting-started-guidance.html

# 通知に関する修正
aws-controltower-AllConfigNotifications SNSトピックがConfigなどで検出するすべてのセキュリティに関係する通知を受信する。
aws-controltower-BaselineCloudTrail 証跡のデータイベントも aws-controltower-AllConfigNotifications に配信される。
詳細なコンプライアンス上の通知を受信するにはaws-controltower-AllConfigNotificationsを購読するとよい。
ドリフトやコンプライアンスの変化をすべてでなく少数受信したいなら aws-controltower-AggregateSecurityNotifications　を購読するとよい。

https://docs.aws.amazon.com/controltower/latest/userguide/sns-guidance.html



# 関連するサービス
関連サービス一覧がここにある。
https://docs.aws.amazon.com/controltower/latest/userguide/integrated-services.html

## SecurityHub
https://docs.aws.amazon.com/controltower/latest/userguide/security-hub.html
AWS Control Towerは、Service-Managed Standardと呼ばれるSecurity Hub標準によってAWS Security Hubと統合されている[Security Hub標準](https://docs.aws.amazon.com/controltower/latest/controlreference/security-hub-controls.html)。

このService-Managed Standardは[AWS Foundational Security Best Practices(FSBP)のサブセット](https://docs.aws.amazon.com/securityhub/latest/userguide/service-managed-standard-aws-control-tower.html#aws-control-tower-standard-controls)になっており、197個のコントロールで構成される（これで十分なのでは？）。一つだけ重複があったので、後で詳細を確認しておこう。([Neptune.4] Neptune DB clusters should have deletion protection enabledが重複している。)

標準を作成したからといってコントロールが有効になるわけではないらしい？  
No controls are enabled automatically when you create this standard in AWS Control Tower.
  
オペレーションはここが参考になる。
https://dev.classmethod.jp/articles/security-hub-service-managed-standard-control-tower/


# AWS FSBP(AWS Foundational Security Best Practices)
AWS FSBPのセキュリティコントロール数は255個だ。
https://docs.aws.amazon.com/securityhub/latest/userguide/fsbp-standard.html

# logging and monitoring
https://docs.aws.amazon.com/controltower/latest/userguide/logging-and-monitoring.html

# ルートユーザのパスワード変更って…
ControlTowerの公式ではパスワード忘れから更新することになってるけど、これ正規プロセス？
https://docs.aws.amazon.com/controltower/latest/userguide/root-login.html


# Control Towerによる推奨コントロール
https://docs.aws.amazon.com/controltower/latest/controlreference/strongly-recommended-controls.html