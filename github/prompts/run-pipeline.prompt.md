---
agent: "Pipeline Orchestrator"
description: "1画面/1共通部品の基本設計MD変換パイプラインを DAG に従って自動実行する。unit を指定して起動する。"
---

`08_編排/manifests/${input:unit}.yaml` を manifest として読み込み、
`pipeline-orchestrator` の実行アルゴリズムに従ってパイプラインを自動実行してください。

- DAG（jobs / depends_on）を解決し、依存が満たされた Job を **S-00 から順に逐次実行** する。
- 各 Job は **Pipeline Step Runner サブエージェントへ委譲**して生成する（Job ごとに新規起動＝コンテキスト隔離）。
- 生成後、`gate` Task（確定的門禁）を実行し、`semantic_review` 対象は **Design QA Reviewer** で語義検証する。
- すべて合格したら `git commit` で検査点を打つ。NG なら `git reset --hard` で回滚し、`max_retries` までリトライする。
- リトライ上限超過時は人間へエスカレーションして停止する。
- 全 Job 完了で `_run_ledger.json` を出力し、結果を報告する。

開始前に必ず `git status` が clean であることを確認してください。
