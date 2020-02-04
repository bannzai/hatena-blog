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
