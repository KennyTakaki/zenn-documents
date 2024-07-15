---
title: "補足資料：JAWS-UG名古屋 2024 Copilot Primer Workshop"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,JAWSUG]
published: true
---

# この記事の目的と使い方

#### まず最初に…本ハンズオンでは、IAMのアクセスキー/シークレットキーを作成する手順があります。ハンズオン終了後、必ず、絶対、確実に削除してください。 
  
この記事は2024/1/23(火)に実施する「JAWS-UG 名古屋 Amazon ECSハンズオン」の補足資料です。ハンズオンの資料は以下です。ハンズオンをスムーズに実施するための追加情報を提供します。
https://catalog.us-east-1.prod.workshops.aws/workshops/d03316be-3c29-49db-8dc3-eb196c1778c9/ja-JP


以降の章立てはワークショップと合わせています。もし手順で不明点や詰まった場合があったら当該の章を参照してみてください。
本ハンズオンではコマンドの待ち時間が発生することが何度かあります。その際にトイレ休憩をとってください。下記の動画も今回のハンズオンのサプリメントとしてよいと思いますので是非ご視聴ください。
CICDまで含めて実施すると、今日の作業時間範囲に収まらない可能性もありますので、目安としてはbackend, frontendそれぞれのServiceを作成して、環境削除を実施するのをお勧めします。
CICD抜きの場合、削除の時間には15分ほどを要するので、この時間に動画を観るのもおすすめです。

https://www.youtube.com/watch?v=-xqg95QBK2M



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
デフォルトのVPCであれば問題なく起動すると思われます。

スタッフの一名の環境ではパブリックIPが自動で割り振られない状態でしたので、その際のトラブルシュート方法を以下に示します。
簡易的な方法として、EIPを取得してパブリックサブネットに配置したEC2インスタンスに割り当てることで通信が可能です。Cloud9はSSMを利用する形式で作成してください。（SSHのアクセスでは少し脆弱かもしれません。NAT構成のネットワークをすでにお持ちの方はプライベートサブネットにインスタンスを配置していただいてもよいと思います。

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

showを押すとSecret access keyが表示されるので、Access Keyとともに手元に記録してください。*** 誰にも教えず、ハンズオンが終わったら削除してください。絶対に。 ***
![Alt text](/images/articles/jawsug-nagoya-copilotprimerworkshop-supplement/create-iamuser9.png)

## Cloud9環境の設定
#### ストレージの増量　に関しては実施する必要がありません。実施しようとするとエラーが発生するはずです。
このハンズオン記載の方法でストレージを増量させる場合、Cloud9のインスタンスを作成する際のOSイメージをAmazonLinux2に設定してください。
事前にテストランした際にはストレージを増やさなくても完走することができました。

# Copilotを用いた TODO アプリの作成
特にありません。
## Application と Evironment の作成
特にありません。
## backend Service の作成
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
下記のコマンドを実行してパイプラインの処理が完了するまでに15分ほどかかります。
```
git add -A
git commit -m 'initial commit'
git push origin main
```

## (オプション)本番環境用アカウントでの準備
特になし。

## Pipeline の作成

以下のコマンドで、git remoteで始まるコマンドのoriginに登録するURLには利用するリージョンが含まれています。ap-northeast-1を利用している場合は差し替えて実行してください。
```
cd ~/environment/copilot-primer-workshop/code
git init
git switch -c main
git remote add origin https://git-codecommit.us-east-1.amazonaws.com/v1/repos/todoapp
```

# 後片付け
特になし。

## 作成したリソースの削除
``` copilot app delete ``` の実行にはかなり時間がかかります。CI/CDパイプラインを作成しない場合でも15分ほどを要します。