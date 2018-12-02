# gqlgenとは
[gqlgen](https://github.com/99designs/gqlgen/)とはGraphQLのSchemaベースで[GraphQL](https://graphql.org/learn/)サーバーの開発を進めるためのライブラリです。最近ではでmercariが[mercari tech conf 2018](https://techconf.mercari.com/2018)のサーバーのプログラムをGraphQLで書いていて、「お」と目を見張りました。今回はそんな **gqlgen** の導入部分を記載していきたいと思います

なおGraphQL自体についてはこの記事では割愛します。  
GraphQLについて知りたい方は公式のページがあるので[こちら](https://graphql.org/learn/)をご覧いただければと

## 本題に入る前に
導入部分を書く前にこの記事を書いた時点でのgqlgenの情報を載せておきます。
[この記事を書く時点での最新のtree](https://github.com/99designs/gqlgen/tree/63fc2753eb74995bcb65f64914bbd913114cf4da)
[この記事を書く時点での公式のドキュメント](https://github.com/99designs/gqlgen/tree/63fc2753eb74995bcb65f64914bbd913114cf4da/docs)

## 本題
[最新のドキュメント](https://gqlgen.com/)はここから確認できます。本記事でもこちらを参考にしながら話を進めます。が、多少割愛してまずはGraphQLとして機能するサーバーを立ち上げることを目標にします。[Getting Started](https://gqlgen.com/getting-started/)をまず見ていきましょう。    

まず[ライブラリのインストール](https://gqlgen.com/getting-started/#install-gqlgen)です。 `dep`等のパッケージマネージャーを使っている人は適宜変えてください。

```bash
$ go get -u github.com/99designs/gqlgen github.com/vektah/gorunpkg
```

さて、インストールが終わったら [gqlgen init](https://gqlgen.com/getting-started/#create-the-project-skeleton) を実行してしまいましょう。あらかじめ用意しておいてGitリポジトリを使っている前提で `git status`でどんなファイルができたか確認します。

```bash
$ gqlgen init
Exec "go run ./server/server.go" to start GraphQL server
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        generated.go
        gqlgen.yml
        models_gen.go
        resolver.go
        schema.graphql
        server/

```


[gqlgen init](https://gqlgen.com/getting-started/#create-the-project-skeleton)の実行により下記のファイルたちが作られました。
それぞれの役割を簡単に記述していきます。
-  schema.graphql
    * GraphQLのスキーマを書く
    * 書かれたスキーマが `gqlgen generate` を経て各 `.go` ファイルに `Golang`のコードとして内容が記載される
-  generated.go 
    * `gqlgen generate`でこのファイルの中にGraphQLの処理に必要な処理が吐かれる
    * `schema.graphql`で記載したスキーマ情報やそれに沿った`Resolver`や`Type`の情報
-  gqlgen.yml
    * `gqlgen generate`の際にプロダクションによってカスタマイズしたい場合に記載する設定ファイル
    * 何が設定できるかは[ここ](https://gqlgen.com/config/)を参照
-  models_gen.go
    * `gqlgen generate`の際にGraphQLの`Type`等を `Golang` の `struct` が書かれる
    * `schema.graphql`に記載されたものからコードが作られる
    * `Resolver` 以外のものがここに吐き出されると思ってもらえたらいいかも。 `Type`,`Input`,`Scalar`,`Enum` and so on...
-  resolver.go
    * `gqlgen generate`の際にGraphQLの`Query`や`Mutation`の処理を書くための`Resolver`が用意される。
    * `Resolver`は各`Query(or Mutation)`を処理するためのメソッドを持っている。それを処理する役割と捉えてもらえると。他のフレームワークやライブラリでもこの用途で`Resolver`という名前で出てくる。
    * `Type` が 1つ以上のfieldを持つ場合もここに吐き出される。
-  server/server.go
    * `go run ./server/server.go` でサーバーを試せるようになっている

さて、これで実はデモを試す最低限の準備はできてしまいました。  
`gqlgen init` の時点でExampleに必要なSchemaやそれに沿ったGolangのコードが生成されています。  
Todoという例でドキュメントには[書かれています](https://gqlgen.com/getting-started/#define-the-schema)。今の状態でこれと同じものができています。便利。

ではローカルホストにサーバーを立ち上げ、 `http:localhost:8080` にアクセスします。
```bash
$ go run ./server/server.go
```

下記のようなページが立ち上がれば成功です。
<img src="https://user-images.githubusercontent.com/10897361/49099121-05320780-f2b4-11e8-8d99-96749093e3f1.png" />

これは[GraphQL Playground](https://github.com/prisma/graphql-playground)をCDNを利用して立ち上げています。**GraphQL Playground**は`GraphQL IDE`という立ち位置みたいですね。このツールでQueryをインタラクティブに確認することができます。  
というわけで下記のQueryを書いて動作を確認してみます。

```GrpahQL
query {
  todos {
    id
  }
}
```

<img src="https://user-images.githubusercontent.com/10897361/49099557-1b8c9300-f2b5-11e8-835f-784b55be8c94.png" />

結果がjsonで帰ってきたのが右側の画面を見て確認できると思います。  
しかし、実行してみると何やらエラーが出ています。エラーが起きずに正しくデータが取得できている場合は`data`の項目に`todos`の結果が返ってきて、`errors`の方はフィールド自体が存在しないはずです。

なぜエラーが返ってきたのかなのですが、gqlgenではQueryを処理する役割のResolverのコードが生成された時点では`panic`を呼ぶコードが埋め込まれています。当たり前のことなのですが、`GraphQL`のインタフェースを通った後の内部の処理はその都度書く必要が出てきます。`gqlgen`ではコード生成されたタイミングで下記のようなテンプレなコードが埋め込まれています。これは `resolver.go` を見て確認することができます。

```go
func (r *queryResolver) Todos(ctx context.Context) ([]Todo, error) {
	panic("not implemented")
}
```

では、ここに`[]Todo` 型の値を返すようにコードを下記のように変更してあげましょう。

```go
func (r *queryResolver) Todos(ctx context.Context) ([]Todo, error) {
	return []Todo{Todo{ID: "1"}, Todo{ID: "2"}, Todo{ID: "3"}}, nil
}
```

雑ですが、配列で3つの要素を返してあげています。  
`$ go run ./server/server.go` を再度実行してみて動作を確認してみましょう。先ほど遠同じQueryを投げてみます。
下記のように表示されていれば成功です。

<img src="https://user-images.githubusercontent.com/10897361/49100244-ccdff880-f2b6-11e8-87fc-580908809b8a.png" />


## 終わりに
これで `gqlgen` を導入して GraphQLサーバーを立ち上げるところまでできました。次回は実際にschema.grpahqlを編集して`gqlgen generate`でコード生成をするといったことも書けたらと思っています。

GitHubにここまでの内容を公開しています。  
https://github.com/bannzai/gqlgen-demo  

branchは `introduction/gqlgen` となっています  
https://github.com/bannzai/gqlgen-demo/tree/introduction/gqlgen

この記事がいいと思ったら、**GitHub**のスターください

おしまい \\(^o^)/