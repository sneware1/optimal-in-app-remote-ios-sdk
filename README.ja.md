# Optimal In-App Remote SDK for iOS

Optimal In-App Remote SDK for iOS は、iOS アプリの遠隔支援を実現するための開発キットです。
この SDK をアプリに組み込むことで、そのアプリを [Optimal Remote](http://www.optim.co.jp/products-detail/top/40) で遠隔支援できるようになります。

Android 版 SDK も近日中に対応予定です。

## 対象環境
 - アプリ動作環境
     1. iOS 6 以降
     2. 上記 OS で動作している iPhone または iPad
     3. 英語、日本語
         - 上記以外の言語環境では英語表記になります
     4. インターネットに接続できるネットワーク環境
 - 開発環境
     1. Xcode 6.1 以降

## この SDK でできること

### 画面共有機能
SDK を組み込んだアプリの画面をオペレーターがリアルタイムに閲覧できます。

### 遠隔操作機能
SDK を組み込んだアプリをオペレーターが操作できます。

### 赤ペン機能
SDK を組み込んだアプリの画面にオペレーターが赤ペンで書き込むことでユーザーに操作を指示することができます。

### 指さし機能
SDK を組み込んだアプリの画面にオペレーターから指マークを表示することでユーザーに操作を指示することができます。

### 音声通話機能
オペレーターがユーザーと VoIP で音声通話しながら遠隔支援を効果的に実施することができます。

## SDK をプロジェクトに導入する
以下の手順を進める前に、開発者登録をしていただき以下をご用意ください。

[詳しい手順はこちらを参照してください。](docs/REGISTRATION.md)

 1. SDK を利用するためのプロファイル・キーペア
 2. オペレーターツール (Windows 版)
 3. オペレーターツールを利用するためのアカウント (ID・パスワード)

### 0. この Git レポジトリをチェックアウトする
ZIP としてダウンロードすると OptimalRemote.framework の構成が不正になるので Git レポジトリとしてチェックアウトしてください。

次にチェックアウトしたディレクトリにある「OptimalRemote.framework.zip」を解凍してください。

### 1. OptimalRemote.framework ディレクトリをプロジェクトに追加する
OptimalRemote.framework には、SDK を利用するのに必要なヘッダファイル・静的ライブラリファイルファイルが含まれています。OptimalRemote.framework をプロジェクトに追加するには、この Git レポジトリに含まれる OptimalRemote.framework ディレクトリを以下を参考に追加してください。

 - [Project Navigator Help: Adding an Existing File or Folder](https://developer.apple.com/library/ios/recipes/xcode_help-structure_navigator/articles/Adding_an_Existing_File_or_Folder.html)

### 2. OptimalRemoteResources ディレクトリ をプロジェクトに追加する
OptimalRemoteResources ディレクトリには、SDK を利用するのに必要な文字列ファイル・画像ファイル一式が含まれています。OptimalRemoteResources ディレクトリをプロジェクトに追加するには、この Git レポジトリに含まれる OptimalRemoteResources ディレクトリを以下を参考に追加してください。

 - [Project Navigator Help: Adding an Existing File or Folder](https://developer.apple.com/library/ios/recipes/xcode_help-structure_navigator/articles/Adding_an_Existing_File_or_Folder.html)

### 3. SDK に必要な Framework へのリンクを追加する
SDK を利用したアプリをビルドするには、以下の Framework へのリンクを追加する必要があります。

 1. AssetsLibrary.framework
 2. AudioToolbox.framework
 3. AVFoundation.framework
 4. CoreMedia.framework
 5. CoreVideo.framework
 6. OpenGLES.framework
 7. SystemConfiguration.framework
 8. Security.framework
 9. libsqlite3.dylib

Framework へのリンクを追加するには以下を参考にしてください。

 - [Project Editor Help: Linking to a Library or Framework](https://developer.apple.com/library/ios/recipes/xcode_help-project_editor/Articles/AddingaLibrarytoaTarget.html)

### 4. SDK に必要なリンカフラグを追加する
SDK はカテゴリクラスを利用しているため、リンカフラグに「-ObjC」を追加してビルドする必要があります。また「-lc++ -lstdc++」を追加してビルドする必要があります。リンカフラグをを追加するには以下を参考にしてください。

 - [Technical Q&A QA1490: Building Objective-C static libraries with categories](https://developer.apple.com/library/mac/qa/qa1490/_index.html)

## SDK の利用するためのチュートリアル
SDK を利用するには、いくらかお決まりのコードを記述する必要があります。

### 1. 端末の画面回転に対応する
アプリが端末の画面対応に対応していない場合、以下の対応は不要です。

SDK が表示する画面は、keyWindow とは別のウィンドウに表示されます。そのため、端末の画面対応に手動で対応する必要があります。

`UIApplicationDelegate` プロトコルの `application:willChangeStatusBarOrientation:duration:` メソッドに以下のようにコードを追加することで対応することができます。

```XxxAppDelegate.m
...
// 1. SDK を利用するためのヘッダをインポートする
#import "OptimalRemote/OptimalRemote.h"
...

- (void)application:(UIApplication *)application willChangeStatusBarOrientation:
  (UIInterfaceOrientation)newStatusBarOrientation duration:(NSTimeInterval)duration {
    ...
    // 2. 画面が回転した時に SDK が表示する画面を回転させる
    [ORIAWindow setOrientation:newStatusBarOrientation withDuration:duration];
}
...
```

### 2. ORIASession クラスのインスタンスを作る
ここでは例として、`UIViewController` クラスの派生クラスの `viewDidLoad` メソッドで `ORIASession` クラスのインスタンスを作ります。`ORIASession` クラスは、SDK で iOS アプリの遠隔支援を実現するためのコアクラスのひとつです。

```XxxViewController.m
...
// 3. SDK を利用するためのヘッダをインポートする
#import "OptimalRemote/OptimalRemote.h"
...
// 4. 
@interface XxxViewController () <ORIASessionControllerAppDelegate>
...
// 5. ORIASession の制御クラスをプロパティに追加する
@property (nonatomic, strong) ORIASessionController *controller;
// 6. ORIASession クラスをプロパティに追加する
@property (nonatomic, strong) ORIASession *session;
...
@end
...
- (void)viewDidLoad {
    [super viewDidLoad];
    ...
    // 7. .profile の内容を以下に貼り付ける
    NSString *PROFILE = @"XXXXXXXX";
    // 8. .key の内容を以下に貼り付ける
    NSString *KEY = @"XXXXXXXX";

    // 9. ORIASession の制御クラスのインスタンスを作る
    self.controller = [ORIASessionController defaultController];
    self.controller.appDelegate = self;

    // 10. ORIASession クラスのインスタンスを作る
    self.session = [ORIASession sessionForProfile:PROFILE signedBy:KEY];
    // VoIP を有効にする (既定では NO)
    self.session.voiceChatEnabled = YES;
    // ヘッドフォンがない場合にスピーカーから音声を出力する (既定では NO)
    self.session.voiceChatOverridesSpeakerWhenNoHeadphones = YES;
    self.session.delegate = self.controller;
    [self.session loadDefaultPointerImages];
}
```

### 3. ボタンがタップされたら ORIASession を開始する
ここでは例として、`UIViewController` クラスの派生クラスが `helpMeButton` という `UIButton` クラスのプロパティを持っていると仮定し、そのボタンがタップされたら ORIASession を開始する、というコードを追加します。

```XxxViewController.m
...
// 11. ORIASession が開始したときのメソッド
- (void)controllerDidOpen:(ORIASessionController *)controller {
    // ORIASession が完了するまでボタンを無効化する
    self.helpMeButton.enabled = NO;
}

// 12. ORIASession が完了したときのメソッド
- (void)controllerDidComplete:(ORIASessionController *)controller remoteConnectionHasEstablished:
  (BOOL)remoteConnectionHasEstablished {
    // ORIASession が完了したのでボタンを有効化する
    self.helpMeButton.enabled = YES;
    // 完了画面を表示する
    if (remoteConnectionHasEstablished) {
        [ORIAUISplashWindow showForCompletion];
    }
}

//  13. helpMeButton がタップされたときのメソッド
- (IBAction)helpMeButtonDidTouchUpInside:(id)sender {
    // 開始画面を表示して ORIASession を開始する
    if (self.controller.canOpen) {
        [ORIAUISplashWindow showWithBlock:^{
            [self.session open];
        }];
    }
}
...
```

これで iOS アプリ側の準備は完了です。

### 4. オペレーターツールと接続する
アプリをビルドしたら、インターネットに接続された端末でアプリを実行し、`helpMeButton` をタップすると「受付番号」が表示されます。オペレーターツールでこの受付番号を入力すると、オペレーターツールとアプリが接続され、オペレーターツールにアプリの画面が表示されます！

以上でチュートリアルは完了です。うまくオペレーターツールと接続できない場合、お問い合わせください。