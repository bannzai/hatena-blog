# Goでenumのswitchをcheckできるツールを作りました

Goでenumの**switch**を**check**できるツール [switchecker](https://github.com/bannzai/switchecker) を作りました。

## switchecker
**switchecker** は Goのソースコード内でswitch文で `enum` を用いた値のパターンマッチングにおいてenumで宣言されている `case` をすべて網羅できているかをチェックしてくれるツールです。と、ここまで言いましたが、`Go` で `enum` ？と思いますよね。厳密には`Go`で`enum`は存在しないです。ここで `enum` と表現してしまっているものは下記のようなコードになります。連続した定数定義と `iota` を使用したりするアレです。 おそらくみなさん一度は `Go enum` でググって出てきたコードではないでしょうか。

```go
type language int

const (
	golang language = iota
	swift
	objectivec
	ruby
	typescript
)

func function(x language) {
	switch x {
	case swift:
		println("swift")
	case ruby:
		println("ruby")
	case golang:
		println("golang")
	case typescript:
		println("typescript")
	case objectivec:
		println("objectivec")
	default:
		panic("unexpected default")
	}
}
```

これをこの記事では形式的に `enum` と表現しちゃいます。やりたいことに対してわかりやすいのと「連続した定数定義と `iota` を使用したりするアレ」って書くのは長いし(このテクニックに名前がついていたら教えて下さい)  

## 用法

例えば先程の例に出したコードを2分割にして `enum.go` と `use.go` と２つに分かれているとします。

**enum.go**
```go
package main

type language int

const (
	golang language = iota
	swift
	objectivec
	ruby
	typescript
)
```

**use.go**
```go
package main

func function(x language) {
	switch x {
	case swift:
		println("swift")
	case ruby:
		println("ruby")
	case golang:
		println("golang")
	case typescript:
		println("typescript")
	case objectivec:
		println("objectivec")
	default:
		panic("unexpected default")
	}
}
```

この場合は `-source` に `enum.go` を渡して、 チェックしたいファイルである `use.go` を `-target` に渡して使用をします。

```shell
$ switchecker -source=enum.go -target=use.go
```

globも使えます
```shell
$ switchecker -source=*.go -target=*.go
```

デフォルトでは `*.go` 渡すようになっているのでこの場合は省略も可能です。
```shell
$ switchecker 
```

実行して成功するとこんな感じです。
```shell
$ ls
enum.go main.go use.go

$ switchecker
Succesfull!! $ switchecker -source=*.go -target=*.go
```

ファイルが増えてきたら `,` 区切りで対象を増加できます。
```shell
$ switchecker -source=enum.go -target=use.go,use2.go
```

そしてhelpです。
```
$ switchecker --help                                                                                      
Usage of switchecker:
  -source string
        Source go files are containing enum definition. Multiple specifications can be specified separated by ,. e.g) *.go,pkg/**/*.go  (default "*.go")
  -target string
        Target go files are containing to use enum. Multiple specifications can be specified separated by ,. e.g) *.go,pkg/**/*.go  (default "*.go")
  -verbose
        Enabled verbose log
```

## 用量
プロダクトで使っているCIに導入してこれにチェックさせるのが一つ。もう一つは単純に手元で走らせるのも一つの使いみちかな。と思います。ただエディタで編集したときにチェック。とかは考えてないです。

## モチベーションとか仕様とか
使いたいシチュエーションとしてはツールを少し雑に作っていたときに `enum` を用いて関数の引数に渡して `switch` で分岐して。みたいな書き方をしていたら、あとから `enum` の要素を増やす必要が出てきたので増やした。そしたら実行時にちょっとバグった。コンパイルでは検知できないけど、ここも検知してほしいな。作ってみるか。と思って作ってみました。  

まあ実のところ `language`は`int`に名前がついただけなやつなので `case 1000` とか書けてしまいこれは `switchecker` には引っかかりません。。現状はこういった `case` は無視して、`golang,swift...`など、連続で定数定義されたものすべてが一つの`switch`の中に入っているか `check` してくれる機能に絞っています。 名前のついていない `case` 等は実装社の責任としています。

あとはトップレベルで宣言されたものだけを現在対応しています。`func` の中で同じ構文で書いていても対応していないです。 `func` の中で宣言するケースはきっと多くないだろう。というのと `ast` 等で底を解析して他の型と区別させるのが大変。という理由です。

どこまでサポートしているか、想定しているか。を厳密に知りたい方はコードを読むのが一番正しいとは思うのでぜひ見てみてください。  
ちなみにテストを見るとサポート範囲のイメージがつかめると思います。

- https://github.com/bannzai/switchecker/tree/master/testdata
- https://github.com/bannzai/switchecker/blob/master/check_test.go
- https://github.com/bannzai/switchecker/blob/master/parser_test.go

## 技術
使ったパッケージや構成をさらっと。  

[go/ast](https://golang.org/pkg/go/ast/) を `-source` の引数の解析部分で [golang.org/x/tools/go/packages](https://godoc.org/golang.org/x/tools/go/packages) を `-target` で渡したファイルの型情報と `-source` から作った構造体の情報とマッピングしてチェックする。そういった流れになっています。

`-target` の型情報取得は[go/types](https://golang.org/pkg/go/types/)でも行けるのかなあ。と思ったのですが[Config.Check](https://golang.org/pkg/go/types/#Config.Check)を使ってみた所、渡した引数のGoファイルの型情報は取得できているが、標準のパッケージではない自作のパッケージ等がimportされている場合はそっちの型情報が取得できずに `Config.Error` が返ってきてしまいました。少し試行錯誤しましたがイマイチimport先の型情報の取得を `go/types` を使って取得するやりかたがわからなくて、`golang.org/x/tools/go/packages` の方ではimport先の型情報まで取得できたのでこちらを使用しています。

## まとめ
switchecker のリポジトリはこちらです。  

[https://github.com/bannzai/switchecker:embed:cite]


個人的に `ast` や 型情報みたいなメタな部分を使って効率化を測るツールを作るのが好きで、 その中でもコード量が少なめかつ割と愛用していきそうななツールができたと思っています。
まだできたばかりなので `CI` にまで組み込めてませんが手元で動かす感じでは期待した通りの動きで満足です。近いうちにCIにも組み込んでみてもっと改善してい後と思います。

`enum` のチェッカー欲しかったんだよなあ。これいいかも。`switchecker` って名前がいいね。と思ったそこのあなた

<p style="font-weight:800; font-size:60px;">
スターください 🌟
</p>

おしまい \\(^o^)/
