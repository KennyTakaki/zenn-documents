---
title: "Cloud9 の代替ソリューションを試してみたらDeployに失敗した"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,Cloud9,SageMaker]
published: true
---

# 概要

コトのいきさつは[みのるんさんがまとめている](https://qiita.com/minorun365/items/f5289163795d5d7b21e2)ようにCloud9が急遽使えなくなってしまって代替案が必要になるケースが出てきた、というものです。
上記のみのるんさんのブログでも記述があるように代替手段が提案されているのですが、このたびaws-samplesにテンプレートとしてソリューションが実装されリリースされたようです。
https://x.com/toshikwa/status/1820749343705698531

このソリューションを一応触ってデプロイぐらいしてみておくかー、と思ったのですがデプロイでこけました。ユーザグループで環境として利用する際にも少しだけ注意が必要かもしれません。

# デプロイ方法
GitHubのページからデプロイ可能、ここでは東京(ap-northeast-1)でデプロイする。

リポジトリは以下。
https://github.com/aws-samples/sagemaker-studio-code-editor-template

1. GitHubでap-northeast-1を選択
![alt text](/images/articles/cloud9-replacement/deploy.png)

2. マネコンに遷移するのでデフォルトのまま進める
![alt text](/images/articles/cloud9-replacement/page01.png)

3. スタックの詳細で設定を行う
- AutoStopIdleTimeInMinute : 自動で環境を停止させるまでの時間（分）
- EbsSizeInGb : EBSのサイズ(GB)
- InstanceType：インスタンスタイプ
![alt text](/images/articles/cloud9-replacement/page02.png)

4. 後段のページは基本的にそのままで作成
ただし、「AWS CloudFormation によって IAM リソースがカスタム名で作成される場合があることを承認します。」のみチェックをいれる。

...なんだ簡単だなと思ったら、デプロイ時にこけました。

# 何が悪かったのか？
**結論を先に書くと、このソリューションはアカウント内にデフォルトVPCが必要です。**  
  
さて、デプロイ時にCloudFormationが止まってしまいました。Create Failedです。
![alt text](/images/articles/cloud9-replacement/deploy-failed.png)

エラーメッセージを引用するとカスタムリソースからエラーが返ってきているようです。カスタムリソースのコード内のリストのインデックス外を参照しているような挙動をしています。
> Received response status [FAILED] from custom resource. Message returned: list index out of range (RequestId: af07e59d-1faa-43df-9d2e-a206a9c03715)

当該エラーに対応するコードを調べてみると、このソリューションはデフォルトVPCがアカウント内に存在することが前提になっているようです。
ソリューション内の[default_vpc_lookup.py](https://github.com/aws-samples/sagemaker-studio-code-editor-template/blob/main/src/default_vpc_lookup.py)を確認してみると、以下のコードが確認できます。out of indexは2行目のコードで発生していますね。
```
    res = ec2.describe_vpcs(Filters=[{"Name": "isDefault", "Values": ["true"]}])
    vpc_id = res["Vpcs"][0]["VpcId"]
```

**デフォルトVPCはセキュリティのプラクティスとして削除することが推奨されていたり、ControlTowerなどから払い出したメンバーアカウントには存在しないことがあります。**
そういったアカウントで作業している場合、本ソリューションはここでこけることになります。

# リカバリー方法
デフォルトVPCを再作成することでリカバリーします。
![alt text](/images/articles/cloud9-replacement/create-default-vpc.png) 

下記画面で「デフォルトVPCを作成」を押す。
![alt text](/images/articles/cloud9-replacement/create-default-vpc.png)

これでデフォルトVPCが作成されました。

# 再度デプロイ
再度スタックをデプロイしてみると成功しました。
![alt text](/images/articles/cloud9-replacement/deploy-succeeded.png)

環境にアクセスしてみると、ちゃんと作成されている様子。よかった。
![alt text](/images/articles/cloud9-replacement/ide.png)