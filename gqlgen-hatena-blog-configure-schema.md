## gqlgenでSchema firstでAPIを作ってみる
[前回](https://bannzai.hatenadiary.jp/entry/2018/11/28/025612)ではgqlgenでサーバーをlocalhostに立ち上げて、[GraphQL Playground](https://github.com/prisma/graphql-playground)から実際に `Query` が実行されるところまで確認できました。

今回は一度コード生成された後にGraphQLのインタフェースを変えたい場合どう実装していくのか書いていこうと思います。  

`gqlgen` は **Schema first** でGraphQLなAPIを作るためのライブラリです。  
`schema.grpahql`を編集して`Go`のコードを再生成するところまで確認していきましょう。  


## 前回
まず[前回](https://bannzai.hatenadiary.jp/entry/2018/11/28/025612)までのまとめなのですが、下記のQueryを実行すると `Todo.id` の一覧が取得できたことまで確認しました。

### Query
```graphql
query {
  todos {
    id
  }
}
```

### Response
```json
{
  "data": {
    "todos": [
      {
        "id": "1"
      },
      {
        "id": "2"
      },
      {
        "id": "3"
      }
    ]
  }
}
```

今回はこの `Todo` に `title` というプロパティを足していきたいと思います。  

## schema.graphqlの編集
初めに `schema.graphql` を編集していきましょう。今回は `Todo` にプロパティを追加していくので下記のスキームに注目します。

```graphql
type Todo {
  id: ID!
  text: String!
  done: Boolean!
  user: User!
}

type Query {
  todos: [Todo!]!
}
```

### type Todo { ...
 上の `type Todo` から始まるものはレスポンスの形式を表しています。 `ID!` 型の `id` や `String!` 型の `text` というプロパティがあるといった具合ですね。これは `gqlgen generate` の後に `models_gen.go` にコードが吐き出されます。

### type Query { ....
下の `type Query` から始まり `todos: [Todo!]` フィールドを持っている構造は `Request` を投げる時・受け取る時のインタフェースの定義になります。これは `gqlgen generate` の後に　`resolver.go` に記述したインタフェースに対応した `Resolver` のコードが吐き出されます。  

### 編集していく
今のままの `Todo` では `id,text,done,user` の4つのフィールドはレスポンスとして返せますが `title` は返すことができません。  
`title` を返せるようにするためにまず `schema.graphql` を編集して `Todo` に `title` を追加しましょう。  

```graphql  
type Todo {
  id: ID!
  text: String!
  done: Boolean!
  user: User!

  title: String! # ここが追加
}
```

`Scehma first` を謳っている`gqlgen`ではスキーマファイルと実装する`GraphQL`のインtらフェースが合うように開発を進めていきます。  
先ほど、`titlte: String!` の部分を追加しました。続いてこの`schema`の情報を`Go` のコードに反映させていきます。

## Goのコードを生成する
前回では`gqlgen init`した結果、Exmapleの`Go`のコードが作成されました。   
追加したフィールドを`Go`に反映させたい場合は`gqlgen generate`を実行します。

```shell
$ gqlgen generate
```

ちなみに短縮して `gqlgen` とだけ打って実行しても構いません。  

```shell
$ gqlgen
```

実行後 `generated.go`, `models_gen.go`が変更されていれば成功です。   
特に `models_gen.go`の中身を確認してみましょう。  
`Todo` に `Title` というプロパティが追加されていることが確認できると思います。  

```Go
 type Todo struct {
	ID    string `json:"id"`
	Text  string `json:"text"`
	Done  bool   `json:"done"`
	User  User   `json:"user"`
	Title string `json:"title"`
 }
```

次に前回と同様に`resolver.go` の `Todos` メソッドを編集していきましょう。  

```Go
func (r *queryResolver) Todos(ctx context.Context) ([]Todo, error) {
	return []Todo{
		Todo{
			ID:    "1",
			Title: "First",
		},
		Todo{
			ID:    "2",
			Title: "Second",
		},
		Todo{
			ID:    "3",
			Title: "Third",
		},
	}, nil
}
```

これで完了です。[GraphQL Playground](https://github.com/prisma/graphql-playground)から結果を確認してみましょう。  

```
$ go run ./server/server.go
```

サーバーが立ち上がったあら下記のQueryを貼り付けて実行してみましょう。

```graphql
{
  todos {
    id
    title
  }
}
```

```json
{
  "data": {
    "todos": [
      {
        "id": "1",
        "title": "First"
      },
      {
        "id": "2",
        "title": "Second"
      },
      {
        "id": "3",
        "title": "Third"
      }
    ]
  }
}
```

画面はこのような具合になっていると思います。

<img src="https://user-images.githubusercontent.com/10897361/49340575-0e8cec80-f685-11e8-9cde-2f706c5a59d6.png" />

## 終わりに
今回はプロパティの追加するというユースケースを通して `gqlgen` の機能を使い `Schema first` で実装をしていきました。

GitHubにここまでの内容を公開しています。  
https://github.com/bannzai/gqlgen-demo  

branchは `fix/add/field` となっています。前回からの差分としてPRを一つ作ってあります。
https://github.com/bannzai/gqlgen-demo/pull/2

この記事がいいと思ったら、**GitHub**のスターください

おしまい \\(^o^)/

