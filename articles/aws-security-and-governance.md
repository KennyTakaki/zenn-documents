---
title: "AWS のセキュリティ&ガバナンスをマルチアカウントでいじり倒してみる 01（OU設計まで）"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,ControlTower,SecurityHub,Inspector,GuardDuty]
published: false
---

# 概要
AWS をマルチアカウントでしっかりと管理したい。セキュリティとガバナンス系のサービスは巷にあふれているものの、実際に触る機会は少ない。また、ContorolTower や SecurityHub のように内部で様々なサービスが利用されている場合に実態をつかみにくくチョットしたチューニングの障壁になるため理解度を上げたい。  
加えて、Findingの集約など各サービス間の連携や、運用を想定した場合にケアすべき仕様もドキュメントレベルでの調査ではモノにするのに限界がある。  
そういうわけでここ最近、これらのサービスを設定/破棄を繰り返し、勘所をつかもうとしている。その際の所感や全体像、懸念点などを残すのがこの記事の目的である。執筆開始時点では記事の構成を十分に検討できていないので散文的な記事をいくつか書いた後に取りまとめることになると思うがその点は容赦いただきたい。

# 目指すところ
本当に体系的に実施するのであれば[AWSセキュリティ成熟度モデル(AWS Security Maturity Model)](https://maturitymodel.security.aws.dev/en/model/)などに沿うのが王道だと思われる。ただしちょっとしんどい。  
今回のスコープはマルチアカウントにフォーカスしたもので、アカウント払い出し直後に実施すべき「rootに対するMFA適用」、「操作時にrootを普段使いしない」、などの基礎的な部分は考慮しない。そのため上記のモデルは参考程度にとどめる。(クラメソさんの成熟とモデルに関する[この記事](https://dev.classmethod.jp/articles/intro-aws-security-maturity-model/)は一読の価値があります。)

まずは大規模な組織でなく、AWSに関わる人が全体で20人程度までの組織を想定しようと思う。[みずほさんの記事](https://aws.amazon.com/jp/builders-flash/202405/mizuho-service-catalog/)のようにセルフサービス型でユーザが自由に選択しながら環境を当時し、開発速度を同時に確保するスタイルは運用しながら目指すとし、最初は以下を満たす程度の実装を行う。

1. 組織が利用する環境に脅威検出を設定する (Amazon GuardDuty)
2. 組織が利用する環境に脆弱性検出を設定する (Amazon Inspector V2)
3. 組織が利用中のリソースを構成管理する (AWS Config) 
4. 組織のセキュリティ状態の自動評価を設定する (AWS Security Hub)
5. 組織に禁則事項を強制させる仕組みを用意する (Service Control Policy)
6. 組織で発生したセキュリティの検出項目を通知可能にする (Amazon EventBridge, Amazon SNS, AWS Lambda)
7. 組織で発生したセキュリティの検出項目はリージョン、アカウントを問わず一元的に確認可能にする (AWS Security Hub)
8. 組織で発生したセキュリティの検出項目は専用アカウントで取り扱う（委任）
9. 組織で発生したログは専用アカウントで保管し、開発アカウントからはアクセスできない状態にする。

ちなみにAWSセキュリティ成熟度モデルでは以下のカテゴリがある。上記はその部分的なものにすぎない。
- セキュリティガバナンス
- セキュリティ保証
- アイデンティティとアクセス管理
- 脅威検出
- 脆弱性管理
- インフラ保護
- データ保護
- アプリケーションセキュリティ
- インシデント対応

# OUの構成
マルチアカウントの管理なので、サービスブロック図を描く前に、まずはOUを設計する。[OUの推奨設計が公式から提供されている](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/recommended-ous-and-accounts.html)ので、これを参考にしてサブセットを利用する。
参考までに各OUの説明を確認しておく。
- Security OU
Log Archive account と Security Tooling (Audit) account を配置する。
- Infrastructure OU
Identity accountm, Network account,Operation Tooling,Backup account などを配置する。IAM Identity Centerなどもここに委任するのがいいみたいだ。（そのためだけにアカウントを作るのも少し面倒だな）
- Sandbox OU
チームや開発者に対して調査、試作を目的とした環境を提供する。
- Workloads OU
サービスを提供する際に本番環境/非本番環境（Testなど）をホストするための環境を配置する。ProdやTestなどのサブOU下にアカウントを配置するとよい。
- Policy Staging OU
SCPやタグポリシーなどを組織に適用する前に検証するために用意する。
- Suspended OU
廃止するアカウントを配置するOU。
- Individual Business Users OU
ビジネスサイドのメンバーがS3を介してデータ交換する場合などに利用する。あまり用意する必要性を感じない。
- Exceptions OU
Workload OU に配置されるべきだが、セキュリティポリシーで例外を必要とするアカウントが配置される。
- Deployments OU
オンプレ環境などを除いて、CI/CDをAWSで実装する場合はWorkloadから切り離してDeployments OU にCI/CD管理の機構を用意するのがよく、そのためのアカウントを配置する。
- Transitional OU
AWSの組織構造などを変更する場合にアカウントを一時的に保持する領域として用意されるOU。
- Business Continuity OU
データとワークロードの分離やBCPの要件が特殊な場合に利用するためのOU。一般的にはInfrastructure OUで運用されるBackupアカウントがあれば事足りる。

# OU構成図
上記期のOUからSecurity, Sandbox, Workloads, Policy Staging, Suspended, Deployments を採択して実装しようと思う。その他は必要になったら都度追加する。
Workloadにはprodとdevを用意して以下のように設計する。
![alt text](/images/articles/aws-security-and-governance/ou.png)
SecurityとSandboxは後段で実施するControlTowerから作成できるのでその他以外は手作業で実施することになる。  
長くなりそうなので、いったんここで切って次の記事に託す。