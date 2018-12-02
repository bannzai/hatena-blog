
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


今のままの `Todo` では `id,text,done,user` の4つのフィールドはレスポンスとして返せますが `title` は返すことができません。  
`title` を返せるようにするためにまず `schema.graphql` を編集して `Todo` に `title` を打ち貸しましょう

```graphql  
type Todo {
  id: ID!
  text: String!
  done: Boolean!
  user: User!

  title: String! # ここが追加分
}
```

`Scehma first` を謳っている`gqlgen`ではスキーマファイルと実装する`GraphQL`のインtらフェースが合うように開発を進めていきます。  
先ほど、`titlte: String!` の部分を追加しました。続いてこの`schema`の情報を`Go` のコードに反映させていきます。

## gqlgen generate

