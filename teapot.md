
## 🍵を入れました
表題の通り[Ocha](https://github.com/bannzai/ocha)を[Teapot](https://github.com/bannzai/teapot)に淹れました
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
- ls -la
```

- `source` が 変更点を検知したいファイル指定です。
- `ignore` が 検知した変更点を無視するファイル指定です。
- `execute` が 変更があったファイルに対して実行するシェルスクリプトです。

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

作りたかった理由としてはファイル変更時に実行したいことってたまにあるなー。でも**Ocha**は自分で**Swift**のコードを改めて書く必要があるなー。もっと気軽に使いたいなー。と思って作った感じですね。



