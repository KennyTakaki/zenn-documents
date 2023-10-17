---
title: "Godot で最初の2Dゲームを作る 04"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---
チュートリアルの総まとめとして2Dゲームを作成してく。第四回目。
https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html

ここまででプレーヤーと敵の準備ができたので、次のパートでは、新しいシーンでそれらをひとつにまとめる。敵がゲームボードの周りでランダムにスポーンして前進するようにし、プロジェクトをプレイ可能なゲームに変える。

## The main game scene

新しいシーンを作成し、Mainという名前のNodeを追加する。Node2DではなくNodeを使う。Node2Dではない理由は、このノードがゲームロジックを扱うコンテナになるからだ。2Dの機能自体はMainに必要ない。そのため一番基底に近いノードでよい。

Instanceボタン（鎖のリンクアイコン）をクリックし、保存したplayer.dscnを選択する。

![Alt text](/images/articles/godot-my-first-2dgame04/addplayer.png)


次に、以下のノードをMainの子として追加し、それぞれ括弧内の名前を付ける

- Timer（MobTimer） : モブがスポーンする頻度をコントロールする
- Timer (ScoreTimer) : 1秒ごとにスコアを増加させる
- Timer (StartTimer) : 開始前にディレイを与える
- Marker2D (StartPosition) : プレイヤーのスタート位置を示す

各TimerノードのWait Timeプロパティを以下のように設定する

- MobTimer: 0.5
- ScoreTimer: 1
- StartTimer: 2

さらに、StartTimerのOne Shotプロパティを "On "に設定し、StartPositionノードのPositionを(240, 450)に設定する。

![Alt text](/images/articles/godot-my-first-2dgame04/tochu.png)

## Spawning mobs
Mainノードは新しいモブをスポーンさせる。モブは画面の端のランダムな場所に現れるようにする。Mainノードの子としてMobPathという名前のPath2Dノードを追加する。Path2Dを選択すると、エディタの上部（ツールバー）に新しいボタンがいくつか表示される。

真ん中「点を空きスペースに追加」を選択し、表示されているコーナーにポイントを追加するためにクリックしてパスを描く。この時、時計回りに描かないと、モブが内側ではなく外側を向いてスポーンしてしまう。点4を画像に配置したら、曲線を閉じるボタンをクリックすれば完成する。

![Alt text](/images/articles/godot-my-first-2dgame04/curve.png)


これでパスが定義されたので、PathFollow2DノードをMobPathの子として追加し、名前をMobSpawnLocationとする。このノードは自動的に回転し、パスに沿って動くので、パスに沿ってランダムな位置と方向を選択するために使うことができる。

![Alt text](/images/articles/godot-my-first-2dgame04/mid-scene.png)


## Main script

ゲームロジックを追加するためにMainにスクリプトを追加する。スクリプトの先頭で、@export var mob_scene：PackedSceneを使って、インスタンス化したいモブシーンを選択できるようにする。

```
extends Node

@export var mob_scene: PackedScene
var score
```
さらりと出てきたけれど。変数の後ろに : を付けてのは型を付与している。下記のBasic useを参照するといい。
https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_exports.html

おそらく、PackedSceneは何かしらのシーンを型として持つという意味だ。ドキュメントを読んでもそんな雰囲気がする。正しくは理解できていない。そのため、この後の手順で具体的なmobのシーンを割り当てることになる。
（GDScriptを触っていて思うのは、通常のプログラミングのようなモジュールシステムが存在しておらず、エディタの機能で依存関係を表現するので少しやりづらさを感じている。慣れが必要だ。）

これでMainノードのインスペクターからからMob Scene プロパティにアクセスできる。（Script Variablesに所属している。）

このプロパティの値は、2つの方法で割り当てることができる

- FileSystem ドックからmob.tscnをドラッグし、Mob Sceneプロパティにドロップする。
- emptyの横の下矢印をクリックし、Load を選択する。mob.tscnを選択する。 

![Alt text](/images/articles/godot-my-first-2dgame04/MobScene.png)

次にPlayerのhitシグナルをMainシーンに接続し、コールバックの関数名をgame_overとして作成する。game_over内では子ノードのScoreTimerとMobTimerを停止させる。
また、新しくゲームを始めるための関数 new_gameも作成し、Playerに初期位置を与えてStartTimerを起動させる。

```
func game_over():
	$ScoreTimer.stop()
	$MobTimer.stop()

func new_game():
	score = 0
	$Player.start($StartPosition.position)
	$StartTimer.start()
```

ここで、各タイマーノード（StartTimer、ScoreTimer、MobTimer）のtimeout()シグナルをメインスクリプトに接続する。StartTimerは他の2つのタイマーをスタートさせ、ScoreTimerはスコアを1増やす。


on_mob_timer_timeout()では、モブのインスタンスを作成し、Path2Dに沿ってランダムな開始位置を選び、モブを動かすように設定する。PathFollow2Dノードはパスに沿って自動的に回転するので、それを使ってモブの方向と位置を選択する。モブをスポーンするとき、各モブの移動速度は150.0から250.0の間のランダムな値を選ぶことにする。
新しいインスタンスは、add_child()を使ってシーンに追加しなければならない。インスタンスを作成するのはinstantizte()だ。

```
func _on_mob_timer_timeout():
	# Create a new instance of the Mob scene.
	var mob = mob_scene.instantiate()

	# Choose a random location on Path2D.
	var mob_spawn_location = get_node("MobPath/MobSpawnLocation")
	mob_spawn_location.progress_ratio = randf()

	# Set the mob's direction perpendicular to the path direction.
	var direction = mob_spawn_location.rotation + PI / 2

	# Set the mob's position to a random location.
	mob.position = mob_spawn_location.position

	# Add some randomness to the direction.
	direction += randf_range(-PI / 4, PI / 4)
	mob.rotation = direction

	# Choose the velocity for the mob.
	var velocity = Vector2(randf_range(150.0, 250.0), 0.0)
	mob.linear_velocity = velocity.rotated(direction)

	# Spawn the mob by adding it to the Main scene.
	add_child(mob)
```

Let's test the scene to make sure everything is working. Add this new_game call to _ready():

```
func _ready():
	new_game()
```

ここまででひとまずゲームの大枠ができているはずだ。
![Alt text](/images/articles/godot-my-first-2dgame04/game.png)

今回はここまで。