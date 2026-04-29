# memory MCP 調査と myBrain運用との相性判定

> created: 2026-04-29
> tags: #調査 #判断 #保留 #MCP #Obsidian

## 話題

Obsidianの運用調べてたら memory MCP の情報に当たって、詳しく見ておきたくなった。
公式実装(Anthropicリファレンス)とObsidian統合派生がいくつかあるのは知ってたけど、中身ちゃんと読んでなかった。
**いまのmyBrain運用(partnerBrain skill + 日付ノート + Obsidian Vault)とマッチするか/補完になるか**を軽く壁打ちしたかった。

## 結論・方針

**寝かせ案件**。今すぐ導入する必要なし。

- 機能的には、いまのmyBrain運用が memory MCP の **下位互換ではなく上位互換** に近い
- memory MCP は「Vault持ってない人向けの永続記憶」が主用途。自分のセットアップは既にそれを上回る情報密度を持ってる
- ただし「補完になり得るニッチ」は存在するので、調査メモは残して将来の選択肢として置いとく

## 公式memory MCP の正体(整理)

- **データモデル**: 知識グラフ
  - Entity(ノード、type付き)
  - Relation(向き付きエッジ、能動形で記述)
  - Observation(エンティティに紐づく事実1行)
- **保存形式**: デフォルト `memory.jsonl` 単一ファイル(JSON Lines)
- **ツール**: `create_entities` / `create_relations` / `add_observations` / `read_graph` / `search_nodes` / `open_nodes` / `delete_*` の9個前後
- **設定**: `MEMORY_FILE_PATH` 環境変数でパス指定。Claude Desktop/Code両対応

→ そのままだと **Obsidianと無縁**(JSONLなのでグラフビューに乗らない)

## Obsidian統合派生3パターン

| 実装 | 保存形式 | Vault親和性 | 性格 |
|---|---|---|---|
| **obsidian-memory-mcp** (YuNaga224) | エンティティ毎の `.md` + frontmatter, `[[Type::Target]]` リレーション | ◎ Vault直書き可 | 公式のMarkdown化フォーク、最小構成 |
| **Basic Memory** (basicmachines-co) | Markdown + 構造化observation記法 + tags | ○ vault重視 | 知識ベース全体を担う重め設計 |
| **MegaMem** (C-Bjorn) | Graphiti(時系列対応グラフDB) + 11+10ツール | △ 別DB併設 | temporal記憶、リッチだが重い |

`obsidian-memory-mcp` の `[[Manager of::Alice]]` 型typed-wikilinkはObsidianのグラフビューにそのまま乗る設計で、ここは賢い。

## myBrain運用との被り/補完判定

### 被り(注意ポイント)

- partnerBrain summary/details が既に「前提知識の構造化レイヤ」を担ってる
  → memory MCPで同じ粒度の事実を持つと **source of truth分裂**
- 自動生成ノードの粒度がバラついて Vault汚染しがち(`愛媛.md` `Marukin.md` `建材.md` …が乱立する未来が見える)
- 人間キュレーションした濃いpartnerBrainと、機械生成のスカスカmemoryが混ざると劣化

### 補完になり得るレイヤ(確度: 推測)

- **動的ステータス記憶**: マルキン基幹刷新の進捗、ArsMagia実装状況、誰といつ何話した — partnerBrainは静的前提、memory MCPは流れる事実、と分担
- **スキル非対応環境のフォールバック**: モバイル/Web版でpartnerBrainスキルが効かない時、memory MCPは起動するクライアントが多い
- **Claude側書き込み専用**: 人間が書くノートとは物理的に分離(`_memory/` サブディレクトリ等)

## 蹴った選択肢

- **即導入(obsidian-memory-mcp + `_memory/` サブディレクトリ)**
  → 試すコスト自体は低いけど、いま困ってる課題と紐付いてない。「導入してから用途探す」は順序逆。Goldilocks原則的に枠が増えすぎ側に倒れるリスク。
- **Basic Memory採用**
  → 知識ベース全体を担う設計。partnerBrain + 日付ノートが既にあるので **役割衝突**。これに乗り換えるなら全部移行する覚悟必要、それは重い。
- **MegaMem採用**
  → temporal graph はかっこいいけど、Graphiti別途立てる時点で運用継続性が下がる。趣味プロジェクト感強くて見送り。
- **公式memory MCP(JSONL)をそのまま使う**
  → Obsidianグラフに乗らない時点で、myBrain環境に置く意味が薄い。

## 判断軸

- **運用継続性**: 層を増やすと続かない。partnerBrainの整備でメンテ余力使い切ってる感ある
- **source of truth原則**: 同じ情報が二箇所にあると必ず劣化する。役割が完全に分けられないなら統合するな
- **Goldilocks**: 「枠は提示しつつ余白残す」。memory MCPはここで枠の追加側になる
- **困ってから入れる**: 「セッション越しに事実忘れてる」で実害が出てから検討で十分

## モヤモヤ・揺れ

- 「補完になり得るニッチ」(動的ステータス記憶/スキル非対応環境フォールバック) は確かに存在する。完全否定するほどでもない
- 自分が知らない別の用途で刺さる可能性も残ってる。**memory MCPを既に運用してる人の事例**を一度見ておくと判断精度上がりそう
- partnerBrain側に「動的ステータス」を書くノートを追加するだけで、memory MCP不要で同じこと達成できる気もする。そうなるとますます要らない
- スマホ/Web版のフォールバック需要は、実際スキルが効かなくて困った頻度次第。今は分からない

## 気づき・ツッコミ

- memory MCP、「Vault持ってない人に永続記憶を持たせる」のが本来の用途。Obsidian運用してる時点でターゲット層から外れてる説あり
- typed-wikilink(`[[Manager of::Alice]]`)構文は Obsidianのグラフビューにそのまま乗るっていう、地味だけど効く設計。覚えとくと別の場面で使える
- partnerBrainの存在で「memory MCPがやろうとしてること」をほぼカバーできてるのが今回の最大の発見。**スキル整備しておいたのが効いてる**
- Claudeに調査投げる→比較表で返してくる→「で、要るの?」って自分で問い直す、の流れが噛み合った。この壁打ちパターンは今後も使えそう
- 「詳しく調べたい」って言ったけど、実際は「今の運用とマッチするか軽く確認したい」だった。Claude側もそれを汲み取って結論先出ししてきた、その方が自分には刺さる
- memory MCPの「relationを能動形で書く」(`works_at`, `manages` 等)ルール、地味に英語前提な設計。日本語Vaultでやると微妙にダサくなりそう

## 参考リンク

- [Knowledge Graph Memory MCP server (公式)](https://github.com/modelcontextprotocol/servers/tree/main/src/memory)
- [obsidian-memory-mcp (YuNaga224)](https://github.com/YuNaga224/obsidian-memory-mcp)
- [Basic Memory (basicmachines-co)](https://github.com/basicmachines-co/basic-memory)
- [MegaMem (C-Bjorn)](https://github.com/C-Bjorn/MegaMem)
- [obsidian-mcp-plugin (aaronsb)](https://github.com/aaronsb/obsidian-mcp-plugin)

## 関連リンク候補

[[2026-04-29-partnerBrain構築とスキル整備]]
[[2026-04-29-obsidian-brainプラグイン化とCowork対応]]
[[2026-04-29-claude-obsidian社内展開構想]]
[[要検討-動的ステータスノートの位置づけ]]
