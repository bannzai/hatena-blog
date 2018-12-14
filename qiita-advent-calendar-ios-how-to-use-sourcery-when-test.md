# テスト書いてますか？
みなさん、こんにちは。突然ですがiOSアプリ開発ででテストを書いていますか？ きっとこの記事を読んでいるのはプロダクトにテストがまだ導入できていなくてこれからテストを始めたい人。テストを書いているが書くことに苦労している人。そういった人たちが多いのではと考えています。今回はそんなテストを書くハードルを `Sourcery` を活用して下げて行こうと思います。

# Sourceryで下げられるハードル
テストを書くハードルといっても様々です。 処理を1つのクラスにまとめて責務分割ができていなかったりグローバル変数やシングルトンの多用等の置き換えの効きづらいものが残っているせいでテストが書けない。といった悩みもあるでしょう。この記事ではどちらかというとこういった悩みを解決した後の話になります。今回はテストを書く上で必要なものを用意する手間を省きテストを書くハードルを下げて行きます。  

### Sourceryについて
ここで改めて `Sourcery` について軽く紹介します。  
[Sourcery](https://github.com/krzysztofzablocki/Sourcery)とはボイラープレートを無くすためのSwift製のライブラリになっています。具体的にこのライブラリがやることは以下のような流れになります。

- Swiftのコードを読み込む。
    - プロダクションのコード等を対象にSourceryに読み込ませます。
- 要素ごとに分解して変数に保持している。
    - 要素とはメソッドやクラスといった単位になります。
- 保持している変数と指定したstencilテンプレートからSwiftのコードを自動で生成する
    - [stencilについて](https://github.com/stencilproject/Stencil)

とりあえず `Swift` のコードを元に `Swift` のコードが組み立てられるツールなんだな。ということがわかってもらえたら良いと思います。  
ここまで聞くとなんだかだんだんとどういう風に楽ができるのか。想像できてきたのではないでしょうか。  
では前置きが長くなってしまったのですが、次のセクションから `Sourcery` の活用法について紹介していきます。


# 例
具体的な `Sourcery` の活用方法を紹介する前に例としてユースケースを想定してテストを一つ書いていきたいと思います。  
まずWebAPIからJSONを取得してそれを元に `View` の描画を行う画面があると想定します。この画面を構成する機能を今回のテスト対象とします。
そしてこの機能は3つのクラスで責務分割されていて実現されていると考えます。 

- ViewController
    - 言わずもしれた `UIViewController` を継承したクラスです
- Presenter
    - `ViewController` で発生・ハンドリングしたイベントごとに処理を行う
- APIClient
    - WebAPIを叩くためのクライアント

そして、`API` のレスポンスは `User` という構造体の形を想定します。

```swift
struct User: Codable {
    let id: Int
    let name: String
}
```

今回は `API` のレスポンスにエラーがあると想定して、 `Result` 型で`API`のレスポンスを表現したいと思います。
まだ `Swift` では標準で `Result` が入っていないので、 Swiftで[採択されたResult型](https://github.com/apple/swift-evolution/blob/master/proposals/0235-add-result.md)よりも簡易なものを定義します。　　

```swift
enum Result<T> {
    case value(T)
    case error(Swift.Error)
}
```

上から順に責務分割されている3つのクラスのコードも載せます。

```swift
protocol APIClient {
    func fetch(endpoint: String, handler: @escaping (Result<User>) -> Void)
}

struct APIClientImpl {
    func fetch(endpoint: String, handler: @escaping (Result<User>) -> Void) {
        // Fetch data from api server
    }
}
```

```swift
protocol PresenterOutput: class {
    func reload(user: User)
    func presentAlert(error: Error)
}
protocol Presenter {
    func fetch()
}
class PresenterImpl: Presenter {
    weak var output: PresenterOutput?
    
    let apiClient: APIClient
    init(apiClient: APIClient) {
        self.apiClient = apiClient
    }
    func fetch() {
        apiClient.fetch(endpoint: "users/1") { [weak self] (result) in
            switch result {
            case .value(let user):
                self?.output?.reload(user: user)
            case .error(let error):
                self?.output?.presentAlert(error: error)
            }
        }
    }
}
```

```swift
class ViewController: UIViewController {
    let presenter: Presenter
    init(presenter: Presenter) {
        self.presenter = presenter
        
        super.init(nibName: nil, bundle: nil)
    }
    required init?(coder aDecoder: NSCoder) { fatalError() }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        presenter.fetch()
    }
}
extension ViewController: PresenterOutput {
    func reload(user: User) {
        // Reload views using user
    }
    
    func presentAlert(error: Error) {
        // Catch error and presenting error with alert view
    }
}
```

処理の流れとしては `viewDidLoad` のタイミングで `Presenter` 経由で `APIClient` を叩き `User` を取得する。となります。

## どこをテストするか・何をテストするか
さて、実装部を作ったところで今度はどこをテストするか明確にしていきます。今回は `PresenterImpl` の `fetch` メソッドをテストする前提でお話を進めます。テスト内容は下記の通りです。

- apiClient.fetch(endpoint:handler:)において、`handler` で `Result.value(User)` が渡ってきた場合に、`self?.output?.reload(user:)` メソッドが呼ばれているかどうか
- apiClient.fetch(endpoint:handler:)において、`handler` で `Result.error(Error)` が渡ってきた場合に、`self?.output?.presentAlert(error:)` メソッドが呼ばれているかどうか

前のセクションでお見せしたのクラスたちは全て `protocol` でインタフェースが定義されています。
上記の挙動のテストのためにこのインタフェースに沿った **実装を偽装したクラス** を作ると便利です。
protocol名+Mockという命名でクラスを作っていきます。

```swift
class APIClientMock: APIClient {
    var fetchEndpointHandlerClosure: ((String, @escaping (Result<User>) -> Void) -> Void)?
    func fetch(endpoint: String, handler: @escaping (Result<User>) -> Void) {
        fetchEndpointHandlerClosure?(endpoint, handler)
    }
}

class PresenterOutputMock: PresenterOutput {
    var reloadUserCalled: Bool = false
    var presentAlertErrorCalled: Bool = false
    
    func reload(user: User) {
        reloadUserCalled = true
    }
    
    func presentAlert(error: Error) {
        presentAlertErrorCalled = true
    }
}
```

- `APIClientMock` は `fetchEndpointHandlerClosure` というプロパティを持ちます。これにより `handler` で `Result.value(User)`  もしくは `Result.error(Error)` を自由に設定ができます。
- `PresenterOutputMock` は どのメソッドが通ったかをプロパティで表現しています。このプロパティを見れば `APIClient` からの戻り値によってどのメソッドが通ったかわかりますね。

この **実装を偽装したクラス** を利用した`PresenterImpl.fetch` のテストを下記に記します。

```swift
func testFetch() {
    XCTContext.runActivity(named: "Presenter called reload(user:) when it fetched user from api") { (_) in
        let apiClient = APIClientMock()
        apiClient.fetchEndpointHandlerClosure = { endpoint, handler in
            handler(.value(User(id: 1, name: "bannzai")))
        }
        
        let output = PresenterOutputMock()
        let presenter = PresenterImpl(apiClient: apiClient)
        presenter.output = output
        
        presenter.fetch()
        
        XCTAssertTrue(output.reloadUserCalled)
    }
    XCTContext.runActivity(named: "Presenter called presentAlert(error:) when it got error from api") { (_) in
        struct AnyError: Error {
            
        }
        
        let apiClient = APIClientMock()
        apiClient.fetchEndpointHandlerClosure = { endpoint, handler in
            handler(.error(AnyError()))
        }
        
        let output = PresenterOutputMock()
        let presenter = PresenterImpl(apiClient: apiClient)
        presenter.output = output
        
        presenter.fetch()
        
        XCTAssertTrue(output.presentAlertErrorCalled)
    }
}
```

このテストは見事成功します。そして、とても簡単ですね。テストを用意することで僕たちは安心感を得られます。しかし、簡単に書くことができたと言っても、多少テストを書くことに面倒さも覚えますね。例えば **実装を偽装したクラス** として用意した `処理を実行したかどうかを調べるクラス` `処理の実装を偽装するためのクラス`を用意することは毎回同じようなコードを書くことになります。仮に実装が変わればこれに追随して `Mock` の調整も必要になることもあります。次第にテストを書くことが億劫になってくるかもしれません。この明らかなボイラープレートコードの書く手間を `Sourcery` を使って無くしていきましょう。

# Mockを自動で用意する
テストに関心がある方は `Mock` という単語を耳にしたことがあると思います。  
ここでいう `Mock` の意味としてはテストしたい対象の内部的挙動を把握するためのオブジェクト、くらいの認識で書いてあります。(正確な理解ではない可能性あり) このセクションで説明することは [GitHub](https://github.com/krzysztofzablocki/Sourcery/blob/master/Templates/Templates/AutoMockable.stencil) にも紹介されているものがあります。ですので、言葉を合わせるといった意味でも `Mock` という単語を今後使っていきます。 `Mock` の意味については別途調べていただけたらと思います。
では `Mock` の準備を `Sourcery` を使用して自動で生成したいと思います。  


## Sourceryを使う
[インストール方法](https://github.com/krzysztofzablocki/Sourcery#installation)はいくつかあるのでお好みで選んでください。インストールが終わればあとはテンプレートを用意して `Sourcery` に用意されているスクリプトを実行するだけになります。今回 `Mock` を作るためのテンプレートは[GitHubにテンプレート](https://github.com/krzysztofzablocki/Sourcery/blob/master/Templates/Templates/AutoMockable.stencil)として用意されているものを使います。  


テンプレートについての細かい説明は [ドキュメント](https://cdn.rawgit.com/krzysztofzablocki/Sourcery/master/docs/index.html) をご覧になってください。  
この中でも下記の行に注目します。


```
{% for type in types.protocols where type.based.AutoMockable or type|annotated:"AutoMockable" %}
```

この行は `Mock` を生成する処理の入り口になっています。 `Sourcery` ではコマンドの引数として `Swift` のコードを渡してあげ、それをパースしてテンプレートに沿った内容を出力してくれます。上記の `stencil` テンプレートの内容で押さえておきたいポイントは以下の3つです。

- `types.protocols`: 引数として渡したSwiftコードの中で `protocol` として存在するものを配列として持っています。ちなみに `types`は配列じゃないです
- `type.based.AutoMockable`: こちらは `AutoMockable` をベースとした `type`(型) 情報を取得してくれます。 
- `type|annotated:"AutoMockable"`: こちらは `AutoMockable` というコメントで `Annotation` が付いているものを取得してきてくれます。

合わせて上記の `for` 文を読むと `protocol` の一覧を取得して `AutoMockable` を `base` とした型・`AutoMockable` とコメントが付いている型 を全て取得してテンプレートからの描画処理が始まることになります。

つまり、上記のテンプレートを使って `Mock` を生成したいのであれば `AutoMockable` を目印としてつけてあげる必要があります。  
`protocol AutoMockable` を用意して準拠していきましょう。

```swift
protocol AutoMockable {
    
}
```

そして、 `Mock` を生成したい `protocol` の `base` として `AutoMockable` を設定してあげます。

```swift
protocol APIClient: AutoMockable {
    func fetch(endpoint: String, handler: @escaping (Result<User>) -> Void)
}

protocol PresenterOutput: class, AutoMockable {
    func reload(user: User)
    func presentAlert(error: Error)
}
```

そして、テンプレートを任意の位置に追加してあげます。
内容は[ここから](https://github.com/krzysztofzablocki/Sourcery/blob/master/Templates/Templates/AutoMockable.stencil)コピペしましょう。  

```shell
$ mkdir templates
$ touch templates/AutoMockable.stencil
```

最後に `sourcery` を実行します。

```shell
$ sourcery --sources ./SourceryDemo/ViewController.swift --templates ./templates/AutoMockable.stencil --output ./SourceryDemo/Generated/Mock.generated.swift 
```

これで `./SourceryDemo/Generated/Mock.generated.swift` というファイルができて、その中に `Mock` が吐き出されていると思います。

## と思ったが
[この記事を書いている時点のAutoMockable](https://github.com/krzysztofzablocki/Sourcery/blob/cb533d709b5c315a6a44f4e9adb0112394014eff/Templates/Templates/AutoMockable.stencil)を使用するとエラーが出てしまいました。
原因はわかったので `PR` をあげました。[あげたPRの内容のAutoMockableのテンプレート](https://github.com/krzysztofzablocki/Sourcery/pull/709)を使っていきます。  
`PR` がマージされるかはわかりませんが、上記の `PR` で想定している動きになります。

というわけで続きなのですが、 `sourcery` の実行後のファイルの中身は以下のようになっていると思います。


```swift
class APIClientMock: APIClient {

    //MARK: - fetch

    var fetchEndpointHandlerCallsCount = 0
    var fetchEndpointHandlerCalled: Bool {
        return fetchEndpointHandlerCallsCount > 0
    }
    var fetchEndpointHandlerReceivedArguments: (endpoint: String, handler: (Result<User>) -> Void)?
    var fetchEndpointHandlerClosure: ((String, @escaping (Result<User>) -> Void) -> Void)?

    func fetch(endpoint: String, handler: @escaping (Result<User>) -> Void) {
        fetchEndpointHandlerCallsCount += 1
        fetchEndpointHandlerReceivedArguments = (endpoint: endpoint, handler: handler)
        fetchEndpointHandlerClosure?(endpoint, handler)
    }

}
class PresenterOutputMock: PresenterOutput {

    //MARK: - reload

    var reloadUserCallsCount = 0
    var reloadUserCalled: Bool {
        return reloadUserCallsCount > 0
    }
    var reloadUserReceivedUser: User?
    var reloadUserClosure: ((User) -> Void)?

    func reload(user: User) {
        reloadUserCallsCount += 1
        reloadUserReceivedUser = user
        reloadUserClosure?(user)
    }

    //MARK: - presentAlert

    var presentAlertErrorCallsCount = 0
    var presentAlertErrorCalled: Bool {
        return presentAlertErrorCallsCount > 0
    }
    var presentAlertErrorReceivedError: Error?
    var presentAlertErrorClosure: ((Error) -> Void)?

    func presentAlert(error: Error) {
        presentAlertErrorCallsCount += 1
        presentAlertErrorReceivedError = error
        presentAlertErrorClosure?(error)
    }

}
```

`APICient` と `PresenterOutput` の `Mock` が吐き出されましたね。これを `Test Target` に `import` して使いましょう。  
ここでもう一つ注意点があるのですが、 `@testable import XXXX` をする必要があります。こちらは適宜 `stencil` テンプレートの方にも記載して `sourcery` コマンドを実行した時に挿入されるようにしてください。  
先ほど手で書いた`APIClientMock`と`PresenterOutputMock` は削除してます。
テストが成功したら無事に `Sourcery` 移行が完了です。

# 終わりに
いかがでしたでしょうか。  
テストがあるととても助かりますが、それを継続して書き続けるのは困難な場合もあると思います。
テスト書く際の手間を減らして、テストをガンガン書いていきましょう。

おしまい \\(^o^)/


