# UserDefaultsに関するライブラリを作りました
iOSアプリ開発をしていると誰もが一度はお世話になったことがある `UserDefaults` に関するライブラリを作りました。この記事はそのライブラリの紹介記事になります。

公式ドキュメント: https://developer.apple.com/documentation/foundation/userdefaults

## UserDefaultsGenerator
[UserDefaultsGenerator](https://github.com/bannzai/UserDefaultsGenerator)というライブラリを作りました。これは `UserDefaults` のKeyとValueの組み合わせを便利に管理するための手法を提供しています。  

早速ですが使い方を説明してどういうライブラリかを紹介していきます。

## 使い方
まずは `udg.yml` という名前のyamlファイルを用意します。  
そして下記の設定項目があるので適切なものを設定していきます。

```yml
- name: numberOfIndent
  type: Int

- name: UserSelectedDarkMode
  type: Bool
  key: DarkMode

- name: XYZ
  type: Array
```

ymlが用意できたら `$ udg generate` というコマンドを実行します。コマンドを実行したら `UserDefaultsGenerator.generated.swift` というSwiftファイルができています。  
事前に**UserDefaultsGenerator**をインストールしておきましょう。 [mint](https://github.com/yonaskolb/Mint)によるインストールが可能です。

```shell
$ mint install bannzai/UserDefaultsGenerator # install
$ udg generate
$ ls UserDefaultsGenerator.generated.swift
UserDefaultsGenerator.generated.swift
```

中身を見るとUserDefaultsに関する関数やenumが生成されています。

```swift
public enum UDGArrayKey: String {
  case XYZ
}
public enum UDGBoolKey: String {
  case UserSelectedDarkMode = "DarkMode"
}
public enum UDGIntKey: String {
  case numberOfIndent
}

// MARK: - UserDefaults Array Extension
extension UserDefaults {
  public func array(forKey key: UDGArrayKey) -> [Any]? {
    return array(forKey: key.rawValue)
  }
  public func set(_ value: [Any]?, forKey key: UDGArrayKey) {
    set(value, forKey: key.rawValue)
    synchronize()
  }
}
// MARK: - Bool Extension
extension UserDefaults {
  public func bool(forKey key: UDGBoolKey) -> Bool {
    return bool(forKey: key.rawValue)
  }
  public func set(_ value: Bool, forKey key: UDGBoolKey) {
    set(value, forKey: key.rawValue)
    synchronize()
  }
}
// MARK: - Int Extension
extension UserDefaults {
  public func integer(forKey key: UDGIntKey) -> Int {
    return integer(forKey: key.rawValue)
  }
  public func set(_ value: Int, forKey key: UDGIntKey) {
    set(value, forKey: key.rawValue)
    synchronize()
  }
}
```

各yamlの設定については下記の用途になっています。

#### yamlの設定
|  Key  |  Description  |  Required/Optional  |
| ---- | ---- | ---- |
|  name  |  生成されるEnumのcaseの名前。case xxxx の xxxx の部分  |  Required  |
|  type  |  どの型で保存取得をするためのものか  |  Required  |
|  key  |  name とは違うkeyで保存したい場合に設定することでEnumのrawValueをここで設定した値にできる  |  Optional  |


あとは生成されたSwiftファイルをiOSのプロジェクトに含めましょう。それで使用することが出来ます。

## メリット/デメリット
このライブラリを使用するメリットは以下が考えられます。

- 単純なyamlの構造でUserDefaultsに関するメソッドが生成できる
- TypeSafeなUserDefaultsの仕組みが簡単に用意できる
  * key の typo 防止
  * keyとvalueの組み合わせの乖離
  * 上記２つを自分で用意しなくて良い
- 用意されるメソッド自体も単純

逆にデメリットは以下が考えられます

- Swiftの方が書きなれている
- yamlとSwiftで同じ構造を表しているものが２つできる。
- yamlファイルの管理が増える

といったところですかね。 
UserDefaultsの数によって辛いか辛くないか。というのは大きく変わってきそうですね。

## 作ったきっかけ
[iOSDC Reject Conference Day2](https://iosdc-reject-conference.connpass.com/event/137296/)というイベントで `UserDefaultsに関するライブラリをライブコーディングします` という発表をしてこのライブラリができました。
こちらのイベントやライブコーディング等に興味がある方は別の[ブログがある](https://bannzai.hatenadiary.jp/entry/2019/09/22/010027)のでそちらをご覧になってください。  

もともと上記のイベントに発表するか。なんのライブラリを作って発表しよう。と30分でだいたいできるところまで持っていけそうなお題を探していたのがきっかけと言えます。
その中で **UserDefaultsGenerator** のようなアプローチを取っているライブラリは無いのでは。ということに気づき作るに至りました。  

しかし、実際に自他共需要があまりないものを作ったり、技術的なチャレンジができないものを作るほどのモチベーションは持っていませんでした。今回は前者の「需要」というものが知りたいな。と感じて事前にTwitterでアンケートを取ったりした結果が結構面白く、かつ需要ももしかしたら生まれるかもな。と思い作成を決めました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">みなさんが開発に携わっているアプリにはおおよそいくつのUserDefaultsのKeyがありますか</p>&mdash; ＼(^o^)／ (@_bannzai_) <a href="https://twitter.com/_bannzai_/status/1173629194510708737?ref_src=twsrc%5Etfw">September 16, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

これがTwitterで集計したアンケート結果です。アンケートの取り始めた当初はもちろんアプリによりますが、冷静に考えたら1~10個とかそんなアプリがほとんどではないだろうか。と思っていました。実際はそのとおりではあったんですけど、2番目に多い結果として `30 よりも多い`だったのが意外でした。

また、申告しもらった中では100個を超える人も2人いました。100ってすごいですよね。すっごい。(語彙  
[発表で使ったissue](https://github.com/bannzai/UserDefaultsGenerator/issues/1)にも書いてあるので気になる方はそちらをご覧になってください。

## 技術的なチャレンジ
前述で「需要」があるから作ろう。がきっかけだったのですが、技術的に試したこともあるのでよければソースコードも呼んでみてください。個人的に初めて使ったな。っていうものは下記の２つでした。また違う機会にブログに個別にまとめようかと思います。

- ライブラリの内部実装で[Stencil](https://github.com/stencilproject/Stencil)を使ってコード生成しました。
  * Stencilの中でも [SwiftGen](https://github.com/SwiftGen/SwiftGen) で使われている [StencilSwiftKit](https://github.com/SwiftGen/StencilSwiftKit) を使用しました。
  * 使っている場所は[ここ](https://github.com/bannzai/UserDefaultsGenerator/blob/6db889ffe7463384787d97729620aaa8e7d12aab/Sources/UserDefaultsGeneratorCore/Generator.swift#L68)
- Swift Package Manager プロジェクトで[GitHub Actions](https://help.github.com/ja/articles/about-github-actions)と[Codecov](https://codecov.io)を使用してのCIの構築

## まとめ
- スターください
- `UserDefaults` のKeyとValueの管理に便利なライブラリ**UserDefaultsGenerator**を作りました
- スターください
- 世の中には多大なUserDefaultsを使用するアプリもあることがわかりました
- スターください
- ライブコーディングでコア機能を開発した動画もあります
- スターください
- 技術的に初めてのことにも取り組めて満足
- スターください
- スターください
- スターください

この `UserDefaultsGenerator` 最高だぜ。ソースも興味ある。俺もこんなライブラリ作りたい。そう思ったそこのあなた。

<p style="font-weight:800; font-size:60px;">
スターください 🌟
</p>

おしまい\\(^o^)/
