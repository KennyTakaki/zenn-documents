---
title: "SSMエージェントを利用したオンプレからのAWSアクセス"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,SSMエージェント,IAM]
published: false
---

# やりたいこと
オンプレミスのマシンからAWSにアクセスしたい。実施したいことはS3からのデータ取得で非常に簡単だ。
オンプレミスのマシンに対してIAMのアクセス制御を付与できれば良いことになる。

以下を参照すると、実施できるような気がしたので調べてみる。
https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/systems-manager-managedinstances.html

>　Systems Manager 向けにハイブリッドおよびマルチクラウド環境を設定すると以下のことを行うことができます。
> ...
> ・AWS Identity and Access Management (IAM) を使ってマシンに実行できるアクションのアクセス制御を一元化します。

## 手順
1. SystemsManager向けにサービスロールを作成する。
![Alt text](/images/articles/aws-iam-role-from-on-premises/service-role01.png)
![Alt text](/images/articles/aws-iam-role-from-on-premises/service-role02.png)
![Alt text](/images/articles/aws-iam-role-from-on-premises/service-role03.png)
2. ハイブリッドアクティベーションを作成する
#### アクティベーションとは
- アクティベーションはアクティベーションコードとアクティベーションIDからなる。  
- アクティベーションコードとアクティベーションIDをSSM Agentのインストール時に指定することでマネージドノード（オンプレミス）からSSMサービスに安全にアクセスできる。
- アクティベーションには有効期限があり、期限切れになると利用できなくなる（登録処理ができなくなる）。
- 登録した全てのオンプレミスサーバは明示的に登録を解除するまで、SystemsManagerのマネージドノードとして登録されたまｍになる。
- CLIからアクティベーションを作る場合はタグ指定できる。このタグはアクティブ化の差異にマネージドノードに割り当てられる(マネコンからタグ付与はできない)
- アクティベーションに対してインスタンス数の制限が紐づく（つまりアクティベーション1：管理対象N<1000）

#### ハイブリッドアクティベーションを作成する
![Alt text](/images/articles/aws-iam-role-from-on-premises/h-activate01.png)
![Alt text](/images/articles/aws-iam-role-from-on-premises/h-activate02.png)
2023-12-14 18:00:00+09:00
![Alt text](/images/articles/aws-iam-role-from-on-premises/h-activate03.png)
![Alt text](/images/articles/aws-iam-role-from-on-premises/install-agent.png)
![Alt text](/images/articles/aws-iam-role-from-on-premises/registered.png)
![Alt text](/images/articles/aws-iam-role-from-on-premises/winproc.png)

プライベートキーの自動ローテーションは設定しておいた方がよさそう

$code = "L9RCEXhgYDcueXjy7Pg9"
$id = "6ac54bee-c014-4e59-8e15-e3967c0a1df2"
$region = "ap-northeast-1"
$dir = $env:TEMP + "\ssm"
New-Item -ItemType directory -Path $dir -Force
cd $dir
(New-Object System.Net.WebClient).DownloadFile("https://amazon-ssm-$region.s3.$region.amazonaws.com/latest/windows_amd64/AmazonSSMAgentSetup.exe", $dir + "\AmazonSSMAgentSetup.exe")
Start-Process .\AmazonSSMAgentSetup.exe -ArgumentList @("/q", "/log", "install.log", "CODE=$code", "ID=$id", "REGION=$region") -Wait
Get-Content ($env:ProgramData + "\Amazon\SSM\InstanceData\registration")
Get-Service -Name "AmazonSSMAgent"
