# ターミナル

## [macOS] ターミナル

標準の **ターミナル.app**（`アプリケーション → ユーティリティ → ターミナル`）を使います。
シェルは **zsh** がデフォルトです。以降の手順で使うパッケージマネージャ **Homebrew** を入れておきます。

```zsh
# コマンドラインツール（git, clang など）
xcode-select --install

# Homebrew（未導入の場合）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# brew を PATH に通す（Apple Silicon）
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

以降 `% ...` は macOS のターミナル、`> ...` は Windows の PowerShell を表します。

## [Windows] Windows ターミナル をインストール

[Microsoft Store](https://apps.microsoft.com/detail/9n0dx20hk701?hl=ja-JP&gl=JP) からインストールします。

## PowerShell をアップデート

Windows ターミナルでは、Powershell を使用します。デフォルトでインストールされている Powershell は最新版でないため、アップデートします。

既存の PowerShell をアンインストールします。

```
winget uninstall PowerShell
```

新規 PowerShell をインストールします。

```
winget install --id Microsoft.Powershell --source winget
```