---
title: "Amazon Q Developer CLI の挙動を自分好みにする-カスタムエージェント機能"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "Amazon Q", "agent"]
published: false
---

# 概要

Amazon Q Developer CLI 便利でとても好きで愛用しています。好きな特徴は、AWS の開発すと親和性がよいこと、ターミナルからでなくても作業が完結すること、セッションウィンドウの情報を save/load できることなどです。今回は利用目的に応じて、Amazon Q Developer CLI (以下、Q Dev CLI) の挙動を柔軟に制御したくなり、[カスタムエージェント](https://docs.aws.amazon.com/ja_jp/amazonq/latest/qdeveloper-ug/command-line-custom-agents.html)の動作を確認したので記録として残します。

ちなみに、私は混乱しやすいのですが **[Amazon Q Developer のエージェント機能](https://aws.amazon.com/jp/blogs/news/streamline-development-with-new-amazon-q-developer-agents/)とは別物**で、意識していないと話がかみ合わなくなってしまいます。

# 検証環境

```
q --version
q 1.16.2
```

# 機能概要

本家の概要には以下のようにあり、Q Dev CLI の挙動を json で変更可能です。

> カスタムエージェントは、さまざまなユースケースに特定の設定を定義することで、Amazon Q Developer CLI の動作をカスタマイズする方法を提供します。各カスタムエージェントは、エージェントがアクセスできるツール、アクセス許可、含めるコンテキストを指定する JSON 設定ファイルによって定義されます。

Agent では特定ツールの事前承認や関連コンテキストの自動ロードの指定が可能となります。私の目的は主に後者で、関連するコンテキストを Agent ごとに調整することでプロンプトに対する出力の精度をあげることです。Steering ファイルや Rule ファイルといったものの切り替えを Q Dev CLI でも可能にしようと考えています。

## コマンド一覧と俯瞰

/agent のサブコマンドの一覧を俯瞰してみると、このコマンドで Agent の追加や編集まで実施できるようです。Agent の設定は json で行いますが、最初に作成するのは generate や edit コマンドで実施すればよいかと思います。

```
  list         List all available agents
  create       Create a new agent with the specified name
  edit         Edit an existing agent configuration
  generate     Generate an agent configuration using AI
  schema       Show agent config schema
  set-default  Define a default agent to use when q chat launches
  swap         Swap to a new agent at runtime
  help         Print this message or the help of the given subcommand(s)
```

さっそく最初の agent を作成してみます。

```
/agent create
```
