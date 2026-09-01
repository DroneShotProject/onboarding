# Android Studio

Android Studio のセットアップは、Windows では必須、macOS では任意です。
macOS で **コマンドライン優先** かつ **エミュレータでビルド・起動できれば十分** な場合は、
Android Studio（GUI）を入れずに SDK コマンドラインツールだけで完結できます。
DroneShotDJI の macOS 向け詳細手順は
[DroneShotDJI/docs/mac-setup-guide.md](https://github.com/DroneShotProject/DroneShotDJI/blob/main/docs/mac-setup-guide.md)
を参照してください。

## [macOS] インストールとセットアップ

### JDK 17

本プロジェクトは **Java 17**（AGP 8.7 / `sourceCompatibility = 17`）です。
`brew` の **formula**（`--cask` ではない）なら `sudo` 不要で入ります。

```zsh
% brew install openjdk@17

# ターミナルから ./gradlew を実行するために JAVA_HOME を通す（formula は symlink されないのでパス直指定）
% echo 'export JAVA_HOME="/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home"' >> ~/.zshrc
% export JAVA_HOME="/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home"
% "$JAVA_HOME/bin/java" -version   # 17.x を確認
```

> GUI 用に Temurin を使う場合は `brew install --cask temurin@17`（`/Library` へ入れるため
> インストール時にパスワードを要求されます）。その場合の `JAVA_HOME` は
> `export JAVA_HOME="$(/usr/libexec/java_home -v 17)"` で取得できます。

### Android SDK（コマンドラインツールのみ）

```zsh
% brew install --cask android-commandlinetools

% cat >> ~/.zshrc <<'EOF'
export ANDROID_HOME="/opt/homebrew/share/android-commandlinetools"
export ANDROID_SDK_ROOT="$ANDROID_HOME"
export PATH="$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator:$ANDROID_HOME/cmdline-tools/latest/bin:$JAVA_HOME/bin:$PATH"
EOF
% source ~/.zshrc

% yes | sdkmanager --licenses
% sdkmanager --install "platform-tools" "platforms;android-35" "build-tools;35.0.0" \
    "emulator" "system-images;android-35;google_apis;arm64-v8a"
```

> `sdkmanager` / `avdmanager` は PATH に Java が無いと `Unable to locate a Java Runtime` で
> 無言終了します。上の PATH に `$JAVA_HOME/bin` を含めてください。

> DroneShotDJI は `abiFilters 'arm64-v8a'` のため、エミュレータは **arm64-v8a** イメージを使います
> （Apple Silicon ではネイティブ動作）。x86/x86_64 イメージだと `INSTALL_FAILED_NO_MATCHING_ABIS` になります。

### [任意] Android Studio（GUI）

SDK Manager / AVD Manager / デバッガを GUI で使いたい場合:

```zsh
% brew install --cask android-studio
```

Android Studio から Gradle を動かす場合は
**Settings → Build, Execution, Deployment → Build Tools → Gradle → Gradle JDK** も **17** に設定します
（この設定はターミナルの `./gradlew` には影響しません）。

## [Windows] [JetBrains Toolbox](https://www.jetbrains.com/ja-jp/toolbox-app/)をインストール

Android Studio をダウンロード・インストールするためのアプリケーションです。

## [Windows] Android Studio をインストール

JetBrains Toolbox から、Android Studio の最新安定版をダウンロードしてインストールします。

## [Windows] Android Studio をセットアップ
  
  ### JDK をインストール
  
  - Oracle JDK 17 または Eclipse Temurin JDK 17 をダウンロード
  - [JDK 17](https://drive.google.com/file/d/18R8wvNg1xR7yf5MYQSu3_LiE5H9wzh0z/view?usp=sharing)
  - インストーラーを実行してインストール
    - デフォルト：`C:\Program Files\Java\jdk-17.x.x`
  
  ### 環境変数を設定
  
  1. `Win + X` → `システム` を開く、もしくは`検索` → `env`
  2. `システムの詳細設定` → `環境変数`
  3. `新規` → システム環境変数を追加
     - 変数名：`JAVA_HOME`
     - 変数値：`C:\Program Files\Java\jdk-17.x.x`（インストールパスに合わせる）
  4. `Path` に `"C:\Program Files\Java\jdk-17.0.18\bin"` を追加
  5. `OK` で完了
  6. PC を再起動
  