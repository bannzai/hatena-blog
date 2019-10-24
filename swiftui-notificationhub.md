# SwiftUIで開発したアプリをOSSにしました
WWDC 2019 で発表された[SwiftUI](https://developer.apple.com/videos/play/wwdc2019/204/) でアプリを作り、昨日AppStoreにもリリースされました。  

NotificationHub: https://apps.apple.com/jp/app/notificationhub/id1484099869?l=en. 

この記事では、どんなアプリを作ったのかとSwiftUI x α の技術について躓いたところや解決の仕方をざっくりまとめます。  
項目によっては後日詳細な記事も書きたいな。と思っています。  

また、この NotificationHubはGitHubでOSSとして公開しています。  
こちらも合わせて見てもらえるとわかりやすいと思います。

記事を書いている時点のコミット: [fd39bd11b5bdb937186b2390f86cdae997b37f36](https://github.com/bannzai/NotificationHub/tree/fd39bd11b5bdb937186b2390f86cdae997b37f36)

## NotificationHub
まずはどんなアプリを作ったのか。から紹介していきます。　　
**NotificationHub** では GitHubの未読の通知をアプリ内で一覧して見れる機能を提供しています。

Webからだとここから見れるものですね

<img width=100px src="https://user-images.githubusercontent.com/10897361/67446542-e5ce6680-f64b-11e9-9ea2-553dece6e097.png" />

これを作るかあ。と思った要因として

1. WatchingしているOSSの通知が多い。目を通しておきたいのも多い
2. 公開したリポジトリに稀にissueやPRが 飛んでくることがある。`1` のこともあって埋もれたりする。

がありました。メールで通知を受け取って確認したり、確認したいタイミングでWebの通知を見てもまあ良かったのですが、メールは件数が多くてあまり通知として来てほしくない。Webの通知画面も導線が少し奥で一覧性があまりよくない。

サクッとアクセスして Organization・個人アカウントごとの通知を一覧できる画面も追加できる。確認したい時に開いて見たいのだけ見る。みたいなアプリがほしいな。と思って作りました。

右上の **+** アイコンみたいなところからどのGitHubアカウントの通知一覧画面を作るか選択することが出来ます。ONになっているやつの通知一覧画面が追加されます。

|  通知一覧  |  **+** ボタン押したあと  |
| ---- | ---- |
|  <img width=300px src="https://user-images.githubusercontent.com/10897361/67378947-04d4e600-f5c3-11e9-9cbd-e4f178ab94b8.jpg" />  |  <img width="300px" src="https://appstoreconnect.apple.com/WebObjects/iTunesConnect.woa/ra/ng/app/1484099869/ios/versioninfo/deliverable" />  |







