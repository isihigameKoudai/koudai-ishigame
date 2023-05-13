---
title: Nuxt.js + Netlifyで爆速構築するサーバーレス開発入門
tags: JavaScript Vue.js serverless フロントエンド Nuxt
author: isihigameKoudai
slide: false
---
## 概要
こんにちは、アジアクエスト株式会社でフロントエンドエンジニア(以下、FE)をやっています、かめぽんです。

最近、VueもNuxtも勢いすごくないですか？？
すごいですよね？！？！
そんでもって、FirebaseやAWSを使ったサーバーレスアーキテクチャのアプリケーションも増えてきていますね！

だけど、「覚えること多くてよくわからん！けど、なんか動かしてみたい！」
と、思う方もいると思うのでぜひこの記事を参考にしていただけたらと思います。

サーバーレスSPAの構築方法はたくさんありますが、まずは手軽にモダンフロントエンド周りの知見をインプットしつつ、Nuxt.jsとNetlifyを用いてサーバーレスSPA環境を爆速で作っていければと思います！

## ターゲット
- 手軽かつ本格的にサービスorサイトを公開してみたい人
- FEもしくはFEになりたい人
- サーバーレスを手軽に爆速で体験してみたい人
- PWA・SPAなどを組んでみたい人

## メリット（どんな知見がたまるか、何ができるようになるか）
- SPA、PWA、SSGなどのモダンフロントエンドの知見
- Netlifyを使ったサーバーレス構築方法
- Nuxtやサーバーレス導入での効率化による、読者の方々のこれからの時間の捻出

##インプット
### Nuxt.js
これはもう説明不要かと思いますが、今もっとも勢いのあるjsフレームワークのVue.jsをベースに作られたユニバーサルなフレームワークです。
導入事例も多数上がっており、先日バージョンアップしたことでさらに勢いを増していてます。
SSR（サーバーサイドレンダリング）が手軽に実装出来たり、スターターテンプレートでのSPA（シングルページアプリケーション）環境構築やNuxtに最適化された便利なモジュールが揃っていたりと開発効率をかなりあげることが出来ます。

<img width="678" alt="スクリーンショット 2018-09-27 2.34.16.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/c4cb7bcd-656f-84f0-8dc4-ff956155da49.png">


Nuxtについては詳しくは[こちら](https://ja.nuxtjs.org/)

### Netlify
Netlifyは静的なサイトのホスティングサービスで、ビルドからデプロイ、ホスティングなどを全て賄ってくれる特徴があります。
また、Githubなどとも連携することができ、masterブランチにマージもしくはプッシュしたタイミングで自動的にデプロイしてくれます。他には、HTTPS化やwebhookによりトリガ機能なども備えています。
とにかく便利です。

<img width="640" alt="スクリーンショット 2018-09-27 1.50.22.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/cc23d0bd-2901-e646-54d2-d6affd5b8b21.png">

Netlifyは[こちら](https://www.netlify.com/)

### サーバーレスアーキテクチャ
サーバーレスアーキテクチャはサーバーを自前で準備せず、Web上にある様々なマイクロサービスと呼ばれるサービスを組み合わせたり、ホスティングサービスなどを利用して構築するアーキテクチャのことです。

従来のようなサーバーの保守や管理が必要なくなるので、そこにかかる人件費や労力・サーバー費用などをコストカットできるメリットがあります。

最近だとAWSやFirebaseなどのサービスを使って構築する例が多く、サーバーの実装がいらない部分が大きいので爆速で開発することもサーバーレスのメリットです。

### SPA ( Single Page Application)
Single Page Application略で、従来はMPA(Multiple Page Application)と呼ばれるページ遷移が発生するごとにリクエストを投げ、htmlを受け取って描写して、、、というUX的にはあまりよろしくない仕組みがありました。
しかし、SPAはhtml自体は単一で、遷移などは全てjavascriptベースにすることで素早いページ遷移を行うことが出来ます。

VueをはじめとするReactやAngularなどによってSPA開発は今や主流となっおり、Web上においても素早いレスポンスを実現しています。

### SSG ( Static Site Generator )
Static Site Generatorの略で、SPAのようなjsベースでページ内がゴリゴリと変化するようなものとは少し異なり、名前の通り静的なサイトを構築するジェネレーターです。

普通の静的サイトの構築と何が違うかというと、Markdown形式のファイルで編集してそれをhtml形式に書き出せることです。また、GatsbyやVuepressなどReactやVueなどのコンポーネント志向な開発で構築できることも魅力の一つで、コンポーネント設計などモダンな開発手法を用いながら最終的には静的なサイトをかき出してくれるのがSSGの良いところです。今回扱うNuxt.jsの中にもSSGの機能が搭載されています。


### PWA ( Progressive Web Apps )
Progressive Web Appの略で、簡単にいえばWebアプリだけどネイティブアプリのような使い心地を実現したアプリのことです。

ServiceWorkerと呼ばれるブラウザの機能によりキャッシュの制御を上手く行い、Webアプリなのにオフラインでも動いたりプッシュ通知が使えるようになったりする技術です。また、ホーム画面にショートカットとして追加することにより、本物のネイティブアプリ同様な使い心地を実現することが出来ます。

PWAについては[こちら](https://qiita.com/edwardkenfox/items/4c0b9550ffa48c1f0445)が参考になります

## 構築
さてインプットも終わり、ここからいよいよサーバーレスSPAの構築に入っていきます。
大きな流れとしては、Nuxt.jsでの構築とgitリポジトリへの追加、そのあとそれをGithubとNetlifyを接続させることで自動デプロイをするというながれになります。
ざっくりですが、以下の通りに進めていきます。

- Nuxt.jsの導入
- Githubリポジトリの準備
- Netlifyでの準備
- 公開


※・・・構築はNode.js,npm(もしくはyarn)、gitが導入されていることを前提に説明します。

### Nuxt導入
まずはじめにNuxtの導入です。以下、コマンドでNuxtのスターターテンプレートの展開からサーバーの立ち上げまで爆速でやってしまいましょう！（とても簡単ですね！）
（ここではnpxでnuxt-create-appや他のモジュールをグローバルインストールしなくても扱えるnpxを事前にグローバルインストールしておくことが必要です。）

```
npx create-nuxt-app [pj-name]
cd [pj-name]
npm install
npm run dev
```

ビルドが完了したらブラウザにアクセス（ http://localhost:3000/ ）してみて、以下のような画面が出ていればNuxtの初期構築は以上です。

<img width="423" alt="スクリーンショット 2018-09-27 0.27.34.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/2eb2e576-f181-17ff-ba60-9e59252c5243.png">

### Githubリポジトリの準備
次に、プロジェクトをgitリポジトリにしGithubなどのGitサービスとリモートの登録をしましょう。
あらかじめ、Githubなどで空のリモートリポジトリを作っておきましょう。以下コマンドでGithubにリポジトリの内容が反映されれば成功です。

```
git init
git commit -m "first commit"
git remote add origin https://github.com/user-name/[pj-name].git
git push -u origin master
```

<img width="992" alt="スクリーンショット 2018-09-27 2.23.45.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/bf626b95-6678-1a38-d3ca-4f735e66cce8.png">


### 構築
このあとは自由にプロダクトを作っていきましょう！
今回の例ではシンプルに以下のような構成にします。
プロダクトが出来たら、gitに反映しておきます。

```
pages/
 ├─ index.vue
 ├─ about.vue
 └─ contact.vue
```

### Netlify設定
使用の前に必ずNetlifyのアカウントを作っておいてください。

ログインが成功するとこのような画面に入ります。
画面右上の「New site from Git」を押すとリポジトリ選択画面に移動します。
<img width="608" alt="n0.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/8824ec44-3562-9682-1bd6-8089494ef54e.png">


先ほどの遷移先がこちらになります。
先ほどNuxtの環境をプッシュした先のgitのサービスを選んでください。今回はGithubを使います。

<img width="768" alt="n2.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/74ac3297-350b-bf95-26fb-35ad0d100a87.png">

Githubを選んだら先ほどプッシュした名前のリポジトリを選択してください。

<img width="534" alt="n1.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/81a7abc5-e2e9-c4d1-38af-0b59670e57c7.png">

そのあとに、リポジトリ内でデプロイの対象となるブランチを指定します。基本的にはmasterブランチを対象とします。こうすることによって、普段developブランチやfeatureブランチなどを使い開発しておき、developからmasterへリポジトリ内をクリーンな状態でプッシュ出来ますし、そのタイミングで自動デプロイすることができるからです。
<img width="757" alt="スクリーンショット 2018-09-27 2.48.43.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/ae0d9bc4-4ddd-8b86-041c-8e5d8aa224e5.png">

ブランチを選択したら、今度はビルドの設定です。
ここはどうのようなフレームワークを使ったか、package.jsonのnpm scriptをどう設定したのかによりますが、今回Nuxtから静的サイトジェネレートするので `nuxt generate` の記述をします。
そして、そのジェネレート先のディレクトリがどこなのかを指定します。`nuxt generate`によって`dist`ディレクトリが生成されるので、`dist`を記述します。

<img width="461" alt="n4.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/b0443c68-0490-74d4-9b98-38ebf669d03d.png">

### 準備はこれでOKです、、、ほんとです
あとは、「Deploy site」のボタンを押してしばらく待ちましょう。
デプロイ中はアドレスの部分が、`Site Deploy in progress`の表記が出ると思いますが、それが、水色のURL表記に変わったらデプロイ終了です。
<img width="736" alt="n6.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/0259d83a-5cc1-c739-bf0c-310b4bbb86f8.png">

### そして、、、
URLのリンクを押すと、ブラウザ上で以下のようにNuxtで作ったサイトが表示されていれば作業は終了です。お疲れ様でした。
<img width="666" alt="n7.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/c5711bcf-9972-dc2d-9555-308fc91130f1.png">

### まとめ
サーバーレスSPAの爆速構築はいかがだったでしょうか？
Nuxt.js + Netlifyで非常に手軽にサイトの環境構築からデプロイまで爆速で作れたと思います。
まだまだ発展途上ではあるかもしれませんが、これからのWeb開発の未来が垣間見えるようなものだと思うので、どんどんプロダクトを出していきましょう！

