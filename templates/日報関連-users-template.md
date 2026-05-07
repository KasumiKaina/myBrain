# Marukin日報 ユーザマッピング

> updated: {YYYY-MM-DD}

このファイルは skill による本人同定に使われる正本データ。
配布対象社員の email と 漢字フルネーム(=Drive上の個人フォルダ名)の対応を管理する。

## 配布対象社員

| email | 漢字フルネーム | 役職 | 上司email | 配布日 | 備考 |
|---|---|---|---|---|---|
| user1@marukin-group.com | {漢字フルネームA} | {役職A} | {上司email} | 2026-05-MM | Phase 0a配布 |
| user2@marukin-group.com | {漢字フルネームB} | {役職B} | {上司email} | 2026-05-MM | Phase 0a配布 |

## IT管理者

| email | 漢字フルネーム | 役職 |
|---|---|---|
| k.torigoe@marukin-group.com | 鳥越花南 | IT管理者 |

## 運用ルール

### 新規追加時

1. 「配布対象社員」表に1行追加
2. Drive ACL で `{漢字フルネーム}\` フォルダに本人アクセス権付与
3. `{漢字フルネーム}\partnerBrain\` 雛形配置(`templates\partnerBrain-shared\` からコピー)
4. partnerBrain の基礎情報をKainaが手動で埋める
5. `updated:` フィールドを更新

### 退職・配布対象外時

1. 「配布対象社員」表から削除
2. 個人フォルダはアーカイブ(削除しない、別領域に移動)
3. `updated:` フィールドを更新

### 命名規則

- 漢字フルネームは Drive 上の個人フォルダ名と**完全一致**(skillはこの文字列でフォルダパスを組み立てる)
- 同姓出現時は「{姓}{名}」のフルネーム形式で衝突回避
- 上司email 列は Phase 1以降の上司閲覧範囲設計で参照(現時点未使用)

## skill側の参照仕様

- skill は起動時に Cowork session env から `user.email` を取得
- 「配布対象社員」表を email 列でルックアップ
- 一致する行の「漢字フルネーム」をフォルダパスに組み込み
- 一致しない場合は「IT管理者」表もチェック(Kaina自身が試運転する場合)
- どちらにも一致しない場合は「初期化されてないユーザ」として fallback フローへ

## 改訂履歴

- {YYYY-MM-DD}: 初版作成 (Phase 0a配布対象2名)
