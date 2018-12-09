# テスト書いてますか？
みなさん、こんにちは。突然ですがiOSアプリ開発ででテストを書いていますか？ きっとこの記事を読んでいるのはプロダクトにテストがまだ導入できていなくてこれからテストを始めたい人。テストを書いているが書くことに苦労している人。そういった人たちが多いのではと考えています。今回はそんなテストを書くハードルを `Sourcery` を活用して下げて行こうと思います。

# Sourceryで下げられるハードル
テストを書くハードルといっても様々です。 処理を1つのクラスにまとめて責務分割ができていなかったりグローバル変数やシングルトンの多用等の置き換えの効きづらいものが残っているせいでテストが書けない。といった悩みもあるでしょう。この記事ではどちらかというとこういった悩みを解決した後の話になります。今回はテストを書く上で必要なものを用意する手間を省きテストを書くハードルを下げて行きます。  

### Sourceryについて
ここで改めて `Sourcery` について軽く紹介します。  
[Sourcery](https://github.com/krzysztofzablocki/Sourcery)とはボイラープレートを無くすためのSwift製のライブラリになっています。具体的にこのライブラリがやることはこんな感じです。

- Swiftのコードを読み込む。
    - プロダクションのコード等を対象にSourceryに読み込ませます。
- 要素ごとに分解して変数に保持している。
    - 要素とはメソッドやクラスといった単位になります。
- 保持している変数と指定したstencilテンプレートからSwiftのコードを自動で生成する
    - [stencilについて](https://github.com/stencilproject/Stencil)

とりあえず `Swift` のコードを元に `Swift` のコードが組み立てられるツールなんだな。ということがわかってもらえたら良いと思います。  
ここまで聞くとなんだかだんだんとどういう風に楽ができるのか。想像できてきたのではないでしょうか。  
では前置きが長くなってしまったのですが、次のセクションから `Sourcery` の活用法を紹介します。

# Mockを自動で用意する
テストに関心がある方は `Mock` という単語を耳にしたことがあると思います。  
ここでいう `Mock` の意味としてはテストしたい対象の内部的挙動を把握するためのオブジェクト、くらいの認識で書いてあります。(正確な理解ではない可能性あり) このセクションで説明することは [Sourceryのwiki](https://github.com/krzysztofzablocki/SourceryWorkshops/wiki/5.Sourcery-Mocks) にも記載があります。ですので、言葉を合わせるといった意味でも `Mock` という単語を今後使っていきます。 `Mock` の意味については別途調べていただけたらと思います。
では `Mock` の準備を `Sourcery` を使用して自動で生成したいと思います。  

## ユースケース
具体的なユースケースを想定して紹介したいと思います。
まずアプリの中にWebAPIからJSONを取得して `View` の更新を行う画面があると想定します。この画面が今回のテスト対象の画面とします。
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

このテストケースを見てどう書けばテストが担保出るか考えていきましょう。

## Sourceryを使う
[インストール方法](https://github.com/krzysztofzablocki/Sourcery#installation)はいくつかあるのでお好みで選んでください。インストールが終わればあとはテンプレートを用意して `Sourcery` に用意されているスクリプトを実行するだけになります。今回は

# テストデータを自動で用意する

# 終わりに