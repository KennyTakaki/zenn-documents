---
title: "SageMaker Studio Code Editor のテンプレートがデフォルトVPC不要になったので試してみる"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,SageMaker,CloudFormation]
published: true
---

# 概要
[前回の記事](https://zenn.dev/frommiddle1/articles/cloud9-replacement)にSAさんが対応してくれました。
https://x.com/toshikwa/status/1821097548645552579

ということで、SageMaker Studio Code Editor が新しくなってデフォルトVPC以外の選択肢を提供してくれるようなのでデプロイしてみます。

# 変更点の確認
リポジトリはこちら。
https://github.com/aws-samples/sagemaker-studio-code-editor-template

ユーザが指定可能なパラメータが４つに増えて、UseDefaultVpcをfalseに指定すると新規のVPCが作成されるようです。
- AutoStopIdleTimeInMinutes : Idle time before auto-stop of Code Editor (disabled if 0)
- EbsSizeInGb : EBS volume size of Code Editor
- InstanceType : Instance type of Code Editor
- UseDefaultVpc : Whether to use the default VPC (true) or create a new one (false)

というわけでUseDefaultVpcをfalseに設定してスタックを作成してみます。
![alt text](/images/articles/sagemaker-studio-code-editor/create-vpc.png)

CodeEditorStack-vpc という名前でVPCが作成され、public、privateサブネットがそれぞれ2つずつ作成されます。
![alt text](/images/articles/sagemaker-studio-code-editor/vpc.png)

無事スタック作成が完了しました。
![alt text](/images/articles/sagemaker-studio-code-editor/succeeded.png)


期待通りIDEにアクセス可能になっています。
![alt text](/images/articles/sagemaker-studio-code-editor/ide.png)

# 作成時間と削除の時間
## 作成時間
Create Start が     2024-08-07 21:34:17 UTC+0900  
Create Complete が  2024-08-07 21:42:18 UTC+0900  

で約8分で作成が完了しました。VPCを新規作成する場合でも10分を見ておけば十分なようです。

## 削除時間
Delete Start が　   2024-08-07 21:49:55 UTC+0900  
Delete Complete が  2024-08-07 21:52:18 UTC+0900　　

3分かからず削除可能です。

# まとめ
デフォルトVPCを必要としないことで、使いやすくなる気がします。作成削除も併せて15分程度なら十分許容範囲かなぁと感じました。