---
title: "Godot で最初の2Dゲームを作る 01"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---

チュートリアルの総まとめとして2Dゲームを作成してく。
https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html

# 概要

大きくは以下のようなことを経験する
- Godotエディタで完全な2Dゲームを作成
- 簡単なゲームプロジェクトを構成
- プレイヤーキャラクターを動かし、スプライトを変更する（アニメーション）
- ランダムに敵を出現させ、動かす
- スコアを数える

プロジェクトの完成品は以下からアクセスできる。
https://github.com/godotengine/godot-demo-projects

コード以外のアセット（audioやimage）は以下から取得できる。
https://github.com/godotengine/godot-docs-project-starters/releases/download/latest-4.x/dodge_the_creeps_2d_assets.zip

# 開発開始

## 新規プロジェクトを作成する
適当な名前を付けて新規プロジェクトを作る。新規プロジェクトのディレクトリにコード以外のアセットを展開する。アセットはdodge_the_creeps_2d_assets.zipを回答すると得られる。artとfontsをプロジェクト直下に展開する。ファイルシステムのドックは以下のようになる。

![Alt text](/images/articles/godot-my-first-2dgame01/assets.png)


このゲームはポートレートモード用に設計されているらしい。そのため、ゲームウィンドウのサイズを調整する必要がある。（ポートレートモードってなんだ？）
Project -> Project Settingsをクリックしてプロジェクト設定ウィンドウを開き、左の列でDisplay -> Windowタブを開きます。そこで、"Viewport Width "を480に、"Viewport Height "を720に設定する。また、ストレッチからモードをcanvas_itemsに設定する。
![Alt text](/images/articles/godot-my-first-2dgame01/view-size.png)
とにかく、ゲームウィンドウのサイズを固定するのが重要なようだ。

## プロジェクトを整理する
このゲームではプレイヤー(Player)、モブ(Mob)、HUD(HUD)の3つのシーンを作成する。HUDはHead Up Display だろうか？
このゲームでは希望が小さいのでres://で参照されるプロジェクトルートフォルダにシーンとスクリプトを保存していく。ファイルドックで様子は確認できる。

# Playerシーンを作成する
最初のシーンではPlayerオブジェクトを定義する。Playerを個別に作成することで、テストが楽になる。

## ノードの構造
はじめに、プレイヤーオブジェクトのルートノードを選択する必要がある。一般的なルールとして、シーンのルートノードは、オブジェクトの望ましい機能、つまりオブジェクトが何であるかを反映する必要があり。ここではArea2Dノードをシーンを選択し、ノードの名前をPlayerに設定する。（ゲームの設計がすごく重要になりそうだ。ある程度パタン化できると楽そうだが…）
Area2Dを使えばプレイヤーに重なったり、ぶつかったりするオブジェクトを検出することができる。
Playerノードに子ノードを追加する前に、編集中に間違えて移動させたりサイズ変更しないようにロックをかける。
![Alt text](/images/articles/godot-my-first-2dgame01/node-lock.png)

#### 命名規則
GDScriptでは
- クラス（ノード）は PascalCase 
- 変数と関数は snake_case
- 定数は ALL_CAPS
を使用する

詳細は以下のスタイルガイドを参照。
https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html#doc-gdscript-styleguide

## Sprite animation（「Sprite」ゲーム作成の文脈での意味がわからない…）

Playerノードをクリックし、子ノードとしてAnimatedSprite2Dを追加する。AnimatedSprite2Dは、プレーヤーの外観とアニメーションを処理する。
AnimatedSprite2Dには、表示できるアニメーションのリストであるSpriteFramesリソースが必要だ。これがない場合、ノードの横に警告マークがでる。
作成するには、InspectorのAnimationタブにあるSprite Framesプロパティからempty をクリックして新規作成すると、SpriteFramesパネルが開く。
ここにアニメーションupを追加し、defaultをwalkとする。アニメーションにはそれぞれアニメーションフレームを付与することができるので、walkにファイルシステムのartからPlayerGrey_walk1.png、PlayerGrey_walk2.png、PlayerGrey_up1.png、PlayerGrey_up2.pnをそれぞれのアニメーションに割り当てる。

また、プレイヤー画像はゲームウィンドウには少し大きすぎるので、縮小する必要がある。AnimatedSprite2Dノードをクリックして、Scaleプロパティを(0.5, 0.5)に設定します。ScaleプロパティはインスペクタのNode2Dの下にある。

![Alt text](/images/articles/godot-my-first-2dgame01/SpriteFrames.png)

最後に、Playerの子ノードとしてCollisionShape2Dを追加する。これでプレイヤーの「ヒットボックス」、つまり衝突範囲の境界が決まる。このキャラクターの場合、CapsuleShape2Dノードが最適なので、インスペクタのShapeの隣にあるemptyをクリックし、CapsuleShape2Dをクリックする。

最終的に以下のようになる。これをシーンとして保存する。ここでは、player.tscnとして保存した。
![Alt text](/images/articles/godot-my-first-2dgame01/result.png)

次回に続く…