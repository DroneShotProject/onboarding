# Android Studio

Android Studio のセットアップは、Windows では必須、macOS では任意です。

## [JetBrains Toolbox](https://www.jetbrains.com/ja-jp/toolbox-app/)をインストール

Android Studio をダウンロード・インストールするためのアプリケーションです。

## Android Studio をインストール

JetBrains Toolbox から、Android Studio の最新安定版をダウンロードしてインストールします。

## Android Studio をセットアップ

- More Actions → SDK Manager → SDK Tools タブ → Android SDK Command-Line Tools (Latest) をチェック → Apply
- [Windows] More Actions → Virtual Device Maneger → Emulator 作成
  - 新しいデバイスを選択
  - Show Advanced Settings から Internal storage を 2048 から 4096 くらいにしておくことをおすすめ  ## 1. JDK（Java開発キット）のセットアップ
  
  ### JDK をインストール
  
  - Oracle JDK 17 または Eclipse Temurin JDK 17 をダウンロード
  - インストーラーを実行してインストール
    - デフォルト：`C:\Program Files\Java\jdk-17.x.x`
  
  ### 環境変数を設定
  
  1. `Win + X` → `システム` を開く
  2. `システムの詳細設定` → `環境変数`
  3. `新規` → システム環境変数を追加
     - 変数名：`JAVA_HOME`
     - 変数値：`C:\Program Files\Java\jdk-17.x.x`（インストールパスに合わせる）
  4. `Path` に `"C:\Program Files\Java\jdk-17.0.18\bin"` を追加
  5. `OK` で完了
  6. PC を再起動
  