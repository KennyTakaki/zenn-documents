---
title: "Godot でゲーム開発がしてみたい 6 (Creating instances)"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: true
---
Godot DocsのIntroductionを一通り見てきたので、Step by Stepの章に着手する。
この記事では以下のトピックからCreating instancesを学習する。これらのトピックは次の章であるYour first 2D gameの準備となる。これらのトピックで基礎がかなり身につくらしい。
- Nodes and Scenes
- Creating instances ← いまここ
- Scripting languages
- Creating your first script
- Listening to player input
- Using signal

https://docs.godotengine.org/en/stable/getting_started/step_by_step/instancing.html
  
### インスタンスを作成する（Creating instances）

前回は、一つのノードをルートに持つ構成で、シーンは木構造を持つノードの集まりであることを確認した。
プロジェクトは任意の数のシーンに分割することができる。この特徴はゲームの様々なコンポーネントを分解して整理するのに役に立つ。
好きなだけシーンを作成して、「テキストシーン」を意味する拡張子.tscnのファイルとして保存することができる。（tscnはtext+sceneだったのか）  
前のレッスンのlabel.tscnファイルがその例だ。これらのファイルは、シーンの内容に関する情報を保持（パック）しているため、私たちは「保持（パック）されたシーン」と呼ぶ。
一度保存したシーンはブループリントとして機能し、他のシーンで何度でも再現することができる。このようにテンプレートからオブジェクトを複製することをインスタンス化と呼ぶ。

インスタンス化されたシーンはノードのように動作する。エディターはデフォルトでそのコンテンツを隠蔽する。あるシーンをインスタンス化すると、対応するノードだけが表示されます。それぞれの複製がユニークな名前を持つことも念頭に置いておく必要がある。

### 実践する（In practice）
以下からサンプルプロジェクトをDLしてきて展開する。
https://github.com/godotengine/godot-docs-project-starters/releases/download/latest-4.x/instancing_starter.zip

展開したプロジェクトをインポートする。Godot4.0で編集されているようで、4.1で開くとちょっと問題があるかもしれない。といった旨のメッセージが出た。
![Alt text](/images/articles/godot-tutorial6/importproject.png)

このプロジェクトには、ボールが衝突する壁を含むmain.tscnとball.tscnの2つのシーンがあって、Mainシーンが自動的に開く。

Mainノードの子として、ボールを追加する。シーンドックで、Mainノードを選択する。そして、シーンドックの上部にあるリンクアイコンをクリックする。このボタンで、現在選択されているノードの子として、シーンのインスタンスを追加できる。
![Alt text](/images/articles/godot-tutorial6/Main.png)


次に、Ballノードのインスタンスをさらに作成する。ボールを選択したまま、Ctrl-D（macOSではCmd-D）を押して複製コマンドを呼び出せる。クリックしてドラッグし、新しいボールを別の場所に移動すると複製できたことがわかる。

ゲームを起動すると複製したボールを確認できて、それぞれが独立して落下する。ボールシーンをテンプレート（クラス）として、インスタンスを複数作成することができた。
![Alt text](/images/articles/godot-tutorial6/multi-balls.png)

### シーンとインスタンスを編集する（Editing scenes and instances）

インスタンスの編集は、個々のインスタンスのとクラスであるシーンそのものの編集と、の2つの方法がある

- 個々のインスタンスの編集はインスペクタを使用して、他のボールに影響を与えずに1つのボールのプロパティを変更することができる。

- シーンの編集はシーンに対するデフォルトのプロパティを修正することができる。つまり、そのインスタンスすべてに影響する。今回でいえば、Ball.tscn シーンを開き、Ball ノードに変更を加えることで、すべての Ball のデフォルトプロパティを変更できる。保存すると、プロジェクト内のすべての Ball インスタンスの値が更新される。

![Alt text](/images/articles/godot-tutorial6/inspector.png)

ゲームを再起動するとボールがはねるようになった！！

次は個々のボールをいじってみる。Ball2に対してGravity Scaleを大きく設定すると、ゲームを起動した際にほかのボールより早く落ちるようになる。
![Alt text](/images/articles/godot-tutorial6/GravityScale.png)

### デザイン言語としてのシーンインスタンス（Scene instances as a design language）

Godotのインスタンスとシーンは優れたデザイン言語で、他のエンジンとは一線を画している。Godotでゲームを作る際には、MVC（Model-View-Controller）やEntity-Relationshipダイアグラムのようなアーキテクチャー・コード・パターンは使わないことが進められている。その代わりに、プレイヤーがゲームの中で目にする要素を想像することから始め、それを中心にコードを構成することができる。
クラス図のようなものを作成したりするとよさそうだ。そういった意味ではゲームのUMLを作ってみるのもいいかも知れない。

オフィシャルでも図（diagram）ができたら、そこに記載されている要素ごとにシーンを作成してゲームを開発することを勧めている。シーンのツリーを構築するには、コードまたはエディターで直接インスタンス化する。コードでやる場合は動的にインスタンス化できるのかな？乱数ベースのコントロールなどはそのように実施できると嬉しい。

### まとめ（Summary）
インスタンス化とは、ブループリント(設計、クラス)からオブジェクトを生成するプロセスである。
利点は以下
- ゲームを再利用可能なコンポーネントに分割できる
- 複雑なシステムを構造化し、カプセル化することができる
- ゲームプロジェクトの構造を自然な方法で考えるための言語として扱える