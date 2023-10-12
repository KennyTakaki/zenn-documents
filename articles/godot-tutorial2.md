---
title: "Godot でゲーム開発がしてみたい 2"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---

前回記事
https://zenn.dev/frommiddle1/articles/godot-tutorial


## 雑談
今までAWSのことに関してアウトプットを補足実施してきたが、プラットフォームをZennに変えてみた。Zennでの記事投稿の体験は素晴らしいな。CLIでの記事管理や、previewサーバが立てれることがとても良い。
あと、前回投稿したGodotに関して記事数を確認すると80程度しかなかった。AWSに関しては5252記事もあるらしい。さて、今回はGodotのエディタに関して学習しよう。


## Godot のエディタ（Godot's editor）
以下を参考に進める。
https://docs.godotengine.org/en/stable/getting_started/introduction/first_look_at_the_editor.html
エディタを立ち上げて最初に表示されるのは Project Manager である。既存プロジェクトのインポートや新しいプロジェクトを作成したりできる。
Asset Library Projects というもう一つのタブはコミュニティによって開発された多くのプロジェクトを含むオープンソースのアセットライブラリから、でもプロジェクトを検索することができる。  
言語切り替えは右上のプルダウンから可能である。

![Alt text](/images/articles/godot-tutorial2/image.png)

ちなみにアセットライブラリはこんな感じで既存の色々なプロジェクトを参考にしつつ進められるようだ。
![Alt text](/images/articles/godot-tutorial2/asset-lib.png)
  
新しいプロジェクトを開くと以下のような画面になる。デフォルトではメニュー、メインスクリーン、プレイ絵テストボタンがウインドウのトップに表示されている。（画像に簡単にメモしたが、ここで思う）
メインスクリーンには2D、3D、スクリプト、AssetLibの4つのメインスクリーンボタンがある。
2D画面はすべてのタイプのゲームに使用する。2Dゲームに加え、2D画面はインターフェイスを構築することになる。
3D画面では、メッシュやライトを操作したり、3Dゲームのレベルをデザインしたりでる。
![Alt text](/images/articles/godot-tutorial2/newproject.png)
  
- 中央はビューポートで、上部にツールバーがある。
- ツールバーにはシーンのノードを移動、拡大縮小、ロックするツールがある。
- ファイルシステムドックにはプロジェクト内のスクリプト、画像、音楽といったファイルが表示される。
- シーンドックにはアクティブ状態のシーンのノードが表示される
- インスペクターでは選択したノードのプロパティを編集することができる。
- ボトムパネルはデバッグコンソール、アニメーションエディタ、オーディオミキサなどが配置されていて普段はスペースを確保するため折りたたまれている。
![Alt text](/images/articles/godot-tutorial2/viewport.png)

ツールバー下のパースボタン（透視投影？）を押すと3Dビューに関するオプションのリストが開く。
（詳細は後で学ぼう）
![Alt text](/images/articles/godot-tutorial2/perspective.png)

スクリプト画面は、デバッガ、豊富なオートコンプリート、内蔵コードリファレンスを備えたコードエディタを提供する。
![Alt text](/images/articles/godot-tutorial2/script.png)


AssetLibではフリーでオープンソースのアドオン、スクリプト、プロジェクトで使用するアセットのライブラリが提供される。
![Alt text](/images/articles/godot-tutorial2/assetlib.png)

  
## 今回はここまで
エディタの機能配置に関して学ぶことができた。