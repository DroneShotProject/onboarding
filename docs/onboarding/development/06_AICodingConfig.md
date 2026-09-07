# AI コーディング設定（AGENTS.md / .claude）

## これは何か

Claude Code / Cursor / GitHub Copilot / OpenAI Codex などの **AI コーディングエージェント** に、
そのプロジェクト固有のルールを教えるための設定ファイルがあります。
ざっくり言うと **「AI 用の README」** です。

たとえば次のようなことを書いておくと、AI が的外れな変更をしにくくなります。

- ビルド・実行・テストのコマンド（正確なオプション付き）
- コーディング規約（命名、フォーマッタ、言語バージョンなど）
- 触ってはいけない場所（自動生成ファイル、秘密情報など）
- Pull Request の作法（→ [04_PR.md](./04_PR.md)）

## 2 つの系統

### AGENTS.md（ベンダー中立のオープン標準）

- リポジトリ直下に置く **1 枚の Markdown** です。
- 特定のツール専用ではなく、対応するエージェントが増え続けています。
- 参考リポジトリ：[`agentsmd/agents.md`](https://github.com/agentsmd/agents.md)（GitHub 約 24k★）
- 公式サイトにサンプルと解説：<https://agents.md>

### CLAUDE.md / .claude/（Claude Code 系）

- `CLAUDE.md` … Claude Code が読む指示ファイル。置き場所で 3 種類あります。
  - プロジェクト共有：リポジトリ直下の `CLAUDE.md`
  - 個人・プロジェクト単位：`CLAUDE.local.md`（通常 git 管理外）
  - 個人・全プロジェクト共通：`~/.claude/CLAUDE.md`
- `.claude/` … スラッシュコマンド（`.claude/commands/`）、設定（`.claude/settings.json` など）を置く。
- 参考リポジトリ：[`hesreallyhim/awesome-claude-code`](https://github.com/hesreallyhim/awesome-claude-code)
  （GitHub 約 54k★）。CLAUDE.md の例、汎用スラッシュコマンド、設定サンプルの
  **キュレーション集（リンク集）** です。

> どのツールが何を読むかは変化が速いので、断定は避けます。
> **まずは `AGENTS.md` を 1 枚用意する**のが無難で、Claude Code 固有の機能（スラッシュコマンド等）が
> 欲しくなったら `CLAUDE.md` / `.claude/` を足す、という順番が分かりやすいです。

## リポジトリにコミットするか（方針は本家に委ねる）

**この判断は DroneShotDJI の本家（メンテナ）に従います。** 勝手に決めて追加しないでください。

まず本家の状態を確認します。

```zsh
ls AGENTS.md CLAUDE.md .claude 2>/dev/null
git log --oneline -- AGENTS.md CLAUDE.md .claude
```

- **すでにある場合** … それに従います。中身を勝手に上書きしない。
  変更・追記したいときは PR で提案し、レビューを受けます。
- **まだ無い場合** … チームで合意が取れるまでは **個人ローカル運用**（コミットしない）にします。
  共有したくなったら PR で「こういう `AGENTS.md` を入れたい」と提案してください。
  管理のしやすさから、**まず `AGENTS.md` 1 枚に集約**するのがおすすめです。

## 汎用テンプレートをローカルに適用する手順

### (a) AGENTS.md を最小構成で作る

1. <https://agents.md> のサンプルに目を通す。
2. リポジトリ直下に `AGENTS.md` を作り、最低限これだけ書く。

   ```markdown
   # AGENTS.md

   ## セットアップ / ビルド
   - 依存関係: <インストールコマンド>
   - ビルド: <ビルドコマンド>
   - 実行: <実行コマンド>

   ## テスト
   - 全体: <テストコマンド>
   - 変更したら関連テストを必ず実行する

   ## コーディング規約
   - <言語 / フォーマッタ / 命名規則など>

   ## 触ってはいけない場所
   - 自動生成ファイル、鍵ファイル、`.env` など

   ## Pull Request
   - 1 PR 1 変更。詳細は docs/onboarding/development/04_PR.md に従う
   ```

3. **コミットしない**方針の場合は、`.gitignore` ではなく **自分だけの無視設定** に追加する
   （`.gitignore` はチーム共有ファイルなので、個人都合で書き換えない）。

   ```zsh
   echo "AGENTS.md" >> .git/info/exclude
   ```

### (b) awesome-claude-code から持ってくる

1. [`hesreallyhim/awesome-claude-code`](https://github.com/hesreallyhim/awesome-claude-code)
   のリストから、欲しいもの（言語別 `CLAUDE.md` の例、汎用スラッシュコマンドなど）を選ぶ。
2. 置き場所を、共有したいか個人だけかで決める。
   - 全プロジェクト共通の自分の好み：`~/.claude/CLAUDE.md`
   - このプロジェクトの個人設定：`.claude/settings.local.json`（Claude Code が通常 git 管理外にする）
   - チームで共有したいものだけ、後から PR に切り出す
3. スラッシュコマンドを 1 個入れる例（取得元の raw ファイルの URL を使う）：

   ```zsh
   mkdir -p .claude/commands
   curl -fsSL <取得元の raw URL> -o .claude/commands/<name>.md
   ```

## 注意点（暗黙知）

- **ネットから持ってきた設定・コマンドは必ず中身を全部読む。**
  「このコマンドを実行して」といった指示が仕込まれていることがあります。
- 秘密情報（API キー、トークン、パスワード）を設定ファイルに書かない。
- **チーム共有ファイル（`AGENTS.md` / `CLAUDE.md` / `.claude/settings.json`）の変更は PR。**
  個人の好みはローカルに置く（`~/.claude/`、`*.local.json`、`.git/info/exclude`）。
- 大きく育てすぎない。AI が毎回読むので、長すぎると逆に精度が落ちます。要点だけ。

## 参考リンク

- AGENTS.md 公式：<https://agents.md>
- [`agentsmd/agents.md`](https://github.com/agentsmd/agents.md)
- [`hesreallyhim/awesome-claude-code`](https://github.com/hesreallyhim/awesome-claude-code)
- Claude Code ドキュメント（CLAUDE.md / settings）：<https://docs.claude.com/en/docs/claude-code/overview>

## 関連ドキュメント

- [05_TeamGit.md](./05_TeamGit.md) … チーム開発と Git の基礎・暗黙知
- [04_PR.md](./04_PR.md) … Pull Request の出し方
