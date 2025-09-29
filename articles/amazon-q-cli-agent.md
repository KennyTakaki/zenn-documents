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

# カスタムエージェントの作成

さっそく最初の agent を作成してみます。

```
/agent create -n agent-my-first
```

コマンド入力すると、エディタが立ち上がって以下のようなエージェントの設定ファイルを編集できます。保存して`Agent agent-my-firs has been created successfully`のメッセージがでたらエージェント作成が完了です。

```
{
  "$schema": "https://raw.githubusercontent.com/aws/amazon-q-developer-cli/refs/heads/main/schemas/agent-v1.json",
  "name": "agent-my-first",
  "description": "",
  ...
}
```

エージェントの設定ファイルはデフォルトではグローバルカスタムエージェントの出力ティレク取りに保存されます。グローバルカスタムエージェントのパスは以下です。

`~/.aws/amazonq/cli-agents/{agent-name}.json`

確認してみると、ファイルが作成されていることがわかります。

```
ls ~/.aws/amazonq/cli-agents/
agent_config.json.example  agent-my-first.json
```

プロジェクトレベルのカスタムエージェントを作成したい場合にはコマンド実行時に `-d` オプションを指定します。プロジェクトレベルのカスタムエージェントのパスは以下です。
`.amazonq/cli-agents/{agent-name}.json`
-d は.amazonq/cli-agents からの相対ディレクトリになっているようで`.`を指定するだけで`.amazonq/cli-agents`にファイルが配置されました。

```
/agent create -n project-agent -d .
```

確認してみるとファイル作成がなされています。

```
ls .amazonq/cli-agents/project-agent.json
.amazonq/cli-agents/project-agent.json
```

エージェントはプロジェクト（ローカル）→ グローバル → ビルトインのエージェントの順に読み込まれます。今回はプロジェクトに対するエージェントを開発していきます。

エージェント作成には generate コマンドも利用ができます。少量のインタラクションに回答すると、generate コマンドがエージェントファイルを出力します。この出力には prompt 部分にシステムプロンプトが設定されます。

```
[my-default-agent] > /agent generate

✔ Enter agent name:  generated-agent
✔ Enter agent description:  このエージェントはTypeSciptでWeb開発するためのものです
✔ Agent scope Local (current workspace)
Select MCP servers (use Space to toggle, Enter to confirm):

```

出力には最初から prompt が記述されています。

```
{
  "$schema": "https://raw.githubusercontent.com/aws/amazon-q-developer-cli/refs/heads/
main/schemas/agent-v1.json",
  "name": "generated-agent",
  "description": "このエージェントはTypeSciptでWeb開発するためのものです",
  "prompt": "You are a specialized TypeScript web development assistant. Help users with TypeScript code, web development best practices, modern frameworks, debugging, and optimization. Focus on providing clean, type-safe solutions and following current TypeScript and web development standards.",
  "mcpServers": {},
　...
}
```
