---
title: "GodotでのUI構築を理解する02"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: false
---
# 概要
GUIのカスタマイズに関して理解する。

# Introduction to GUI skinning
ゲームにとって、プレイヤーにわかりやすく、有益で、しかも視覚的に楽しいユーザーインターフェイスを提供することは不可欠です。コントロールノードは、箱から出してすぐに機能的な外観を備えていますが、常に独自性とケース特有のチューニングの余地があります。この目的のために、GodotエンジンにはGUIスキニング（またはテーマ設定）のシステムがあり、カスタムコントロールを含むユーザーインターフェースのすべてのコントロールの外観をカスタマイズすることができます。

ゲームのユニークな外観を実現するだけでなく、このシステムにより、開発者はアクセシビリティ設定を含むカスタマイズオプションをエンドユーザーに提供することができます。UIテーマはカスケード方式で適用されます（つまり、親コントロールから子コントロールに伝搬されます）。つまり、フォント設定や色覚異常ユーザー向けの調整を一箇所で適用し、UIツリー全体に影響を与えることができます。もちろん、このシステムはゲームプレイの目的にも使用できます。ヒーローベースのゲームでは、選択したプレイヤーキャラクターに応じてスタイルを変更できますし、チームベースのプロジェクトでは、サイドに異なるフレーバーを与えることができます。

## Basics of themes
スキニング・システムは Theme リソースによって駆動されます。すべてのGodotプロジェクトは、組み込みのコントロールノードで使用される設定を含む固有のデフォルトテーマを持っています。これによって、コントロールは箱から出してすぐに、独特の外観を持つようになります。しかし、Themeは設定を記述しているだけで、その設定をそれ自身を表示するために必要な方法で使用するのは、まだ個々のコントロールの仕事です。これは、独自のカスタムコントロールを実装する際に覚えておくことが重要です。
(Godotエディター自体もデフォルトのテーマに依存している。しかし、Godotプロジェクトと同じようには見えません。なぜなら、デフォルトのテーマの上に、独自に大きくカスタマイズされたテーマが適用されるからです。)

## Theme items
テーマに格納されるコンフィギュレーションはテーマ・アイテムで構成される。各項目は一意の名前を持ち、以下のデータ型のいずれかでなければなりません。

- Color
フォントや背景によく使われる色の値。色はコントロールやアイコンの変調にも使用できる。
- Constant
コントロールの数値プロパティ（BoxContainerのアイテムの分離など）、またはブーリアンフラグ（Treeの関係線の描画など）に使用できる整数値。
- Font
テキストを表示するコントロールが使用するフォントリソース。フォントには、サイズと色を除くほとんどのテキスト表示設定が含まれている。その上、整列とテキストの方向は個々のコントロールによって制御される。
- Font size
テキストを表示するサイズを決めるためにフォントと一緒に使われる整数値。
- Icon
通常、アイコンを表示するために使用されるテクスチャリソース（Buttonなど）。
- StyleBox
スタイルボックス リソースは、UI パネルの表示方法を定義する設定オプションのコレクションです。スタイルボックスは多くのコントロールで背景やオーバーレイに使用されるため、これは Panel コントロールに限定されません。

## Theme types
....
いったんここで学習をやめる。なぜならば、ここから先は装飾のはなしだからだ。まずはデフォルトのＵＩでゲームを作って、そのあとに装飾の話を理解して肉付けするとよいと考えている。