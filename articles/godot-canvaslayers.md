---
title: "GodotのCanvasLayersを理解する"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---

# 描画に関する基本を理解する
Godotでモノを描画する際の順序や、描画したものに対する変更（スライドさせたり、差し替えたり）を制御する方法を学びたい。

## CanvasLayers
CanvasLayersを利用すると、2Dのレンダリングレイヤーを制御できる。CanvasLayerに対して数値が設定されて、数値が大きなレイヤーは数値が小さいレイヤーより上に描画される。このレイヤー毎に何かしらの作用を行える。テクニカルタームでいえば、CanvasLayerはそれぞれにトランスフォームを持っており、他のレイヤーのトランスフォームから独立している。例えば、UI専用のレイヤーを用意すれば、背景の変化などから切り離すことができる。

公式の説明を参照すると以下のようなイメージになる。
![Alt text](/images/articles/godot-canvaslayers/LayerAbstruct.png)

CanvasLayers は、ノードの描画順序を制御するために必要必須ではなく、ノードが他のノードの「前」または「後ろ」に正しく描画されるようにする標準的な方法は、シーンパネル内のノードの順序を操作することだ。シーンパネルで上にあるほど後ろに描画されるのでこれで制御を行う。
エディタの機能で制御する以外には、2Dノードには描画順序を制御するためのCanvasItem.z_indexプロパティがあり、こちらのプロパティにスクリプトでアクセスすればよい。

# Viewport and Canvas Transform(表示領域とキャンバスの状態変化)

## Canvas transform
すべてのキャンバスレイヤーはトランスフォーム（平行移動、回転、スケールなど）を持ち、Transform2Dとしてアクセスできる。
デフォルトではノードはレイヤー0に描画される。ノードを別のレイヤーに配置するには、CanvasLayer ノードを使う。

## Global canvas transform
ビューポートにはグローバル・キャンバス・トランスフォーム（Transform2D）があります。これはマスター・トランスフォームで、すべてのキャンバスレイヤーのトランスフォームに影響する。

## Stretch transform
ビューポートにはストレッチ・トランスフォーム（Stretch Transform）がある。このトランスフォームは内部的に使用されますが、各ビューポートで手動で設定することもできる。

## Window transform
ルートビューポートはWindowである。Windowのコンテンツを拡大縮小して配置するために、各Windowにはウィンドウトランスフォームが容易されている。

## Transform order
CanvasItemのローカル座標を実際のスクリーン座標に変換するには、次の一連の変換を適用する必要があります：
全ての変換は右から左へと向かう。Globalなものから変換がてきようされる（っぽい）。右から左は基本はアフィン変換（つまり行列の内積）、左から右は逆アフィン変換だ。
![Alt text](/images/articles/godot-canvaslayers/order.png)


## 最後に
まだいまいちしっくり来ていないが、概要はつかめた。