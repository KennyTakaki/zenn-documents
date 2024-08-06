---
title: "AWS Step Functions から YouTube Data API を呼び出してみる"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,Youtube,StepFunctions]
published: true
---

# 概要
気になっていたけど触っていなかったサービス、機能の一つにAWS StepFunctionsの外部API呼び出しがある。試しにGoogleの提供するYoutube Data APIをStep Functionsから呼び出してみようと思う。

# AWS Step Functionsのどこに興味があるか？
なぜAWS Step Functionsに興味があるかというと、Lambda関数の置き換えになると考えているからだ。コードを書かずにシステムが組めるのであればそれに越したことはない。
例えば、Athenaなどと連携する際にAWS Step Functionsはクエリ結果の待機を状態として持てるなど、AWSの他のサービスとの連携も行いやすく機能を把握しておくことでシステム設計の難易度や実装量が下がると感じている。外部連携もここに任せられるのであればEventBridge等をトリガとしたワークフロー形成がとても楽になるはずだ。

# Youtube Data API
取得するデータはなんでもよいのだが、ここではサービスとしてなじみ深いYoutubeを操作してみたい思う。[Youtube Data API](https://developers.google.com/youtube/v3/docs?hl=ja) を利用するとYouTube Web サイトで通常実行する機能を、自分の Web サイトやアプリケーションに統合することができる。  

今回はなじみの深い[検索](https://developers.google.com/youtube/v3/docs/search?hl=ja)をAPIから実行してみる。キーワードに合わせた動画のリストが得られるのが期待値だ。  

Youtube Data API はプロジェクト内でコールできる上限がユニットで定義されており、[デフォルトの1日の総呼び出しユニットは10,000ユニット](https://developers.google.com/youtube/v3/guides/quota_and_compliance_audits?hl=ja)が提供されている。
> YouTube Data API が有効になっているプロジェクトでは、1 日あたりのデフォルトの割り当てが 10,000 ユニットになっています。

今回利用する[検索のAPIは1コールで100ユニットを消費する](https://developers.google.com/youtube/v3/determine_quota_cost?hl=ja)。
![alt text](/images/articles/stepfunctions-youtubeapi/searchcost.png)


適当に叩いてみたサンプルのコマンドは以下。qでクエリを指定するとともに、最新のものを取得するためにアップロード時刻を条件に加える場合はpublishedAfterを利用する。今回は簡易な検証のため、OAuth2.0による認証は行わず API Keyで実施する。
```sample_command.sh
 curl -G \
     -d key=YOUR_API_KEY\
     -d type=video \
     -d part=snippet \
     -d q=ソフトテニス\
     -d type=video\
     -d publishedAfter=2024-08-05T00:00:00Z \
     "https://www.googleapis.com/youtube/v3/search"
```
  
:::details サンプルレスポンス
{
  "kind": "youtube#searchListResponse",
  "etag": "ZipimXSsB3q7kxBmpD00zaAEvHQ",
  "nextPageToken": "CAEQAA",
  "regionCode": "JP",
  "pageInfo": {
    "totalResults": 71,
    "resultsPerPage": 1
  },
  "items": [
    {
      "kind": "youtube#searchResult",
      "etag": "FvpNUYg3B1pSPjC0AQpQKFPuA78",
      "id": {
        "kind": "youtube#video",
        "videoId": "583RQuIYbKg"
      },
      "snippet": {
        "publishedAt": "2024-08-05T09:00:06Z",
        "channelId": "UCmkqivrWCRprV01OU5o1diw",
        "title": "2024年 全日本実業団ソフトテニス選手権大会 男子 準決勝 第1対戦 広岡宙・長江光一(NTT西日本) 対 中本圭哉・鈴木琢巳(福井県庁A)",
        "description": "2024年 男子第69回 女子第68回 全日本実業団ソフトテニス選手権大会 主催：（公財）日本ソフトテニス連盟、長浜市、 ...",
        "thumbnails": {
          "default": {
            "url": "https://i.ytimg.com/vi/583RQuIYbKg/default.jpg",
            "width": 120,
            "height": 90
          },
          "medium": {
            "url": "https://i.ytimg.com/vi/583RQuIYbKg/mqdefault.jpg",
            "width": 320,
            "height": 180
          },
          "high": {
            "url": "https://i.ytimg.com/vi/583RQuIYbKg/hqdefault.jpg",
            "width": 480,
            "height": 360
          }
        },
        "channelTitle": "公益財団法人日本ソフトテニス連盟",
        "liveBroadcastContent": "none",
        "publishTime": "2024-08-05T09:00:06Z"
      }
    }
  ]
}
:::

# State Machine を作成して実行する
1. Step Functions のサービスからステートマシンを作成を選択して、blank を選択する。
![alt text](/images/articles/stepfunctions-youtubeapi/select-blank.png)

2. Call third-party APIをドラッグしてステートマシンに追加
![alt text](/images/articles/stepfunctions-youtubeapi/callthirdparty.png)

3. 追加したワークフローのタスクを編集
- API パラメータ
https://www.googleapis.com/youtube/v3/search
- メソッド
GET
- Authentication  
今回の場合は利用しないいが、入力が必須なのでダミーで作成する。以下説明文のその理由後ご確認くださいからEventBridge接続を作成し、適当なリソースを作成してそのリソースを指定する。
> Step Functions の HTTP エンドポイントの認証は、EventBridge Connection リソースを使用して処理されます。その理由をご覧ください。EventBridge コンソールで新しい接続を作成するか、または以下で既存の接続を選択できます。

![alt text](/images/articles/stepfunctions-youtubeapi/createeventbridge.png)
- 詳細パラメータを設定
クエリパラメータの覧に以下を入力（curlで実行した際のオプション相当）
{
  "key": "Your API KEY",
  "type": "video",
  "part": "snippet",
  "q": "ソフトテニス",
  "publishedAfter": "2024-08-05T00:00:00Z"
}

4. 作成する
画面右上の作成からワークフローを作成する。このとき必要なIAMポリシーを作成してくれ、ポリシーを紐づけたロールが作成される。
![alt text](/images/articles/stepfunctions-youtubeapi/StepFunctionsIAM.png)

5. 実行する
WFを実行するとAPIが叩かれて、レスポンスを確認することができる。
![alt text](/images/articles/stepfunctions-youtubeapi/exec.png)


# 所感
- コードを書かなくていいのは楽だと感じた。
- ランタイムのバージョンを意識しなくてよいのもいい点だなと感じた。
- OAuth2.0 対応も今後試してみたい
- 結果を後段のDBに保存する部分も近々実装してみたい
- IAMで生えるポリシー（StepFunctionsInvokeHttpEndpointScopedAccessPolicy-hash）を確認すると以下のようなポリシーになっていた。
呼び出すAPIに対してInvokをAllowすることができ、セキュアだと感じた。


``` stepfunctions-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Sid": "InvokeHttpEndpoint1",
            "Action": [
                "states:InvokeHTTPEndpoint"
            ],
            "Resource": [
                "arn:aws:states:us-east-1:730335331323:stateMachine:*"
            ],
            "Condition": {
                "StringEquals": {
                    "states:HTTPEndpoint": [
                        "https://www.googleapis.com/youtube/v3/search"
                    ],
                    "states:HTTPMethod": [
                        "GET"
                    ]
                }
            }
        }
    ]
}
```