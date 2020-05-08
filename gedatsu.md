# 解脱してAutoLayoutのエラーログをスラスラ読もう
解脱(Gedatsu)をすればAutoLayoutのエラーログもスラスラ読めるようになります

| 解脱前 | 解脱後 |
| ---- | ---- |
|  <img width="100%" src="https://raw.githubusercontent.com/bannzai/Gedatsu/master/docs/autolayout.png" />  |  <img width="100%" src="https://raw.githubusercontent.com/bannzai/Gedatsu/master/docs/gedatsu.png" />  |


## 解脱を入れるには
CocoapodsでもSwiftPackageManagerでもCarthageでもマニュアルインストールでもいいですが、まずは[GedatsuのInstallセクションを見て](https://github.com/bannzai/gedatsu#install)、Gedatsuをプロジェクトにインストールしていきましょう

## 解脱するには
解脱するのは[簡単](https://github.com/bannzai/gedatsu#usage)です。`AppDelegate.application:didFinishLaunchingWithOptions:`に `Gedatsu.open` をして悟りを開くだけです。 `DEBUG` フラグで囲むのを忘れないように

```swift
#if DEBUG
import Gedatsu
#endif

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    #if DEBUG
    Gedatsu.open()
    #endif
    return true
}
```

## 解脱の仕組み
どうやって私が解脱まで到達できたのか少し紹介します。まず `Gedatsu` のエントリポイントとなる関数を見てみましょう。ここでは標準エラー出力までに途中に処理を挟んでいます。次にUIKitのPrivate APIをフックしている関数を見ます。

### Gedatsu.open  

コードは[ここ](https://github.com/bannzai/Gedatsu/blob/97860af03a9ccc8693eb9b2166171ec5a85073c1/Sources/Gedatsu/UIViewExtension.swift#L14)です

```swift
internal func open() {
    _ = dup2(STDERR_FILENO, writer.writingFileDescriptor)
    _ = dup2(reader.writingFileDescriptor, STDERR_FILENO)
    source = DispatchSource.makeReadSource(fileDescriptor: reader.readingFileDescriptor, queue: .init(label: "com.bannzai.gedatsu"))
    source.setEventHandler { ... }
    source.activate()
```

普段iOS開発たちでは使わない関数たちばかりですね。僕も初めて使いました。ここらへんの関数たち説明を書こうと思ったのですが思ったよりボリューミーだったので途中から挫折しちゃいました。(てへ

なので簡易的な説明だけ書いておきます。参考のリンクも貼っておくので興味があったら調べてみてください。簡易的な説明すると 1つめの `dup2` で `writer.writingFileDescriptor` を通して標準エラー出力に書き込めるように。2つ目の `dup2` で 標準エラー出力の書き込み先を `reader.writingFileDescriptor` にしています。`DispatchSource.makeReadSource` では監視したいファイルディスクリプターを渡して、`setEventHandler` で監視対象のイベントのハンドリング、`activate` で監視の開始をしています。

- [標準エラー出力](https://wa3.i-3-i.info/word11638.html)
- [dup](http://www.kis-lab.com/serikashiki/C/C04.html)
- [DispatchSource.makeReadSource](https://developer.apple.com/documentation/dispatch/dispatchsource/2300104-makereadsource)

あと途中まで詳しく書こうとして断念した文章をおいておきます。情報量はあまり変わりませんがもう少し丁寧な言葉づかいになっています。

<details>
<summary> 途中まで詳しく書こうとして断念した文章</summary>

dup2` はファイルディスクリプタを複製するシステムコールの関数です。`duplicate` の略ですね。`2` はきっと引数の数です。ファイルディスクリプタはファイルに対して割り当てられている識別子です。参照先のファイルを表すポインタみたいなやつです。少し説明を省略しますが、1つめの `dup2` で `writer.writingFileDescriptor` を通して標準エラー出力に書き込めるように。2つ目の `dup2` で 標準エラー出力の書き込み先を `reader.writingFileDescriptor` にしています。

readerやらwriterといったものは [Pipe](https://developer.apple.com/documentation/foundation/pipe)のラッパーです。例えばReaderの中身だとこんな具合です。Pipeで用意されたファイルに書き込まれると(2つ目のdup2を思い出してください)ファイルが読み込めるようになります。
```swift
internal class ReaderImpl: Reader {
    let pipe: Pipe = Pipe()
    var writingFileDescriptor: Int32 { pipe.fileHandleForWriting.fileDescriptor }
    var readingFileDescriptor: Int32 { pipe.fileHandleForReading.fileDescriptor }

    func read() -> Data {
        pipe.fileHandleForReading.availableData
    }
}
```

次に [DispatchSource.makeReadSource](https://developer.apple.com/documentation/dispatch/dispatchsource/2300104-makereadsource) で行っていることは `reader.readingFileDescriptor` を監視するために渡しています。きっと指定したファイルディスクリプタに向いているファイルに変更がある場合にデータを読み込むように割当られる仕組みなんでしょう(雑な理解。`source.setEventHandler` で変更を検知したときに実行したい処理内容を書きます。この場合はAutoLayoutでエラーが起きた時点のデータからログを整形してコンソールに流す。それ以外は整形せずにコンソールに流す。といったことをしています。最後の `source.activate()` で監視を始める流れになっています。


通常何かしらのエラーのログや出力は標準エラー出力書かれてコンソール上に出てきます。もっと噛み砕くと 
1. Process X < 標準エラーにログ書くぞ
1. 標準エラー < Process X から書かれるぞ！書かれてるぞ！
1. 標準エラー < 書かれたからコンソール上に出してくれ
みたいな流れです。

</details>

### UIView.engine:willBreakConstraint:dueToMutuallyExclusiveConstraints:

2つ目のUIKitのPrivate APIのフックです。コードは[ここ](https://github.com/bannzai/Gedatsu/blob/97860af03a9ccc8693eb9b2166171ec5a85073c1/Sources/Gedatsu/UIViewExtension.swift#L14)です
こちらはシンプルにObjective-CのRuntime APIを Swiftから呼び出してメソッドを入れ替えることで途中の処理を挟んでいます。

```swift
internal static func swizzle() {
    guard let from = class_getInstanceMethod(UIView.classForCoder(), NSSelectorFromString("engine:willBreakConstraint:dueToMutuallyExclusiveConstraints:")) else {
        fatalError("Could not get instance method for UIView.engine:willBreakConstraint:dueToMutuallyExclusiveConstraints:")
    }
    guard let to = class_getInstanceMethod(UIView.classForCoder(), #selector(UIView._engine(engine:constraint:exclusiveConstraints:))) else {
        fatalError("Could not get instance method for UIView.\(#selector(UIView._engine(engine:constraint:exclusiveConstraints:)))")
    }
    method_exchangeImplementations(from, to)
}
```

このPrivate API UIKitCoreの中のシンボルに含まれていました。 下記のコマンドで確認ができます。
```bash
$ nm /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Library/Developer/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/System/Library/PrivateFrameworks/UIKitCore.framework/UIKitCore | grep engine:willBreakConstraint:dueToMutuallyExclusiveConstraints:
```

上記の2つを組み合わせて AutoLeyoutの制約のエラーが発生したときにコンソールに任意のエラーメッセージを出力する仕組みを実現しています。


## 解脱した人の声

https://twitter.com/fromkk/status/1258347357646643200?s=20

## まとめ
Gedatsuを望む方リポジトリはこちらです。  
https://github.com/bannzai/Gedatsu/  

そして解脱をした人もこれからする人も  
<p style="font-weight:800; font-size:60px;">
スターください 🌟
</p>

おしまい\\(^o^)/
