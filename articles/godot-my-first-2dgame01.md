---
title: "Godot で最初の2Dゲームを作る 01"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: false
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


