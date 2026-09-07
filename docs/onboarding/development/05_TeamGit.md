# チーム開発と Git

## このドキュメントのゴール

Git の個々のコマンド（[02_Branch.md](./02_Branch.md) / [03_Commit.md](./03_Commit.md) /
[04_PR.md](./04_PR.md)）の前に、**なぜそのやり方をするのか**という考え方と、
チームで共有されている「暗黙のルール（暗黙知）」をまとめます。

1 人で開発しているときは、`main` に直接コミットしても困りません。
しかしチームでは、全員が同じ `main` を共有します。ここに「お作法」が必要になります。
初めての人ほど、まずここを読んでおくと後で楽になります。

## なぜ `main` を直接触らないのか（いちばん大事）

- `main` はチームの **「正（せい）」** です。全員がここを基準にコードを読み、
  ビルドやデプロイもここから行います。壊れていると全員が止まります。
- 複数人が同時に `main` を書き換えると、あとで変更を **統合（merge / マージ）** するときに
  **衝突（コンフリクト）** が起きます。同じ行を別々に直していると、Git は
  「どちらが正しいか」を判断できず、人間が手で解決することになります。
- さらに、`main` がどんどん進んでいるのに、自分だけ **古い `main`** の上で何日も作業を続けると、
  最後にまとめて合流させるときに大量の衝突が一気に出ます。
  イメージとしては、**締め切り直前に全員分の変更をいっぺんに突き合わせる**ようなもので、
  非常に大変です。

> 対策はシンプルです。**`main` から作業用のブランチを切って、そこで作業する。
> そして `main` の更新はこまめに取り込む。** これだけで上のトラブルはほぼ防げます。

## 作業の 2 択：ブランチ or フォーク

`main` を汚さずに作業する方法は、大きく 2 つです。

### A. 共有リポジトリでブランチを切る（DroneShotProject 推奨）

- 前提：`DroneShotProject/DroneShotDJI` に **直接 push できる権限** がある（メンバーに追加済み）。
- このとき `origin` は **本家リポジトリそのもの** を指します。
- 作業ごとに `main` からブランチを切り、そのブランチを push して Pull Request を出します。

### B. フォークして作業する

- push 権限がない場合や、OSS のように「まず自分のコピーで作業したい」場合はこちら。
- 自分の GitHub アカウントに **フォーク（コピー）** を作ります。
- このとき `origin` は **自分のフォーク**、本家は `upstream`（アップストリーム）という名前で追加します。
- 作業手順の違いは後述の「フォーク運用の場合の差分」を参照してください。

いま自分の `origin` がどちらを指しているかは、次のコマンドで確認できます。

```zsh
git remote -v
```

`origin ... DroneShotProject/DroneShotDJI` なら A、`origin ... <自分のユーザー名>/DroneShotDJI`
なら B です。

## ブランチ戦略：`main` を汚さない

考え方は 1 つだけです。

- **自分のローカルの `main` は自分で編集しない。** 常に本家 `main` の「鏡（ミラー）」として保つ。
- 何か作業するときは、**必ず `main` から新しいブランチを切って**、その作業ブランチの上だけで
  コミットする。

```text
main    ──●──●──●──●──●        ← ここは常に本家のコピー。直接コミットしない
             \        ↑
              ●──●──●  merge   ← feature/xxx で作業して、PR 経由で main に戻す
              feature/xxx
```

作業ブランチは「使い捨て」です。PR がマージされたら消して、また `main` から切り直します。

## 基本サイクル（共有リポジトリ版）

### 1. 作業を始める前に `main` を最新にする

これは毎回やります。**朝に顔を洗うのと同じ**くらい、当たり前の最初の一歩です。

```zsh
git checkout main
git pull origin main
```

### 2. `main` からブランチを切る

ブランチ名の付け方は [02_Branch.md](./02_Branch.md) を参照してください。

```zsh
git checkout -b feature/task-name
```

### 3. 作業して、こまめにコミットする

コミットの粒度やメッセージの書き方は [03_Commit.md](./03_Commit.md) を参照してください。

### 4. push して Pull Request を出す

```zsh
git push origin feature/task-name
```

このあとの PR 作成手順は [04_PR.md](./04_PR.md) を参照してください。

## 作業中に `main` が進んだら：取り込み方

自分が作業している間に、他の人の PR がマージされて `main` が進むことがあります。
**早めに・小さく取り込むほど、衝突は小さくて済みます。** 数日に 1 回ではなく、
気づいたら取り込む、くらいの感覚で構いません。

### 方法 1：merge（初心者はまずこれ）

```zsh
git checkout main
git pull origin main
git checkout feature/task-name
git merge main
```

安全で分かりやすい方法です。取り込んだ記録が «Merge branch 'main' into ...» という
コミットとして残りますが、チーム開発では問題ありません。

### 方法 2：rebase（履歴をきれいに保ちたい人向け・参考）

```zsh
git checkout feature/task-name
git fetch origin
git rebase origin/main
```

コミット履歴が一直線になって見やすくなりますが、**すでに push 済みのブランチを rebase すると
`git push --force`（正確には `--force-with-lease`）が必要**になります。
仕組みが分かるまでは方法 1 で十分です。

### 衝突（コンフリクト）が出たとき

慌てないでください。衝突は「事故」ではなく **日常** です。

1. `git status` で、衝突しているファイルを確認する。
2. そのファイルを開くと、次のような印が入っています。

   ```text
   <<<<<<< HEAD
   自分の変更
   =======
   取り込もうとしている側の変更
   >>>>>>> main
   ```

3. `<<<<<<<` `=======` `>>>>>>>` の行ごと消して、**正しい最終形** に手で書き直す。
4. 直したファイルを `git add` する。
5. merge なら `git commit`、rebase なら `git rebase --continue` で続行する。

判断に迷ったら、抱え込まずにレビュアーやチームに聞いてください。
「どっちを残すべきか分からない」は、遠慮なく聞いていい質問です。

## フォーク運用の場合の差分

フォークで作業する場合（前述の B）は、最初に一度だけ本家を `upstream` として登録します。

```zsh
git remote add upstream git@github.com:DroneShotProject/DroneShotDJI.git
git remote -v
```

作業を始める前に、**自分のフォークの `main`** を本家に追従させます。

```zsh
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

そのあとは共有リポジトリ版と同じで、`main` からブランチを切って作業し、
`git push origin <ブランチ名>` してから、**フォーク → 本家** への Pull Request を出します。

## チームの暗黙知（明文化）

言葉にされていないけれど、チームで共有されている前提です。

- **作業開始の第一歩は必ず `git pull origin main`。** 古い `main` の上で作業を始めない。
- **ブランチは 1 タスク 1 本。** 長生きさせない（目安：数日以内に PR まで持っていく）。
- **`main` に直接 commit / push しない。** `git push origin main` は打たない。
- **`git push --force` は自分の作業ブランチだけ。** 共有ブランチや `main` には絶対にしない。
- `git pull` する前に、自分の変更は commit するか `git stash` で退避しておく。
- **コミット前に `git status` と `git diff` を必ず見る**（[03_Commit.md](./03_Commit.md) の再掲）。
- 他人のブランチに勝手に push しない。手伝うときも一声かける。
- 大きい変更ほど、早めに **Draft（下書き）PR** を作って方向性を共有する。
  「全部完成してから見せる」は、手戻りが大きくなりがち。
- 鍵ファイル・`.env`・ビルド生成物はコミットしない（`.gitignore` を確認する）。
- コンフリクトは日常。**早め・小口** で取り込んで潰す。

## 困ったときのコマンド集

- 変更を一時退避する / 戻す：

  ```zsh
  git stash
  git stash pop
  ```

- 「さっきの操作を取り消したい」…まず落ち着いて、戻り先を探す：

  ```zsh
  git reflog
  ```

- ローカルの変更を **全部捨てて** `main` をリモートと同じ状態に戻す（**不可逆・要注意**）：

  ```zsh
  git checkout main
  git reset --hard origin/main
  ```

- 今どのブランチにいる？ 何が変わってる？：

  ```zsh
  git branch
  git status
  ```

## 関連ドキュメント

- [02_Branch.md](./02_Branch.md) … ブランチの命名規則と作成手順
- [03_Commit.md](./03_Commit.md) … コミットの粒度とメッセージ
- [04_PR.md](./04_PR.md) … Pull Request の出し方とレビュー
- [06_AICodingConfig.md](./06_AICodingConfig.md) … AI コーディング設定（AGENTS.md / .claude）
