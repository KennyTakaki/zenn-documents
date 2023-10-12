---
title: "Godot でゲーム開発がしてみたい 3"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: false
---

前回
https://zenn.dev/frommiddle1/articles/godot-tutorial2

前々回
https://zenn.dev/articles/godot-tutorial/edit

## Learning new features

今回はここの章を読んでいく。  
https://docs.godotengine.org/en/stable/getting_started/introduction/learning_new_features.html

Godotは機能豊富なゲームエンジンで、学ぶべきことがたくさんある。この章では、オンラインマニュアルや組み込みのコードリファレンスを利用したり、オンラインコミュニティに参加して新しい機能やテクニックを学んだりする方法が説明される。

### Making the most of this manual

何か新しい紺瀬tプとや機能を学ぶときには、オンラインマニュアルの関連ページを調べたり、検索バーでキーワードを調べることになる。
https://docs.godotengine.org/en/stable/getting_started/introduction/learning_new_features.html

Godotが提供するクラスリファレンスにはエディタからもアクセスすることができる。ヘルプ-> ヘルプを検索 or F1(Windows)でオープンすることができる。クラスリファレンスはオンラインでもオフラインでも利用が可能。
![Alt text](/images/articles/godot-tutorial3/editor-manual.png)
![Alt text](/images/articles/godot-tutorial3/search-help.png)
  
クラスリファレンスでは以下を知ることができる
- クラスが継承階層のどこに存在するか
- クラスの役割と利用例の概要
- クラスのプロパティ、メソッド、シグナル、列挙型、定数についての説明
- クラスをより知るためのマニュアル・ページへのリンク

### Learning to think like a programmer¶
プログラムの素養を持つことを推奨していて以下のいずれかを選択してやるといいと記述がある。
- Harvard university の CS50
https://cs50.harvard.edu/x/  
内容としては以下のようなコースらしい。行き詰ったら、ここに帰ってきてもいいかも。
> このコースは、今日の新しい言語の基礎となっているC言語という、伝統的でありながら普遍的な言語から始まり、関数、変数、条件分岐、ループなどについて学ぶだけでなく、コンピュータそのものがどのように動作しているのか、メモリなどについても学びます。コースの最後には、データベースにデータを保存するSQL、ウェブやモバイルアプリを作成するHTML、CSS、JavaScriptを紹介します。コースは最終プロジェクトで締めくくられます。
- 退屈なことはPythonにやらせよう（書籍）
https://automatetheboringstuff.com/

### Learning with the community
行き詰った場合には以下を利用することができる。
- Godotのコミュニティ
Redditのような掲示板からDiscord、TwitterのようなSNSが紹介されている。
https://godotengine.org/community/
- QA サイト
https://ask.godotengine.org/
  
#### 質問をする際の作法
- 目的を説明する
- エラーがある場合は正確なエラーメッセージを共有する
- コードが含まれている場合はコードサンプルを共有する
- シーンドック（Scene dock） のスクリーンショットを共有する（スマホでとるな、観にくい）
（この項目からもわかるようにSceneが人間がゲームのコンポーネントを理解するうえで重要な単位なんだと思う）
- ゲームの動作を動画にして共有する
- 安定版のGodotを使っていない場合は利用しているバージョンを明記する

### Community tutorial
有志が提供しているチュートリアルがあるそうだ。入門を超えて、RPGなどのチュートリアルがほしい場合は以下のページを参照するといい。
https://docs.godotengine.org/en/stable/community/tutorials.html#doc-community-tutorials
