## 🍵を入れました
表題の通り[Ocha](https://github.com/bannzai/ocha)を[Teapot](https://github.com/bannzai/teapot)に入れました。  
この記事では[Teapot](https://github.com/bannzai/teapot)を紹介したいと思います。

## Teapotについて
**Teapot**はSwiftで作ったライブラリです。 macOSのCLIのコンソール上で動くプログラムになります。 **Teapot**は指定した(1以上の)ファイルの変更を検知して、予め設定しておいたシェルスクリプトを実行します。 Teapotでできることはファイルの何らかの変更(編集・削除等)をもとに任意のシェルスクリプトを簡単に走らせることができる。そういった役割のプログラムです。 たとえばローカルでブログを編集し保存した時に `$ git add ` して `git commit` をする等を自動で行うことができます。

## インストール
[Mint](https://github.com/yonaskolb/Mint) 経由でインストールできます。

```bash
$ mint install bannzai/teapot
```

## 使い方
**Teapot**の使い方を簡単に紹介します。 Teapotでは`teapot.yml`という設定ファイルが必要になってきます。下記のように `ignore`, `source`, `execute` の3要素があります。  

```yaml
source:
- build/*
- Sources/*.*
- Sources/Teapot/*.*
ignore:
- ".git"
- ".gitignore"
- tests/*
execute: 
- ls -la __FILE__
- echo $HOME
```

- `source` が 変更点を検知したいファイル指定です。
- `ignore` が 検知した変更点を無視するファイル指定です。
- `execute` が 変更があったファイルに対して実行するシェルスクリプトです。
- `__FILE__` は変更されたファイルのパスが渡ってきます。

これは `$ teapot setup` でカレントディレクトリに作成することができます。

```bash
$ teapot setup
🍵 Teapot setup completion. You can edit ./teapot.yml 🍵
```

そして、この用意された `teapot.yml` を自分の都合に合わせて編集して保存します。これで設定ファイルの準備ができました。続いて `$ teapot start` をすることで `teapot` が動き出します。

```bash
$ teapot start
🍵 Teapot start 🍵
```

`$ ls -a` を実行するように `execute` に設定していた場合、`./Sources/Teapot/main.swift` を再度保存した場合に下記のように標準出力されます。

```bash
$ teapot start                                                        
🍵 Teapot start 🍵
-rw-r--r--  1 bannzai  1522739515  367  7  6 22:43 /Users/bannzai/develop/oss/Teapot/Sources/Teapot/main.swift
```

## なぜ Teapot なのか
以前に[Ocha](https://github.com/bannzai/ocha)というライブラリを作って[ブログ](https://bannzai.hatenadiary.jp/)にも記事を書きました。今回の **Teapot** は **Ocha** を `import` して実現しています。**Ocha** が入っているので **Teapot** にしました。  

作りたかった理由としてはファイル変更時に実行したいことってたまにあるなー。でも**Ocha**は自分で**Swift**のコードを改めて書く必要があるなー。もっと気軽に使いたいなー。と思って作った感じです。  

細かく制御したい場合は **Ocha**, 気軽にシェルスクリプトを走らせたい場合は **Teapot** という使い分けができると思います。



## 素敵なロゴを描いてもらいました
[TeapotのREADME](https://github.com/bannzai/Teapot)を見るとわかるのですが素敵なロゴを[井上乃彩 / Noa Inoue](https://twitter.com/noa_design51)さんに描いていただきました。実は**Ocha**の時も井上さんにロゴを描いてもらっていました。今回の**Teapot**にもロゴが欲しかったので引き続きお願いさせてもらいました。**Ocha**との親和性も考えられててとても面白くかわいいロゴになっておりとても良いです。今回の自分の作品である `Ocha` → `Teapot` の歴史的な流れがあって、それがロゴというわかりやすい形で関連付けされているのが自分にとっても嬉しいです。

噂のロゴです。

[f:id:bannzai:20190707000120p:plain]

## おまけ
実はすでに[overlook](https://github.com/wess/overlook)という似たようなツールが存在しています。が、これを動かしたりはしていないです。これを知ったタイミングが少なくとも**Ocha**を完成させたあとだったので**Teapot**も思いついちゃったし作りきっちゃおう。と思って結果的に車輪の再発明になりました。overlookのほうはちゃんとウォッチしていないので詳細な機能の比較はできないのですが、自分でコードを知っているかどうかで扱いやすさや安心感は違うので車輪の再発明だとしても無駄ではなかったかなと思っています。


## 最後に
今回は[Teapot](https://github.com/bannzai/Teapot)というライブラリを紹介しました。これも自作である[Ocha](https://github.com/bannzai/ocha)というライブラリに強く関連されています。**Teapot**は`Swift`で動くCLIツールとなっており、簡易的な`yaml` を用意することで簡単に変更されたファイルに対してシェルスクリプトを実行できるようになっています。コード量自体はそんなに多くないのですらっと読めるかもしれないです。**Teapot** を使ってバグ等を発見したら日本語でいいので[issue](https://github.com/bannzai/teapot/issues)や[PR](https://github.com/bannzai/teapot/pulls)を作ってもらえたら嬉しいです。

あとOSSにロゴ乗っているの最高です。テンション上がります。

このライブラリが便利！最高！クール！ロゴかわいい！と思ったそこの貴方へ

<p style="font-weight:800; font-size:60px;">
スターください 🍵
</p>

おしまい \\(^o^)/
