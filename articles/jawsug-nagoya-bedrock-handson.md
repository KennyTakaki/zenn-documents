---
title: "生成系AI(Amazon BedRock)のハンズオンに参加　JAWS-UG名古屋"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,JAWS,BedRock]
published: false
---
# 生成系AIのハンズオンイベントに参加（JAWS-UG名古屋）

JAWS-UG名古屋の生成系AIのハンズオンに参加してきた。
https://jawsug-nagoya.doorkeeper.jp/events/164942

Amazon Bedrockと呼ばれる、かなり新しいサービスの体験会だ。
https://aws.amazon.com/jp/bedrock/

## 当日ハンズオンの資料

当日のハンズオン資料は以下。

1. Claude準備
https://gist.github.com/tsukumonasu/964c9012166490b7094093872a912fc3

2. その他準備
https://gist.github.com/tsukumonasu/b42ef1469522a3fd0bf16a6fb2b4f69f

3. Bedrock
https://gist.github.com/tsukumonasu/96b7aa2f3167da16bd80c59687384516

4. Claudeスマホアプリ
https://gist.github.com/tsukumonasu/aec4d5a872c5da4ff9898cbc0fc85494

5. Stable Diffusionスマホアプリ
https://gist.github.com/tsukumonasu/92c203380179cb939f2b12dfdcec96cc

6. Kendra
https://gist.github.com/tsukumonasu/03bd51b9a03412566718442b9a9234c6

## PlayGround

### テキスト対話
対話型のモデルとチャットで遊んでみました。「おなかが減った」と話しかけると「おなかが減ったということは、お腹が空いたということですね」と有名なネットミーム風に返事をくれました。かなり体をいたわってくれます。
![Alt text](/images/articles/jawsug-nagoya-bedrock-handson/chat-sample1.png)
Metaモデルはなぜかうまく動かなくって少し残念でした。また別途トライしてみよう。
![Alt text](/images/articles/jawsug-nagoya-bedrock-handson/chat-sample2.png)

### 画像生成
画像生成も試してみます。Stable Diffusionですね。「style of japonese」に反応してくれてかわいい猫の絵が出力されました。
![Alt text](/images/articles/jawsug-nagoya-bedrock-handson/sd-cloud.png)

手元にあるStable diffusion Web UIでも画像出力してみました。クラウドのほうが処理能力が高いのか、かなり画像生成が速かったです。RTX 3050ti 程度だと足元にも及ばないということですね。生成するオプションがリッチであれば、クラウドだけで完結できるかなとおもったのですが、Stable diffusion Web UIのほうが細かな設定ができる様子。LoRAの指定やimage2imageなどのオプションは見当たらなかったです。
![Alt text](/images/articles/jawsug-nagoya-bedrock-handson/SD-Web.png)

## 未消化のハンズオン
時間切れでスマホアプリ作成を完了することができなかったのが少し残念だった。どこかで別途時間をとってやってみようかな。