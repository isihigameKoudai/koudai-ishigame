---
title: 光の速さでデプロイ ~ Vue + Firebase hosting ことはじめ ~
tags: Vue.js Firebase JavaScript TypeScript フロントエンド
author: isihigameKoudai
slide: false
---
## 概要
こんにちは、都内でフロントエンドエンジニアをやっています[かめぽん](https://twitter.com/kamepon_fe)です。最近は、Next.js + Firebaseで個人プロダクトを作っているんですが、Firebaseが便利すぎてお米が３杯食べられるくらいに感動しています。
何と言っても、スピード感が出せるのは非常に大きいなと感じています。特に、デプロイまでは本当にあっという間で、ものの数分でデプロイができます。
今回、Vue-cliとfirebase hostingを使ってプロジェクトの立ち上げからデプロイまでを、ササッと体験してもらえればと思います。

## ターゲット
- 光の速さでデプロイしたい人
- Firebase実は使ってない人
- Vue好きな人

## メリット
- SPAを光の速さでデプロイ出来る
- firebase hostingが出来るようになる
- Vue-cli触れる

## 本題
### 開発の流れ
今回の開発の流れは以下のように進めていきます。
事前に、firebase-toolsとVue-cliをグローバルインストールしておいてください。

- vue-cliでプロジェクトの立ち上げ
- firebaseプロジェクトの作成
- プロジェクト内でfirebaseの設定
- デプロイ

### vue-cliでプロジェクトの立ち上げ

なにはともあれ、Vue-cliでプロジェクトを立ち上げましょう！以下コマンドにて、プロジェクトが一撃で作れます。

```
vue create [プロジェクト名]
```

すると、色々質問されると思うのですが任意で進めてみてください。
今回は`Manually select features`を選択し、好きな設定でカスタマイズしていきます。
スペースキーで選択・非選択、エンターで決定できます。
<img width="362" alt="スクリーンショット 2019-05-22 20.55.42.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/014482ff-018a-5214-0607-43a624dbc83c.png">

設定はまだまだ続きます、テストやLinterの設定などをお好みで選択していってください。
<img width="568" alt="スクリーンショット 2019-05-22 20.57.10.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/965c027e-620f-b043-1d58-321b0d601c4b.png">

最後までいき、以下のような画面が出たらプロジェクト作成完了です！

<img width="395" alt="スクリーンショット 2019-05-22 21.00.45.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/cebbe784-c479-36bf-c6df-dc630287cf6a.png">

### firebaseプロジェクトの作成
次にFirebaseの設定です。<https://console.firebase.google.com/u/0/> こちらのサイトにアクセスしてみてください。

すると以下のような画面が出るので、赤枠の部分を押してFirebaseプロジェクトを新規作成してみてください。

<img width="1230" alt="スクリーンショット 2019-05-22 22.19.34.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/c31ff5f6-17b8-b194-95ed-54615b74e28d.png">

プロジェクトの名前を打ち込み、、、
<img width="577" alt="スクリーンショット 2019-05-22 22.19.53.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/546be533-0a41-f504-76a4-685778a9fbbd.png">


プロジェクト作成を押すだけで、、、
<img width="565" alt="スクリーンショット 2019-05-22 22.20.11.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/8ad44d99-dc43-d41a-8c58-f387e1cc4606.png">

プロジェクトの作成が完了しました。
<img width="615" alt="スクリーンショット 2019-05-22 21.25.00.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/d4e51ebe-cadb-2e3a-28af-349335df5e55.png">



### プロジェクト内でfirebaseの設定
はじめに、プロジェクト内でfirebaseを使えるようにするために以下コマンドを売ってみてください。（事前にfirebase-toolsをグローバルインストールしておいてください）

```
firebase init
```
すると、こんな感じでターミナルが燃えていると思います。firebaseがもついろんな機能がありますが、今回は`Hosting`を選んでください。
<img width="564" alt="スクリーンショット 2019-05-22 21.20.13.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/056e769a-e810-dd44-3e6c-72bc8e3800b0.png">

そしたら、あらかじめ作っておいたプロジェクトが表示されるのでそれを選んでください。
<img width="498" alt="スクリーンショット 2019-05-22 23.07.07.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/83631bcb-4723-5a4c-62eb-e9f6bac6058d.png">

そして、firebaseの設定に関する質問を進めていくと以下のような画面が出るので、ここまでくればfirebaseプロジェクトとVue-cliで作ったプロジェクトをつなぎこむことが出来ました。

次に、Vue-cli側でアプリのデプロイをします。その前に`firebase.json`の設定を少し変えます。Vue-cliのビルドしたファイルとfirebaseで読むときの設定を変えにいきます。`npm run build`でVueの資材を吐き出せるのですが、その吐き出したファイルをホスティングするためにその対象ディレクトリを指定します。

```md
{
  "hosting": {
    "public": "dist", //publicからdistへ
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ]
  }
}

```

設定は以上になります！

### デプロイ

そして、ここからデプロイに移っていくわけなんですが、その前にVue-cliの資材をビルドします。以下コマンドを打てみてください。

```
npm run build
```

すると、`dist`配下にファイルが流れ込んできたのがわかると思います。これらが、ホスティングする対象ファイルたちです。


<img width="199" alt="スクリーンショット 2019-05-22 23.25.49.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/a68cb71f-425d-ce6c-ae13-9c41d2fc97a9.png">


次に、以下魔法のコマンドを打つとホスティングされます。

```
firebase deploy
```

この画面が出たらデプロイが完了です！なんて簡単なんでしょうか。
<img width="565" alt="スクリーンショット 2019-05-22 21.30.16.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/02fc12af-4f8b-b3bb-df48-98b17b1c1850.png">

そして、ホスティングのURLにアクセスしてみて、吐き出した画面が表示されれば成功です！

<img width="835" alt="スクリーンショット 2019-05-22 23.33.55.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/da39ceee-794c-a3dd-e8ad-f546cbe55caa.png">




## まとめ
いかがだったでしょうか？非常に簡単でスピーディにデプロイが出来たと思います。実案件でも良いですが、爆速でプロトタイプを作りたいときなどにも非常におすすめなのでジャンジャン使っていきましょう！それでは！


