---
title: 【入門】Nuxt.js + Contentful + Netlify で始める、JAMstack な CMS 構築
tags: JavaScript Vue.js JAMstack contentful Nuxt
author: isihigameKoudai
slide: false
---
## 概要

こんにちは、アジアクエスト株式会社でフロントエンドエンジニアをやってます、[かめぽん](https://twitter.com/kamepon_fe)です。最近はエンジニアリングだけでなく **Vue** や **Nuxt** の導入支援などもしていたので、徐々に Vue を使ってくれる人が増えつつあり、組織的にもこれからどんどんフロントエンド技術を一層浸透させていきたいなと思っております。

最近、JAMstack やヘッドレス CMS という言葉を聞きますが、CMS と聞いて真っ先に思い浮かべるのは **WordPress** だと思います。人によっては好き嫌いあって僕もそんなに得意と言うか好きな技術ではありません。だけど、CMS の需要はかなりあるのでどうしたらもっと扱い回しやすいものがあるか、そんな中「ヘッドレス CMS」がいいんじゃ無いか、もっと話を大きくすると「**JAMstack**」と呼ばれる思想がエンジニアあるいは組織やクライアントに対して良い影響を与え、プロダクトの新規開発だけでなく長期的に成長させていく上で大事なのではと思ってます。

以前、[Nuxt.js + Netlify で爆速構築するサーバーレス SPA 開発入門](https://qiita.com/isihigameKoudai/items/e3b136e9964f1d30d73d)と言う記事を書かせていただきました。そこでは Nuxt+Netlify での静的サイトジェネレート機能でコンポーネント設計で作った資材を静的なファイルに書き出し、自動デプロイするものでした。それだけで完結できればいいのですが、**記事を更新して逐一それを反映させたい！という要望**がそこそこありました（用は CMS、つまり WordPress）。ただ、色々ツラミだったので今回はヘッドレス CMS である Contentful でコンテンツマネジメントしながら、静的サイトとして自動で反映しようと思います。

フロントエンドがまだまだモノを作りやすくなっていますので、本記事が更にキャッチアップの手助けになれば幸いと思い書かせていただきました。今回の制作物は[Github](https://github.com/isihigameKoudai/nuxt-contentful)にも公開していますので、もしわからない場合は真似しながら作ってみてください。

（本記事で内容のインプットからワークまでの内容を記載しているため少々長いですが、一通り読めば基礎的な内容のインプットからJAMstackでの開発が体験できますので、どうかお付き合いいただければと思います。）

## ターゲット

- 手軽かつ本格的にサービス or サイトを公開してみたい人
- JAMstack やサーバーレスなどに興味があり、爆速で体験してみたい人
- WordPress などの従来の CMS ではなく、ヘッドレス CMS で手軽に構築したい人

## メリット

- JAMstack や API ベースでのサービス設計の基礎的な内容の理解
- Netlify や Contentful の簡単な使い方の理解
- Nuxt.js を使った開発の体験

## インプット

### JAMstack

**JAMstack** とは、**javascript, API, Markup** の頭文字をとった造語で、クライアントサイド javascript、再利用可能な API、マークアップによって構成されたアーキテクチャの総称です。

最近のフロントエンドは javascript での構成要素が大半をしめるサービスが多くなり、いずれもアプリケーションの状態やリクエストあるいはレスポンスのハンドリングが js によって制御されています。
また、それによってサーバーサイドは API を返すための機構としての役割が大きくなっています。さらに、特定のクライアントサイドに依存しない 再利用可能な API にすることで、サービスをスケールさせやすくなり且つ疎結合で様々なクライアントの要望に答えられるので横展開もしやすくなります。
マークアップはジェネレーター等により静的な資材として吐き出されたテンプレートになります。リクエストが来てからテンプレートを組立てるよりも静的なデータの方がレスポンスが早いですし、メンテする必要性がグッと減ります。

この概念の登場により、以下のメリットを受けられます。

- より優れたパフォーマンス・・・事前ビルドされ CDN ホスティングされた資材による高いパフォーマンス
- より高いセキュリティ・・・API への抽象化、高度なサードパーティ製 API の活用など
- 安価でスケールしやすい・・・お互いに低依存なので、各所でのチューニングやスケーリングなどが自在
- 良い開発体験・・・サーバー・クライアント間の低依存化、コンテンツ・マーケティングの両方をメンテする必要がなくなる

### Nuxt.js

これはもう説明必要無いかもしれませんが、Vue.js をベースに作られたユニバーサルなフレームワークです。サーバーサイドレンダリングの実装、SPA 開発、静的サイトジェネレートなどが手軽に実行でき規約ベースで開発できるので、エンジニアのレベルによるルールの破綻回避や様々な環境にスピーディに柔軟に対応出来ることが強い部分になっています。create-nuxt-app コマンド一発ですぐに始められます。
<img width="535" alt="スクリーンショット 2018-11-21 0.27.48.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/393142df-ee90-0499-0cd5-34388e1a22ff.png">


### Contentful

「ヘッドレス CMS」と呼ばれる API ベースで構成された CMS で、ビューの機能は一切持たずコンテンツの管理に徹しています。クライアント側から API を呼び出しレスポンスを返すと言うシンプルな構成なので、スケールしやすく、クライアント側に依存しないのでコンテンツの管理に徹することがで出来ます。Contentful 自体は PC/SP だけで無いたくさんのプラットフォームに対して配信する API があるのでクライアントサイドに依存せず開発することも可能としています。
<img width="1011" alt="スクリーンショット 2018-11-21 0.30.47.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/90412164-c325-b1c0-43df-090416c45975.png">



### Netlify

Netlify は静的サイトの高機能なホスティングサービスで、ビルド・デプロイ・ホスティングを全て引き受けてくれます。大きな特徴としては Git サービスと連携することで、指定したブランチに対してコミットまたはマージすることで自動検知しデプロイまで自動で行ってくれます。また、HTTPS 化やドメイン取得などが同画面内で完結するので本番用の作業も簡単に出来ます。
今回は Webhook 機能を使い Contentful からのフックを受け付けて自動デプロイする機能も実装します。


<img width="749" alt="スクリーンショット 2018-11-21 0.29.50.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/b58ac93b-cda7-ceab-54fb-2e72a1e057df.png">

## 構築手順

### Nuxt の導入と Git リポジトリの追加

なにはともあれ create-nuxt-app。
コマンドが使えない場合は事前に`yarn global add @vue/cli`で vue-cli をインストールして、`create-nuxtapp`コマンドを使えるようにしましょう。

```
create-nuxt-app sample-app
cd sample-app
```

次に、Git リポジトリの準備をします。事前に Git リポジトリを Github 等で準備しておきましょう。

```
git init
git add .
git commit -m "コミットメッセージ"
git remote add origin <GitリポジトリURL>
git push -u origin master
```

そこから、良しなにに nuxt 内のコンテンツを作っていきます。今回の場合はあとでContentfulのソースを流し込みますがいったん仮の実装をします。（コードは一部抜粋です）

```html:components/card

<template>
  <article class="card">
    <nuxt-link
      :to="{ name: 'blog-slug'}"
      class="wrapper"
    >
      <h1 class="card_title">記事１</h1>
      <p class="card_text">記事の内容。hogehogehogehogehoge</p>
      <p class="card_date">2018/8/2</p>
    </nuxt-link>
  </article>
</template>
```


```html:pages/index

<template>
  <section class="index">
    <card
      v-for="i in 5"
      :key="i"
    />
  </section>
</template>
 <script>
import Card from '~/components/card.vue'
 export default {
  components: {
    Card
  }
}
</script>

```

```html:pages/blog/\_slug

<template>
  <section class="slug">
    <h1 class="slug_title">
      タイトル
    </h1>
    <p class="slug_date">2018/8/2</p>
    <div>
      記事の内容。hogehogehogehogehogehogehogehoge
    </div>
  </section>
</template>
```

### Contentful の設定とコンテンツ用意

まずはじめに Contentful でログインしましょう。 Github アカウントで簡単にログイン出来ます。

<img width="549" alt="スクリーンショット 2018-11-21 0.52.42.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/5f5ab747-a4f2-fc5a-7a5a-b75e3001cb2e.png">

コンテンツのモデルを追加します。
画面上部のContent modelタブから開き、Add content typeボタンを押すと以下のようにModelを作成することができます。
<img width="768" alt="スクリーンショット 2018-11-21 0.55.59.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/c0c15378-17f5-3cb6-d4ec-d06282f567ca.png">

モデルはレコード1件分のカラムを設定できるので、テキストやメディア・位置情報などを簡単に持たせることが出来ます。
<img width="758" alt="スクリーンショット 2018-11-21 1.00.15.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/7a972d2b-2686-9a4d-60a6-f123dae381de.png">

以下、サンプルですが今回はこんな感じで行きます。
<img width="652" alt="スクリーンショット 2018-11-21 1.01.46.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/e011b1a0-f0de-3c35-b4a8-54524b3c2c56.png">

  先ほどのモデルを使ってコンテンツを追加します。今回は、Nuxt 側でリストで表示したいのでいくつか記事を追加してみましょう。Contentタブを押し、Add entryボタンでコンテンツをどんどん追加していきます。
記事がいくつか追加できたら Contentful 側でのやることは終わりです。

<img width="767" alt="スクリーンショット 2018-11-21 1.03.11.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/10f9938f-bdd9-8b94-403f-3ca6accf33a9.png">

ここからContentfulをnuxtプロジェクトに入れていきます。
contentfulをyarnでaddします！

```
yarn add --dev contentful
```

以下、設定ファイルを準備していきます。今回は環境ごとに環境変数を設置するため`dotenv`を導入します。基本的にenvファイルはオンライン上にあげないため、別途ホスティングサービスで環境設定をする必要がありますが、開発ではdotenvで環境設定を作って試します。

```js:lib/config.js

require('dotenv').config()

function getValidConfig(configEnv, keys) {
  let { config, missingKeys } = keys.reduce(
    (acc, key) => {
      if (!configEnv[key]) {
        acc.missingKeys.push(key)
      } else {
        acc.config[key] = configEnv[key]
      }
      return acc
    },
    { config: {}, missingKeys: [] }
  )

  if (missingKeys.length) {
    throw new Error(`Contentful key is missing : ${missingKeys.join(', ')}`)
  }
  return config
}

module.exports = {
  getConfigForKeys(keys) {
    const configEnv = {
      CTF_BLOG_POST_TYPE_ID: process.env.CTF_BLOG_POST_TYPE_ID,
      CTF_SPACE_ID: process.env.CTF_SPACE_ID,
      CTF_CDA_ACCESS_TOKEN: process.env.CTF_CDA_ACCESS_TOKEN
    }
    return getValidConfig(configEnv, keys)
  }
}

```
Contentfulにアクセスするためnodeモジュールを取り込み、雛形を作ります。  

```js:plugins/contentful.js
const contentful = require('contentful')
const defaultConfig = {
  CTF_SPACE_ID: process.env.CTF_SPACE_ID,
  CTF_CDA_ACCESS_TOKEN: process.env.CTF_CDA_ACCESS_TOKEN
}

module.exports = {
  createClient(config = defaultConfig) {
    return contentful.createClient({
      space: config.CTF_SPACE_ID,
      accessToken: config.CTF_CDA_ACCESS_TOKEN
    })
  }
}
```

nuxtのコンフィグファイルでは主に、Contentfulの取り込みとgenerateの設定、envの設定を行います。特にジェネレートの設定は大事で、SSRやSPAだけであれば不要ですが、ジェネレートは動的ルーティングを解釈出来ないので、動的ルーティングが受け持つパターンの数だけジェネレートが必要です。

```js:nuxt.config.js
const pkg = require('./package')
const { getConfigForKeys } = require('./lib/config.js')
const ctfConfig = getConfigForKeys([
  'CTF_BLOG_POST_TYPE_ID',
  'CTF_SPACE_ID',
  'CTF_CDA_ACCESS_TOKEN'
])

const { createClient } = require('./plugins/contentful.js')
const cdaClient = createClient(ctfConfig)

module.exports = {
  // 省略、、、
  generate: {
    routes() {
      return cdaClient
        .getEntries(ctfConfig.CTF_BLOG_POST_TYPE_ID)
        .then(entries => {
          return [...entries.items.map(entry => `/blog/${entry.fields.slug}`)]
        })
    }
  },
  env: {
    CTF_SPACE_ID: ctfConfig.CTF_SPACE_ID,
    CTF_CDA_ACCESS_TOKEN: ctfConfig.CTF_CDA_ACCESS_TOKEN,
    CTF_BLOG_POST_TYPE_ID: ctfConfig.CTF_BLOG_POST_TYPE_ID
  }
}

```
  
```:.env
CTF_SPACE_ID=<Contentfulで記載されているspace_id>
CTF_CDA_ACCESS_TOKEN=<Contentfulで記載されているアクセストークン>
CTF_BLOG_POST_TYPE_ID=<contentfulで記載されているポストタイプ>

```

ここからは、Contentfulで追加したコンテンツの内容をコンポーネントに流し込むため、Componenntsとpages配下の内容を書き換えます。

```js:pages/index.vue

<template>
  <section class="index">
    <card
      v-for="(post,i ) in posts"
      :key="i"
      :title="post.fields.title"
      :id="post.sys.id"
      :date="post.sys.updatedAt"
    />
  </section>
</template>

<script>
import Card from '~/components/card.vue'
import { createClient } from '~/plugins/contentful.js'

const client = createClient()
export default {
  transition: 'slide-left',
  components: {
    Card
  },
  asyncData({ env, params }) {
    return client
      .getEntries(env.CTF_BLOG_POST_TYPE_ID)
      .then(entries => {
        return {
          posts: entries.items
        }
      })
      .catch(console.error)
  }
}
</script>
```

```html:components/card.vue

<template>
  <nuxt-link
    :to="{ name: 'blog-slug', params: {
      sys: id
    }}"
    class="wrapper"
  >
    <article class="card">
      <h1 class="card_title">{{ title }}</h1>
      <p class="card_text">{{ id }}</p>
      <p class="card_date">{{ date }}</p>
    </article>
  </nuxt-link>
</template>
<script>
export default {
  props: {
    title: {
      type: String,
      default: ''
    },
    id: {
      type: String,
      default: ''
    },
    date: {
      type: String,
      default: ''
    }
  }
}
</script>

```

```html:pages/blog/_slug.vue

<template>
  <section class="slug">
    <h1 class="slug_title">
      {{ article.fields.title }}
    </h1>
    <p class="slug_date">{{ article.sys.updatedAt }}</p>
    <div>
      {{ article.fields.body.content[0].content[0].value }}
    </div>
  </section>
</template>
<script>
import { createClient } from '~/plugins/contentful.js'

const client = createClient()
export default {
  props: {
    id: {
      type: String,
      default: ''
    }
  },
  transition: 'slide-right',
  async asyncData({ env, params }) {
    return await client
      .getEntry(params.sys)
      .then(entrie => {
        return {
          article: entrie
        }
      })
      .catch(console.error)
  }
}
</script>

```

### Nelify 設定と Webhook 連携

  次に、Nelify の設定に入ります。こちらも事前に Github でログインしておいていください。そうしたら、Github に準備したリポジトリと接続するため、設定を進めていきます。以下が、簡易的ではありますがNetlifyの初期設定になります。ちょっとわかりづらい時は[こちら](https://qiita.com/isihigameKoudai/items/e3b136e9964f1d30d73d)を参考にしてください。

- プッシュしたリポジトリ内容を Netlify に取り込むため、`New site from Git` を選択します。
- プッシュしたリポジトリの Git サービスを選択します。（Github でログインしてない場合はログインを求められる可能性があります。）
- あとは、デプロイ時のコマンド、ターゲットブランチ、デプロイ後の参照ディレクトリを入力または選択していきます。

**Contentful と Netlify の Webhook 連携**
  ここからがミソです。Netlify 側で Contentful の環境変数を設定します。ローカルで開発していた時は.env と言うファイルに環境変数を記述していたのですが、Github にあげる際は env ファイルは gitignore に登録しているためそのままだとアップロードされずにジェネレーが失敗しつつ Contentful と接続できません。なので、Netlify 側で環境変数を設定してあげる必要があります。設定自体は key と value を env ファイルの通りに設置するだけなので簡単です。

Netlify側でenvファイルを参照しながら記述してください。先ほどのNetlifyの初期設定の続きでできます。
<img width="835" alt="netlify5.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/c6449643-f9a7-734a-9f05-8aa82f3e9c18.png">

設定が完了するとWeb hooksの情報が記載されます
<img width="768" alt="webhook0.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/6a4b2c80-cbfb-947a-33b3-d5e0e0474158.png">

先ほど、記載されているWebhook URLをContentfulの管理画面でWebhookの設定をします。
Settings > Webhooksで移動し、画面右側のNetlifyを選択します。そこで先ほどのWebhook URLをコピペしcreate webhookのボタンを押せば終わりです。
<img width="900" alt="webhook2.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/27d80a1e-f1e0-992f-b541-b840c4235370.png">

これでWebhook連携は終わりです。
そうすると、Contentfulにコンテンツを追加したタイミングでNetlifyのWebhook URLを叩きます。そして、新たに資材構築をスタートし、新しいコンテンツの内容で逐一ビルドからデプロイまで行います。

今までは、静的サイトは能動的に反映させなければいけませんでした。また、サーバーからロジックとビューがスパゲッティになったテンプレートエンジンでhtmlをいちいち吐き出すのもしんどい状態でした。
ですがこれからは自動でコンテンツが追加・削除・更新されたタイミングで静的データを吐き出すと言う、**静的なんだけど静的じゃない**ちょっと不思議で便利なアーキテクチャになってます。

## まとめ

今回は Nuxt.js・Contentful・Netlify で、お手軽に CMS を構築することが出来ました。これらのサービスはお互いに違うものではありますが、JAMstack と呼ばれるアーキテクチャの元、複数のサービスを組み合わせてプロダクトを作ることが体験出来たと思います。

もちろん、サービスのアーキテクチャは**プロダクトの規模や事業フェーズ・ユーザーの数やアクセス数などにより、適した形**があると思います。ですが、JAMstack は**手軽にスタートでき、スケールさせやすい**ので非常に有用なアーキテクチャと言えます。また、**それぞれを「疎結合」にしておくことは、「壊しやすいサービス」を作る上で非常に大事**な考え方です。

フロントエンド業界の技術動向は非常に変化が激しく、数年もしくは数ヶ月前の技術が廃れたりバージョンが違うと挙動が違ったり、などあります。その中で、フレームワークやライブラリに依存しすぎる構成は新しい技術に対応させる際に大きな負債に成りかねません。じゃあ、「技術を追わなければいい」となると、古い技術を使い続けた結果、今度はメンテが全く出来なかったりそもそも開発できる人がいなかったりと言う問題になります。ちなみに言うと適切な技術を用いなければ適切な人材も集まりづらくなるので、新しい技術をキャッチアップしないのは非常に高いリスクになります。
ですので、Nuxt.js などのフレームワークやこういった新しアーキテクチャの導入は最初小さくスタートしてみると良いかもしれません。

**「人間がもっとも集中すべきところはどこか？」**
それを日々問いながらサービスと作っていくときっと良い未来が作れると僕は思います。

**結局最後はエモさなので**

少し話がそれてしまいましたが、JAMstack 楽しんでいきましょう！

## 参考
https://jamstack.org/
https://qiita.com/nori-k/items/1e789651ec154fdd8bd8
https://qiita.com/hitsuji-haneta/items/7e41bf5cdfde55b826a4


