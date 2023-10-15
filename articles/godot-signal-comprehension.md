---
title: "Godotのシグナルをきちんと理解する"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [game,Godot]
published: false
---

# 目的
現在Godotを学習中だ。シグナルに関してチュートリアルで一通りさらったが、もう一段理解を深めるためにおさらいをしようと思う。

概念的な理解～コードレベルでの理解までを行う。

## 概念的な理解
シグナルはシーン（ノードの集合、オブジェクトと理解するとよい）間での相互作用を実演するためにGodotが提供する機構だ。例えばシューティングゲームにおいて
- 自機（player）
- 敵機（enemy）
- アイテム（ボム）

の３つがあった際に「自機がボムに触れた際に画面内の敵機が全て消える」といったケースを考えれば、それぞれが相互に作用する必要があるのは想像にたやすい。
  
さこういった相互作用の仕組みをGodotではシグナルと呼ばれる機構で実現している。シグナルはソフトウェア開発で利用されるデザインパターンの一種である observer pattern を利用して構築されている。そこでまずobserver patternを理解する。

ちなみにデザインパターンはGang of Four(GoFと略される)と呼ばれる優秀なプログラマによって作られたオブジェクト指向プログラミングにおける再利用性の高いコーディングのパターンのことだ。ソフトウェアエンジニアなら耳にしたことがある人も多いと思う。observer patternも含めて全23種類あるので、気になる人は調べてみるといい。

### observer pattern
observer pattern ソフトウェア開発で利用されるデザインパターンの一種だ。observer patternにおいて登場人物は以下の2種類が存在する。
- observer（何かが起こった時に知りたいモノ）
- subject（何かが起こった時に通知するモノ）

observerは何かが起こったと誰かに通知された際実行する処理を持っている。ここではこの処理のことをonNotifyと呼ぶ。

Subjectは何かが起こった時に通知する相手のリストとそのリストを外から変更（追加/削除）するため
方法を提供する。ここでは追加方法をaddObserver、削除方法をremoveObserverと呼ぶ。
そして、Subjectはリストに入っている相手に対しての通知方法を持っている。ここではこの通知方法をnotifyと呼ぶ。
  
ここまでの理解を図にすると以下のようになる。

![Alt text](/images/articles/godot-signal-comprehension/observer-subject.png)

これを先ほどのシューティングゲームの当てはめると以下のような例が考えられる。敵機(enemy)がアイテム（bomb）に対してaddObserverを呼び出して自身を通知先として登録する。

![Alt text](/images/articles/godot-signal-comprehension/register.png)

Observerは（ここでは記述していないが）プレイヤーと触れた際に通知を発行する。通知が発行された際にObserverはonNotifyを実行する。この例では、例えば自身を除去して、スコアなどがあればそれを更新する等の処理が発生する。（もちろんスコアも別途シーンとして作成し、シグナルを介して敵機から通知を発行するように作るのが自然だろう）

![Alt text](/images/articles/godot-signal-comprehension/vanish.png)

## シグナルの実装方法

https://zenn.dev/frommiddle1/articles/godot-tutorial10


## 参考ページ
- Signaに対する概要が記述されている
https://docs.godotengine.org/en/stable/getting_started/introduction/key_concepts_overview.html#

- hoge