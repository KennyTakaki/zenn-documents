---
title: "Godotで多重スクロール（Parallax Scrolling）を実装する"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---

# 概要
Godotで多重スクロールを実装する。背景をレイヤー管理して、レイヤーごとに異なる速度で動かすことで疾走感などを演出する技法だ。

以下の記事でも簡単に触れていて、「４．ゲーム特有の表現技法を学ぶ」の中で触れた。
https://zenn.dev/frommiddle1/articles/godot-from-now-on

完成系のイメージは以下になる。赤い月が表示されているのが最背面、その手前に★が動いているレイヤーがあり、最前面にスプレー塗装したようなレイヤーがある。これらをレイヤー管理し、別々の論理を適用している。速度災害にも、★のレイヤーは上下左右の動きに対応しているのに対し、最前面は左右の動きにのみ対応させている。
![Alt text](/images/articles/godot-parallax-scrolling/parallax-scrolling.gif)

# 実装方法
面白いことに、差分背景はGodotの組み込みクラス（ノード）として容易されている。シーンの実装例としては以下のようになる。
![Alt text](/images/articles/godot-parallax-scrolling/scenes.png)

それぞれのクラスは以下のようなっており、ParallaxBackground内にParallaxLayerを入れ込んでいけばインスペクターの設定項目で速度差などを設定可能になる。
- World： Node2D
- bckground： ParallaxBackground
- space: ParallaxLayer
- Sprite2D: Sprite2D
...
- Player:CharacterBody2D

例えばspaceに対してMotionのScaleを0,0に設定すれば背景を固定可能であるし、
![Alt text](/images/articles/godot-parallax-scrolling/space.png)
（spaceのインスペクター）

starsに対してMotionのScaleを0.8,0.8に設定すれば、Characterの動きに対してx方向、y方向ともに0.8の速度で移動する。
![Alt text](/images/articles/godot-parallax-scrolling/stars.png)
（starsのインスペクター）

![Alt text](/images/articles/godot-parallax-scrolling/gas.png)
（gasのインスペクター）

Mirroringに関してはSpaceは固定させるので不要だが、ループさせたい場合にはViewPortの解像度に合わせてやれば問題なく動作する。
![Alt text](/images/articles/godot-parallax-scrolling/2D.png)

画像はプロジェクトのViewportを1920,1080に設定して、3枚とも同じサイズで背景を用意して利用した。各背景のサイズを変えることもできると思うが、ループさせる場合は各サイズが整数倍になるようにしたほうが管理が楽だと思う。

# 別実装の案
単純なシーンの前後関係だけでもParallax Scrollingは実装できると思う。背景の前後を単純にシーンの前後で表現して、それぞれの背景にスクリプトを割り当てて速度差分を表現する論理を記述すればよい。
が、組み込みクラスとして提供されているので上記のクラスを利用するのが賢いと思う。

# 参考
以下の動画が非常に参考になった。
https://www.youtube.com/watch?v=1k0hKUtZrq0