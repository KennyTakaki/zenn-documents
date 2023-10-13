---
title: "Godot でゲーム開発がしてみたい 10 (Listening to player input)"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: false
---
Godot DocsのIntroductionを一通り見てきたので、Step by Stepの章に着手する。
この記事では以下のトピックからListening to player inputtを学習する。これらのトピックは次の章であるYour first 2D gameの準備となる。これらのトピックで基礎がかなり身につくらしい。

Nodes and Scenes
Creating instances
Scripting languages
Creating your first script
Listening to player input
Using signal ← いまここ

https://docs.godotengine.org/en/stable/getting_started/step_by_step/signals.html

## シグナルを使う（Using signals）
この章では、シグナルについて見ていく。シグナルは、ボタンが押されるなど、ノードに特定のことが起こったときにノードが発するメッセージだ。他のノードはそのシグナルに接続し、イベントが発生したときに関数を呼び出すことができる。
（シグナルに対して、コールバックを登録するイメージだ）

シグナルはGodotに組み込まれた委譲の仕組みで、あるゲームオブジェクトが他のゲームオブジェクトの変化に反応する際に、お互いを参照することなく反応することができる。シグナルを使うことで、結合を制限し、コードを柔軟に保つことができる。

例えば、画面上にプレイヤーの体力を表すライフバーがあるとする。プレイヤーがダメージを受けたり、回復薬を使ったりすると、その変化をバーに反映させたいとする。そのために、Godotではシグナルを使う。

### シーンの準備（Scene setup）
ゲームにボタンを追加するには、新しいメインシーンを作成し、Buttonと、最初のスクリプトを作成するレッスンで作成したsprite_2d.tscnシーンの両方を含めます。

基本的には今までと同様の作業でNode2DシーンにSprite2DとButtonシーンを追加する。ボタンにはTextプロパティがあるのでインスペクターからToggle motionと入力しておく。

![Alt text](/images/articles/goto-tutorial10/addbutton.png)

### Connecting a signal in the editor
ここでは、ボタンのpressシグナルをSprite2Dに接続し、その動きをオン・オフする新しい関数を呼び出す。スクリプトをSprite2Dノードにアタッチする必要があります。

ノードドックでシグナルを接続することができる。Buttonノードを選択し、エディタの右側にあるインスペクターの隣にある Nodeというタブをクリックする。

![Alt text](/images/articles/goto-tutorial10/Button-Node.png)

ここで、シグナルをSprite2Dノードに接続します。このノードには、Buttonがシグナルを発したときにGodotが呼び出す関数、Receiverメソッドが必要だ。これは自動でエディターが生成してくれる。慣習として、これらのコールバックメソッドには"_on_node_name_signal_name "という名前をつけます。ここでは、"_on_button_pressed "とする。

![Alt text](/images/articles/goto-tutorial10/on-button-pressed.png)

詳細ビューでは、任意のノードや組み込み関数に接続したり、コールバックに引数を追加したり、オプションを設定したりすることができる（今はまだ使わなそうだが、ボタンを押したときにボタンのプロパティを渡すなども可能なのだろう）。ウィンドウの右下にあるAdvancedボタンをクリックすることで、モードを切り替えることができる。

設定を行うと、node_2dのスクリプトが表示される。新しいメソッドが追加されており、左側の余白に接続アイコンが表示されます。

![Alt text](/images/articles/goto-tutorial10/callback-added.png)

passキーワードの行を、ノードの動きを切り替えるコードに置き変えることでボタンによる制御が可能だ。

Sprite2Dは、_process()関数のコードで制御されている。Godotには、処理のオンとオフを切り替えるメソッドが用意されていて、Node.set_process()だ。Nodeクラスの別のメソッドであるis_processing()は、アイドル処理が有効な場合にtrueを返します。この値を反転させるには not キーワードを使用する。
コードは以下のような感じだ。

```sprite_2d.gd
extends Sprite2D

var speed = 400
var angular_speed = PI


func _process(delta):
	rotation += angular_speed * delta
	var velocity = Vector2.UP.rotated(rotation) * speed
	position += velocity * delta


func _on_button_pressed():
	set_process(not is_processing())
```

これでボタンを押すことで、アイコンの停止、動作を切り替えることができるようになった。
![Alt text](/images/articles/goto-tutorial10/Icon-Rotation-_DEBUG_-2023-10-13-19-47-28.gif)


### コードを経由してシグナル接続を行う（Connecting a signal via code）
エディタを使用する代わりに、コードでシグナルを接続することができる。これは、スクリプト内でノードを作成したり、シーンをインスタンス化するときに必要だ。（もっと動的にゲームを制御する場合に有効だということ。）

GodotにはTimerノードというノードがあり、スキルのクールダウン時間や武器のリロードなどを実装するのに便利だ。Sprite2Dに対して子ノードとしてTimerノードを追加する。
![Alt text](/images/articles/goto-tutorial10/Timer.png)

TimerノードのインスペクターからAutostartプロパティをオンに設定し、Sprite2Dのコードに戻る。
ノードをコードで接続するには、2つの実装が必要だ。
- Sprite2Dからタイマーへの参照を取得する。
- Timerのtimeoutシグナルでconnect()メソッドを呼び出す。
(コードをつかってシグナルに接続するには、見張りたいシグナルのconnect()メソッドを呼び出す必要がある。今回はTimerのtimeoutシグナルをリッスンすることになる)


シーンがインスタンス化されたときにシグナルを接続したいので、ノードが完全にインスタンス化されたときにエンジンによって自動的に呼び出されるNode._ready()組み込み関数を使用する。

```snipet
func _ready():
	var timer = get_node("Timer")
	timer.timeout.connect(_on_timer_timeout)
```

get_node()関数は、Sprite2Dの子ノードを見て、その名前でノードを取得する。例えば、エディタでTimerノードの名前を "BlinkingTimer "に変更した場合は引数をかえて、get_node("BlinkingTimer")と呼び出す必要がある。
Timerの timeoutシグナルをスクリプトがアタッチされているノードに接続する（プロパティのように見える。Timer.timeoutってSignalを継承しているとかか？）。タイマーがタイムアウトしたら、関数_on_timer_timeout()を呼び出す。この関数をスクリプトの一番下に追加して、Sprite2Dの表示を切り替えるのに使う。

```
func _on_timer_timeout():
	visible = not visible
```
visible プロパティは、ノードの可視性を制御するブール値です。v
シーンを実行すると、Sprite2Dが1秒間隔で点滅する。


### Custom signals
スクリプトでカスタムシグナルを定義できる。例えば、プレイヤーの体力が0になったときにゲームオーバー画面を表示したいとする。そのためには、体力が0になったときに diedまたは health_depleted という名前のシグナルを定義すればいい。

```
extends Node2D

signal health_depleted

var health = 10
```
(シグナルは起こったばかりの出来事を表すので、一般的にその名前には過去形の動作動詞を使う。)

作成したシグナルはビルトインされたシグナルと同じように機能する。シグナルはノードタブに表示され、他のシグナルと同じように接続できる。

```
func take_damage(amount):
	health -= amount
	if health <= 0:
		health_depleted.emit()
```

シグナルには、オプションで1つ以上の引数を宣言することができる。引数名は括弧で囲んで指定する。

```
extends Node

signal health_changed(old_value, new_value)

var health = 10
```

シグナルとともに値を放出するには、emit()関数の追加引数として値を追加する。

```
func take_damage(amount):
	var old_health = health
	health -= amount
	health_changed.emit(old_health, health)
```

### 最後に
もう一度シグナルの章はおさらいする必要がある。理解が甘い…