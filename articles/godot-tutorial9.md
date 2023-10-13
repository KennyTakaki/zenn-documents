---
title: "Godot でゲーム開発がしてみたい 9 (Listening to player input)"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---
Godot DocsのIntroductionを一通り見てきたので、Step by Stepの章に着手する。
この記事では以下のトピックからListening to player inputtを学習する。これらのトピックは次の章であるYour first 2D gameの準備となる。これらのトピックで基礎がかなり身につくらしい。
- Nodes and Scenes
- Creating instances 
- Scripting languages
- Creating your first script 
- Listening to player input ← いまここ
- Using signal

https://docs.godotengine.org/en/stable/getting_started/step_by_step/scripting_player_input.html


### プレイヤーの入力を待ち受ける（Listening to player input）
sprite_2d.gdのコードを修正しながらプレイヤーの入力を受け付ける。

Godotには、プレイヤーの入力を処理するための2つの主要なツールがある

- 組み込みの入力コールバック：主に_unhandled_input()関数だ
_unhandled_input()は、_process()と同じく、プレイヤーがキーを押すたびにGodotが呼び出す組み込みの仮想関数だ（アンスコ始まり）。これは、Spaceキーを押してジャンプするような、毎フレーム起こらないイベントに反応するために使うツールだ。
- Inputシングルトンシングルト：グローバルにアクセス可能なオブジェクト。Godotはスクリプトの中でいくつかのオブジェクトへのアクセスを提供している。これは、毎フレーム入力をチェックするのに適したツール。
（毎フレームユーザの入力を受け付けるってどんなだ？移動系の矢印の長押しとか？）

今回のケースではプレーヤーが毎フレーム回転したいのか移動したいのかを知る必要があるので、ここではInputシングルトンを使う。

新しい変数directionをユーザからの入力保存に使いrotetionに反映する。_process()関数で、rotation += angular_speed * deltaの行を以下のコードに置き換える。全体としては以下のようになる。

```sprite_2d.gd
extends Sprite2D

var speed = 400
var angular_speed = PI
# コンストラクタ
func _init():
	print("Hello, world!")
# コールバック
func _process(delta):
	var direction = 0
	if Input.is_action_pressed("ui_left"):
		direction = -1
	if Input.is_action_pressed("ui_right"):
		direction = 1
	rotation += angular_speed * direction * delta	
	var velocity = Vector2.UP.rotated(rotation) * speed
	position += velocity * delta
	print(position)

```

このフレームでキーが押されたかどうかをチェックするために、Input.is_action_pressed()を呼び出している。このメソッドは入力アクションを表すテキスト文字列を受け取り、アクションが押された場合はtrueを、そうでない場合はfalseを返す。
（このロジックだと、右左を同時押ししていたら右に曲がるはずだ）

"ui_left "と "ui_right "はGodotプロジェクトで定義済みの値だ。それぞれ、プレイヤーがキーボードの左と右の矢印、またはゲームパッドのDパッドの左と右を押したときにトリガーされる。
このあたりは決まり事なので慣れるしかないな。

補足：「プロジェクト」→「プロジェクト設定」で「入力マップ」タブをクリックすると、プロジェクトの入力アクションを確認・編集できるらしい。

### 上を入力した際に動かす（Moving when pressing "up"）

キー入力で速度を発生させるように修正する。コードとしては以下のような感じだ。
上方向に割り当てられた文字列は"ui_up"らしい。

```sprite_2d.gd
extends Sprite2D

var speed = Vector2.ZERO
var angular_speed = PI
# コンストラクタ
func _init():
	print("Hello, world!")
# コールバック
func _process(delta):
	var direction = 0
	if Input.is_action_pressed("ui_left"):
		direction = -1
	if Input.is_action_pressed("ui_right"):
		direction = 1
	rotation += angular_speed * direction * delta
	
	var velocity = Vector2.ZERO
	if Input.is_action_pressed("ui_up"):
		velocity = Vector2.UP.rotated(rotation) * speed
	
	position += velocity * delta
	
	print(position)
```

動作は以下のような感じだ。
![Alt text](/images/articles/godot-tutorial9/Icon-Rotation-_DEBUG_-2023-10-13-15-02-01.gif)

### まとめ
スクリプトがシーンを継承するというのもなんとなくわかった。結局のところGodotが提供する組み込みノードとしてのふるまいを拡張するための機構がスクリプトだということだろう。作りたいモノに必要そうな動作を想像して、それを実現するための機構がないか探すのがよさそうだ。
自前のクラスとエンジンを接続するにはアンスコ始まりのメソッドか、シングルトン（グローバル変数のような存在）でアクセスするらしい。後者は基本的に_process()の中で
扱うことになるはずだ。
  
次回はSignalの扱いを学ぶことになる。