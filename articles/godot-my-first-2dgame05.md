---
title: "Godot で最初の2Dゲームを作る 05"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---
チュートリアルの総まとめとして2Dゲームを作成してく。第五回目。
https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html

ある程度ゲームがかたちになったのでHeads up Displayを作成していく。ゲームのUI部分でPlayerの操作を除けば、この章がユーザインターフェースとなる。

# Heads up display 

スコアや「ゲームオーバー」のメッセージ、リスタート・ボタンなどを表示するユーザー・インターフェイス（UI）を追加する。

新しいシーンを作って、HUDという名前のCanvasLayerノードを追加する。「HUD」は「Heads-up Display」の略で、ゲーム・ビューの上にオーバーレイとして表示される情報表示だ。

CanvasLayerノードを使うと、UIエレメントをゲームの上のレイヤーに描画できるので、表示される情報がプレイヤーやモブなどのゲーム・エレメントに隠れることがない。

HUDは以下の情報を表示する必要がある：

- ScoreTimerによって変更されるスコア
- ゲームオーバー や Get Ready!などのメッセージ
- ゲームを開始するための Start ボタン

UI要素の基本ノードはControlだ。UIを作成するには、2種類のControlノード（LabelとButton）を使用する。

HUDノードの子として以下を作成する。
- Labelノード（ScoreLabel）
- Labelノード（Message）
- buttonノード（StartButton）
- Timerノード（MessageTimer）

デフォルトのフォントは読みにくいので、各LabelのインスペクターのTheme Overridesのfontsにアセットのfontsからフォントを選択し適応する。それでも小さいので、font SizesプロパティからFont Sizeを64pxに指定する。

各要素をゲーム内の好きな位置に配置する。GUIで往査しない場合はアンカープリセットを利用する。ここでは触れないので本家を参照すること。原点基準で位置を設定できる機能らしい。
![Alt text](/images/articles/godot-my-first-2dgame05/arrange.png)

MessageTimer で Wait Time を 2 に設定し、One Shot プロパティを On に設定する。

ここまできたらHUDにスクリプトをアタッチする。

MessageにGetReadyなどのメッセージを表示するために以下の関数を追記する。
```
func show_message(text):
	$Message.text = text
	$Message.show()
	$MessageTimer.start()
```

プレイヤーが負けたときの処理も実装する。以下のコードでは、2秒間「ゲームオーバー」を表示し、タイトル画面に戻り、短いポーズの後、「スタート」ボタンを表示する。

```
func show_game_over():
	show_message("Game Over")
	# Wait until the MessageTimer has counted down.
	await $MessageTimer.timeout

	$Message.text = "Dodge the\nCreeps!"
	$Message.show()
	# Make a one-shot timer and wait for it to finish.
	await get_tree().create_timer(1.0).timeout
	$StartButton.show()
```

進出の機能が出てきたので説明を張っておく
- await
シグナルが発生するまで待つ、という予約語だ。以下を参照するといい
https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html#awaiting-for-signals-or-coroutines
- get_tree()
- create_timer(sec)
当該シーンのツリーが管理するワンショットのタイマーを作成する。sec経過でtimeoutのシグナルが発生する。（簡易的にsleepみたいなのを実装する場合に便利そうなイディオムとして覚えておこう）
当該ノードを含むシーンの木を返す
- reload_current_scene
現在アクティブなシーンを再読み込みする


この関数はプレイヤーが負けたときに呼び出される。2秒間 "Game Over "を表示した後、タイトル画面に戻り、短いポーズ(1秒)の後、Start ボタンを表示する。

さらにMessageTimerのtimeout()シグナルとStartButtonのpressed()シグナルを接続し、新しい関数に以下のコードを追加する。

```
func _on_start_button_pressed():
	$StartButton.hide()
	start_game.emit()

func _on_message_timer_timeout():
	$Message.hide()
```

## Connecting HUD to Main
HUDをMainシーンにつなぎこむ。Playerシーンと同じように、MainでHUDシーンをインスタンス化する。
![Alt text](/images/articles/godot-my-first-2dgame05/hud-main.png)

次に、HUDをメインスクリプトに接続する必要があります。そのためには、Mainシーンにいくつか追加する必要がある。

HUDのNodeタブからstart_gameシグナルをMainのnew_game()に紐づける。

new_game()の呼び出しを_ready()から削除することを忘れないでください。

new_game()で、得点表示を更新し、"Get Ready "メッセージを表示する：

![Alt text](/images/articles/godot-my-first-2dgame05/start_game_main.png)

new_gameに対してスコアとメッセージを更新する処理を追加し、game_overにshow_game_overの処理を追加する。スコア更新の関数にも、HUDの操作を追記する。

```new_game追加分
$HUD.update_score(score)
$HUD.show_message("Get Ready")
```

```game_over追加分
$HUD.show_game_over()
```

```_on_score_timer_timeout追加分
$HUD.update_score(score)
```

## Removing old creeps
ゲームの基本動作は作成できたが、もう少しブラッシュアップさせる。ゲームオーバーになった際に、前の回のゲームのモブが残っていることがあるのでこれらを消す。全てのモブを対象にしたいのでgroupと呼ばれる機能を利用する。

Mobシーンでルートノードを選択し、インスペクタの隣にある Node タブをクリックします（ノードのシグナルがある場所と同じです）。シグナルの隣にあるグループをクリックし、新しいグループ名を入力して追加をクリックする。

これで、すべてのモブが「mobs」グループに入る。次に、Mainのnew_game()関数に以下の行を追加する。

```
get_tree().call_group("mobs", "queue_free")
```

call_group()関数は、グループ内のすべてのノードに対して指定された関数を呼び出す。

この時点でゲームはほぼ完成した。次で最後のパートでは、背景やループする音楽、キーボード・ショートカットを追加して、少し磨きをかける。