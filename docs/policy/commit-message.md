# コミットメッセージ方針

## 目的
コミットメッセージの型（type）と書式を固定し、変更履歴の可読性と検索性を高める。

## 書式
```
type(scope): summary
```

- `type`: 変更の主目的（必須）
- `scope`: 影響範囲（任意、推奨）
- `summary`: 短い要約（必須、日本語可）

## 種別
一般的な慣習に合わせ、追加は `feat` / `docs` に含める。`add` は使わない。

- `feat` 新機能・挙動追加
- `fix` バグ修正
- `docs` ドキュメント
- `refactor` 仕様変更なしの整理
- `perf` 速度/省メモリ改善
- `test` テスト
- `build` ビルド/依存関係
- `ci` CI設定
- `chore` 雑務（ビルド以外）
- `revert` 取り消し

## 範囲
- 基本は 1 つに絞る（一般的な慣習）。
- 複数スコープは使わない。

例: `godot`, `data`, `sim`, `tools`, `docs`, `build`

## 要約
- 日本語OK。
- 命令形または体言止めで短く書く。

## 種別の優先順位
主目的で選ぶ。

1. `fix`
2. `feat`
3. `perf`
4. `refactor`
5. `docs`
6. `test`
7. `build`
8. `ci`
9. `chore`
10. `revert` （特別扱い。単独で使う）

## 例
```
feat(godot): バトル開始時のドロー処理を追加
fix(data): レア度テーブルの重複を修正
docs: 仕様概要を追加
refactor(sim): ターン処理を関数分割
```
