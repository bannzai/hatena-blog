# SwiftUIで開発したアプリをOSSにしました
WWDC 2019 で発表された[SwiftUI](https://developer.apple.com/videos/play/wwdc2019/204/) でアプリを作り、昨日AppStoreにもリリースされました。  

NotificationHub: https://apps.apple.com/jp/app/notificationhub/id1484099869?l=en. 

また、この NotificationHubはGitHubでOSSとして公開しています。  
こちらも合わせて見てもらえるとわかりやすいと思います。

リポジトリ: [NotificationHub](https://github.com/bannzai/NotificationHub)  
記事を書いている時点のコミット: [fd39bd11b5bdb937186b2390f86cdae997b37f36](https://github.com/bannzai/NotificationHub/tree/fd39bd11b5bdb937186b2390f86cdae997b37f36)   

この記事ではどんなアプリを作ったのか紹介と、このプロダクトで使用している技術的な部分をさらっと紹介してちょっとだけ使ってみた感想とか書いていきます。SwiftUI + αを紹介していきますが、このリポジトリを見たらその技術の使い方や、少なくとも **NotificationHub** ではどうやっているのかを知ってもらえたら困った時に参考にできる状態であると思います。
躓いた部分もいっぱいあって最終的に今の形に落ち着いたのですが、それらを書いていくとブログがすごい長さになると思うので軽くかいつまんで書いていきます。項目によっては後日詳細な記事も書きたいな。と思っています。  

## NotificationHub
まずはどんなアプリを作ったのか。から紹介していきます。  
**NotificationHub** では GitHubの未読の通知をアプリ内で一覧して見れる機能を提供しています。

Webからだとここから見れるものですね

<img width=100px src="https://user-images.githubusercontent.com/10897361/67446542-e5ce6680-f64b-11e9-9ea2-553dece6e097.png" />

これを作るかあ。と思った要因として

1. WatchingしているOSSの通知が多い。目を通しておきたいのも多い
2. 公開したリポジトリに稀にissueやPRが 飛んでくることがある。`1` のこともあって埋もれたりする。

がありました。メールで通知を受け取って確認したり、確認したいタイミングでWebの通知を見てもまあ良かったのですが、メールは件数が多くてあまり通知として来てほしくない。Webの通知画面も導線が少し奥で一覧性があまりよくない。

サクッとアクセスして Organization・個人アカウントごとの通知を一覧できる画面も追加できる。確認したい時に開いて見たいのだけ見る。みたいなアプリがほしいな。と思って作りました。

右上の **+** アイコンみたいなところからどのGitHubアカウントの通知一覧画面を作るか選択することが出来ます。ONになっているやつの通知一覧画面が追加されます。主要画面は下記の２つですね

|  通知一覧  |  **+** ボタン押した時  |
| ---- | ---- |
|  <img width=300px src="https://user-images.githubusercontent.com/10897361/67378947-04d4e600-f5c3-11e9-9cbd-e4f178ab94b8.jpg" />  |  <img width="300px" src="https://user-images.githubusercontent.com/10897361/67485662-7f763200-f6a5-11e9-9f3b-1c7b059afc2b.png" />  |

あとはタップするとお知らせに対応したGitHubのPRやIssueのページが開きます。
機能としては以上になりますね。

次にこのリポジトリで採用した技術について書いていきます。

## Swift Package Manager
**NotificationHub** ではサードパーティ製のツール導入のために **Swift Package Manager** を採用しました。
Swift Package Manager も 今年のWWDCでXcodeからライブラリを導入できてiOSアプリ開発でも使える。と紹介されていました。  
**NotificationHub** では [Nuke](https://github.com/kean/Nuke) と [OAuthSwift](https://github.com/OAuthSwift/OAuthSwift) の導入に使いました。

**Nuke** では画像をリモートサーバーから取得して表示する用途で使っています。  
特に躓いた点はありませんでしたが、Swift 5.0 から入った `Result` 型に `Combine` 拡張として `public var publisher: Result<Success, Failure>.Publisher { get } ` が生えていたことに地味に感動しました。[Result.Publisherを使っている場所はここ](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Frameworks/NotificationHubCore/Image/ImageLoader.swift#L30)  画像取得してViewに表示するまでの主要なロジックは下のclass達を見れば分かるかと思います。

- [ImageLoader.swift](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Frameworks/NotificationHubCore/Image/ImageLoader.swift)
- [ImageLoaderView.swift](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Components/Image/ImageLoaderView.swift)
- [ImageLoaderViewModel.swift](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Components/Image/ImageLoaderViewModel.swift)

**OAuthSwift** では[GitHub API v3](https://developer.github.com/v3/)を使用するためにOAuth2のフローを踏んでAccessTokenを取得するために使用しています。このライブラリはもともとiOSアプリで使用する場合はUIKitを使用する前提のライブラリです。SwiftUIでUIKitの世界に踏み込む場合は [UIViewControllerRepresentable](https://developer.apple.com/documentation/swiftui/uiviewcontrollerrepresentable)、もしくは[UIViewRepresentable](https://developer.apple.com/documentation/swiftui/uiviewrepresentable) を使用します。これらはググれば使い方等が普通に出てくるのであえてここでは解説しません。話を戻してOAuthSwiftでは `UIViewController` を使用するのが都合が良かったため `UIViewRepresentable` 経由で `UIViewController` を使用しています。

`UIViewController` を使うついでにiOS13からStoryboardでUIが組まれた `UIViewController` も、その `UiViewController` のクラスのinitializerが使用できるAPIが生えていたので[使うことにしました](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Screen/OAuth/OAuthView.swift#L17)。新しく生えたAPIは [instantiateInitialViewController](https://developer.apple.com/documentation/uikit/uistoryboard/3213988-instantiateinitialviewcontroller)と[instantiateViewController](https://developer.apple.com/documentation/uikit/uistoryboard/3213989-instantiateviewcontroller) ですね。使ってみた感想は素直にええやん。ってなりましたね。Storyboardを使っていてiOS13対応ができるアプリ開発者の皆さん。使っていきましょう。

あと書きながら思い出したのですが、 **Swift Package Manager** 経由で `OAuthSwift` はそのまま使えませんでした(1ヶ月ほど前は)。自分でも原因調査ちゃんとしたかもはや記憶がさだかではないのですが、暫定対応として無理やりビルドを通すようにしたブランチを本家からforkして **NotificationHub** では使用しています。たぶんPRにメモがきしてあるあたり `OTHER_SWIFT_FLAG` に値が設定できなくて積んだからforkしたみたいな感じらしいですが1mmも覚えていません。ここで躓いたかたはよかったら見てみてください。[forkしたブランチから上げたPR](https://github.com/bannzai/OAuthSwift/pull/1)

OAuthSwiftを使っている場所は[ここ](https://github.com/bannzai/NotificationHub/tree/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Screen/OAuth)を見れば全体的にどう使っているのか分かると思います。

## Redux
**NotificationHub** では最終的にアーキテクチャとしてReduxを採用しました。**NotificationHub**ではEmbededFramework化しており、[Redux用のフレームワーク、ディレクトリが用意されています](https://github.com/bannzai/NotificationHub/tree/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Frameworks/NotificationHubRedux)。さて、ここで参考にしたOSSを紹介したいと思います。SwiftUI x Reduxを実現する上で [SwiftUIFlux](https://github.com/Dimillian/SwiftUIFlux) というライブラリを大いに参考にしました。これを **Swift Package Manager** 経由でインストールすることも考えたのですが、ObservableObjectになっている[Store](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Frameworks/NotificationHubRedux/Store.swift) に [objectDidChange](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Frameworks/NotificationHubRedux/Store.swift#L15) ほしいなあ。とか、スレッド管理少しロジック足したいなあ。とか思ったりして移植という形で記述してあります。ちなみに `objectDidChange` の方は特に下手するとBad Practiceの可能性もあります。誰か `objectDidChange` 使わなくてもいい感じに **NotificationHub** の機能を保ってくれるPRを送ってください(他力本願)。 

**SwiftUIFlux** で感心したのが `@EnvironmentObject` の使い方です。僕は1ファイルにまとめてしまったので[そっちのリンクを貼って](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Frameworks/NotificationHubRedux/Render/RenderableView.swift)紹介します。 `SwiftUI` では通常 `View` プロトコルに準拠して `View` の構成を決めていきますが、この `RenderableView` を準拠することで `Redux` で管理されている全ての情報源である `Store` が `@EnvironmentObject` 経由で `DI` されることになります。これは一度DIしてしまえば同じ階層下にいる細かい `View` にも `@EnvironmentObject` が受け継がれ、`@EnvironmentObject` の宣言をすることで変更通知を受け取ることができるようになります。さらにこの `Store` は `ObservableObject` であり、 `@Published public var state: State` が変更されるたびに `RenderableView` に準拠している `View` に変更通知が行きます。少し話が飛びますが、これにより **Single Source of Truth** を実現する上での強制力も上がり、アプリ全体での記述方法の統一化、`View` の構成のパターン化もできそうだな。と思いました。もちろん上手くコントロールしなければならない場合もあります。そうしたい時に私がとった選択は SwiftUIでよく出てくるような `ViewModel` みたいなものに[一度Storeのイベントをハンドリングさせて必要なイベントだけ `View` に伝えるという方法を選択しています](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Screen/Root/RootView.swift#L14)。

`Redux` を採用した経緯は `@ObservableObject` や `@State`, `@Binding` を通した変更通知のみだと苦しいな。と感じたからです。  具体的に厳しいな。という場面があって採用したのですが、これも長くなりそうなのでここでは書きません。さらっとだけ書くと **NotificationHub** では横のページングに応じて `NavigationBar` のタイトルをリポジトリの名前にするというロジックを入れているのですが、ページングした時に親までページングのイベントを伝えつつ、親の `View` を更新しない。という書き方が `@State` や @ObservableObject` を使用した場合に素直に思いつかずに `Redux` の採用に踏み切りました。最終的には `NavigaitonBar` のタイトルを[変更するロジックはこうなり](https://github.com/bannzai/NotificationHub/blob/fd39bd11b5bdb937186b2390f86cdae997b37f36/NotificationHub/Components/NavigationBar/NavigationBarTitleModifier.swift#L16)、また自分が実現したい機能も実現できたので採用してよかったです。

というのとあまりこのアーキテクチャ最高！っていうのは思考停止なので言いたくないのですが、 `SwiftUI` と `Redux` というものはある程度親和性があるのではないのだろうか。と感じていたりします。 `Single Source of Truth` の実現しやすさとか。

## Bitrise
OSSのアプリケーションで使ううえでは無料ということもありCIサービスは[Bitrise](https://bitrise.io) を採用しました。  
主にPRごとのテストとApp Store Connectへのアプリのバイナリのアップロードを自動化しています。  
今回始めてiOSアプリをOSSに試験的にしてみたこともあり他のアカウントが勝手に build とか出来ないかなー。とか思っていたのですが、BitriseのGUI上からはたぶん出来なさそう。ということがわかりました。次もiOSアプリをOSS化する時にも使えそうです。秘匿情報はを隠す必要はもちろんありそうですが...

あとBitriseでArchiveのステップを特に意識せずにArchiveできてしまいました。xcodebuildするだけでSwift Package Managerからライブラリ落として来てくれたりしてくれのかな。と思いました。便利だな。と思って何も検証も確認もしてなかったのですが、Swift Package Manager を使用したプロダクトでもBitriseからアプリのバイナリをアップロードするのは簡単でした。

このアプリにBitriseを導入した時にAppleアカウントの2FAで躓いた知見を[記事]((https://bannzai.hatenadiary.jp/entry/2019/10/22/005621))にしました。よければこちらもご覧になってください。

## まとめ
結局技術的な部分についてダラダラといつまでも書いてしまいそうなので強引ですがここで一旦記事を終わりにします。  
お気づきの方もいるでしょうがこの **NotificationHub** は **SwiftUI** 使って何かアプリ作れるかなー。っていうのが原点で始まりました。少し言い訳めいてますが、それもあっていわゆる `UX` 的な設計はだいぶ適当になりがちです。とはいえ欲しい機能は実装できたので個人的には満足しているというのは本音です。とはいえちょっとずつブラッシュアップはしていきます。もういくつかこうしたほうがいいな。と考えている点はあるので。

個人的な **SwiftUI** で小さなアプリケーションを組んでみた感想はまあやっぱりまだまだ足りない機能はあるのはもちろん、ワークアラウンド的な解決策も多く、Xcodeのエラーもよくわからない場所によくわからない感じに出たりして、目玉のPreviewも `@Binding` とかが絡むとすぐ動かなくなったりとかして良くない点をあげるといっぱいあります。既存のアプリに導入 **すべき** 家と言われればNOだし、新規アプリに関しても現状の **SwiftUI** で現実的に実装が出来る範囲で使う分には面白いとは思う。って感じの答えになりそうです。

と、散々だめな部分を言っておいてなんですが、やっぱり新しいパラダイムとして未来は感じています。宣言的UIや **Single Source of Truth** という方針も個人的には好きです。単純にUIをコードで書けるのは気持ちいですし、安定はしてませんがPreviewもやはり便利です。あと今は手を付けませんがCatalystも良いですね。Mac App も書いてみたいです。成熟していくにつれ便利にはなっていくはずなので出来ることが広がっていく今後に期待ですね。

最後にこの記事に書いてあることや **NotificationHub** いいじゃん。OSSも後で見よう。そう思ったそこのあなた。

<p style="font-weight:800; font-size:60px;">
スターください 🌟
</p>

おしまい\\(^o^)/
