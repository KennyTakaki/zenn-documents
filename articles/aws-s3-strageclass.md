---
title: "S3ストレージクラスの整理(Intelligent-Tiering)"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "S3"]
published: true
---

# 目的

家族写真が増えてきて、どこにアーカイブを持っていこうかなぁと検討していたのですが、AWSが手になじんている
こともあり結局S3に保存することにしました。iCloud、GooglePhoto、Amazon Photoなどとの比較はこちらでも比較しているのですが、
https://speakerdeck.com/kennytakaki/jia-zu-noxie-zhen-dong-hua-dokonibao-cun-sitemasuka-jawsug-nagoya-20260131
S3のストレージクラスとIntelligent-Tieringの料金と仕様を事前にまとめておきたいので、記事を作成した。

# ストレージクラスと料金

表にすると以下のようになる。ドル円は155円で計算。Archive Access クラスにはうまみがないのでDeep Archive Accessだけを所定期間で有効にするのがよさそうだ。

| 階層 (Tier)            | 料金 (USD/GB) | 月額 (USD / 1TB) | 日本円目安 (1TB/月) | アクセス速度 | 自動移行    |
| ---------------------- | ------------- | ---------------- | ------------------- | ------------ | ----------- |
| Frequent Access        | $0.025        | $25              | **約3,875円**       | ミリ秒       | ―           |
| Infrequent Access      | $0.0138       | $13.8            | **約2,139円**       | ミリ秒       | ○           |
| Archive Instant Access | $0.005        | $5               | **約775円**         | ミリ秒       | ○           |
| Archive Access         | $0.0045       | $4.5             | **約698円**         | 3〜5時間     | ×（要設定） |
| Deep Archive Access    | $0.002        | $2               | **約310円**         | 約12時間     | ×（要設定） |

# コード

cdkのコードで記述するとこんな感じだ。

```ts:s3-it.ts
import * as cdk from 'aws-cdk-lib/core';
import * as s3 from 'aws-cdk-lib/aws-s3';
import { Construct } from 'constructs';

export class PhotoSystemStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, 'PhotoDataBucket', {
      bucketName: `Photo-data-${cdk.Aws.ACCOUNT_ID}-${cdk.Aws.REGION}`,
      versioned: true,
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
      removalPolicy: cdk.RemovalPolicy.RETAIN,
      intelligentTieringConfigurations: [
        {
          name: 'archive-config',
          // Archive Access クラスにはうまみがないのでコメントアウト、使う場合はとる
          // archiveAccessTierTime: cdk.Duration.days(90),

          //Deep Archive Accessだけを所定期間で有効にする。半年。
          deepArchiveAccessTierTime: cdk.Duration.days(180),
        },
      ],
    });
  }
}
```
