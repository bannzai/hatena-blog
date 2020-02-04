# Goでenumのswitchをcheckできるツールを作りました

Goでenumの**switch**を**check**できるツール [switchecker](https://github.com/bannzai/switchecker) を作りました。

## switchecker
**switchecker** は Goのソースコード内でswitch文で `enum` を用いた値のパターンマッチングにおいてenumで宣言されている `case` をすべて網羅できているかをチェックしてくれるツールです。  
と、ここまで言いましたが、`Go` で `enum` ？と思いますよね。厳密には`Go`で`enum`は存在しないです。ここで `enum` と表現してしまっているものは下記のようなコードになります。連続した定数定義と `iota` を使用したりするアレです。 おそらくみなさん一度は `Go enum` でググって出てきたコードではないでしょうか。

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

これをこのブログでは形式的に `enum` と表現しちゃいます。やりたいことに対してわかりやすいのと「連続した定数定義と `iota` を使用したりするアレ」って書くのは長いし(このテクニックに名前がついていたら教えて下さい)  

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

## モチベーション
使いたいシチュエーションとしてはツールを少し雑に作っていたときに `enum` を用いて関数の引数に渡して `switch` で分岐して。みたいな書き方をしていたら、あとから `enum` の要素を増やす必要が出てきたので増やした。そしたら実行時にちょっとバグった。コンパイルでは検知できないけど、ここも検知してほしいな。作ってみるか。と思って作ってみました。  

まあ実のところ `language`は`int`に名前がついただけなやつなので `case 1000` とか書けてしまいこれは `switchecker` には引っかかりません。。現状はこういった `case` は無視して、`golang,swift...`など、連続で定数定義されたものすべてが一つの`switch`の中に入っているか `check` してくれる機能に絞っています。 名前のついていない `case` 等は実装社の責任としています。


