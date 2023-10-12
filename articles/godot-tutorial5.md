---
title: "Godot でゲーム開発がしてみたい 5（Nodes and Scenes）"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game, Godot]
published: true
---
これまではGodot DocsのIntroductionを一通り見てきたので、Step by Stepの章に着手する。
この記事では以下のトピックからNodes and Scenesを学習する。これらのトピックは次の章であるYour first 2D gameの準備となる。これらのトピックで基礎がかなり身につくらしい。
- Nodes and Scenes
- Creating instances
- Scripting languages
- Creating your first script
- Listening to player input
- Using signals

https://docs.godotengine.org/en/stable/getting_started/step_by_step/index.html

## ノードとシーン（Nodes and Scenes）
Godotで作るゲームはシーンで構成されたツリーであり、各シーンはノードのツリーである。
ここでは、これらの構成要素を深堀する。

### ノード（Nodes）
ノードはゲームの基本的な構成要素です。レシピの材料のようなものだ。画像を表示したり、音楽を再生したり、カメラを表現したり、様々な種類がある。

全てのノードは以下の特徴を持つ
- 名前
- 編集可能なプロパティ
- ノードは毎フレーム更新されるコールバックを受け取る
- 新しいプロパティや関数で拡張することができる
- ノードは子ノードとして別のノードに追加できる

最後の特徴は重要だ。ノードは結合することでツリーを形成します。そしてこの特徴はプロジェクトを整理するための強力な機能です。異なるノードは異なる機能を持つので、それらを組み合わせることで、より複雑な動作が生まれる。前に見たように、Characterという名前のCharacterBody2DノードとSprite2DノードとCamera2DノードとCollisionShape2Dノードを使用して、カメラが追うプレイアブルキャラクターを作成できます。

### シーン（Scenes）
上述のキャラクターのように、ノードでツリーを構成するとき、私たちはこの構成をシーンと呼ぶ。一度保存されると、シーンはエディターで新しいノードタイプのように機能し、既存のノードの子として追加することができます。その場合、シーンのインスタンスは内部が隠された1つのノードとして表示される。
（ノードでシーンをつくって、ツリー内では作成したシーンをノードとして扱える。シーンはノードを抽象化するための仕組みと理解したらいいかな？）

シーンを使うとゲームのコードを好きなように構成できる。ノードを組み合わせることで、走ったりジャンプしたりするゲームキャラクター、ライフバー、操作可能なチェストなど、カスタムで複雑なノードタイプを作成できる。

Godotエディターは基本的にシーンエディターで、2Dや3Dのシーンやユーザーインターフェイスを編集するためのツールを多く備えている。Godotプロジェクトには、必要なだけシーンを含めることができる。エンジンが必須とするのは、アプリケーションのメインシーンとして利用する1つのシーンだけだ。これは、あなたやプレイヤーがゲームを実行するときに、Godotが最初にロードするシーンだ。
（別の言い方をすれば、メインシーンだけ作成すればGodotでゲームを作成したともいえる。エントリポイント、Main関数のようなものなのだろう。）

ノードのように動作するだけでなく、シーンには次のような特徴がある：
- 上述のキャラクターのように一つのルートノードを持つ（CharacterBody2Dの継承関係の大元を指していると思われる）
- ローカルドライブに保存して、後でロード可能（ノードはできないのか、というかノードはGodotから見た時には組み込みなのか）
- シーンのインスタンスは、好きなだけ作成できる。上述のキャラクターシーンから作成したキャラクターを、5人、10人とゲームに登場させることができる。

シーンはあくまでクラスのようなものらしい。シーンからインスタンスを作成する、という行程があるみたいだ。

### 最初のシーンを作ってみる（Creating your first scene）

最初にノード1つのシーンを1つ作ってみる。

空のシーンではシーンドックに、ルートノードを追加するためのオプションが表示されます。
- 2D Scene : Node2Dノードを追加
- 3D Scene : Node3Dノードを追加
- User Interface : Controlノードを追加
- Other Node : 任意のノードをルートノードとして選択可能
空のシーンでは、Other Node は、Sceneドックの左上にある + ボタンを押すのと同じで、通常、現在選択されているノードの子として新しいノードを追加する。
![Alt text](/images/articles/godot-tutorial5/root-node.png)
  
Add Child Nodeボタンか、Other Node を選択してルートノードを作成する。シーンに対してLabelノードを追加して、画面にテキストを書くことが目的だ。

検索窓にLabelと入れると、所望のノードが見つかる。  
![Alt text](/images/articles/godot-tutorial5/search-label-node.png)

これでLabelノードが追加された。
  
シーンの最初のノードを追加すると、いろいろなことが起こる。Labelは2Dノードタイプなので、シーンは2Dワークスペースに変わる。Label は、ビューポートの左上に、選択された状態で表示さる。ノードは左側の Scene ドックに表示され、ノードのプロパティは右側の Inspector ドックに表示される。
![Alt text](/images/articles/godot-tutorial5/LabelNode.png)

### ノードの属性を編集する（Changing a node's properties）

次のステップでは、Labelの Textプロパティを変更する。Hello World に変更してみる。

ビューポートの右にあるインスペクタ・ドックに向かいます。Textプロパティの下のフィールドをクリックして、Hello Worldと入力する。テキストを入力すると、ビューポートにテキストが描画される。

![Alt text](/images/articles/godot-tutorial5/Text-HelloWorld.png)


ツールバーの移動モードを選択していると、Labelノードをビューポート内で移動させることができる。
![Alt text](/images/articles/godot-tutorial5/Move-Text.png)


### シーンを実行する（Runnning the scene）
画面右上のシーン再生ボタンを押すか、F6（macOSではCmd + R）を押してシーンを実行することができる。

![Alt text](/images/articles/godot-tutorial5/Runnnig-Scene.png)

シーンを保存するポップアップが表示されるので、label.tscnとして保存する。ウィンドウ上部の res:// パスは、プロジェクトのルートディレクトリを表し、リソースパスを意味する。
![Alt text](/images/articles/godot-tutorial5/Save-Scene.png)
  
無事にシーンを起動することができた。ウィンドウを閉じるか、F8キー（macOSではCmd + .）を押して実行中のシーンを終了することができる。
![Alt text](/images/articles/godot-tutorial5/HelloWorld.png)

### メインシーンを設定する（Setting the main scene）
テストシーンを実行するには、Play Sceneボタンを使う。その隣にある別のボタンで、プロジェクトのメインシーンを設定して実行できる。F5（macOSではCmd + B）を押して実行できる。

![Alt text](/images/articles/godot-tutorial5/SelectMain.png)
  
![Alt text](/images/articles/godot-tutorial5/Select-Label-As-Main.png)
  
エディターはメインシーンのパスをプロジェクトのディレクトリにあるproject.godotファイルに保存する。このテキストファイルを直接編集してプロジェクト設定を変更することもできるが、Project -> Project Settings ウィンドウを使って変更することもできる。

### 終わりに
おそらく最小要件のゲームを作ることができた。起動して、HelloWorldと表示するだけのゲームだ。構成要素は単一のLabelノードからなるシーンだ。