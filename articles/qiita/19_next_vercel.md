---
title: Vercel + Next.js + micro CMSで作る、jamstackなCMS構築
tags: React Next.js JAMstack CMS フロントエンド
author: isihigameKoudai
slide: false
---
## 概要

こんにちは、都内でフロントエンドエンジニアをしてます[かめぽん](https://twitter.com/kamepon_fe)です。先日、便利なツールしか開発しないことで有名なあのZEITの名前が変わりVercelになった、というニュースは記憶に新しいかと思います。どんどん使い勝手がよくなっていくということで注目していきたいわけですが、昨今ではjavascrptあるいはtypescriptを中心としたアプリケーションが主流になっている中で、jamstackと呼ばれるアーキテクチャが注目されていると思います。jamstack自体は[過去に執筆した記事](https://qiita.com/isihigameKoudai/items/3e45ade7c438176a4cc9)でも紹介していて、基礎的な内容はそちらで確認してみてください。

以前は[Nuxt.js + Contentful + Netlifyの組み合わせ](https://qiita.com/isihigameKoudai/items/3e45ade7c438176a4cc9)でやってみましたが、今回はVercelのプロダクトと国産のヘッドレスCMSで有名なmicro CMSを使って、jamstackなブログサイトを作ってみたいと思います。

![top1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/4c82059b-f307-abca-e61d-879f9e510466.png)

jamstackな構成は気になっている人も多かと思いますが、コンテンツとViewを切り離せることやパフォーマンスの観点でもかなり強いので、この記事で体系的な知識群として参考にしてもらえれば、と思います。今回の記事は割とボリュームが多いですが、一通りやればjamstackなcms構築はなんとなく肌感は掴めると思うので、ぜひ最後まで読んでみてください。

### アークテクチャ

![top2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/f119d73a-798f-6821-ec1f-11203ad602d8.png)


- Next.js
- Vercel (旧ZEIT now)
- micro CMS

今回使う構成は以上の通りで、クライアントサイドはNext.jsを使い、コンテンツマネジメントはmicro CMS、そしてVercelでホスティングをします。そして、コンテンツの公開時や変更がされた場合micro CMSからVercelに対してWebhookでPOSTをし、最新の情報を取り込んだ静的コンテンツとしてホスティングされる、という流れを作っていきます。


URLは以下の様に設計していきます。`/`と`/about`は完全に静的なコンテンツであるため、今回は `/blogs`と`/blogs/:id`について書いていきます。

```
/
/about
/blogs // コンテンツを一覧で表示する
/blogs/:id // 該当するコンテンツの詳細を表示する
```

## micro CMSの準備

最初にコンテンツの管理をしますが、事前にmicro CMSのアカウントを作成し専用のワークスペースを作っておいてください。

### APIの設定

micro CMSでやることは、APIのエンドポイントの作成、スキーマの設定、コンテンツの作成になります。管理画面に入ってもらい、画面左上のコンテンツの右側の+ボタンを押します。

<img width="1258" alt="cms1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/ca5acd3e-6ad7-b185-46ae-4148883c3231.png">

そうするとAPIの設定に入っていくので画面に従って入力していきます。APIのエンドポイントはNext.js側のURL設計と合わせるため同じ名前にします。


### APIスキーマの設定

APIスキーマの設定で、リスト形式とオブジェクト形式の選択がありますが今回はリスト形式を選択します。次にスキーマを設定していきますが、記事のタイトル（title）、日付（date）、本文コンテンツ（content）の３つを用意します。一項目につきフィールドID、表示名、種類

<img width="1030" alt="cms2.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/a8e91607-9123-5a0b-50f9-99fd4cf0ebfa.png">


### コンテンツの作成

APIスキーマの設定が終わると以下の画面が出ると思います。ここから記事を増やしていきます。

<img width="1108" alt="cms3_1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/c5807671-a15f-ce42-eeee-48311c2bed24.png">


とりあえず３つほど作ってみます。

<img width="1098" alt="cms3_2.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/e9ed3b95-0999-923f-f4ae-47bfce2de473.png">


## Next.jsの準備

ここからは実際にコンテンツを受け取り表示するNext.js部分を準備していきます。実際に作ったソースは[Github](https://github.com/isihigameKoudai/next-kits/tree/with-micro-cms)に載せておきますので参考にしてみてください。

ディレクトリ構成は以下の様にしています。ちなみに、Next.jsでは[ダイナミックルーティング](https://nextjs.org/docs/routing/dynamic-routes)をするときに `blogs/[id].tsx`の様に、ファイル名を[]で括ることで動的ルーティングに対応します。また、[]で括られた名前をコード内で取得することが出来ます。後で紹介します。

```
components/ //共通コンポーネントの置き場
service/
 └─blogs.ts micro //CMSにアクセスするためのAPIユーティリティ
pages/
 ├─ index.tsx
 ├─ about.tsx
 └─ blogs/
		 ├─ index.tsx  // 記事コンテンツの一覧表示ページ
		 └─ [id].tsx  // 単体記事のページ
```

### next.config.jsの設定

micro CMSにリクエストを送る際に、`X_API_KEY`をセットします。

```jsx
require("dotenv").config({path: './.env'}); //プロジェクト直下においた環境変数ファイルを読み込む
...
module.exports = {
	...
	env: {
    X_API_KEY: process.env.X_API_KEY
  }
	...
};
```

X_API_KEYはmicro CMSのAPIリファレンスで確認することが出来ます。それをプロジェクト直下の `.env`ファイルに記述しnext.configでセットします。

<img width="1080" alt="cms4.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/5222b5f8-30e6-aa9f-9ae1-72680cb0a5d6.png">


### Service層の準備

Service層では主にAPI取得に関する機能を提供します。getBlogsは記事の全権取得で`/blogs/`で使い、getBlogByは`/blogs/:id`で使います。

```tsx
import axios from 'axios';

const X_API_KEY: string = process.env.X_API_KEY || '';

export const getBlogs = () => (
  axios.get('https://sample-cms.microcms.io/api/v1/blogs', { headers: {
    'Content-type': 'application/json',
    'X-API-KEY': X_API_KEY
  }})
)

export const getBlogBy = id => (
  axios.get('https://sample-cms.microcms.io/api/v1/blogs/' + id, { headers: {
    'Content-type': 'application/json',
    'X-API-KEY': X_API_KEY
  }})
)
```

### コンポーネントの準備

#### componets/templates/navigations.tsx

各ページで使うナビゲーションメニューを用意します。

```tsx
import * as React from 'react';
import Link from 'next/link'

const containerStyle = {
  display: 'flex'
}

const itemStyle = {
  padding: 10
}

const Navigation: React.FC = () => {
  return (
    <div style={containerStyle}>
      <Link href="/">
        <p style={itemStyle}>Index</p>
      </Link>
      <Link href="/about">
        <p style={itemStyle}>About</p>
      </Link>
      <Link href="/blogs">
        <p style={itemStyle}>Blogs</p>
      </Link>
    </div>
  )
}
export default Navigation
```

#### pages/blogs/index.tsx

blogの一覧ページでやることは、serivece/blogsのgetBlogsでmicro CMSに保存している記事一覧を取得し、ページに表示させます。今回はコンテンツが正しく取得できるか検証するためスタイルの実装は最低限にしています。

Netx.jsで新しく追加された[getStaticProps](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation)でページコンポーネントが表示される前のタイミングでデータをフェッチします。getStaticPropsは前のバージョンで使われていたgetInitialPropsに変わるもので、プリレンダリング専用の関数になります。jamstack構成や静的サイトで取り扱う場合に使うと良いでしょう。

もう一つ、[getServerSideProps](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering)もNext.jsの新しいバージョンから推奨してますがこちらはサーバーサイドレンダリングで作動するものなので今回は使いません。

記事一覧を動的にリンク化させるために、`<Link href="/blogs/[id]" as={`/blogs/${id}`}></Link>` 　の記述をします。これを行うことによりパスに記事のidを付与することが出来ます。

```jsx
import { NextPage } from 'next';
import Link from 'next/link'
import React from 'react';

import { getBlogs } from '../../service/blogs';
import Head from '../../components/templates/head';
import Navigation from '../../components/templates/navigation'

...
const BlogItem: React.FC<BlogItemType> = props => {
  const { id, title , date } = props.items;

  return (
    <div style={ BlogItemStyle }>
      <Link href="/blogs/[id]" as={`/blogs/${id}`}>
        <div>
          <span>{ date }</span>
          <span>{ title }</span>
        </div>
      </Link>
    </div>
  )
}

const Blogs: NextPage = (props: any) => {
  const { contents } = props;

  return (
    <div className="blog-container">
      <Head title="Blogs page" description="" url="" ogImage="" />
      <Navigation />
      {
        contents.map( item => <BlogItem items={ item } key={ item.id } />)
      }
    </div>
  )
}

export const getStaticProps = async () => {
  const { data } = await getBlogs();
  return { props: { contents: data.contents } };
}

export default Blogs;
```

#### pages/blogs/[id].tsx

ここでは、`/blogs/index.tsx`から記事をクリックしたときに遷移するページです。こちらでも先ほど説明した `getStaticProps`を使います。このページではパスにidが含まれるので、getStaticPropsでparamsを取得します。そこからパスに含まれるidを取得しgetBlogBy関数でmicro CMSに保存している記事を取得します。

そして、もう一つgetStaticPathsとありますが、こちらはダイナミックルーティングがSSGでは対応していないため、next export時にデータをフェッチして生成するファイルのパスを教えてあげるメソッドになります。こちらを設定しないとjamstackな静的コンテンツを意図した分出力出来なくなるので注意しましょう。

```jsx
import { NextPage } from 'next'

import Head from '../../components/templates/head';
import Navigation from '../../components/templates/navigation'
import { getBlogBy, getBlogs } from '../../service/blogs'

interface BlogItemType {
  id: String,
  createdAt: String,
  updatedAt: String,
  title: String,
  date: String,
  content: String
}

const BlogsItemPage: NextPage<BlogItemType> = (props) => {
  const { title, date, content } = props;
  return (
    <>
      <Head title="Blogs page" description="" url="" ogImage="" />
      <Navigation />
      <section>
        <h2>{ title }</h2>
        <p>{ date }</p>
        <div>{ content }</div>
      </section>
    </>
  )
}

export const getStaticPaths = async () => {
  const { data } = await getBlogs();
  const paths = data.contents.map(item => `/blogs/${item.id}`);
  return {
    paths,
    fallback: true
  }
}

export const getStaticProps = async ({ params }) => {
  const { id } = params;
  const { data } = await getBlogBy(id);
  return { props: { ...data }}
}

export default BlogsItemPage;
```

ここで一旦動作確認するために、`npm run dev` (nextでも良い)でローカル環境で試してみます。（[npm scriptsの例はこちら](https://github.com/isihigameKoudai/next-kits/blob/cfdbe3361753a30cc0e9cc56f4bf632e358200fd/package.json#L6)）

`/blogs/`にアクセスしてみてmicro CMSで準備したデータが取れていれば成功です。

<img width="959" alt="next1.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/7bce182d-6aef-1e8b-8475-70bb3e1e5f0b.png">

一覧が表示されたらどれか記事をクリックしてみて、以下の様に詳細ページに飛べたら成功です。

<img width="1272" alt="next2.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/6a287a29-1cc3-9cfb-496d-d279340bc734.png">

#### 静的サイトジェネレート

先ほど作ったソースがSSG（静的サイトジェネレート）で、正しく出力されるか [npm run export](https://github.com/isihigameKoudai/next-kits/blob/cfdbe3361753a30cc0e9cc56f4bf632e358200fd/package.json#L12)で確認します。コマンドを実行すると以下の様にファイルが書き出され、ダイナミックルーティング部分もid付きで名前がついていれば成功です。実際に書き出されるファイルはデフォルトでoutディレクトリに書き出されます。

![next3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/f04c4fbb-073a-b352-782d-e81042d553c5.png)


## Vercelの準備

Next.jsとmicro CMSでクライアント部分とコンテンツ部分が出来たところでホスティング部分をやっていきます。事前にVercelのアカウントを作り管理画面に入れる様にしておいてください。

import ProjectからあらかじめGithub上に上がっている本プロジェクトを取り込みます。

![vercel1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/082595fa-8ddb-c21f-1114-16b85f2baf1b.png)

途中で作ったディレクトリをVercel上でビルド・デプロイするときの設定をします。

 BUILD COMMANDではSSGをするコマンド、OUTPUT DIRECTORYは実際の生成物、DEVELOP COMMANDは開発用のコマンドを入力します。OUTPUT DIRECTORYで一点注意があり、SSGをするときoutディレクトリに出力されますが、ここでoutを指定してしまうとbuild-manifest.jsonが見つからないとエラーが出るので.next（npm run buildで出力されるディレクトリ）を指定しましょう。

<img width="761" alt="vercel4.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/07f0e5ff-75cc-b066-505b-fb156fe1e5e7.png">

VercelのメニューでSetting > Environment Variablesに遷移します。ローカル環境にてdotenvでX_API_KEYを設定したと思いますが、Vercel上でservice/blogsのAPIを正しく動かすために環境変数を瀬定しましょう。

![vercel5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/4a2abeb6-f606-18ca-207d-bb3d430444a0.png)


Next.jsをVercel上で動かすための設定は一旦完了です。ビルド＆デプロイが自動で走ると思うのでDeploymentのメニューでビルトステータスがReadyになっていることが確認できればビルド成功しています。

<img width="1095" alt="vercel6.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/bf5717fb-0913-eb7a-a5dc-a88614b9f36a.png">

該当するデプロイ結果を確認するために、Visitで実際のデプロイされた資材をみに行ってみましょう。以下の様な画面が出ていれば成功です。 

 <img width="930" alt="vercel7.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/01a6efd2-48ce-acc4-e267-b199e26541a9.png">

## Webhookの設定

先ほどまではNext.jsをVercel上で動かせる様にしただけなのですので、ソースコードのプッシュをすれば再ビルドはかかりますが、micro CMSで記事コンテンツを更新してもサイトの方には反映されません。

それでは実運用で使えないので、コンテンツのステータスが変わったタイミングでデプロイを実行させる様にします。具体的にはmicro CMS側でコンテンツのステータスを変えたタイミングでVercelに対してWebhookでPOSTリクエストを送ります。そのリクエストを受け取ったVercel側で再ビルドされ、その時点で最新のコンテンツを取得した状態でデプロイされます。

### Vercel側の設定

Settings > Git Integration > Deploy Hooksで新しくフックの設定をします。フックの名前と該当するブランチの名前を設定し、Create Hookボタンを押します・

![webhook1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/fe2950f7-f8d9-af49-bb0c-8867b308ef5f.png)


そうするとWebhookのAPIが表示されるのでクリップボードにコピーしましょう。

![webhook2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/a67b04bf-bd8d-cc0e-c01c-7f30c039c097.png)

### micro CMS側の設定

 今度はmicro CMS側の設定ですが、API設定 > Webhookに遷移し、カスタム通知を選択します。

ここで先ほどコピーしたWebhook URLとフックを作動させるタイミングを選択します。今回は、コンテンツの公開時・下書き保存時・削除時にフックを飛ばす様に設定します。micro CMSの設定はこれだけになります。

![webhook3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/17815ab6-ea98-e15d-efbe-14dfbdea216e.png)


### 動作確認

ここから動作検証をしていくので、試しにコンテンツを追加してみます。

<img width="827" alt="webhook3_2.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/9085688a-4306-1fff-984c-0f606e6dac71.png">


コンテンツを追加したらビルドがすぐに自動的に走るので、VercelのPreview Deploymentの画面を確認します。するとステータスがBuildingになっていると思います。これがmicro CMSからフックで自動的にビルド＆デプロイされている証拠になります。

![webhook4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/c2ab1b50-e058-5dfe-7fd7-b0b09fd6fe7f.png)


ステータスがReadyに変わったらサイトを確認してみましょう。以下の様に一覧ページに追加したコンテンツや、そのコンテンツのページに遷移出来たら成功です。これで、jamstackな静的サイトが一通り作れたと思います。

一覧ページ
![webhook5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/b35c6d6b-98e0-c2e7-c870-3699afde3f79.png)

詳細ページ
![webhook6.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/0ad93d3f-0cca-9f4f-4344-a3f48f9e5300.png)


## まとめ

- SSGのNext.jsを使うときはダイナミックルーティング、getStaticProps・getStaticPathsに注意使用
- jamstackであってもなくても、Vercel + Next.jsでかなり開発フローが楽になる
- ヘッドレスCMSは非常に便利で使い勝手が良いので、アーキテクチャに柔軟性を持たせることができる。

いかがだったでしょうか、ちょっと長かったかもしれませんが、jamstackなCMSを一通り作れたと思います。このアーキテクチャは特定のフレームワークに依存するものではなく、壊しやすく作れるので導入する要件によっては強力な一手になるんじゃないかと思います。
