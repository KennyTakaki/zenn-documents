---
title: "Amazon GuardDutyがリージョンサービスであることを再確認した"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,GuardDuty]
published: true
---

# 目的とサマリ

GuardDutyはリージョンサービスであり、有効にするリージョンを選択することができる。
この際に、ちょっとした議論になった。
1. 有効にしたリージョンのみで検出を行うのか
2. 有効にしたリージョンに検出結果があつまるのか

1 のつもりだったのだが、ふいに議論になった際に自信を持てなかったので確認する。

# サマリ
結果は以下だった。
- Amazon GuardDuty は有効にしたリージョンに対して検出を行う。
- 検出結果はリージョンを超えない。ユーザが結果集約の実装をする必要がある。

# 情報ソース
## Amazon GuardDuty のドキュメント
[このページで](https://docs.aws.amazon.com/ja_jp/guardduty/latest/ug/guardduty_settingup.html#guardduty_enable-gd
)有効にしたリージョンで検出を開始するとあった。

> Once enabled, GuardDuty will immediately begin to monitor for security threats in the current Region.

## Amazon GuardDutyに関するよくある質問
[このページ](https://aws.amazon.com/jp/guardduty/faqs/?nc1=h_ls)のQAに以下のようにある。リージョンごとの結果はGuardDutyの機能では他のリージョンに飛ばないことになっている。
> GuardDuty はリージョン別のサービスです。複数のアカウントが有効となっていて、複数のリージョンが使用されている場合でも、GuardDuty のセキュリティの検出結果は、基盤となるデータが生成されたリージョンと同じリージョンに残ります。このため、分析されたすべてのデータがリージョンに基づいており、AWS リージョンの境界を超えないことを保証します。

# 実際に試してみようとしたが…
CLIから[こちら](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/guardduty/create-sample-findings.html)のAPIでFindingsを発生させてみようとしたが、そもそもGuardDutyを有効化していないとdetecter-idが発行されないのでAPIを叩くこともできなかった。

強制的にGuardDutyがFindingsを検知するインシデントを起こそうと思って、[こちらのテスター](https://github.com/awslabs/amazon-guardduty-tester)を利用しようとしたが、やはり説明でGuardDutyをOnにしてから使えとあった。

# まとめ
ひとまず自身の認識が間違えていないことがわかった。