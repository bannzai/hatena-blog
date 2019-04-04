記事のタイトルはアドベントカレンダーのタイトルをそのまま使わせてもらいました
僕が `alias`登録して使っている中で最も使用頻度が高かったものをご紹介します。  

## git push

`Git`　はもはやデファクトと言っていいくらい普及しているバージョン管理ツールですね。  
`hub` コマンド等 `Git` の関連ツールのコマンドもとても便利ですね。  
そんな `Git` なのですがリポジトリに `push` する際に `git push` の代わりに使っている `alias` を紹介します。  


```shell
alias gcb="git rev-parse --abbrev-ref HEAD" # current branch
alias gp="gcb | xargs git push origin"
alias gpf="gcb | xargs git push origin --force-with-lease"
```

例えば `GitHub` でリポジトリを `Fork` しているような人だと初回の　`git push` は下記のようにエラーが出ると思います。

```shell
$ git push                                                             
fatal: The current branch test/branch has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin your-branch-name
```

上のコマンドを実行するか、省略形で `git push -u origin your-branch-name` とすれば解決するのですが、初回の `push` のたびにブランチ名を入力するのは面倒です。紹介した `alias` を使えばいつのタイミングでも　`gp` と打つだけでリモートリポジトリに変更点を `push` できます。これでブランチ名の入力の手間が省けましたね。やったね


## cd
これはちょっとした `tips` なのですが、よく使うので紹介します。  
紹介する `alias` は下のようなものです。

```shell
alias 0="cd /Users/username/development/my-oss"
alias 1="cd /Users/username/development/product"
alias 2="cd /Users/username/blog/"
```

さて、これはセクションのタイトルにある通り `cd` コマンドを実行する `alias` となっています。  
自分の都合にもなってしまうのですが、業務での開発・趣味での開発。また、複数言語・複数プラットフォーム と同じローカルの環境で行き来することがとても多いです。 ディレクトリを移動したい時にいちいちディレクトリを入力するのはとても手間です。この `alias` を登録しておけば `0` と入力して Enter を押すだけで `cd /Users/username/development/my-oss` に移動します。そして、この数字は普段コマンドの最初として入力することがないものなので心なしか `typo` も少なくていいです。もちろん `cd` 以外のコマンドを登録しておくのもありでしょう。実は僕も `0` は上記の `gp` コマンドを割り当ててます。しかし、手軽にやれてしまうので破壊的なコマンドの登録はやめておきましょう。オススメしません。

## 終わりに
いかがでしたでしょうか。
`awk` コマンドや `function` を使った多少複雑なものも便利なのですが、今回は簡易なものを組み合わせたちょっとしたシェルコマンドの `alias` をご紹介させていただきまいた。

この記事に

