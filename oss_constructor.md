## GoでOSS書いてみました
表題の通りこの記事では `Go` で書いた `OSS` である `constructor` を紹介します。

## constructor
[constructor](https://github.com/bannzai/constructor) は `Go`製の`Go`のコードを生成してくれるシンプルなCLIツールになっています。 **constructor**はいわゆるコンストラクタ関数を生成してくれます。

例えば struct `Foo` が `Bar` という stirng型のフィールドが定義されている場合を考えます。

```go
type Foo struct {
  Bar string
}
```

これをファイルに保存して、 `constructor generate` を実行します。すると下記のようなコンストラクタが自動で生成されます。

```go
func NewFoo(bar string) Foo {
    return Foo {
        Bar: bar,
    }
}
```

**constructor** を使う流れ、大まかな機能はこれで以上です。

## なんで作ったの
今回このOSSを書いたモチベーションは`Go`の構造体を安全に `new` したかったからです。ここでいう安全とはstructを使用する側は定義されているフィールドをすべて使用してインスタンス化したい。そうでない場合はコンパイルエラーになってほしい。そういった意味の安全です。確認のためにベタにstructを書いた場合に置き得る想定している課題を書いていきます。。例には先程の `Foo` を使用します。

```go
func main() {
  foo := Foo {
    Bar: "#main",
  }
}
```

こちらは `Foo` のインスタンスを作って変数に代入しています。こういうコードがいくつかプロジェクト内に存在していたとしましょう。ここで `Foo` にフィールドを一つ追加します。

```go
type Foo struct {
  Bar string
  Piyo int // <- New field
}
```

`Piyo` というフィールドを追加しました。さて、このフィールドは `0` 埋めじゃない値が入っている必要がある。そういうフィールドにしないとだめだと仮定します。その場合各 `Foo` を使っている該当コードを検知する術はありますでしょうか。grep だったり UnitTestかもしれませんね。しかしこれらは見つけるのにはいささか人に依存した方法になります。つまりミスや対応し忘れを招く可能性が大きくなります。 **constrructor** はこの問題の早期発見、またはどのフィールドが `Required` か `Optional` かの判断材料を提供できるライブラリになっています。

**constructor** で生成されるコードは基本はstructが持っているフィールドを網羅する形で関数が引数を取って、構造体に代入して返す関数を作ります。`constructor generate` を再度実行することで `Piyo` フィールドを追加したときも先程の `NewFoo` の関数は引数を取る数が変わることになります。

```go
func NewFoo(bar string, piyo int) Foo {
    return Foo {
        Bar: bar,
        Piyo: piyo,
    }
}
```

最初からこの `NewFoo` を使って `Foo` をインスタンス化していた場合のことを想像しましょう。 `Piyo` フィールドが増えた場合は関数の引数が足りなくてコンパイルエラーになります。つまり`Go`のプログラムが実行できない明らかな間違い状態にすることができます。残念ながら `0` 埋めじゃない値が入っているというロジカル部分の保障はできませんが、それでも単純な間違いを一つ減らすことができたことに価値があると思います。

```go
func main() {
  foo := NewFoo(
    "#main",
    1000,
  )
}
```

しかし、これだと初期化したくないフィールドをコンストラクタに含めたくない気持ちも、まあわかります。(実は`Go`でも**初期化したくない**には出会ったこと無いのですが) そこでコンストラクタの対象から外せる `ignore_constructor`というフィールドにつけられるタグを用意してあります。下記のように対象から外したいフィールド(Piyo)にタグをつけるだけで `NewFoo` メソッドの引数から除外されます。

```go
type Foo struct {
  Bar string
  Piyo int `ignore_constructor:true`
}
```

```go
func NewFoo(bar string) Foo {
    return Foo {
        Bar: bar,
    }
}
```

### 補足
**constructor** を使うで3つの形式のフォーマットのファイルが必要になってきます。

1. `.go`ファイルです。コンストラクタを作る上で`.go`で書かれた`struct`が必要です。
2. `.yaml`ファイルです。どのファイルから読み込んで、どこに出力するか等の設定情報が必要です。
3. `.tpl`ファイルです。`NewFoo`関数をどのように構成するかのテンプレートファイルが必要です。詳しくは[template](https://golang.org/pkg/text/template/)

2, 3 のファイルは `constructor setup` コマンドで設定ファイルの雛形を手に入れたり、テンプレートを入手することができます。

[README](https://github.com/bannzai/constructor)もぜひ見てみてください。