# 解脱してAutoLayoutのエラーログをスラスラ読もう
解脱(Gedatsu)をすればAutoLayoutのエラーログもスラスラ読めるようになります

| 解脱前 | 解脱後 |
| ---- | ---- |
|  <img width="100%" src="https://github.com/bannzai/Gedatsu/blob/master/docs/autolayout.png" />  |  <img width="100%" src="https://github.com/bannzai/Gedatsu/blob/master/docs/gedatsu.png" />  |


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
どうやって私が解脱まで到達できたのか少し紹介します。まず `Gedatsu` のエントリポイントとなる関数を見てみましょう。  
https://github.com/bannzai/Gedatsu/blob/099c3c610d72e6501b9d012c8bd78b92fd0cae96/Sources/Gedatsu/Gedatsu.swift#L16  

```swift
internal func open() {
    _ = dup2(STDERR_FILENO, writer.writingFileDescriptor)
    _ = dup2(reader.writingFileDescriptor, STDERR_FILENO)
    source = DispatchSource.makeReadSource(fileDescriptor: reader.readingFileDescriptor, queue: .init(label: "com.bannzai.gedatsu"))
    source.setEventHandler { ... }
    source.activate()
```
普段iOS開発たちでは使わない関数たちです。僕も初めて使いました。`dup2` はファイルディスクリプタを複製するシステムコールの関数です。`duplicate` の略ですね。`2` はきっと引数の数です。ファイルディスクリプタはファイルに対して割り当てられている識別子です。参照先のファイルを表すポインタみたいなやつです。少し説明を省略しますが、1つめの `dup2` で `writer.writingFileDescriptor` を通して標準エラー出力に書き込めるように。2つ目の `dup2` で 標準エラー出力の書き込み先を `reader.writingFileDescriptor` にしています。

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

通常何かしらのエラーのログや出力は標準エラー出力に吐かれます。もっと噛み砕くと 
1. Process X < 標準エラーに書くぞ
1. 標準エラー < 書かれるぞ！  
1. 標準エラー < 書かれたから端末のほうで読み取って出力して

次に [DispatchSource.makeReadSource](https://developer.apple.com/documentation/dispatch/dispatchsource/2300104-makereadsource) で行っていることは `reader.readingFileDescriptor` を監視するために渡しています。きっと指定したファイルディスクリプタに向いているファイルに変更がある場合にデータを読み込むように割当られる仕組みなんでしょう(雑な理解。`source.setEventHandler` で変更を検知したときに実行したい処理内容を書きます。この場合はAutoLayoutでエラーが起きた時点のデータからログを整形してコンソールに流す。それ以外は整形せずにコンソールに流す。といったことをしています。最後の `source.activate()` で監視を始める流れになっています。


