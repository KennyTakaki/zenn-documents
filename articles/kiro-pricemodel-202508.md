---
title: "kiroの料金プランが変更になるようなので調べたら初期ボーナスが消費されていた…"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","kiro"]
published: false
---

# 概要
kiroを本格的に触り始めるか～、と思っていたらXでSimSta2さんが料金プランの変更を示唆するポストをしていた。課金がはいると怖いので、なるべく早く調べてみた。
https://x.com/shimagaji2/status/1956495182020722854

(本家のアナウンスはこちら)[https://kiro.dev/blog/pricing-plans-are-live/]

# 結論

# 詳細
## 対象となる人
Q Developerのサブスクリプションがないユーザが新しい料金モデルに移行している。Google、Github、AWS BuilderIDアカウントでログインしている場合は適用対象だそうだ。私はGithubのアカウントでログインして、WaitList組に配布されたコードを利用しているので適用対象だと思われる。

この場合は「月間Vibeリクエスト50件とSpecリクエスト0件が利用可能」らしい。

ただし、全ユーザーを対象として、新料金プラン利用開始から14日間まで有効な「Specリクエスト100件とVibeリクエスト100件のウェルカムボーナス」があるらしいので、すぐには課金が入らない様子。

## 利用状況の確認方法
バージョン0.2.13以上のkiroを利用する必要がある。バージョンはHelp->Aboutから表示可能。

![alt text](/images/articles/kiro-pricemodel-202508/kiro-version.png)

kiroのエディタのプロフィールアイコンをクリック。
![alt text](/images/articles/kiro-pricemodel-202508/prof.png)

利用状況のモニタリングが可能です。想定通り私はフリープランで、...あれ？すでにvibeが半分以上消費されている？
![alt text](/images/articles/kiro-pricemodel-202508/consume.png)

Upgrade Planからアクセスするとプランが選べます。Kiro Pro PlanなんかはQ Developer Proと同じ料金ですね。しかしながら、Pay-per-use overageとなっており、リクエスト数上限を超える課金が入るようです。ちょっと怖いですね。
![alt text](/images/articles/kiro-pricemodel-202508/plans.png)

## Q Developer Proを付与したアイデンティティでログイン実施

Kiro PlanではどのプランでもPay-per-useが記述されているので少し怖いなと感じたので、Q developer Proを付与しているIAM Identity CenterのIDで再ログインしてみた。
![alt text](/images/articles/kiro-pricemodel-202508/changeidentity.png)

この状態でプロフィールを確認してみると以下のような状態になります。クエリの消費量などが明示されていないので、（おそらく）Kiro Planからの対象外として認識されているようです。

![alt text](/images/articles/kiro-pricemodel-202508/identity-center.png)

## 再ログイン時のセッションに関して
ちなみにkiroとやり取りをしたセッションに関してはIDE側に残っているようで、切り替えをしても失われていなかったです。
![alt text](/images/articles/kiro-pricemodel-202508/session.png)

# まとめ
