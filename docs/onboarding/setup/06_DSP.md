# DSP

リポジトリをクローンします。

`C:\Users\<Username>\dev\github` の下に作業用のディレクトリを用意し、そこに clone するのがベストプラクティスです。

```zsh
mkdir -p C:\Users\<Username>\dev\github
cd C:\Users\<Username>\dev\github
git clone git@github.com:DroneShotProject/DroneShotDJI.git
```

## Android Studio でのビルド手順

Android Studio で本リポジトリを開き、JDK 17 を Gradle に割り当てて、必要に応じて `release` ビルドで APK を生成し、実機へインストールします。

### 前提条件

- Android Studio（最新の安定版推奨）
- Git
- GitHub SSH アクセス（`DroneShotProject/DroneShotDJI` を `git@github.com:...` で clone できること）

### 1. プロジェクトを開く

1. Android Studio を起動する。
2. **Open** でリポジトリ内の `frs-dsp-as` フォルダを開く。ここが Gradle のプロジェクトルートです。
3. 初回に「信頼できるフォルダ」などのダイアログが出た場合は、**DroneShotDJI を信頼**して進める。

初回同期で Android SDK や Build Tools が不足している場合は、案内に従い SDK Manager でインストールしてください。`frs-dsp-as/local.properties` に SDK パスが書き込まれるのが一般的です（コミット対象外）。

### 2. Gradle 用 JDK 17 の設定

プロジェクトは Java 17 互換です。Android Studio の Gradle が別バージョンを参照している場合は、次の手順で 17 を選択します。

1. メインメニュー **File → Settings**（macOS では **Android Studio → Settings…**）。
2. **Build, Execution, Deployment → Build Tools → Gradle** を開く。
3. **Gradle JDK** で **Download JDK…** を選ぶ。
4. **Version 17** を選び、ダウンロードして OK で確定する。
5. 反映されない場合は Android Studio を再起動する。

ここまでの Gradle JDK 設定は Android Studio から Gradle を動かすときのものです。ターミナルで `./gradlew` を実行する場合は、シェル側でも Java 17 を使えるように `JAVA_HOME` を設定する必要があります。

### 3. ビルドバリアントを release にする

Debug ビルドには LeakCanary などデバッグ専用依存が含まれるため、通常の利用や端末へのインストールは `release` を選びます。

1. メインメニュー **Build → Select Build Variant…**。
2. `:dsp` モジュールの **Active Build Variant** を **release** にする。

### 4. APK の生成

1. メインメニュー **Build → Generate App Bundles or APKs → Generate APKs**。
2. 完了後、APK は通常次の場所に出力されます。

`frs-dsp-v1-app/build/outputs/apk/release/`

実際のファイル名はビルド設定により異なる場合があります。

### 5. 実機へのインストール

1. USB ケーブルで端末を接続する。
2. 端末側で開発者向けオプションと USB デバッグを有効にする。
3. Android Studio のツールバーから端末を選択し、**Run ‘dsp’**（緑の実行ボタン）を押す。

### 6. コマンドラインから release APK を作る場合

`./gradlew` は Android Studio の Gradle JDK 設定を使いません。`JAVA_HOME` が 11 のままだと `Android Gradle plugin requires Java 17` で失敗することがあります。

#### 例: Android Studio でダウンロードした JDK を使う

```bash
cd frs-dsp-as
export JAVA_HOME="$HOME/.jdks/<jdk-17-directory>"
./gradlew :dsp:assembleRelease
```

実際のディレクトリ名は環境によって異なります。

#### 例: Ubuntu で OpenJDK 17 を使う

```bash
sudo apt install openjdk-17-jdk
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
cd frs-dsp-as
./gradlew :dsp:assembleRelease
```

生成物は `frs-dsp-v1-app/build/outputs/apk/release/` 配下を確認してください。

### 7. 実機接続時の注意

- 端末の開発者向けオプションを有効にする。
- データ転送対応の USB ケーブルを使う（充電専用ケーブルでは認識されない場合があります）。
- 端末で USB デバッグの許可を出す。

### 8. うまくいかないとき

- Gradle 同期エラー: JDK が 17 になっているか、SDK / NDK の不足メッセージを確認する。
- `Android Gradle plugin requires Java 17` / `You are currently using Java 11`（ターミナル）: Android Studio 側は 17 でも、シェルの `JAVA_HOME` が 11 のままの可能性があります。
- 署名エラー: `frs-dsp-as/gradle.properties` や `frs-dsp-v1-app/build.gradle` の署名設定がローカル環境に適合しているか確認してください。

