---
title: "AWSConfig"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# 概要

AWS Config が苦手なので概念を整理する。以降AWS ConfigはConfigと省略する。

# 要素
https://docs.aws.amazon.com/config/latest/developerguide/config-concepts.html#config-recorder
## Configuration Redorder（Configuration Itemsに対する設定）
Configで記録を残す対象のリソースの種類などを指定する。いつでも変更することが可能だが、Configを利用する際に最初に設定する必要がある。

## Delivery Channel（記録するためのチャネル）
Config が記録する対象のリソースに変更があった場合に通知する。

## Configuration Items（Configで記録するものの最小構成要素）
ある時点でのAWSリソースの設定値（属性）を記録したもの。Configuration Itemの構成要素は、メタデータ、属性、関係性、現在の設定値、関連イベントを含む。Configは記録対象のリソースに変更があった場合にはいつでもConfiguration Itemを作成する。記録頻度は指定することが可能である（イベントドリブンにやることも、時間指定もできる）。

## Configuration History（あるリソースにおける時系列を表現）
Configuration history は任意の期間の Configuration Items のコレクション。Configは記録対象のリソースタイプに関して自動的にConfiguration HistoryのファイルをS3に配信する。
Configuration Historyを利用すると、ある利s－スのすべての過去の設定を確認することができる。

## Configuration Snapshot（AWSアカウントに対する一時点を表現）
Configuration snapshot はアカウントに存在する対象リソースのConfiguration Itemのコレクション。アカウントに不要なリソースなどを探すために利用することができる。ある時点での、Configuration Item の集まり。Configuration snapshotはＳ３に配信させることが可能・

## Configuration Stream（変更を配信するためのストリーム）
Configuration StreamはConfigが記録対象としているConfiguration Itemsの自動更新リスト。リソースが作成、修正、削除された時にConfigがConfiguration StreaにConfiguration Itemを追加する。Configration Streamは選択したSNSトピックを用いて動作する。AWSリソースの設定変更が行われたことをトリガにして外部システムを更新したりするのに役に立つ。


## AWS Config Rules
AWSリソースやAWSアカウント全体に対して必要とされる構成設定を表現する。リソースがルールのチェックをパスしない場合にはAWS Configはリソースに非準拠のフラグをたてて、ConfigはSNSを通じて通知する。
Conigルールの評価結果は以下4種類
- COMPLIANT：条件に合致しパスした場合
- NON_COMPLIANT：条件に合致せずパスしなかった場合
- ERROR：必須/オプションのパラメータのいずれかが有効でない、正しい型でない、または書式が正しくない
- NOT_APPLICABLE：ルールの適用ができないリソースを除外するために利用される。

## Trigger Types
Trigger TypeはConfig ルールの一部と定義されて、以下のタイプを含む。
- Configuration changes：ルールのスコープに一致するリソースが存在し、構成変更があった場合に評価する。スコープには以下を含めることができる。
    - 1つ以上のリソースタイプ
    - リソースタイプとリソースIDの組み合わせ
    - タグ
    - 記録されたリソースが作成、更新、削除されたときのいずれか
- Periodic：選択した頻度で実行
- Hybrid：上記２つの組み合わせ

## Evaluation models
2種類のConfig rulesの評価モデルがある
- Proactive：リソースをデプロイ前に評価する。COMPLINTかNON_COMPLIANTを評価する。
- Detective：デプロイ済みのリソースを評価する。

# ControlTowerから利用する場合の統合
Audit アカウントの Config のアグリゲータからリソースが確認可能。
