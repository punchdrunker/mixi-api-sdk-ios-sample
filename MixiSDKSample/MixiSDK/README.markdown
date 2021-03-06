はじめにお読みください。
=======================

はじめに
--------

mixi API SDK for iOSは開発者の方がiOSネイティブのmixiアプリをできる限り簡単に開発することができるように開発されました。

### 特徴

mixi API SDK for iOSの特徴は、以下の通りです。

- 個人・法人に関わらず、個人パートナー登録すればどなたでも開発可能
- OAuth 2.0の認証/認可手順の実装が不要
- Tokenの取得/更新を自動化
- APIコールを統一的な手順で実行可能
- シングルサインオンが可能でユーザ認可時のパスワード入力不要

### サポート端末

本SDKにてサポートするiOS端末は以下の通りです。

- iPhone（iOS4.0以降搭載） **1

**1 iPadについては、対応未定

### 利用可能なAPI

- mixiアプリ
  - People API
  - Groups API
  - Photo API
  - Request API
  - Communication Feed **2
- Graph API **3
  - People API
  - Groups API
  - People lookup API
  - Voice API
  - Updates API
  - Check API
  - Photo API
  - Message API
  - Diary API
  - Check-in API
  - Profile Image API

**2 Communication Feedについては、今後提供予定

**3 基本的に提供されているすべての Graph API が利用可能です。

なお、Geolocation APIについてはTouch版/モバイル版と異なり、弊社への許諾なしに利用することが可能です。

SDKダウンロード
---------------

mixi API SDK for iOS を利用するには、以下のファイルをダウンロードしてください。
ダウンロードすると、mixi API SDK for iOS の利用規約に同意したものと見なされます。

<table>
  <tr><th>pckage</th><th>version</th><th>size</th><th>date</th></tr>
  <tr><td><a href="http://developer.mt.mixi.co.jp/connect/appli/spec/ios/download/mixiIOSSDK-__REPLACE_VERSION__.zip">mixiIOSSDK-__REPLACE_VERSION__.zip</a></td><td>v__REPLACE_VERSION__</td><td>508KB</td><td>__REPLACE_TODAY__</td></tr>
</table>

### 更新履歴

__INCLUDE(Classes/MixiSDK/History.txt)__

アプリケーションの登録
----------------------

本SDKを用いてiOSアプリを開発するための手順を説明します。

### mixiアプリ

SDKを利用するには、あらかじめ Partner Dashboard にてアプリケーションが登録されている必要があります。まず、Partner Dashboard の mixiアプリ登録ページにて登録を行なってください。その際、対応デバイスの項目で「スマートフォンに対応(iOS版)」にチェックをすることでSDKが利用可能になります。

### Graph API

SDKを利用するには、あらかじめ Partner Dashboard にてアプリケーションが登録されている必要があります。まず、Partner Dashboard の mixi Graph APIのサービス登録ページにてアプリケーションを登録をしてください。

その際、"起動URIスキーム"を以下を参考に登録してください。

### 起動URIスキーム

ユーザがmixiサイト上のアプリケーション一覧やフィードをクリックした際に、WebブラウザからiOSアプリケーションが起動されることになります。この時に利用されるのが、起動URIスキームです。
初期値は以下の値に設定されています。

- mixiapp-<APP_ID>://run

この起動URIスキームの<APP_ID>の部分は変更可能です。できるだけ変更されることをお勧めします。変更を行う場合は、Partner Dashboardのアプリ設定ページから変更してください。
この起動URIスキームを実際にアプリケーションが受け取るためには、アプリケーションの Info.plist に適切な値を設定する必要があります。

プロジェクトの作成
------------------

SDKをダウンロードして、プロジェクトのソースディレクトリ内の適切な場所に解凍しておく必要があります。

### iOSプロジェクトの作成

通常のiOSプロジェクト作成手順に従ってプロジェクトを作成します。

### 必要なフレームワークの追加

mixi API SDK for iOSは次のフレームワークを必要とします。

- CFNetwork.framework
- Security.framework
- SystemConfiguration.framework

### 起動URIスキームの追加

Partner Dashboardに登録した起動URIスキームをプロジェクトに追加します。

これで開発を始める準備は整いました。次からは、いよいよAPIの利用方法を説明していきます。

初期化と認可処理
----------------

mixi API SDK for iOS を利用したアプリケーションを実際に開発する際のコードの記述方法について説明します。

### ヘッダファイルの追加

SDKを利用する場合は次のヘッダファイルをインポートしてください。

<pre><code>#import "MixiSDK.h"</code></pre>

### 初期化

mixi APIの呼び出しに使用するMixiクラスはシングルトンクラスです。インスタンスは次のようにして取得できます。

<pre><code>[Mixi sharedMixi]</code></pre>

ただし、APIを実行する前に一度シングルトンインスタンスを初期化しておく必要があります。
UIApplicationDelegate#application:didFinishLaunchingWithOptions:
例えばGraph APIを使用する場合、メソッド内で次のように記述するといいでしょう。
（全ての引数はアプリケーションの設定に合わせて変更してください）

<pre><code>Mixi *mixi = [[Mixi sharedMixi] setupWithType:kMixiApiTypeSelectorGraphApi 
                                    clientId:@"ab12c345de6789f12345" 
                                      secret:@"a1b2c3456d789ef0123ghi4567jklmn89op01qrs"
                                       appId:@"A12BCDE3F4.jp.co.mixi.apisdk"];
[mixi restore];
[mixi reportOncePerDay];</code></pre>

### アプリの起動を通知

アプリの起動を通知するコードをapplicationWillEnterForeground:に追記します。

<pre><code>- (void)applicationWillEnterForeground:(UIApplication *)application {
    [[Mixi sharedMixi] reportOncePerDay];
}</code></pre>

このコードの追記は任意ですが、サービスの改善のために協力していただけると幸いです。
アプリケーションの情報は一切送信されません。

### 認可

mixiアプリのAPIを利用するためには、ユーザにAPI利用のための認可を行なってもらう必要があります。そのための画面を表示するのが authorize: メソッドです。

<pre><code>[mixi authorize:@"r_profile", @"r_diary", @"w_diary", nil];</code></pre>

このメソッドを呼び出すことで、mixi公式アプリを利用してユーザーに認可を促す画面が表示されます。

さらに、公式アプリで行われる認可の結果（アクセストークンなど）を受け取るために
UIApplicationDelegate#application:openURL:sourceApplication:annotation:
メソッドに次のような処理を追加しておきます。

<pre><code>NSError *error = nil;
NSString *apiType = [[Mixi sharedMixi] application:application openURL:url sourceApplication:sourceApplication annotation:annotation error:&error];
if (error) {
　　// エラーが発生しました
}
else if ([apiType isEqualToString:kMixiAppApiTypeToken]) {
　　// 認可処理に成功しました
}
else if ([apiType isEqualToString:kMixiAppApiTypeRevoke]) {
　　// 認可解除処理に成功しました
}</code></pre>

上記により認可が完了してればシングルトンオブジェクトはアクセストークンを保持し、APIを実行できる状態になっています。

### 認可状態の確認

現在の認可状態を確認するには、以下のメソッドを利用します。

<pre><code>Mixi#isAuthorized</code></pre>

通常は、認可画面は初回のみ表示され、2回目以降はスキップすることが可能です。isAuthorizedメソッドはそのための確認処理を行います。既に認可済みであれば YES が返り、未認可であれば NO が返ります。

### 認証解除

以下のメソッドを呼び出すことで認可状態の解除を行います。なお、mixi API SDK for iOS を利用する場合はユーザ保護の観点から、この認証解除機能を必ず実装してください。

<pre><code>Mixi#revoke</code></pre>

APIの利用
---------

API にアクセスするメソッドは複数ありますが、代表的なメソッドはMixi#sendRequest:delegate: です。本メソッドを利用して、統一的な利用で様々なAPIをコールすることが可能です。

ここでは、APIを呼び出すためのsendRequest:delegate:メソッドの簡単な例を紹介します。各APIに必要なパラメータの意味や戻り値など個々のAPIに関する詳細情報については、SDKに添付されているリファレンスを参照してください。

APIはエンドポイントとパラメーターを設定した<code>MixiRequest</code>インスタンスを、<code>Mixi#sendRequest:delegate:</code>メソッドに渡すことで実行します。
API実行前に認可が完了しているかどうかを確認して、未認可の場合は先に認可しておきます。認可に失敗した場合はmixi公式アプリがインストールされていないか、最新版ではない可能性があるので、AppStoreのmixi公式アプリのページを開きます。

<pre><code>if ([mixi isAuthorized]) {
    MixiRequest *request = [MixiRequest requestWithEndpoint:＠"/people/＠me/＠friends"];
    [mixi sendRequest:request delegate:mixiDelegate];
}
else if (![mixi authorizeForPermission:＠"r_profile"]) {
    MixiWebViewController *vc = MixiUtilDownloadViewController(self, @selector(closeDownloadView:));
    vc.orientationDelegate = self;
    [self presentModalViewController:vc animated:YES];
}</code></pre>

API実行結果はsendメソッド群の引数に与えていた<code>MixiDelegate</code>プロトコルを実装したデリゲートで処理します。
例えば、APIの実行結果をAlertViewで表示するには次のようなデリゲートメソッドを定義したクラスのインスタンスをsendRequest:delegate:メソッドの引数に与えます。

<pre><code>- (void)mixi:(Mixi*)mixi didFinishLoading:(NSString*)data {
    MixiUtilShowMessageTitle(data, ＠"実行結果");
}</code></pre>

上記以外のデリゲートメソッドについては<code>MixiDelegate</code>のドキュメントを参照してください。

mixiアドプログラムAPI
---------------------

mixiアドプログラムiOSアプリ版のAPIを利用することで、mixiアドプログラムをiOSアプリ版mixiアプリに組み込むこと ができます。ここでは、mixiアプリにどのようにmixiアドプログラムiOSアプリ版の機能を実装するかを説明いたします。

### 表示イメージ

mixiアドプログラムiOSアプリ版の機能を利用するには、アプリ上で以下の「mixiアドプログラム専用枠」を表示してください。

専用枠サイズ： 横幅100%×縦37px（縦表示、横表示の場合とも）

専用枠には上記の図の1～3の場所に、リンクが設置されます。なお表示位置と表示内容は変更することができません。

### 制限事項

mixiアドプログラムiOSアプリ版APIは、どなたでもご利用になれます。なお、お支払い対象となるのはmixiにiOSアプリ版mixiアプリのお申し込みいただき、弊社が承認したmixiアプリのみとなります。

### 利用手順

mixiアドプログラムの機能はiOS SDKではMixiADBannerViewクラスで管理されています。mixiアドプログラムを利用するにはInterface Builderを利用して指定された位置にMixiADBannerViewインスタンスを表示するか、MixiADBannerViewインスタンスを画面に表示するコードをプログラム内に含めてください。

複数の画面でMixiADBannerViewを表示する場合は、MixiADBannerViewの共有オブジェクトを利用するといいでしょう。次は共有オブジェクトを利用してMixiADBannerViewを表示する例です。

<pre><code>- (void)loadView {
    [super loadView];
    [[MixiADBannerView sharedView] addOn:self.view]:
}
</code></pre>

デバイスの向きに応じて表示を変える場合はuseOrientationプロパティをYESに設定するか、shouldAutorotateToInterfaceOrientation: メソッド内で明示的に orientation プロパティを設定してください。

<pre><code>― (BOOL)shouldAutorotateToInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation {
    [MixiADBannerView sharedView].orientation = interfaceOrientation;
}
</code></pre>

\subsection 実装後の申請

アプリケーションの実装が完了した後、Ad Hocビルドしたバイナリをメールに添付し、下記の通りお送り下さい。

- 宛先: contact-mixiapps@mixi.jp
- 件名: 【iOSアプリ版mAPテスト実行ファイル申請】アプリ名/SAP名
- 内容: リリース予定のアプリケーションのAd Hocビルドしたバイナリ（もしメール上のファイルサイズが添付ファイルを含めて10MBを越える場合は、ダウンロード先・方法を指定してください）

なお、Ad Hoc配布先のデバイスとして下記のUDIDを登録してください。

- 978b464c4afe7b6eacb712a36a86ac68f7da5ab6

ファイルを送付後、下記のページからmixiアドプログラムiOSアプリ版にお申し込みください。

http://developer.mixi.co.jp/appli/policies/map/guidelines/
