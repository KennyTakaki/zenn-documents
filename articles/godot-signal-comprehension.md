---
title: "Godotのシグナル（カスタムシグナル）を理解する"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---

# 目的
現在Godotを学習中だ。シグナルに関してチュートリアルで一通りさらったが、もう一段理解を深めるためにおさらいをしようと思う。特にカスタムシグナルに関してはTutorialが舌足らずだったので一通りサンプル実装を施して挙動を確認してみる。カスタムシグナルのサンプルがほしい場合はカスタムシグナルのサンプル実装まで飛ぶといい。

概念的な理解～コードレベルでの理解までを行う。


# 理解とサンプル実装
## 概念的な理解
シグナルはシーン（ノードの集合、オブジェクトと理解するとよい）間での相互作用を実現するためにGodotが提供する機構だ。例えばシューティングゲームにおいて、
- 自機（player）
- 敵機（enemy）
- アイテム（ボム）

の３つがあった際に「自機がボムに触れたとき画面内の敵機が全て消える」といったケースを考えれば、それぞれが相互に作用する必要があるのは想像にたやすい。
  
こういった相互作用の仕組みをGodotではシグナルと呼ばれる機構で実現している。シグナルはソフトウェア開発で利用されるデザインパターンの一種の observer pattern を利用して構築されている。そこでまずobserver patternを理解する。

ちなみに、デザインパターンはGang of Four(GoFと略される)と呼ばれる優秀なプログラマによって作られたオブジェクト指向プログラミングにおける再利用性の高いコーディングのパターンのことだ。ソフトウェアエンジニアなら耳にしたことがある人も多いと思う。observer patternも含めて全23種類あるので、気になる人は調べてみるといい。

### observer pattern
observer pattern はソフトウェア開発で利用されるデザインパターンの一種だ。observer patternにおいて登場人物は2種類が存在する。
- observer（何かが起こった時に知りたいモノ）
- subject（何かが起こった時に通知するモノ）

observerは何かが起きて、それが通知された際に実行する処理を持っている。ここではこの処理のことをonNotifyと呼ぶ。

Subjectは何かが起こった時に通知する相手のリストとそのリストを外から変更（追加/削除）するための方法を提供する。ここでは追加方法をaddObserver、削除方法をremoveObserverと呼ぶ。
そして、Subjectはリストに入っている相手に対しての通知方法を持っている。ここではこの通知方法をnotifyと呼ぶ。
  
ここまでの理解を図にすると以下のようになる。

![Alt text](/images/articles/godot-signal-comprehension/observer-subject.png)
これを先ほどのシューティングゲームの当てはめると以下のような例が考えられる。敵機(enemy)がアイテム（bomb）に対してaddObserverを呼び出して自身を通知先として登録する。

![Alt text](/images/articles/godot-signal-comprehension/register.png)

bombは（ここでは記述していないが）プレイヤーと触れた際に通知を発行する。通知が発行された際にObserverはonNotifyを実行する。この例では、例えば自身を除去して、スコアなどがあればそれを更新する等の処理が発生する。（もちろんスコアも別途シーンとして作成し、シグナルを介して敵機から通知を発行するように作るのが自然だろう）

![Alt text](/images/articles/godot-signal-comprehension/vanish.png)

## シグナルの実装方法
さて、チュートリアルで学習した際にはボタンがプレイヤーに作用した（ボタンを押すとプレイヤーの動きが止まる）
https://zenn.dev/frommiddle1/articles/godot-tutorial10

まず最初にエディタでSprite2DノードとButtonノードを用意し、エディタの機能でButtonノードにSprite2Dを接続した。手順の中でpressed()シグナルの接続先をSprite2Dのスクリプトに向けることで関係性を作成した。関係性を作成したときに自動でSprite2Dに_on_button_pressed()関数が作成された。これは図にすると以下のように理解できる。これまでのonNotifyが_on_button_pressedであり、pressedがnotifyに相当する。
![Alt text](/images/articles/godot-signal-comprehension/editor.png)

editorの機能で暗黙にaddObserverが実行されていることになる。（設定を削除すれば、removeObserverの呼び出しに等しい）
しかしながら、このままでは動的な登録、削除が実現できない。そこでコード上で接続する標準的な方法が提供されている。これがconnectメソッドだ。ノードが保持するシグナルはconnectメソッドを提供している。observer側はこのメソッドに呼び出してほしいメソッド（つまりコールバック関数:onNotify）を指定して登録する。なのでtimerノードのtimeoutに_on_timer_timeoutを登録する際は以下のようになる。
（_on_timer_timeoutは呼び出すわけではないので、呼び出しの()が不要なことには注意が必要だ。）
```
timer.timeout.connect(_on_timer_timeout)
```
なかなかシンプルな機構だと思う。

## カスタムシグナル
自身でシグナルを作成することも可能だ。この機能を使いこなすことが、ゲーム開発の第一歩になると思う。新規シグナルを作成する場合はsignal signal-nameを対象ノードのスクリプトに記述する。このように実装すると、healthが0以下になった際に、このノードはhealth_depletedを通知する。observer側ではhealth_depleted.connect(call_back)としてコールバックを登録しておけばよい。
```
extends Node2D
signal health_depleted
var health = 10

func take_damage(amount):
	health -= amount
	if health <= 0:
		health_depleted.emit()
```

#### シグナルの引数

signal signal-name(value1, value2)と記述することで、引数をもったシグナルを作成することができる。（例は引数二つの場合だ）。ここでは具体例として以下のように定義してみよう
signal on_health_changed(old_value,new_value)

例えば上記の場合はobserver側のコールバックを以下のように定義しておく必要がある。
```
func _on_health_changed(old_value,new_value):
	print(old_value)
	print(new_value)
```
つまりコールバックの実装者（observerのonNotify）が、シグナルの引数を知っておく必要がある。

## カスタムシグナルのサンプル実装
Node2D以下にSprite2Dノードを3つ配置したシーンを作成する。そして、それぞれSubject,Observer01,Observer02と名前を付ける。シーンドックは以下のようになる。
![Alt text](/images/articles/godot-signal-comprehension/scenedoc.png)

これらのノードにそれぞれ以下のスクリプトをアタッチする。

  
Subjectでは引数あり/なしのカスタムシグナルをそれぞれ定義する。_processの中でシグナルを発火させる条件を更新する関数を呼び出し、条件に一致する場合にはシグナルを排出する。イメージとしては引数なしのシグナルは5秒ごとに、引数ありのシグナルは10秒ごとに排出される。
```Subject.gd
extends Sprite2D

signal my_sample_signal

signal my_sample_signal_args(arg1,arg2)

var condition1 = 0
var condition2 = 0

func _init():
	print("Hello, world! I'm Subject.")

func deal_cond1(delta):
	condition1+=delta
	if condition1 > 5:
		my_sample_signal.emit()	
		condition1 = 0

func deal_cond2(delta):
	condition2+=delta
	if condition2 > 10:
		my_sample_signal_args.emit("Well done!!",condition2)
		condition2 = 0

func _process(delta):
	deal_cond1(delta)
	deal_cond2(delta)

```

Observer01では引数なしのシグナルを受信し、受信したことをコンソールに出力する。
```Observer01.gd
extends Sprite2D

func _init():
	print("Hello, world! I'm Ovserver1.")

func _on_my_sample_signal():
	print("Observed1 got notify from Subject !!")

func _ready():
	var subj = get_node("../Subject")
	subj.my_sample_signal.connect(_on_my_sample_signal)

```

Observer02では引数ありのシグナルを受信し、受信した引数をコンソールに出力する。
```Observer02.gd
extends Sprite2D

func _init():
	print("Hello, world! I'm Ovserver2.")

func _on_my_sample_signal_args(message, sum_of_delay):
	print("Observed2 got notify from Subject !!")
	print(message)
	print(sum_of_delay)

func _ready():
	var subj = get_node("../Subject")
	subj.my_sample_signal_args.connect(_on_my_sample_signal_args)

```

上記の実装を施して再生すると狙い通りシグナルの授受が行われていることが確認できた。
![Alt text](/images/articles/godot-signal-comprehension/result.png)


# まとめ
おおむね、以下の理解で問題なさそうだ。
- カスタムシグナルを作成する場合はsignalキーワードでSubjectに実装を行う
- ObserverはSubjectのシグナル（プロパティ）のconnectメソッドを呼び出す
- connectの呼び出し引数にコールバックを指定する
- シグナルが引数を持つ場合は、指定するコールバックがその引数を受け取るように実装する

シグナルは理解できたと思う。