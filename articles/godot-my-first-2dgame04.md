---
title: "Godot で最初の2Dゲームを作る 04"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: false
---
チュートリアルの総まとめとして2Dゲームを作成してく。第三回目。Enemyの作成を行う。
https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html

ここまででプレーヤーと敵の準備ができたので、次のパートでは、新しいシーンでそれらをひとつにまとめる。敵がゲームボードの周りでランダムにスポーンして前進するようにし、プロジェクトをプレイ可能なゲームに変える。

## The main game scene

新しいシーンを作成し、Mainという名前のNodeを追加する。Node2DではなくNodeを使う。Node2D出ない理由は、このノードがゲームロジックを扱うコンテナになるからだ。2Dの機能自体はMainに必要ない。そのため一番基底に近いノードでよい。

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

