# DSP

リポジトリをクローンします。作業用ディレクトリの下に clone するのがベストプラクティスです。

### [macOS]

```zsh
% mkdir -p ~/dev/github
% cd ~/dev/github
% git clone git@github.com:DroneShotProject/DroneShotDJI.git
```

### [Windows]

`C:\Users\<Username>\dev\github` の下に作業用のディレクトリを用意します。

```powershell
> mkdir -p C:\Users\<Username>\dev\github
> cd C:\Users\<Username>\dev\github
> git clone git@github.com:DroneShotProject/DroneShotDJI.git
```

## [macOS] エミュレータでビルド・起動する（実機なし）

実機がなくても、Android エミュレータでビルドが通ることを確認できます。
前提は [04_AndroidStudio.md](04_AndroidStudio.md) の **[macOS] インストールとセットアップ**
（JDK 17 / `JAVA_HOME` / SDK コマンドラインツール / `ANDROID_HOME`）が済んでいること。
詳細は
[DroneShotDJI/docs/mac-setup-guide.md](https://github.com/DroneShotProject/DroneShotDJI/blob/main/docs/mac-setup-guide.md)
にあります。

本アプリは **タブレット・横画面** 想定なので、タブレットプロファイルで AVD を作ります。

```zsh
% cd ~/dev/github/DroneShotDJI
% echo "sdk.dir=$ANDROID_HOME" > frs-dsp-as/local.properties

# エミュレータ（AVD）を作成（devices.xml の Error が出ても AVD は作成されます）
% echo no | avdmanager create avd -n dsp_tablet \
    -k "system-images;android-35;google_apis;arm64-v8a" -d pixel_tablet
% printf 'hw.initialOrientation=landscape\nhw.keyboard=yes\nhw.ramSize=4096\n' \
    >> ~/.android/avd/dsp_tablet.avd/config.ini

# 起動
% emulator -avd dsp_tablet -no-snapshot -no-boot-anim -gpu swiftshader_indirect &
% adb wait-for-device
% adb shell 'while [[ "$(getprop sys.boot_completed)" != "1" ]]; do sleep 1; done'

# ビルド → エミュレータへインストール → 起動
% cd frs-dsp-as
% ./gradlew :dsp:assembleDevDebug
% ./gradlew :dsp:installDevDebug
# debug には LeakCanary のランチャーも入るため MainActivity を明示的に起動
% adb shell am start -n com.frs.dsp/.MainActivity
```

- flavor は `dev` / `prod`、ビルドタイプは `debug` / `release`。エミュレータ確認は `devDebug` で十分です。
- `debug` もリポジトリ同梱のキーストアで署名されるため、追加の署名設定は不要です。
- APK 出力先: `frs-dsp-v1-app/build/outputs/apk/dev/debug/`
- `INSTALL_FAILED_NO_MATCHING_ABIS` が出たら、AVD が x86/x86_64 イメージです。
  本アプリは `abiFilters 'arm64-v8a'` なので **arm64-v8a** イメージで作り直してください。
- 起動確認: `adb shell dumpsys window | grep mCurrentFocus` に
  `com.frs.dsp/com.frs.dsp.home.HomeActivity` が出ればホーム画面まで到達しています。

## 実機をつないで VS Code のターミナルだけでビルド・インストールする

Android Studio（GUI）を使わず、**スマホ／タブレットの実機を PC につなぎ、VS Code の
ターミナルにコマンドを打つだけ**でアプリをビルドして実機にインストールする方法です。
プログラミングが初めての人向けに、順番どおりに進めれば終わるように書いています。

前提:

- [05_Git.md](05_Git.md) までが終わっていて、`DroneShotDJI` をクローン済み。
- [04_AndroidStudio.md](04_AndroidStudio.md) の JDK 17 と Android SDK（`adb` を含む
  `platform-tools`）のセットアップが済んでいる。ターミナルで `adb --version` が
  エラーなく表示されれば準備 OK。
- 実機は **Android 11 以降**。本アプリは **タブレット・横画面** 想定です
  （スマホでも動きますが、画面レイアウトは横画面タブレット向けです）。

> `adb`（Android Debug Bridge）は、PC から実機やエミュレータを操作するためのコマンドです。

### 1. VS Code でプロジェクトを開き、ターミナルを出す

1. VS Code のメニュー **ファイル → フォルダーを開く…**（macOS は **File → Open Folder…**）で、
   クローンした `DroneShotDJI` フォルダを開く。
2. メニュー **ターミナル → 新しいターミナル**（**Terminal → New Terminal**）を選ぶ。
   日本語キーボードのショートカットは `Ctrl + Shift + @`。
3. 画面下部にターミナルが開きます。以降のコマンドはすべてここに 1 行ずつ貼り付けて Enter します。

### 2. 実機側で「USB デバッグ」をオンにする（初回だけ）

メーカーによってメニュー名が少し違いますが、だいたい次の流れです。

1. 実機の **設定 → デバイス情報** を開き、**ビルド番号** を **7 回連続でタップ**する。
   「これでデベロッパーになりました」と出れば成功。
2. **設定 → システム → 開発者向けオプション** を開く。
3. **USB デバッグ** をオンにする。
4. ワイヤレス接続を使うなら **ワイヤレスデバッグ** もオンにする。

### 3. 実機を PC につなぐ

USB（かんたん・確実）とワイヤレスの 2 通りがあります。**まずは USB を試す**のがおすすめです。

#### 3-A. USB でつなぐ

1. **データ転送に対応した** USB ケーブルで実機と PC をつなぐ
   （充電専用ケーブルだと認識されません）。
2. 実機に「USB デバッグを許可しますか？」というダイアログが出たら、
   **「このコンピュータからのUSBデバッグを常に許可する」** にチェックして **許可** を押す。
3. ターミナルで接続を確認する。

   ```zsh
   adb devices
   ```

4. 次のように `device` と表示されれば接続 OK です。

   ```text
   List of devices attached
   R5CT30XXXXX     device
   ```

- 何も表示されない → ケーブルや USB ポートを替える。実機の画面ロックを解除する。
- `unauthorized` と表示される → 実機の許可ダイアログを見落としています。ケーブルを挿し直し、
  ダイアログで **許可** を押す。直らなければ次を実行してから `adb devices` をやり直す。

  ```zsh
  adb kill-server
  adb start-server
  ```

#### 3-B. ワイヤレス（Wi-Fi）でつなぐ

PC と実機を **同じ Wi-Fi** につないでおきます（Android 11 以降）。

1. 実機で **開発者向けオプション → ワイヤレスデバッグ** を開き、オンにする。
2. **「ペア設定コードによるデバイスのペア設定」** をタップする。
   `192.168.x.x:xxxxx` という **IP アドレス:ポート** と、**6 桁のペア設定コード** が表示される。
3. ターミナルでペア設定する（数字は実機の画面に合わせて書き換える）。

   ```zsh
   adb pair 192.168.x.x:xxxxx
   ```

   `Enter pairing code:` と聞かれたら実機に出ている 6 桁を入力する。
   `Successfully paired` と出れば成功。

4. ワイヤレスデバッグ画面の **上部に表示されている** `IP アドレスとポート`
   （ペア設定用とは **別のポート番号**）を使って接続する。

   ```zsh
   adb connect 192.168.x.x:yyyyy
   ```

   `connected to 192.168.x.x:yyyyy` と出れば OK。`adb devices` にも表示されます。

> 一度 USB でつないでから Wi-Fi に切り替えるだけなら、USB 接続中に `adb tcpip 5555` を実行し、
> ケーブルを抜いてから `adb connect <実機のIP>:5555` でもつなげます。
> 実機の IP は **設定 → デバイス情報 → 端末の状態** で確認できます。

### 4. SDK のパスをプロジェクトに教える

`frs-dsp-as/local.properties` に Android SDK の場所を書きます（このファイルは Git 管理対象外）。
**Android Studio で一度でもこのプロジェクトを開いたことがあれば作成済み**なので、その場合は
この手順を飛ばして構いません。

macOS / Linux:

```zsh
cd ~/dev/github/DroneShotDJI
echo "sdk.dir=$ANDROID_HOME" > frs-dsp-as/local.properties
```

Windows で手動作成する場合は、`frs-dsp-as/local.properties` を新規作成し、次の 1 行だけ書きます
（区切りは `\` ではなく `/` でよい。ユーザー名は自分の環境に合わせる）。

```text
sdk.dir=C:/Users/<ユーザー名>/AppData/Local/Android/Sdk
```

### 5. ビルドして実機にインストールし、起動する

macOS / Linux:

```zsh
cd frs-dsp-as
./gradlew :dsp:assembleDevDebug
./gradlew :dsp:installDevDebug
# debug には LeakCanary のランチャーも入るため MainActivity を明示的に起動
adb shell am start -n com.frs.dsp/.MainActivity
```

Windows（PowerShell）は `./gradlew` の代わりに `.\gradlew.bat` を使います。

```powershell
cd frs-dsp-as
.\gradlew.bat :dsp:assembleDevDebug
.\gradlew.bat :dsp:installDevDebug
adb shell am start -n com.frs.dsp/.MainActivity
```

- 初回はライブラリのダウンロードで数分かかります。`BUILD SUCCESSFUL` が出れば成功です。
- `installDevDebug` で実機にアプリが入ります。実機のアプリ一覧では「dsp」を探してください。
- flavor は `dev` / `prod`、ビルドタイプは `debug` / `release`。実機での動作確認は `devDebug` で十分です。

### 6. うまくいかないとき

- `adb: command not found` / `'adb' は認識されていません`:
  [04_AndroidStudio.md](04_AndroidStudio.md) の PATH 設定（`platform-tools` を通す）が未完了です。
- `INSTALL_FAILED_NO_MATCHING_ABIS`:
  本アプリは `arm64-v8a` のみ対応です。最近の実機はほぼ arm64 なので通常は起きません。
- 端末を複数つないでいるとき: `adb devices` に出るシリアルを指定して
  `adb -s <シリアル> shell am start -n com.frs.dsp/.MainActivity` のように実行します。
  Gradle は接続中の全端末にインストールしようとするため、実機 1 台だけにしたいときは他を切断します。
- ワイヤレス接続がすぐ切れる: PC と実機が同じ Wi-Fi か確認し、`adb connect <IP>:<ポート>` を再実行。
  実機がスリープすると切れることがあります。
- `Android Gradle plugin requires Java 17`:
  ターミナルの `JAVA_HOME` が 17 になっていません。[04_AndroidStudio.md](04_AndroidStudio.md) を参照。

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

#### 例: macOS で Homebrew の Temurin 17 を使う

```zsh
brew install --cask temurin@17
export JAVA_HOME="$(/usr/libexec/java_home -v 17)"
cd frs-dsp-as
./gradlew :dsp:assembleDevRelease   # または :dsp:assembleDevDebug
```

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

