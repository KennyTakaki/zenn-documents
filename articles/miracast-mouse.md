---
title: "WindowsPCをサブモニタにするためにMiracastで拡張した画面にマウスカーソルが表示されない"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Miracast]
published: true
---

# 課題
Windows PC が２台あり片方をサブディスプレイとして利用する場合に、Miracastでメイン機からサブ機に接続する方法がとれる。私の環境でこれを実施するとサブディスプレイとして利用するPC側でマウスカーソルが表示されない状態になりとても不便であった。適当に設定をいじったら解消したので手順を残しておく。

# 手順
マウスのプロパティでポインターの軌跡を表示するとマウスカーソルがサブディスプレイ側で表示される。
![alt text](/images/articles/miracast-mouse/mouse-prop.png)

マウスのプロパティを表示するための操作は以下をたどるとよい。
1. コントロールパネル
2. ハードウェアとサウンド
3. デバイスとプリントの下にあるマウスを選択
![alt text](/images/articles/miracast-mouse/dev-print.png)
4. ポインタオプションのタブを選択

# 参考
下記のページを参考にしながら適当にオペレーションしてみた。
https://support.airserver.com/support/solutions/articles/43000516929-my-mouse-pointer-disappears-when-mirroring-to-airserver-what-can-i-do-to-fix-this-