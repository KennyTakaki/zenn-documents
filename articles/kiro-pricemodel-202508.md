---
title: "kiroの新料金プランを調べたら、初期ボーナスがしれっと消費されていた件"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "kiro"]
published: true
---

# 概要
「kiroをそろそろ本格的に触ってみよう」と思っていた矢先、Xで [SimSta2](http://x.com/shimagaji2) さんが料金プラン変更の可能性を示唆する投稿をしていました。  
課金が発生すると怖いので、早速調べてみました。

- 該当ポスト: https://x.com/shimagaji2/status/1956495182020722854  
- 本家アナウンス: [Pricing plans are live](https://kiro.dev/blog/pricing-plans-are-live/)

# 先に結論

- **新料金プランの対象者**  
  Q Developer サブスクリプションを持たないユーザー（Google / GitHub / AWS BuilderID ログイン）が対象。  
  月間利用枠は「Vibe 50件・Spec 0件」。  
  新料金開始後14日間は全ユーザーに「Spec 100件・Vibe 100件」のウェルカムボーナスあり。

- **利用状況の確認方法**  
  Kiro v0.2.13 以上で、エディタ右上プロフィールから確認可能。  
  自分の利用枠と消費状況が可視化される。  
  Upgrade Plan から有料プラン選択可だが、全プランで超過利用は Pay-per-use 課金。

- **Q Developer Pro IDでのログイン効果**  
  IAM Identity Center の Q Developer Pro 付与IDでログインすると、Kiro Plan の利用枠表示が消え、（おそらく）課金対象外扱いになる。

- **セッション保持**  
  アカウント切り替えをしても、IDE 上の Kiro セッションは維持される。

- **注意点**  
  無料枠消費後は自動的に従量課金になるため、利用頻度の高いユーザーは特に注意。

# 詳細

## 対象となるユーザー
Q Developer のサブスクリプションを持たない場合、新料金モデルに移行します。  
Google / GitHub / AWS BuilderID アカウントでログインしている場合は対象。  
（筆者は GitHub アカウントで WaitList 時のコードを利用しており、対象でした）

- 月間利用枠: **Vibe 50件 / Spec 0件**
- 新料金開始から14日間はウェルカムボーナスとして「Spec 100件・Vibe 100件」付与

## 利用状況の確認方法
- **必要バージョン**: Kiro v0.2.13 以上（Help → About で確認可能）  
  ![バージョン確認](/images/articles/kiro-pricemodel-202508/kiro-version.png)

- **手順**  
  1. エディタ右上のプロフィールアイコンをクリック  
     ![プロフィール](/images/articles/kiro-pricemodel-202508/prof.png)  
  2. 利用状況をモニタリング可能  
     （筆者はフリープランで、すでにVibeを半分以上消費済み…）  
     ![利用状況](/images/articles/kiro-pricemodel-202508/consume.png)

- **プラン変更**  
  Upgrade Plan から有料プランを選択可能。  
  ただし **全プランで超過利用は Pay-per-use 課金**。  
  ![プラン選択](/images/articles/kiro-pricemodel-202508/plans.png)

## Q Developer Pro IDでのログイン
Kiro Plan の全プランには Pay-per-use があるため不安を感じ、Q Developer Pro を付与している IAM Identity Center ID で再ログインしてみました。  
![ID切替](/images/articles/kiro-pricemodel-202508/changeidentity.png)

この状態でプロフィールを確認すると、利用枠や消費量の表示がなく、（おそらく）課金対象外として認識されているようです。  
![Pro適用後](/images/articles/kiro-pricemodel-202508/identity-center.png)

## 再ログイン時のセッション
アカウントを切り替えても、IDE 上のセッションは保持されており、作業内容は失われませんでした。  
![セッション保持](/images/articles/kiro-pricemodel-202508/session.png)


# 今後と感想
Amazon Q Developerの利用促進が目的なのか、若干kiroのPlanが不利に見えました。今後、この状態で利用して問題なく使えるかなどは不明です。
個人的にはQ Developerをサブスクリプションしていたらkiroも利用可能（or その逆）だといいなと感じているのですが、どうなるんでしょうね？