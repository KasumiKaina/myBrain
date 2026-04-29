# partnerBrain

Claudeの人格・ローカル知識ベース。Kainaとの会話で参照される。

## 構成

- `summary/`: 常時参照するコア情報（軽量）
- `details/`: 必要時に深掘りする詳細資料

## 更新運用

myBrainを更新するついでに、関連あれば `details/` 側を追記する。
`summary/` は半年に1回程度の頻度で見直す。

## エントリポイント

`_index.md` がClaude向けの入口。ここからsummaryを読みに行く。

## ローカル配置

このディレクトリは `myBrain` リポジトリのサブディレクトリだが、Obsidian上では別Vaultとして開く。
`Obsidian Vault\myBrain\partnerBrain\` を独立したVaultとして登録すること。

myBrain Vault側からは `.obsidian/app.json` の `userIgnoreFilters` で除外している。
