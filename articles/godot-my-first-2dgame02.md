---
title: "Godot で最初の2Dゲームを作る 02"
emoji: "😺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: false
---
チュートリアルの総まとめとして2Dゲームを作成してく。第二回目。Playerのコーディングを行う。
https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html

# Coding the player
この章では、Playerの動きとアニメーションを追加し、衝突を検出するように設定する。
そのためには、組み込みのノードからは得られない機能を追加する必要があるので、スクリプトを追加する。

```player.gd
extends Area2D

@export var speed = 400 # How fast the player will move (pixels/sec).
var screen_size # Size of the game window.
# Called when the node enters the scene tree for the first time.
func _ready():
	pass # Replace with function body.


# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta):
	pass
```

export キーワードを指定するとInspectorで値を設定できるようになる。ノードの組み込みプロパティのように調整をしたい場合に便利な宣言方法だ。インスペクターから値を編集するとスクリプト側の記述が上書きされてしまうので注意。

スクリプトには以下の要素を追加する。1,2に関してはこれまでのチュートリアルで経験した内容だ。
1. 入力をチェックする
2. 指定された方向に動く
3. 適切なアニメーションを再生する


#### 入力をチェックする
このゲームでは、4方向の入力をチェックする。入力アクションはプロジェクト設定の Input Map で定義する。ここで、カスタムイベントを定義して、異なるキー、マウスイベント、その他の入力を割り当てることができる。このゲームでは、矢印キーを4方向にマッピングする。
まずmove_rightという文字列に対して右方向キーを割り当てる。

![Alt text](/images/articles/godot-my-first-2dgame02/set-right.png)

最終的に以下のようなマッピングとなればよい。
![Alt text](/images/articles/godot-my-first-2dgame02/mapping.png)

#### 補足：
各入力アクションには1つのキーしかマッピングしていないが、複数のキー、ジョイスティックボタン、マウスボタンを同じ入力アクションにマッピングすることができる。

キーが押されているかどうかは、Input.is_action_pressed()を使って検出することができ、押されていればtrue、押されていなければfalseを返す。
（Inputは毎フレームユーザの入力を取得したい場合に利用する機構だ）

まず、velocity（速度ベクトルだ）を(0, 0)に設定する。デフォルトでは、プレーヤーは動かない仕様だ。まずユーザの入力をチェックし、足し引きして合計の方向を求める。例えば、右と下を同時に押した場合、速度ベクトルは(1, 1)となる。この場合、水平方向と垂直方向の動きを足しているので、プレイヤーは水平方向にだけ動くよりも斜め方向に速く動くことになる。
これを解決するのがnormarizedメソッドだ。このメソッドを呼び出せばベクトルの長さを1にすることができる。その状態でspeedパラメータを掛けわせることで所望のスピードを算出可能となる。つまり、斜めに早く動くことがなくなる。
AnimatedSprite2Dのplay()やstop()を呼び出せるように、プレイヤーが動いているかどうかもチェックする。（どうやらplay(),stop() メソッドで動きの制御が可能らしい。）

#### 補足：
$の記号はget_nodeメソッドの短縮形として機能する。つまり
$AnimatedSprite2D.play() は get_node("AnimatedSprite2D").play() と等しい。
GDScriptでは、$は現在のノードからの相対パスでノードを返し、ノードが見つからない場合はnullを返す。AnimatedSprite2Dは現在のノードの子なので、$AnimatedSprite2Dを使うことができる。
(平行なノードにはどのようにアクセスするのだ？orそういった設計はしないほうがいいのか？)

これで移動方向が決まったので、プレーヤーの位置を更新できる。また、clamp()を使って画面から離れないようにすることもできます。クランプとは、値をある範囲に制限することだ。process関数の一番下に以下を追加します。

```
# delta is size of flame which is given as a _process argument
position += velocity * delta
position = position.clamp(Vector2.ZERO, screen_size)
```

## アニメーションの切り替え

ここまででPlayerが動作するようになったので、AnimatedSprite2Dが再生するアニメーションを方向によって変える必要がある。ここでは、プレイヤーが右に向かって歩く「walk」アニメーションを用意した。このアニメーションは、flip_hプロパティを使って水平方向に反転させなければならない。また、"up" アニメーションがありますが、これは flip_v を使って垂直方向に反転させ、下方向に移動させます。以下のコードを_process()関数の最後に配置する。

```
if velocity.x != 0:
	$AnimatedSprite2D.animation = "walk"
	$AnimatedSprite2D.flip_v = false
	# See the note below about boolean assignment.
	$AnimatedSprite2D.flip_h = velocity.x < 0
elif velocity.y != 0:
	$AnimatedSprite2D.animation = "up"
	$AnimatedSprite2D.flip_v = velocity.y > 0
```
animationプロパティにアニメーションの名前を指定することで切り替えが可能なようだ。
また、flip_v,flip_hに値を指定することで左右、上下の反転が可能なようだ。

コードの以下の部分は少し特徴的で、この式は比較式なのでtrue/falseを返す。
```
velocity.x < 0
```

### 衝突の準備

Playerに敵との衝突を検知させるために、カスタムシグナルを定義する。ここでは引数なしのシグナルでhitと宣言する。

```
signal hit
```

これは「hit」というカスタム・シグナルを定義したもので、プレーヤーが敵と衝突したときに排出される（emitメソッドを利用する）。衝突の検出にはArea2Dを使う。Playerノードを選択し、Inspectorタブの隣にある Nodeタブをクリックすると、プレーヤーが出せるシグナルのリストが表示されます：

![Alt text](/images/articles/godot-my-first-2dgame02/body-entered-signal.png)

これをPlayerノードに接続し、コールバックを以下のように記述する。
```
func _on_body_entered(body):
	hide() # Player disappears after being hit.
	hit.emit()
	# Must be deferred as we can't change physics properties on a physics callback.
	$CollisionShape2D.set_deferred("disabled", true)
```

敵がplayerにぶつかるたびにシグナルが排出されるので。playerの衝突を無効にして、hitシグナルが何度も発動しないようにする必要がある。
（上記まではbody_enteredと呼ばれるnotifyの中でhitというカスタムシグナルを発生させているという理解だ。）

最後に、新しいゲームを始めるときにplayerをリセットするために呼び出せる関数を追加することだ。

```
func start(pos):
	position = pos
	show()
	$CollisionShape2D.disabled = false
```

playerに関しては上記で終わりだ。

最終的にスクリプトは以下のようになる

```player.gd
extends Area2D

signal hit

@export var speed = 400 # How fast the player will move (pixels/sec).
var screen_size # Size of the game window.
# Called when the node enters the scene tree for the first time.
func _ready():
	screen_size = get_viewport_rect().size
	hide()
	
# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta):
	var velocity = Vector2.ZERO # The player's movement vector.
	if Input.is_action_pressed("move_right"):
		velocity.x += 1
	if Input.is_action_pressed("move_left"):
		velocity.x -= 1
	if Input.is_action_pressed("move_down"):
		velocity.y += 1
	if Input.is_action_pressed("move_up"):
		velocity.y -= 1

	if velocity.length() > 0:
		velocity = velocity.normalized() * speed
		$AnimatedSprite2D.play()
	else:
		$AnimatedSprite2D.stop()
		
	position += velocity * delta
	position = position.clamp(Vector2.ZERO, screen_size)

	if velocity.x != 0:
		$AnimatedSprite2D.animation = "walk"
		$AnimatedSprite2D.flip_v = false
		# See the note below about boolean assignment.
		$AnimatedSprite2D.flip_h = velocity.x < 0
	elif velocity.y != 0:
		$AnimatedSprite2D.animation = "up"
		$AnimatedSprite2D.flip_v = velocity.y > 0


func _on_body_entered(body):
	hide() # Player disappears after being hit.
	hit.emit()
	# Must be deferred as we can't change physics properties on a physics callback.
	$CollisionShape2D.set_deferred("disabled", true)
	
	
func start(pos):
	position = pos
	show()
	$CollisionShape2D.disabled = false

```