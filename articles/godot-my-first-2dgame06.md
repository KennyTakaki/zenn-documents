---
title: "Godot で最初の2Dゲームを作る 06"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---

チュートリアルの総まとめとして2Dゲームを作成してく。第六回目。ひとまず最終回。次は3Dにするか何にするか悩んでいる。
https://docs.godotengine.org/en/stable/getting_started/first_2d_game/index.html

# Finishing up
ここまでで、ゲームの機能はすべて完成した。これからは以下は、ゲーム体験を向上させるための味付けだ。

## Background
デフォルトのグレーの背景はあまり魅力的ではないので、色を変えたい。これを行う一つの方法は、ColorRectノードを使うことだ。
他のノードの後ろに描画されるように、Mainの下の最初のノードにする。ColorRectにはプロパティが1つしかなく、Colorです。好きな色を選び、ビューポート上部のツールバーかインスペクタで、"Layout" -> "Anchors Preset" -> "Full Rect "を選択し、画面を覆うように設定する。

背景画像があれば、TextureRectノードを代わりに使って追加することもできます。

## Sound effects
サウンドと音楽は、ゲーム体験に魅力を加える最も効果的な方法です。ゲームアセットフォルダには、2つのサウンドファイルがあります：BGM用の "House In a Forest Loop.ogg "と、プレイヤーが負けた時用の "gameover.wav "です。

Mainの子として、AudioStreamPlayerノードを2つ追加します。そのうちの1つにMusic、もう1つにDeathSoundという名前をつけます。それぞれ、Streamプロパティをクリックして、"Load "を選択し、対応するオーディオファイルを選択します。

全てのオーディオは、Loop設定を無効にして自動的にインポートされます。音楽をシームレスにループさせたい場合は、Stream fileの矢印をクリックして、Make Uniqueを選択し、Stream fileをクリックして、Loopにチェックを入れます。

音楽を再生するには、new_game()関数に$Music.play()を追加し、game_over()関数に$Music.stop()を追加します。

最後に、game_over()関数に$DeathSound.play()を追加します。

## Keyboard shortcut
ゲームはキーボード操作で行うので、キーボードのキーを押してゲームを開始できれば便利です。これは、Buttonノードの Shortcut プロパティで実現できます。

前のレッスンでは、キャラクターを動かすための4つの入力アクションを作成しました。同じような入力アクションを作成して、スタートボタンにマッピングします。

Project" -> "Project Settings "を選択し、"Input Map "タブをクリックします。移動の入力アクションを作成したのと同じように、start_gameという新しい入力アクションを作成し、Enterキーのキーマッピングを追加します。

もしコントローラーがあれば、コントローラーのサポートを追加しましょう。コントローラーを取り付けるかペアリングして、コントローラーサポートを追加したい各入力アクションの下で "+"ボタンをクリックし、それぞれの入力アクションにマッピングしたい対応するボタン、Dパッド、スティックの方向を押します。

HUDシーンで、StartButtonを選択し、InspectorでShortcutプロパティを見つけます。ボックス内をクリックして新しいShortcutリソースを作成し、Events配列を開いてArray[InputEvent] (size 0)をクリックして新しい配列要素を追加します。


スタートボタンが表示されたら、それをクリックするか、Enterキーを押してゲームを開始する。

これで、Godotでの初めての2Dゲームが完成した。


## ひとまず完成
2Dゲームは完成した。学んだことは多いが、チュートリアルを章か下に過ぎない。