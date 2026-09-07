# AGENTS.md

AI コーディングエージェント（Claude Code / Cursor / Codex など）と人間の貢献者が、
このリポジトリで作業するときの共通ルール。

## このリポジトリについて

- [DroneShotProject/DroneShotDJI](https://github.com/DroneShotProject/DroneShotDJI)
  開発に参加するための **日本語のオンボーディング手順書**。コードはなくドキュメントのみ。
- ドキュメントの場所と番号付け:
  - `docs/onboarding/setup/` … 環境構築（`01_`〜）
  - `docs/onboarding/development/` … 開発フロー（`01_`〜）
- ルート `README.md` に両ディレクトリの目次テーブルがある。

## ブランチとコミット

- **作業は必ず `main` から新しいブランチを切って行う。** `main` に直接コミットしない。
- ブランチ名は `feature/` `fix/` `docs/` `issue/` 接頭辞 + 小文字ハイフン区切り
  （詳細は `docs/onboarding/development/02_Branch.md`）。
- **コミットは論理的変更ごとに最小単位で分ける。** 無関係な変更を混ぜない。
- コミットメッセージは日本語。`Add` / `Update` / `Fix` / `Refactor` などで始めるか
  「〜を追加」「〜を修正」形式（詳細は `docs/onboarding/development/03_Commit.md`）。
- コミット前に必ず `git status` と `git diff` を確認する。
- すでに push 済みのコミットは原則 amend / rebase しない。

## Pull Request

- **PR の base は必ず `main`。** 他のトピックブランチを base にしたスタック PR を作らない
  （base ブランチだけにマージされて `main` に届かない事故を防ぐため）。
- 依存する変更がある場合は、先行 PR が `main` にマージされてから次の PR を出す。
- 1 PR = 1 論理変更。小さく保つ（詳細は `docs/onboarding/development/04_PR.md`）。
- ドキュメントを追加・リネーム・削除したら、`README.md` の目次テーブルも同じ PR で更新する。
- push / PR 作成は、ユーザーから明示的に指示されたときだけ行う。

## ドキュメントの書き方

- 見出しは `#` タイトル → `##` セクション。文体・粒度は既存ファイル（特に
  `docs/onboarding/development/02_Branch.md` / `04_PR.md`）に合わせる。
- 読者はプログラミング初心者。専門用語には短い言い換えや例えを添える。
- **コマンドは必ずフェンス付きコードブロックに入れる。** 実行コマンドは ```zsh、
  出力例は ```text を基本にする。
- OS で分岐する手順は見出しに `[Windows]` / `[macOS]` を付ける。付いていない箇所は共通。
- ユーザーが書き換える値は `<...>` で示し、「`<>` は消して実行する」と注記する。
- ドキュメント間リンクは相対パス（例: `./03_Commit.md`）。追加時にリンク切れがないか確認する。

## やってはいけないこと

- 秘密情報（API キー、トークン、`.env`、鍵ファイル）をコミットしない。
- `.gitignore` を個人都合で書き換えない（個人的な無視は `.git/info/exclude`）。
- `git push --force` は自分の作業ブランチのみ。`main` や共有ブランチには行わない。
- 既存ドキュメントの番号を、必要なく振り直さない。
