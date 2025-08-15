---
title: "KiroでAmazon流『Working Backwards』を意識してタスク定義してみた"
emoji: "🔥"
type: "tech"
topics: ["aws", "kiro"]
published: true
---

# はじめに
ついに、長かったKiroのWaiting Listから解放されました！  
「[Code with Kiro Hackathon](https://kiro.devpost.com/?trk=dccd318a-a012-40c6-bffb-bd0a6216646d&sc_channel=el)」にも登録済みなので、作品投稿を見据えてまずはKiroの基本機能に触れてみることにします。今回は、最初のステップとして**タスク定義まで**を実際に試してみました。

---

# なぜ「Working Backwards」なのか
Kiroの大きな特徴は**Spec Coding**。  
これは、やりたいことを文章で伝えると、以下のような流れで実装に必要な資料を自動生成してくれるアプローチです。

1. 要件定義（`requirements.md`）  
2. 設計（`design.md`）  
3. 実装手順（`tasks.md`）

これを聞いたとき、私は直感的に「Amazon流の**Working Backwards**と相性が良さそう！」と感じました。

Working Backwardsは「顧客から逆算して開発する」という方法論で、プロダクト開発の出発点を**顧客視点のゴール設定**に置きます。  
この考え方をKiroに適用すれば、より意味のある要件が作れそうです。

---

## Working Backwardsの5つの質問
この手法では、まず以下の5つの問いに答えることから始めます。

1. **Who is the customer?**  顧客は誰で、どんな洞察があるか？
2. **What is the problem/opportunity?**  顧客が直面している課題やチャンスは何か？
3. **What is the solution?**  解決策は何で、その中で最も重要な価値は何か？
4. **How do we describe it?**  解決策と体験をどう説明するか？
5. **How do we test it?**  顧客とどうテストし、どう成功を測るか？

この質問に答えたうえでプレスリリースやFAQを作成し、そこから開発に入るのが本来の流れです。  
今回はそこまでやらず、質問への回答を**Kiroへのインプット**として使いました。

---

# 実践：株式推奨システムを題材に
題材として選んだのは、私が興味のある**株式投資支援システム**。  
おすすめ銘柄や売買タイミングまで提示してくれる仕組みを想定し、自分と似た投資スタイルの人物像を設定して回答を作成しました。

- **和文回答**: [five-questions.jp.md](https://github.com/KennyTakaki/tikkiro-prfaq/blob/main/five-questions.jp.md)  
- **英文回答**: [five-questions.md](https://github.com/KennyTakaki/tikkiro-prfaq/blob/main/five-questions.md)

---

# Kiroの出力結果

## 1. 要件定義（Requirements）
回答を基に生成された要件定義がこちら。  
[requirements.md](https://github.com/KennyTakaki/kiro-dev01/blob/main/.kiro/specs/intelligent-stock-recommendation/requirements.md)

特に印象的だったのは、**5つの質問の意図がしっかり反映されていること**です。  
機能要件は以下の6つにまとめられました。

- データ駆動型株式推奨機能
- 将来の価格予測モデリング
- 証拠に基づく投資理由提示
- 実際の取引検証とパフォーマンス追跡
- パーソナライズド投資プロファイル管理
- 包括的なデータ統合と独自分析

---

## 2. 設計（Design）
次に設計を生成。  
[design.md](https://github.com/KennyTakaki/kiro-dev01/blob/main/.kiro/specs/intelligent-stock-recommendation/design.md)

初期出力では、Secrets管理やイベント制御が不足していたため、以下の要望を追加で出しました。

- **イベント制御レイヤー追加**（Amazon EventBridge, AWS Step Functions想定）
- **シークレット管理レイヤー追加**（AWS Parameter Store, Secrets Manager想定）
- 技術スタックは可能な限りTypeScriptで統一し、モノレポ管理
- 基本はサーバーレスで構築し、コスト最小化を優先

この一回の指示で、ほぼ想定どおりの修正版を得られたのは感動ポイントです。

---

## 3. 実装手順（Tasks）
最後に、設計からタスクを自動生成。  
[tasks.md](https://github.com/KennyTakaki/kiro-dev01/blob/main/.kiro/specs/intelligent-stock-recommendation/tasks.md)

驚いたのは、**要件とタスクのトレーサビリティ**が自動で確保されていたことです。  
さらに、「クリティカルパス付きのタスク依存関係図」まで生成してくれたため、進行管理の見通しが格段に向上しました。
これらの情報をグラフ化してください、と頼むとタスク間の依存関係を図示してくれて、より親切な実装手順が作成されました。

---

# まとめと今後
今回、Working Backwardsの考え方をインプットにしてKiroで要件→設計→タスク生成までを試しました。  
結果として、短時間で品質の高いドキュメントを得られ、開発の初動がかなり効率化されることを実感しました。

ただし、外部APIの仕様や運用コスト面での検討が残っているため、実装は一旦保留。  
時間が取れたときに、タスク以降の工程も試してみたいと思います。

---

💡 **ポイント**
- Kiroは「思考→要件→設計→タスク」の流れを一気に作れる
- Working Backwardsの質問はインプットに最適
- 修正依頼の反映が驚くほどスムーズ
