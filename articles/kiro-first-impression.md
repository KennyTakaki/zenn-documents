---
title: "KiroでAmazon風にWorkingBackwardsを意識してtask定義してみた"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws","kiro"]
published: false
---

# 概要
やっとkiro のWaiting Listから解放された。[Code with Kiro Hackathon](https://kiro.devpost.com/?trk=dccd318a-a012-40c6-bffb-bd0a6216646d&sc_channel=el)には登録できているので、作品投稿を念頭に置きながらKiroを触っていこうと思い、まずはtask定義までを実施してみた。

# アプローチ
Kiroの特徴はSpec Codingを採用していることで、従来のvibe codingとは一線を画す。Spec Cotindを簡単に説明すると、ユーザのやりたいことをkiroにインプットして、要件（requirements.md）を生成し、設計（design.md）に落として、実装手順(tasks.md)の工程を作成することでProductionを意識した実装ができるというものだ。この特徴を聞いたときに、直感的にAmazonのWorking Backwardsとの相性がよさそうだと感じた。  
他にも要件を生成する際にKiroに与えるインプットの在り方として、方法論的には[USDM](chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/https://www.juse.or.jp/sqip/workshop/report/attachs/2013/sqip6-d-1.pdf)などが考えられるが、今回は親和性のよさそうなAmazon風の方法を採用することにした。

## ５つの質問
さて、Working Backwardsの中では以下5つの質問に対する答えを考えることから始まる。

1. Who is customer, and what insights do we have about them?
2. What is the prevailing customer problem or opportunity?
3. What is the solution and the most important customer benefit?
4. How do we describe the solution and the experience to customers?
5. How do we test the solution with customers and measure success?

和訳：
1. 顧客とは誰であり、彼らについてどのような洞察を得ているでしょうか？
2. 顧客が直面する主な問題や機会は何でしょうか？
3. 解決策とは何か、そして最も重要な顧客の利益は何でしょうか？
4. 解決策と顧客体験をどのように顧客に説明しますか？
5. 顧客と解決策をテストし、成功を測定する方法はどのようなものでしょうか？

これを練って、プレスリリース・FAQを作成してからプロダクト開発を行う、というのがWorking Backwordsのやり方だ。「顧客」という言葉がすべての質問に出てくるのが、なんともAmazonらしいと思う。今回はプレスリリースやFAQまでは整備せずに、上記の質問に対する答えを用意することで要件定義に対するインプットを作成してみた。

## ５つの質問（サンプル）
私は株式投資にも興味があって、自分用におすすめの銘柄や、その売買タイミングまで提示してくれるシステムがあったらいいなと考えて私と似た属性の人を想像しながら質問に回答してみた。この文書をもとに、kiroに要件定義を行ってもらってみた。
#### 和文：
https://github.com/KennyTakaki/tikkiro-prfaq/blob/main/five-questions.jp.md

#### 英文： 
https://github.com/KennyTakaki/tikkiro-prfaq/blob/main/five-questions.md


# 出力結果 
## 要件定義（requirements）
要件定義の出力は以下のようになった。
https://github.com/KennyTakaki/kiro-dev01/blob/main/.kiro/specs/intelligent-stock-recommendation/requirements.md


出力では、機能要件としては以下６つの要件が定義されて、５つの質問で記述した回答の意図をしっかり反映してくれていると思う。
- 要件1：データ駆動型株式推奨機能
- 要件2：将来の価格予測モデリング機能
- 要件3：証拠に基づく投資理由機能
- 要件4：実際の取引検証とパフォーマンス追跡機能
- 要件5：パーソナライズド投資プロファイル管理機能
- 要件6：包括的なデータ統合と独自分析機能

## 設計（design）
これをもとに設計を作成してもらった。

最終的な出力が[こちらで](https://github.com/KennyTakaki/kiro-dev01/blob/main/.kiro/specs/intelligent-stock-recommendation/design.md)、ArchitectureのHigh-Level Architectureはほぼ私が想像していたような構成を射抜いてくれている。
![alt text](/images/articles/kiro-first-impression/highlevel-architecture.png)
ただし、初期出力では、External APIsのコール時に利用するシークレットを格納する管理部（Secrets Mangement）や、それらのAPIをコールするイベントに関する管理部がなく、それらを追加する旨の依頼をして設計を修正してもらった。

以下のような指示を一回出すだけで、想定通りの修正をしてくれたと考えている。（料金に関するいちゃもんを別途つけて、デザインにコストの章を付けてもらったが、どの程度正しいかは未検証）

英：
Regarding the high-level architecture, please add a data collection event control layer within the data processing layer. It is assumed that services such as Amazon EventBridge and AWS Step Functions will be used.
Please add a layer for handling secrets in external API integrations. It is assumed that AWS Systems Manager Parameter Store or AWS Secrets Manager will be used.
Regarding the technology stack, please create it with TypeScript as the default wherever possible. Align the language with the frontend and manage it as a monorepo.
For other technologies, the basic policy is to build the system using a serverless architecture. For example, cold starts in AWS Lambda are acceptable. Even if the system’s response time is somewhat slower, the goal is to choose technologies that minimize system maintenance costs.


和：
ハイレベルアーキテクチャに関して、データ処理層におけるデータ収集のイベントコントロールの層を追加してください。
Amazon EventBridgeやAWS StepFunctionsなどの利用が想定されます。 外部API連携におけるシークレットを扱う層を追加してください。SystemsManagerのParameter StoreやSecrets Managerの利用を想定しています。
技術スタックに関して、可能な限りTypeScriptを前提として作成してください。フロントエンドと同じ言語に寄せて、モノレポとして管理します。
その他の技術に関して、基本的にはサーバレスでシステム構築したいです。例えばLambdaのコールドスタートなどを許容します。システムのレスポンスが多少悪くてもよいので、
システムの保持コストが最小化されるような技術選定を方針とします。


## 実装手順(Task)
上記のDesignをもとにTaskを生成してもらった。Taskを作成してみて驚いたが、要件に対するトレーサビリティが取れていた。Kiroにこれをグラフで可視化して、と頼むとクリティカルパス付で図示してくれた。とても良い機能だし、これをベースに実装していくことで、構成管理が非常に楽になると感じた。

https://github.com/KennyTakaki/kiro-dev01/blob/main/.kiro/specs/intelligent-stock-recommendation/tasks.md

# まとめ
WorkingBackwardsの考え方をもとにしてKiroに指示を与えてtaskまでを生成してみた。Design, Taskともに一定品質のものが得られたと考えている。
実装に関しては、呼び出すThird PartyのAPIの候補の仕様に関わる問題と、システムの保持に関する料金が発生するのでいったんペンディングする。
ある程度時間が取れるときにTask以降を操作してみようと思う。