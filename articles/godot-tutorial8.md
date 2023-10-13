---
title: "Godot でゲーム開発がしてみたい 8 (Creating your first script)"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---
Godot DocsのIntroductionを一通り見てきたので、Step by Stepの章に着手する。
この記事では以下のトピックからCreating your first scriptを学習する。これらのトピックは次の章であるYour first 2D gameの準備となる。これらのトピックで基礎がかなり身につくらしい。
- Nodes and Scenes
- Creating instances 
- Scripting languages
- Creating your first script ← いまここ
- Listening to player input
- Using signal

スクリプトがゲームに動きをもたらすので、この章はしっかりと理解したい。

https://docs.godotengine.org/en/stable/getting_started/step_by_step/scripting_first_script.html

## 最初のスクリプトを作る（Creating your first script）

この章では、GDScriptを使ってGodotのアイコンをぐるぐる回す最初のスクリプトをコーディングする。

### プロジェクト準備（Project setup）
まっさらな状態から始めるために、新しいプロジェクトを作成する。プロジェクト内にSprite2Dのシーンを作成する。
![Alt text](/images/articles/godot-tutorial8/Sprite2D.png)

Sprite2Dノードを表示するには、テクスチャが必要なので追加する。
![Alt text](/images/articles/godot-tutorial8/empty.png)

ファイルドックから適応したいアイコンを Drug & Drop して処理可能。
![Alt text](/images/articles/godot-tutorial8/IconAdded.png)

これでスクリプトを適用するシーンが作成できた。

### 新規スクリプトを作る（Creating a new script）
シーンドックからSprite2Dを選択してスクリプトをアタッチしていく。TemplateはObject:Emptyに設定する。言語はGDScriptに設定する。
![Alt text](/images/articles/godot-tutorial8/AttachScript.png)
![Alt text](/images/articles/godot-tutorial8/MakeFile.png)
  
全てのGDScriptファイルは暗黙的にクラスである。extendsキーワードはこのスクリプトが継承または拡張するクラスを定義する。今回はSprite2Dを継承しているので、その継承元であるNode2D,CanvasItem,Nodeといった継承クラスも含む。

### Hello, World!
スクリプトに以下のような記述を行う。__init(): の部分はコンストラクタを表現する。
```sprite_2d.gd
func _init():
	print("Hello, world!")
```
このSprite2Dをプロジェクトのメインシーンとして登録して実行すると以下のようになる。
![Alt text](/images/articles/godot-tutorial8/RunHelloWorld.png)


### 回転させる（Turning around）
ノードを移動、回転させるためのスクリプトを作成する。そのために、スクリプトに2つのメンバ変数を追加する。移動速度はピクセル毎秒、角速度はラジアン毎秒だ。スクリプトを次のようにする。
```sprite_2d.gd
extends Sprite2D

var speed = 400
var angular_speed = PI
func _init():
	print("Hello, world!")
```
メンバ変数はスクリプトの一番上、extends行の後、関数の前に置く（なんだかC言語みたいだ）。このスクリプトがアタッチされているすべてのノードインスタンスは、speedプロパティとangular_speedプロパティの独自のコピーを持つことになる。
補足：
Godotの角度はデフォルトではラジアン単位で動作するが、代わりに度単位で角度を計算したい場合は、組み込み関数とプロパティが利用できる。

アイコンを動かすには、ゲームループの中で毎フレーム、アイコンの位置と回転を更新する必要がある。これを実現するために、Nodeクラスの_process()仮想関数を使う（基本的にどのNodeもこの仮想関数は利用できるということだ）。Sprite2DのようにNodeクラスを継承したクラスでこの関数を定義すると、Godotは毎フレームこの関数を呼び出し、最後のフレームからの経過時間であるdeltaという引数を渡す。

この_process()という関数はかなり重要だ、シーンのコールバックに相当する。deltaと呼ばれる変数にも留意する必要がある。通常、ゲームはフレームレートを60fps（1秒に60コマ）のスピードで進めようとするが、ゲームの動作環境によっては低下する可能性がある。もしくはゲームのリアリティを上げようとした場合にレートが上がる可能性もある。つまり、_process()が呼び出される間隔は常に可変ということだ。移動距離など、時間に依存する更新を実施する場合に、このdeltaは必須となる。

deltaの単位が明示されていなかったので、下記のコードではダンプするようにしている。
（コメントは#で実施する。オフィシャルを見るとコメントアウトの場合は#とコードの間にスペースを入れるなと記述もあった。）

```sprite_2d.gd
extends Sprite2D

var speed = 400
var angular_speed = PI
# コンストラクタ
func _init():
	print("Hello, world!")
# コールバック
func _process(delta):
	print(delta)
	rotation += angular_speed * delta	

```

funcキーワードは新しい関数を定義できる。関数の名前と引数を括弧で囲んで書いてコロンで定義を終え、その後に続くインデントされたブロックが関数の内容や命令となる。Pythonの記述と同じだ。
（サラッと出てきているけど、PIも組み込みの予約語なのだろう。）

補足に記述があったが、気になっていたアンスコ始まりの関数はGodotのエンジンと通信するためにオーバーライドできる関数らしい。自前で定義する関数にはアンスコを先頭につけてはだめだ。

rotationはNode2Dから継承しているプロパティで、これをもとにオブジェクトの角度がきまる。

![Alt text](/images/articles/godot-tutorial8/Rotation.png)

ダンプしたdeltaの値が0.00833333333333になっている。おそらくこれはゲームが120fpsで動いているということだろう（1/0.00833333333333≒120）。そしてdeltaの単位は秒だ。
動作させているゲームのウィンドウをつかんで移動させると、以下のようにdeltaの値が変化した。
訳67.5fps ～　120fps強が出ている。平均では118fpsだ。特徴的なのは★を付けているところで150fpsを出そうとしている。おそらく、一定期間での平均を120fpsに近づけるように動作しているのだろう。
0.01481333333333
0.007915
0.009315
0.008437
0.00666666666667　★
0.00770233333333
0.008098
0.008138

### Moving forward
スクリプトを以下のように変更する。

```sprite_2d.gd
extends Sprite2D

var speed = 400
var angular_speed = PI
# コンストラクタ
func _init():
	print("Hello, world!")
# コールバック
func _process(delta):
	rotation += angular_speed * delta	
	var velocity = Vector2.UP.rotated(rotation) * speed
	position += velocity * delta
    print(position)
```

関数内で変数を切るとローカル変数になる。（スコープの仕様はPythonとおんなじ感じかな？）
サンプルコードの理解をすると、
- var velocity は速度を格納する２次元配列
- Vector2.UP はオブジェクト進行方向に対する単位ベクトル
- rotated(rotation) メソッドで、進行方向に対してrotation分だけ進行方向を変更したベクトルを得る
- speedを乗算することで、進行方向を変えた単位ベクトルを速度の大きさを持つベクトルに変える
- position変数はVector2の組み込みプロパティで、モノの位置を表現するので、これを更新する

Vector2.UPのように相対座標系での真上を簡単に取れるのがうれしいな。こういうところがエンジンを使う大きなメリットなんだろう。

さて、とりあえずこの章の目的はクリアすることができた。

![Alt text](/images/articles/godot-tutorial8/Icon-Rotation-_DEBUG_-2023-10-13-13-07-44.gif)

次はユーザ入力の受付だ。