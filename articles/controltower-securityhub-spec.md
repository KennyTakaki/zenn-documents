---
title: "Control TowerとSecurity Hubの関係性の整理"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,ControlTower,SecurityHub]
published: true
---

# 概要
AWSの組織構造にセキュリティ上の検出機構を適用させたかったので Control Tower を調べた。[Control Towerは多くのサービスと統合されており](https://docs.aws.amazon.com/controltower/latest/userguide/integrated-services.html)、私にとっては複雑に感じたので記事を作成する形で整理する。今回は特に Security Hub の関係を整理する。

# AWS Security Hub とは？
AWS Security Hub は、AWS のセキュリティ状態を包括的に把握することが可能で、業界標準やセキュリティベストプラクティスに照らした AWS 環境評価を行うことができる。この業界標準やセキュリティベストプラクティスはセキュリティ標準（Security Standard）と呼ばれ、それぞれのセキュリティ標準はセキュリティコントロール（Security Control）と呼ばれる検出項目の集合で構成される。
https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards.html

AWSで用意している[セキュリティ標準は以下の6つが存在](https://docs.aws.amazon.com/securityhub/latest/userguide/standards-reference.html)し、運用する組織に合わせて選択する。組織に対してのセキュリティ上の要求がわからない場合は、まず AWS Foundational Security Best Practices (FSBP) standard を採択すればよいと思う。

- AWS Foundational Security Best Practices (FSBP) standard
- CIS AWS Foundations Benchmark
- National Institute of Standards and Technology (NIST) SP 800-53 Rev. 5
- Payment Card Industry Data Security Standard (PCI DSS)
- AWS Resource Tagging Standard
- Service-managed standards（後述する）

# Control Tower との関係性
Control Tower は 前述のセキュリティ標準のうち [Service-managed standards](https://docs.aws.amazon.com/securityhub/latest/userguide/service-managed-standards.html) という形でサービス独自のセキュリティ標準を提供している。  
この標準は[Service-Managed Standard: AWS Control Tower](https://docs.aws.amazon.com/securityhub/latest/userguide/service-managed-standard-aws-control-tower.html)と命名されており、Control Towerの組み込み機能として提供されている。

Service-Managed Standard: AWS Control Tower の実態は AWS Foundational Security Best Practices (FSBP) standard の サブセットで、ユーザはこのサブセットのなかから利用したいコントロールを選択して独自の標準を運用することができる。
https://docs.aws.amazon.com/controltower/latest/userguide/security-hub-controls.html

# Service-Managed Standard: AWS Control Tower の利用方法
基本的には下記のブログに記述の操作を行うことになる。クラメソさんのブログにはお世話になる。
https://dev.classmethod.jp/articles/security-hub-service-managed-standard-control-tower/

# 2つの標準の差
2つの標準の差を確認してみると、どちらか一方にしかないコントロールが存在した。正確なサブセットではないみたいだ。

- [Service-Managed Standard: AWS Control Tower のコントロール一覧(255個)](https://docs.aws.amazon.com/securityhub/latest/userguide/service-managed-standard-aws-control-tower.html)
- [AWS Foundational Security Best Practices (FSBP) standard のコントロール一覧(197個)](https://docs.aws.amazon.com/securityhub/latest/userguide/fsbp-standard.html)

:::details Service-Managed Standard: AWS Control Tower に存在しないコントロール
[AppSync.2] AWS AppSync should have field-level logging enabled
[Backup.1] AWS Backup recovery points should be encrypted at rest
[CloudFront.1] CloudFront distributions should have a default root object configured
[CloudFront.3] CloudFront distributions should require encryption in transit
[CloudFront.4] CloudFront distributions should have origin failover configured
[CloudFront.5] CloudFront distributions should have logging enabled
[CloudFront.6] CloudFront distributions should have WAF enabled
[CloudFront.7] CloudFront distributions should use custom SSL/TLS certificates
[CloudFront.8] CloudFront distributions should use SNI to serve HTTPS requests
[CloudFront.9] CloudFront distributions should encrypt traffic to custom origins
[CloudFront.10] CloudFront distributions should not use deprecated SSL protocols between edge locations and custom origins
[CloudFront.12] CloudFront distributions should not point to non-existent S3 origins
[CloudFront.13] CloudFront distributions should use origin access control
[Config.1] AWS Config should be enabled and use the service-linked role for resource recording
[DataFirehose.1] Firehose delivery streams should be encrypted at rest
[DMS.6] DMS replication instances should have automatic minor version upgrade enabled
[DMS.7] DMS replication tasks for the target database should have logging enabled
[DMS.8] DMS replication tasks for the source database should have logging enabled
[DMS.10] DMS endpoints for Neptune databases should have IAM authorization enabled
[DMS.11] DMS endpoints for MongoDB should have an authentication mechanism enabled
[DMS.12] DMS endpoints for Redis should have TLS enabled
[DocumentDB.4] Amazon DocumentDB clusters should publish audit logs to CloudWatch Logs
[DocumentDB.5] Amazon DocumentDB clusters should have deletion protection enabled
[DynamoDB.6] DynamoDB tables should have deletion protection enabled
[DynamoDB.7] DynamoDB Accelerator clusters should be encrypted in transit
[EC2.24] Amazon EC2 paravirtual instance types should not be used
[EC2.51] EC2 Client VPN endpoints should have client connection logging enabled
[ECS.9] ECS task definitions should have a logging configuration
[EFS.6] EFS mount targets should not be associated with a public subnet
[EKS.3] EKS clusters should use encrypted Kubernetes secrets
[EKS.8] EKS clusters should have audit logging enabled
[ElastiCache.1] ElastiCache Redis clusters should have automatic backup enabled
[ElastiCache.2] ElastiCache for Redis cache clusters should have auto minor version upgrade enabled
[ElastiCache.7] ElastiCache clusters should not use the default subnet group
[ElasticBeanstalk.3] Elastic Beanstalk should stream logs to CloudWatch
[EMR.2] Amazon EMR block public access setting should be enabled
[FSx.1] FSx for OpenZFS file systems should be configured to copy tags to backups and volumes
[FSx.2] FSx for Lustre file systems should be configured to copy tags to backups
[Macie.1] Amazon Macie should be enabled
[Macie.2] Macie automated sensitive data discovery should be enabled
[MQ.2] ActiveMQ brokers should stream audit logs to CloudWatch
[MQ.3] Amazon MQ brokers should have automatic minor version upgrade enabled
[NetworkFirewall.2] Network Firewall logging should be enabled
[NetworkFirewall.9] Network Firewall firewalls should have deletion protection enabled
[Opensearch.10] OpenSearch domains should have the latest software update installed
[PCA.1] AWS Private CA root certificate authority should be disabled
[Route53.2] Route 53 public hosted zones should log DNS queries
[RDS.7] RDS clusters should have deletion protection enabled
[RDS.14] Amazon Aurora clusters should have backtracking enabled
[RDS.16] RDS DB clusters should be configured to copy tags to snapshots
[RDS.24] RDS Database clusters should use a custom administrator username
[RDS.34] Aurora MySQL DB clusters should publish audit logs to CloudWatch Logs
[RDS.35] RDS DB clusters should have automatic minor version upgrade enabled
[Redshift.3] Amazon Redshift clusters should have automatic snapshots enabled
[Redshift.15] Redshift security groups should allow ingress on the cluster port only from restricted origins
[S3.19] S3 access points should have block public access settings enabled
[SageMaker.4] SageMaker endpoint production variants should have an initial instance count greater than 1
[ServiceCatalog.1] Service Catalog portfolios should be shared within an AWS organization only
[StepFunctions.1] Step Functions state machines should have logging turned on
[Transfer.2] Transfer Family servers should not use FTP protocol for endpoint connection
[WAF.1] AWS WAF Classic Global Web ACL logging should be enabled
[WAF.6] AWS WAF Classic global rules should have at least one condition
[WAF.7] AWS WAF Classic global rule groups should have at least one rule
[WAF.8] AWS WAF Classic global web ACLs should have at least one rule or rule group
[WAF.12] AWS WAF rules should have CloudWatch metrics enabled
:::

:::details  AWS Foundational Security Best Practices (FSBP) standard に存在しないコントロール
[CloudTrail.6] Ensure the S3 bucket used to store CloudTrail logs is not publicly accessible
[EC2.22] Unused Amazon EC2 security groups should be removed
[KMS.4] AWS KMS key rotation should be enabled
[Lambda.3] Lambda functions should be in a VPC
[MQ.5] ActiveMQ brokers should use active/standby deployment mode
[MQ.6] RabbitMQ brokers should use cluster deployment mode
[S3.17] S3 general purpose buckets should be encrypted at rest with AWS KMS keys
:::

# 結論
Service-Managed Standard: AWS Control Tower に存在しないコントロールのうち、CloudFrontに関わるものやRDSの削除保護のコントロールは重要だと感じた。そのため ControlTower側からService-Managed Standard: AWS Control Towerで設定するのでなく、SecurityHub側からFSBPを指定して組織にて提供しようと思う。  
(補足：いまの開発範囲であれば、Service-Managed Standard: AWS Control Tower のDMSなんかは不要かな。)

# ドキュメントの間違い
下記のページのコントロール一覧に重複があった。
https://docs.aws.amazon.com/securityhub/latest/userguide/service-managed-standard-aws-control-tower.html
![alt text](/images/articles/controltower-securityhub-spec/image-2.png)

サービスの実装側を確認してみると  
[SH.Neptune.3] Neptune DB cluster snapshots should not be public  
が存在しているので、おそらくドキュメント側の間違いかな？
![alt text](/images/articles/controltower-securityhub-spec/ct.png)