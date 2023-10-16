---
title: "Godot で最初の2Dゲームを作る 03"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---
チュートリアルの総まとめとして2Dゲームを作成してく。第三回目。Enemyの作成を行う。
https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html


# Creating the enemy
この章では敵モブを作成する。モブは画面の橋でランダムに生まれ、ランダムな方向を選び、一直線に進む。モブシーンを作成し、インスタンス化することで、ゲーム内でドd区立したモブをいくつでも作成できる。（つまりシーンをクラスとして利用するということだ）


## Node setup

以下のような構造で新規シーンを作成する。
RigidBody2D (named Mob)
　|_AnimatedSprite2D
　|_CollisionShape2D
　|_VisibleOnScreenNotifier2D

Mobノードを選択し、インスペクタのRigidBody2DセクションのGravity Scaleプロパティを0に設定する。

RigidBody2Dセクションのすぐ下のCollisionObject2Dセクションで、Collisionグループを展開し、Maskプロパティの中の1のチェックを外す。これでモブ同士が衝突しないようになる。
![Alt text](/images/articles/godot-my-first-2dgame03/mob-setting.png)

AnimatedSprite2Dをセットアップする。今回は、飛ぶ、泳ぐ、歩くの3つのアニメーションが存在している。artフォルダには、それぞれのアニメーションの画像が2枚ずつ入っている。

3つのアニメーションのAnimation Speedプロパティに対して3を設定する。

プレイヤーの画像と同様に、モブの画像も縮小する。AnimatedSprite2DのScaleプロパティを(0.75, 0.75)に設定する。

プレイヤーシーンと同様に、衝突用にCapsuleShape2Dを追加します。Shapeを画像に合わせるには、Rotation Degreesプロパティを90に設定する必要があります（Inspectorの「Transform」の下）。

## Enemy script
Mobにスクリプトを追加して_readyに以下を記述する.

```
func _ready():
	var mob_types = $AnimatedSprite2D.sprite_frames.get_animation_names()
	$AnimatedSprite2D.play(mob_types[randi() % mob_types.size()])
```

まず、AnimatedSprite2Dのframesプロパティからアニメーション名のリストを取得している。これは、3つのアニメーション名["walk", "swim", "fly"]を含む配列を返す。
そのあと乱数をもとに、表示するアニメーション指定し、再生（play）している。

最後の実装は、モブがスクリーンから離れたときに自分自身を削除するようにすることだ。VisibleOnScreenNotifier2Dノードのscreen_exited()シグナルをモブに接続し、次のコードを追加します。ここではシグナルが届いた際のコールバック（onNotify）のみを実装する。

```
func _on_visible_on_screen_notifier_2d_screen_exited():
	queue_free()
```