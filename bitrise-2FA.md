# bitriseでAppleIDの2段階認証を突破してiOSアプリを申請
個人開発していたiOSアプリで、bitriseを使ってiOSアプリをApp Store Connectにあげるまで少しややこしいプロセスを踏んだので備忘録がてらブログを書こうかと

## AppleIDの2段階認証
主なハードルはこの **AppleIDの2段階認証** です。少し前に AppleIDの2段階認証が必須になりました。個人でアプリを開発して申請しようとすると自身のアカウントが **Account Holder** の役割を担うはずなので、2段階認証の設定が必須です。

[ここに書いてある](https://developer.apple.com/support/authentication/)
> Only developers with the Account Holder role (formerly the “Team Agent”) in the Apple Developer Program, Apple > Developer Enterprise Program, or iOS Developer University Program need to enable two-factor authentication. 
> Developers who are registered for a free account or who have other team roles are not required to enable two-factor authentication.

この2段階認証を有効にした状態でCIからApp Store Connectにアプリのバイナリをあげるときに、端末に届いた6桁のチェックディジットをCIで使われているマシンから入力する術がなく、App Store Connectにアプリのバイナリをアップロードできねえ。と言うことで困ってました。

たぶん一番素直で恒久的な対策としてはもう一つAppleIDを2段階認証を無効化した状態で作って、そいつに **Account Holder** よりも弱い開発者の権限を与えて、CIではその弱い権限のアカウントでログインしてApp Store Connectにアプリをアップロードする。ってやり方もありますが、なんかアカウントもう一つ作るの嫌だな。と思って **Account Holder** を持つアカウントで App Store Connectにbitrise経由でアプリのバイナリをアップロードする方法を調べました。

## 解決策
[bitriseの公式ブログ](https://blog.bitrise.io/app-store-connect-2fa-solved-on-bitrise)にそのやり方が乗っていました。これで解決。やったね。って思ったのですが、それでも少しつまづいたので簡単にまとめます。(順不同だったり、逆に順番が関係するものもあるかも。そこまでは未検証)

1 . Provisioning等をbitriseにあげておく。具体的な方法はここでは書きませんが。WorkflowのCode Signing から確認してください。

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67211821-59a71e00-f456-11e9-80b2-76934cdd6a10.png" />

2 . [Appleのアカウントの管理ページ](https://appleid.apple.com/account/manage)に行って **App用パスワード** と言う項目からパスワードを発行しましょう。後ほど使うので適当な名前をつけたらパスワードを何処かにメモっておきます。

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67213725-798c1100-f459-11e9-8964-ed77f33ee2ed.png" />

3 . Deploy to iTunesConnectを使います。事前にアプリのArchiveはしてある状態で使うので、Archiveのステップよりも後に追加しましょう。ちなみにArchiveのバイナリのExport形式は **app-store** である必要があります。

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67211663-15b41900-f456-11e9-865a-955e50b42653.png" />

4 . Account Holderの権限を持つ AppleIDとPasswordを **Deploy to iTunesConnect** に必要な変数として入力します。

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67212190-ef42ad80-f456-11e9-895a-6ff74d090c49.png" />

5 . 同じく**Deploy to iTunesConnect** に **App Apple ID** か **App Bundle ID** を入力しましょう。どちらかが必須項目です。

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67212370-3fba0b00-f457-11e9-9e67-6ac5b5b99318.png" />

6 . 同じく**Deploy to iTunesConnect** で **Application Specific Password** を入力しましょう。これは `2` の  [Appleのアカウントの管理ページ](https://appleid.apple.com/account/manage) でメモったパスワードを使用します。[fastlaneで内部的に使用する](https://docs.fastlane.tools/best-practices/continuous-integration/#application-specific-passwords) みたいですね。fastlaneの環境変数 `FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD` に該当するものだと思います。

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67213878-c07a0680-f459-11e9-80a5-ce486f0320c0.png" />

7 . 画面変わって [bitriseのブログ](https://blog.bitrise.io/app-store-connect-2fa-solved-on-bitrise)にも載っていた方法に移ります。[マイページのApple Developer Connection](https://app.bitrise.io/me/profile#/apple_developer_account)から **Account Holder** の権限を持つAppleIDでログインしましょう。

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67213099-662c7600-f458-11e9-8b00-f5075f971a51.png" />

8 . ログインが終わったら同じ画面上にアカウント情報が出ていると思います。右上の ... リーダーをクリックして **Re-authentificate (2SA/2FA)** を選択します。その後に2段階認証のフローが始まるので画面に沿って入力してください。

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67213034-4432f380-f458-11e9-83e7-e872ded9bd83.png" />

9 . 画面が変わってアプリケーションの設定ができる画面に行きます。Teamタブを選択して ** Connected Apple Developer Portal Account ** から先ほど接続して Apple アカウントを選択します。これでこのbitriseに登録しているアプリケーションでは、ここで設定した2段階認証済みのSessionを持ったAppleIDのアカウントが使われることになります(という意味になると思っています)

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67213416-f23e9d80-f458-11e9-963d-f0b4518a5e1a.png" />
<img width="320px" src="https://user-images.githubusercontent.com/10897361/67213526-1c905b00-f459-11e9-8943-88e1ce2d3a53.png" />

あとは個人的に Teamが複数あったのでTeamIDかTeam Name が入力が必要でしたが、上記の 0~9までのステップを行うことで **2段階認証が有効化されていないAppleIDの開発者アカウント** の用意をしなくてもbitriseからiOSアプリをApp Store Connectにアップロードすることに成功しました。やったね


### 補足
> 6 . 同じく**Deploy to iTunesConnect** で **Application Specific Password** を入力しましょう。これは `2` の  [Appleのアカウントの管理ページ](https://appleid.apple.com/account/manage) でメモったパスワードを使用します。[fastlaneで内部的に使用する](https://docs.fastlane.tools/best-practices/continuous-integration/#application-specific-passwords) みたいですね。fastlaneの環境変数 `FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD` に該当するものだと思います。

と書いてありますが、これは直接関係なさそうですが必要なステップになります。  
というのもこのステップを実施しなかった場合は、下記のエラーに遭遇してしまいました。エラーメッセージから読み取った内容を解釈して **Application Specific Password** を用意して、**Deploy to iTunesConnect** に設定したところ上手くいった。ので、おそらく bitriseで使われているfastlaneが `FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD` を使うような処理になっているのでしょう。

<img width="320px" src="https://user-images.githubusercontent.com/10897361/67221791-08535a80-f467-11e9-8b47-f325b5a2fb00.png" />

```
[15:29:39]: Making sure the latest version on App Store Connect matches '1.0' from the ipa file...
[15:29:40]: '1.0' is the latest version on App Store Connect
[15:29:41]: Uploading binary to App Store Connect
[15:30:07]: [Transporter Error Output]: Sign in with the app-specific password you generated. If you forgot the app-specific password or need to create a new one, go to appleid.apple.com (-22938)
[15:30:07]: Transporter transfer failed.
[15:30:07]: 
[15:30:07]: Sign in with the app-specific password you generated. If you forgot the app-specific password or need to create a new one, go to appleid.apple.com (-22938)
[15:30:07]: 
[15:30:07]: Your account has 2 step verification enabled
[15:30:07]: Please go to https://appleid.apple.com/account/manage
[15:30:07]: and generate an application specific password for
[15:30:07]: the iTunes Transporter, which is used to upload builds
[15:30:07]: 
[15:30:07]: To set the application specific password on a CI machine using
[15:30:07]: an environment variable, you can set the
[15:30:07]: FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD variable
```



## まとめ
Organizationであれば  **2段階認証が有効化されていないAppleIDの開発者アカウント**  もわりと気軽に用意できそうですが、**個人**という単位だとなんとなく自分が管理するアカウントを増やす行為自体抵抗がある方もいると思います。  
そんな方には上記のプロセスを踏めばアカウントを新しく用意しなくても済むよ。という記事でした。

おそらく上記の方法だと2段階認証のSessionが切れる時があると思うので、その場合は都度 `8.` のステップで `Re-authentificate` を設定する必要があると思います。私自身まだ問題に直面していないのでこれはあくまで予想になります。

今回は少し興味本位でbitrise上の設定だけでうまくできないかな。と思い、権限の弱いアカウント新しく作るとか。fastlaneの環境変数をローカルでベタに用意したりとか。という方法は選択せずにやりました。思ったよりも踏む手順が多いのとこの方法は特に正攻法というわけではないと思っているので、特にこだわりがなければ個人開発者でも **2段階認証が有効化されていないAppleIDの開発者アカウント** を発行してしまうほうが楽だし恒久対応になるな。と思っているところです。[fastlaneでも簡単に対応するにはこれ](https://docs.fastlane.tools/best-practices/continuous-integration/#separate-apple-id-for-ci)的な書き方として載っているくらいなのでより一般的な方法でもあるのでしょう。AppleIDがCI上等でも扱いやすいようなアカウント管理になるようになってくれると大変助かりますね。今後に期待です。

冒頭でも軽く触れましたがこれは個人的な **備忘録** の用途が一番大きいので少し走り書きで自分が後から見返してもわかる程度の粒度で書いております。間違ったことを書いている、とは思ってはないのですが参考にする際は個人的な備忘録ということも考慮してみてください。細かい説明漏れがある可能性はあります。

おしまい \\(^o^)/


