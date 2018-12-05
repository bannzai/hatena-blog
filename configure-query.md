## gqlgenでQueryを追加していこう
[前回](https://blog.hatena.ne.jp/bannzai/bannzai.hatenadiary.jp/edit?entry=10257846132679158230)はフィールドの追加を行なっていきました。今回は`Query`の追加を行なっていきましょう。

## 追加するする前に
`schema.graphql` の下記の部分に注目します。

```
type Query {
  todos: [Todo!]!
}
```

`type Query` のなかに `todos: [Todo!]!` の記述が確認できます。   
この `Query` というキーワードはGraphQLのリクエストを受け取るエントリーポイントの役割を果たすキーワードとなっています。   
この `type Query` 下にアプリケーションで必要となる `Query` (上記でいうと `todos: [Todo!]!`) を追加していくことでリクエストを送る際に記述できる `Query` のキーワードを増やしていくことができます。

では、実際に `type Query` の下に `Query` を増やしていきましょう(ややこしい)

## Query を追加する
`gqlgen` はスキーマファーストのライブラリです。  
ですので、APIのインタフェースの変更がある場合はまず`schema.graphql` を編集していきましょう。  
今回はクライアントが私たIDから `Todo` を一つ取得する `todo(id: ID!)` という `Query` を追加していきたいと思います。  

```
type Query {
  todos: [Todo!]!

  todo(id: ID!): Todo! # ここを追加した
}
```