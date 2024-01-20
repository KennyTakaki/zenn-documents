---
title: "補足資料：JAWS-UG名古屋 2024 Copilot Primer Workshop"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,JAWSUG]
published: true
---

# この記事の目的と使い方
この記事は2024/1/23(火)に実施する「JAWS-UG 名古屋 Amazon ECSハンズオン」の補足資料です。ハンズオンの資料は以下です。ハンズオンをスムーズに実施するための追加情報を提供します。
https://catalog.us-east-1.prod.workshops.aws/workshops/d03316be-3c29-49db-8dc3-eb196c1778c9/ja-JP

以降の章立てはワークショップと合わせています。もし手順で不明点や詰まった場合があったら当該の章を参照してみてください。

#### 当日ハンズをフォロー可能なメンバー
JAWS-UG 名古屋のスタッフがフォローできます。気軽に手を挙げたり声をかけて下さい。

# 導入
特にありません。利用するリージョンに注意してください。

## ワークショップのゴール
特にありません。

## Event Engine を使う場合のコンソールログイン
このページは飛ばしてください。今回はJAWS主催のイベントなので、参加者の皆様が個々人のAWSアカウントを利用することになります。

## Copilot CLIとは
特にありません。Tipsとして、複数アカウントを単一のブラウザで同時に開く手段として「Firefox Multi-Account Containers」が便利です。Firefoxを利用している方はブラウザに追加してみると面白いかもしれません。

# Cloud9とIAMの準備
特にありません。
## 開発用アカウントにCloud9環境の作成
Cloud9のインスタンスを構築する際に注意が必要です。Cloud9はEC2インスタンスをベースとして稼働します。このEC2インスタンスがSSMのエンドポイントと通信可能な設定とする必要があります。

簡易的な方法としては、EIPを取得してパブリックサブネットに配置したEC2インスタンスに割り当てることで通信が可能です。Cloud9はSSMを利用する形式で作成してください。（SSHのアクセスでは少し脆弱かもしれません。NAT構成のネットワークをすでにお持ちの方はプライベートサブネットにインスタンスを配置していただいてもよいと思います。

1. EIPを取得する
![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/get-eip.png)

![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/allocate.png)

2. Cloud9を立ちあげる
![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/c9.png)

ネットワークセッティングではSSMを選択して、VPCのパブリックサブネットに配置する。
![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/c9nw.png)

下までいってCreateする。
![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/c9create.png)

3. Cloud9のインスタンスにEIPを割り当てる
EC2のコンソールに戻ってEIPのAssociate Elastic IP addressを選択する。
![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/eipalloc.png)

![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/associate.png)

これでCloud9の環境が作成できるはずです。

## 開発用アカウントにIAMユーザを作成
マネージメントコンソールでIAMユーザを作成してアクセスキーを発行するまでの手順がハンズオン資料作成時から変わっているようなので以下に手順を記述します。
（余談ですが、永続的な Access Key と Secret Access Keyの利用は推奨されるものではないので、実環境では別の方法を利用したほうがよいです。今回は短時間で削除する前提でこれを利用します。）
 
![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser.png)

![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser2.png)

![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser3.png)

![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser4.png)

![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser5.png)

![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser6.png)

![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser7.png)

![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser8.png)

showを押すとSecret access keyが表示されるので、Access Keyとともに手元に記録してください。
![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser9.png)

## Cloud9環境の設定
#### ストレージの増量　に関しては実施する必要がありません。実施しようとするとエラーが発生するはずです。
（このハンズオン記載の方法でストレージを増量させる場合、Cloud9のインスタンスを作成する際のOSイメージをAmazonLinux2に設定してください。）

# Copilotを用いた TODO アプリの作成
特にありません。
## Application と Evironment の作成
特にありません。
## baackend Service の作成
各コマンドでそこそこ時間を要します。リソースを作成するコマンドは5~7分ほどかかるので、トイレ休憩に都合がよいです。


``` copilot svc init ``` の対話シェルは以下の順で実施されます。（ハンズオン資料の工程の１行目と２行目が逆です）

``` 
Which service type best represents your service's architecture? : Backend Service を選択して Enter
What do you want to name this service?: backend
Which Dockerfile would you like to use for backend? : backend/Dockerfile を選択して Enter
```

## frontend Service の作成
``` copilot svc init ``` の対話シェルは以下の順で実施されます。（ハンズオン資料の工程の１行目と２行目が逆です）

``` 
Which service type best represents your service's architecture? : Load Balanced Web Service を選択して Enter
What do you want to name this service?: frontend
Which Dockerfile would you like to use for frontend? : frontend/Dockerfile を選択して Enter
```

# CI/CDパイプラインの作成
To write
## (オプション)本番環境用アカウントでの準備
To write
## Pipeline の作成
To write

# 後片付け
To write
## 作成したリソースの削除
``` copilot app delete ``` の実行にはかなり時間がかかります。CI/CDパイプラインを作成しない場合でも15分ほどを要します。