# UserDefaultsに関するライブラリを作りました
iOSアプリ開発をしていると誰もが一度はお世話になったことがある `UserDefaults` に関するライブラリを作りました。この記事はそのライブラリの紹介記事になります。

公式ドキュメント: https://developer.apple.com/documentation/foundation/userdefaults

## UserDefaultsGenerator
[UserDefaultsGenerator](https://github.com/bannzai/UserDefaultsGenerator)というライブラリを作りました。これは `UserDefaults` のKeyとValueの組み合わせを便利に管理するための手法を提供しています。  

早速ですが使い方を説明してどういうライブラリかを紹介していきます。

## 使い方
まずは `udg.yml` という名前のyamlファイルを用意します。  
そして下記の設定項目があるので適切なものを設定していきます。

```yml
- name: numberOfIndent
  type: Int

- name: UserSelectedDarkMode
  type: Bool
  key: DarkMode

- name: XYZ
  type: Array
```

ymlが用意できたら `$ udg generate` というコマンドを実行します。コマンドを実行したら `UserDefaultsGenerator.generated.swift` というSwiftファイルができています。  
事前にUserDefaultsGeneratorをインストールしておきましょう。 [mint](https://github.com/yonaskolb/Mint)によるインストールが可能です。

```shell
$ mint install bannzai/UserDefaultsGenerator # install
$ udg generate
$ ls UserDefaultsGenerator.generated.swift
UserDefaultsGenerator.generated.swift
```
