---
title: Vue CLI UIが想像以上に便利だった話
tags: Vue.js cli JavaScript フロントエンド 開発環境
author: isihigameKoudai
slide: false
---
# 概要
最近、vue-cliがバージョンアップしていて、ふーんとか思いながら流してたんですが、vue-cli uiという機能があることを教えてもらい改めて調べて動かしたら結構感動してしまったので、記事にしてみました。cli-uiどうなん？って思った方のお役に立てていただければと思います。

# プロジェクトを始める
## いつものCLI

とりあえずcliをグローバルインストール！！

```
npm install -g @vue/cli
```

以下コマンドで、バージョンが3.0以上になっているのを確認しましょう。

```
vue --version
```

以下、コマンドでvueのプロジェクトを始めましょう

```
vue create [project-name]
```

ここから、設定やらエコシステムやら選んでいきます。

<img width="350" alt="スクリーンショット 2018-06-22 18.10.37.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/c9a98b68-48e0-5c4c-5acc-bc28b06a8b5d.png">

ここまでは、普通の便利なCLIですね。この後はいつものエディタでいつも通り開発していきます。<img width="327" alt="スクリーンショット 2018-06-22 18.11.58.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/7827c6cd-8cff-32a8-48a4-ae5544d967f1.png">



## CLI UI
ここからがGUIでvueのプロジェクトを作っていきます。
以下のコマンドをうってみてください。

```
vue ui
```

そうすると、ブラウザでなにやら画面が出て来ますね！
![スクリーンショット 2018-06-22 18.17.03.png](https://qiita-image-store.s3.amazonaws.com/0/105049/7cedac0f-9d5f-7b46-5e35-4c734c2a9e1c.png)

これが、vue-cliのui環境です！
ここから、プロジェクトを作成していきます。
プロジェクト名やディレクトリなどを設定していきます。
![スクリーンショット 2018-06-22 18.17.53.png](https://qiita-image-store.s3.amazonaws.com/0/105049/5ce4ef7f-1c32-361d-df6a-abb70db0a42c.png)

BabelとかPWAとか Vuexとかの設定もポチポチするだけで、簡単に構築できます。
（めちゃくちゃ便利ですね）
![スクリーンショット 2018-06-22 18.19.48.png](https://qiita-image-store.s3.amazonaws.com/0/105049/edf33cf0-03fc-453a-7c13-e3786d946738.png)

ESLint周りの設定も簡単です！
![スクリーンショット 2018-06-22 18.20.34.png](https://qiita-image-store.s3.amazonaws.com/0/105049/a81ad8c0-9a56-3719-ef63-fe6103180d31.png)

色々設定したら後はプロジェクトの構築ですね！
ここは少し時間を要します。（ワクワクしますね）
![スクリーンショット 2018-06-22 18.20.50.png](https://qiita-image-store.s3.amazonaws.com/0/105049/8e599cab-6e70-49e0-9f32-f5ca17d3d94d.png)


プラグインの管理や追加もコードに手を出すことなく出来るようになってます！（涙目）
![スクリーンショット 2018-06-22 18.35.20.png](https://qiita-image-store.s3.amazonaws.com/0/105049/5c0153df-7da5-7e30-cc20-f4238901a90d.png)

さてさて、ここからアプリを動かしていくわけですが、どうやって動かすのか、、、
もちろんいつも通りターミナルでコマンド打っても起動するのですが、cli-uiでは以下のように、serve やbuildと言った項目から「タスクの実行」ボタンを押すだけで、アプリが簡単に起動します。便利というかここまで再現したか！という感じですね。
<img width="1005" alt="スクリーンショット 2018-06-23 18.12.17.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/5bc5309e-eb71-92d0-4c6b-70899e61b242.png">
ここまで、vue-cliの新しい機能について説明しました。vueの人気さと共に便利ツールがどんどん生まれてきて、vueの活用の幅が広がってきますし初学者がたくさん増えるといいなと思いました。
