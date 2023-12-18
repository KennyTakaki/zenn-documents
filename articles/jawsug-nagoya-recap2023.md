---
title: "JAWS-UG 名古屋 2023年 参加記録"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,JAWS,ReCAP]
published: true
---

# JAWS-UG名古屋2023/12参加記録

## 	AWS Step Functionsの新機能「Call third-party API」を試してみた
#### JAWS-UG 名古屋　三浦さん

Step Functionsから外部APIを呼び出せるようになったので速攻で試してみた、というLTでした。
Lambdaを利用せずローコードで外部連携できるサーバレスシステムが構築できるのは魅力的だと感じました。
一方で、一部IAMロールの設定をきちんと理解しておかないと外部APIコールができないようです。設定時の癖を把握しておけば便利に実装可能なんだなと理解できました。

![Alt text](/images/articles/jawsug-nagoya-recap2023/miura.jpg)

## 現地初参加の体験談	
#### 加藤寛士さん
re:Inventに現地参加してAWS Heroな皆様に後押しされてJAWS-UG名古屋での登壇にこぎつけたそうです。現地参加でコネクションを広げて、日本でもその勢いを保っているのがすごいですね。
自分たちの中だけでの活動では限界があって、外に出ていく必要があると強く感じたそうで、自分が利用したことのないサービスでLTを企画されました。
AmazonQや、それと連動するApplication Composer を使ってみたが、マシンスペックが少し足らなかったようでこれから活用の帰路を探るそうです。
AmazonQに関しては「AmazonQにVSCodeないでファイルを作ってくれ」とお願いしたそうですがかたくなに断られるようです。（ちょっとこれは難易度が高いかもしれないなぁ）
「このアカウントのLambda一覧をみせてくれ」に対しては一覧の取得方法を提案されたそうでリストそのものは提示されなかったとのこと。問い合わせ方や使い方を工夫する必要がありそうですね。
  
加藤さん個人敵にはこれからコミュニティ参加を活発に行いたいそうです。すごいモチベーションがあがってていいなぁと思いました。現地参加素晴らしい。

![Alt text](/images/articles/jawsug-nagoya-recap2023/kato_ok.jpg)

## 現地の体験談とAurora Limitless Databaseについて	
#### 榊原慶太さん
AWS Jamに参加したという報告をなさっていました。日本人2名と海外エンジニア2名の系4任でGenerativeAIに関する課題をこなしたそうです。
短時間での深いディスカッションができなかったのが心残りだそうです。もっと積極的に周りのスタッフに質問すればよかったと反省なさってました。

普段全く運動しないのにre:Invent 5K runにも参加されたそうで色々楽しまれていますね。

Network-JAWS 勉強会 in Lasvegas にも参加。現地交流の価値が高いとのこと。
Japan Night で興味を持った Aurora Limitless Database に関して帰国後に勉強してLTに結びついてまし。

「DBシャーディングをマネージドに行ってくれる」のが素晴らしい。シャーディング（書き込みの水平スケーリング）を実現する機構で、これをAWSがマネージドに提供してくれるそうです。オフィシャルの記事を見るとトランザクションも保証してくれるようです。書き込みが多いDBをRDBMSで設計する場合に利用したいなと感じました。

![Alt text](/images/articles/jawsug-nagoya-recap2023/sakakibara.jpg)

## AI関連のアップデートについて	
#### 川路さん
キーノートからの引用でAWSが生成系AIイノベーションセンターを作った、という点から導入されました。AmazonはGAFAの中では生成系に一手遅れている印象があったのですが、そうではないというメッセージを感じました。
すごい速度でAI関連のニュースを駆け抜けていただきました。10分のLTで資料が数十枚あったので走馬灯のような情報量でした。9分過ぎた時点で33p、半分程度の消化率には笑いました。
![Alt text](/images/articles/jawsug-nagoya-recap2023/kawaji0.jpg)
印象的だったのはAgents for Bedrockです。AIは人間の使い方が重要なのですが、その使い方を補助する機能だと理解しました。例えば、プロンプトに同時に２つの質問を入れたとして、それをAIが理解しやすいように裏では２つに分解してAIにインプットなどがあるそうです。
![Alt text](/images/articles/jawsug-nagoya-recap2023/kawaji.jpg)

## RDS／Aurora関連のアップデート
#### hmatsu47さん
安心のhmatsu47さんによるRDS関連のアップデート紹介です。
- RDSがIBMのDb2に対応
- Aurora Limitless Database シャーディングしてMulti-Writer対応になった
- Limitless Database : Time Sync Serviceに依存している
- Data APi(Aurora Servless v1)とAppSyncの連携強化（なぜVer1なんだ？）
- RDS / Aurora でRedshift との　Zero-ETL統合 （複雑なETLの設定なしにニアリアルタイムデータ連携）
  （この件はよくわからないからあとで聞いてみよう）


## QuickSightのアップデート	
#### JAWS-UG 田中さん
- QuickSightがBigQueryとも連携するようになった。SPICEのアップデートで不足分の自動購入機能が追加されて容量枯渇でのエラーがなくなったので運用しやすくなったとのこと。ただし使いすぎの監視は必要。
- Cost Esplorer → QuickSight が可能になった
- Custom Week Start：週始まりを月曜始まりに設定することができる。視認性が向上しそうですね。
- Amazon Q in QuickSight：自然言語での操作が可能になった。（英語のみ対応）
　①自然言語を用いてビジュアル作成/編集可能  
　②計算フィールドの作成  
　③データストリームの作成

## Amazon S3 Express One Zone の話+ 現地体験談
#### Kazuno Fukudaさん
コラボさんからの参加です。いつも会場提供ありがとうございます。
Amazon S3 Express One Zone はStandardに比べて専用HWを利用することで10倍の性能を提供するそうです。通常のバケットよりもコストメリットもありそうなので、今後利用シーンもありそうですね。DRなど抜きにして高速にR/Wしたい場合に最適そう、とコメントされていました。
実際にLambda（Python）からのデータハンドリングを試してみたところ、Upload、Downloadでは３～４倍の性能差が出たそうです。一時的なオブジェクトを高速にハンドリングしたい、という場合にはいいかなぁ。

## JAWS UG 名古屋 2023年を振り返る
#### JAWS-UG Katzさん
JAWS-UG名古屋の闇、集計していないアンケートをQuickSightで可視化してくれました。Ｑまではマラソンのダメージでたどり着けなかったようですが、サブ３ってすごすぎますね。サブスリーのコミュニティービルダーーって上野さんだけなんじゃないだろうか。。。？


## Trusted AdvisorのAPI or IICのAPI
#### JAWS-UG 山下さん
IICってなんだろうと思ったら旧SSOのことを指しているんですね。たしかにIAM　アイデンティティセンターってアイデンティティが二回入っていて早口言葉のようです。いまだにSSOと呼んでしまうので、これからはIICと呼ぶ地ならし運動に参加しようと思います。

re:Invent前に日本でたっぷり吞んでから向かうのは脱帽です。re:Invent組で共通するのは無尽蔵の体力だと思います。