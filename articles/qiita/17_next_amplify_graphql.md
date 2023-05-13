---
title: Next.js + AWS Amplify + Graphqlで作るサーバーレスアプリケーション環境構築
tags: JavaScript TypeScript Next.js React フロントエンド
author: isihigameKoudai
slide: false
---

## 概要
こんにちは、先日AWSのre:Inventで発表されたAWS Amplify DataStoreがめちゃくちゃ便利すぎて驚きを隠せない、都内でフロントエンドエンジニアをしています[かめぽん](https://twitter.com/kamepon_fe)です。
今まで、VueやNuxtでの開発がメインでやってきておりましてReact、Nextでの開発経験が少なくまたTypescriptでなにか出来ないかなと探しておりました。
最近だとモバイル開発のためのAWSのサービスをインテグレートしたAWS版firebaseのような[Amplify](https://aws.amazon.com/jp/amplify/)というサービスが出ています。内容をみてみると、何やら爆速でサーバーレスアプリケーションが出来そうだなと感じたので、Next.js + AmplifyでTodoアプリを作ってみました。巷では、Nuxt.js + firebaseの組み合わせの記事がかなり多かったですが、こちらのアーキテクチャでも実装出来たのでその方法を体系的にまとめて見ました。
Next + Amplifyでの開発がなんとなくわかるようになると思うので、ぜひ最後まで読んでいただけると嬉しいです。

### Next.jsとは

<img width="945" alt="top_next.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/4add50a6-cae3-af29-86ae-8ad471a0d799.png">

こちらはもはや説明不要かもしれませんが、ZEIT社が開発したサーバーサイドレンダリング対応のwebアプリケーションを構築できるReact製フレームワークです。

pages配下の自動ルーティングやダイナミックルート、SPA/SSRに始まり静的サイトジェネレートなアプリはもちろん、最近だとゼロコンフィグでTypescriptがそのまま使えたり、AMP対応、`api`ディレクトリによるapiの実装などかなりDXがよくなってきています。もちろん導入も手軽にできるので開発スピードを格段に高めることが出来ます。

### AWS Amplifyとは

<img width="1119" alt="top_amplify.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/8339585e-fe94-63f8-c485-47a940c6945e.png">

AWS AmplifyはAWSのサービスを仕様したmBaasの一種で、webアプリケーション作成、設定、開発をかなり簡単にすることが出来、スケーラブルでもあるためサービスの規模に応じてオートスケールさせることも可能です。似たようなサービスではgoogle社のfirebaseがあります。バックエンドの資材を自分で準備しなくてもAmplifyのコマンドで必要なソースコードやモデルなどを準備してくれます。本来であれば、設定やプロビジョニング、分析などを全て自前で行っていかなければいけませんが、選択したもに関してAmplifyはそれらを管理してくれます。認証、オフラインデータ、解析、プッシュ通知、AR/VR、botなどを必要に応じてAWSサービスをアプリケーションに対してインテグレートします。

### Next.js + Amplifyは何が嬉しいのか
#### 初期開発スピードの爆速化
昨今では、マーケットの変化が非常に早くユーザーニーズも目まぐるしく変わっています。それに加えて、サービスやプロダクトにおける提供すべきUXも何が適切かわかりにくくなってきている中で、ビジネスサイドと開発サイドで共通認識として持っていなければならないのが**仮説検証を高速に回し、フィードバックをUXに還元する**ことです。[DX時代とその未来における「ユーザーエクスペリエンス」についての基本を抑える](https://note.com/ishigamekoudai/n/nd3f250311c9a)という記事を書かせていただいたのですが、サービスのローンチだけでなくそこに至る検証もスピード感を持って行うことが重要です。そこで、技術選定やアーキテクチャをどうするのかは悩む部分だと思いますが、いかに早く社会実装するかという観点で見るのそれ自体は早く解決すべき問題です。もちろん軽視すべき問題ではないというのが前提です。

そういった中で、Next.js + Amplifyの組み合わせはそれを解決することができると考えています。両者の環境構築で必要な時間は、独自で設計に応じてAWSサービスの選定をしたり構築することに比べても時間がかからずすぐにlocalhostなどで確認出来ますし、必要に応じてバックエンドサービスを提供してくれます。

#### 適切な型やモデルを定義した上でのオートスケール

Next.jsではゼロコンフィグでTypescriptを動かせますし、AmplifyではAppSyncというサービスを含んでいてGraphqlでのデータのやりとりをします。そこで、必要なデータの型を定義してくれるのでフロントエンドとバックエンドで共通で型を使用することが容易です。
フレームワークが何かよりも最重要ビジネスルールは何かを抑える方が大事で、それを定義した上でオートスケールできるのでデータの保全性を担保しつつ希望に応じて対応することが出来ます。

## Next.jsのセットアップ
早速Next.jsのセットアップを始めていきます。

### ディレクトリ構成
最終的なディレクトリ構成は以下のようになっています。実際のディレクトリから必要な部分だけ掲載してますので、全ファイルを確認したい場合は[Github](https://github.com/isihigameKoudai/next-kits/tree/with-amplify)にコードを準備してますのでぜひ参考にしてみてください。`amplify`と`graphql`ディレクトリに関しては以降の`AWS Amplifyのセットアップ`にて自動生成されます。

```
├─ amplify
│  ├─ #current-cloud-backend
│  │  ├─ amplify-meta.json
│  │  ├─ api
│  │  │  └ todo
│  │  │   ├─ build
│  │  │   ├─ parameters.json
│  │  │   ├─ resolvers
│  │  │   ├─ schema.graphql
│  │  │   ├─ stacks
│  │  │   └─ transform.conf.json
│  │  └─ backend-config.json
│  ├─ backend
│  │  ├─ amplify-meta.json
│  │  ├─ api
│  │  │  └ todo
│  │  │   ├─ build
│  │  │   ├─ parameters.json
│  │  │   ├─ resolvers
│  │  │   ├─ schema.graphql
│  │  │   ├─ stacks
│  │  │   └─ transform.conf.json
│  │  ├─ awscloudformation
│  │  │  └ nested-cloudformation-stack.yml
│  │  └─ backend-config.json
│  └─ team-provider-info.json
├─graphql/
│ ├─queries.ts
│ ├─mutations.ts
│ ├─subscriptions.ts
│ └─schema.json
├─pages/
│ ├─index.tsx
│ └─todo.tsx
├─components/
│ └─templates/
│   ├─head.tsx
│   └─navigation.tsx
├─store/
├─aws-exports.js
├─package.json
├─.gitignore
├─next.config.js
├─next-env.d.ts
└─tsconfig.json
```
### 必要なモジュールのインストールとpackage.jsonの編集
作業ディレクトリが出来たら、以下コマンドで必要なモジュールの準備をしましょう。

```
npm install --save react react-dom next
npm install --save-dev @types/node @types/react
```

インストール出来きたら、package.jsonのscriptsを以下のように編集します。

```json:package.json
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
```

### 各種コンポーネントの準備
ここでは、共通で使うコンポーネントの定義をします。ナビゲーション用とhead部用のコンポーネントを準備します。

```tsx:navigations.tsx

import * as React from 'react';
import Link from 'next/link'

const Navigation: React.FC = () => {
  return (
    <div>
      <Link href="/">
        <p>Index</p>
      </Link>
      <Link href="/about">
        <p>About</p>
      </Link>
      <Link href="/todo">
        <p>Todo</p>
      </Link>
    </div>
  )
}
export default Navigation

```

head用のコンポーネントに関してはお好みで設定してみてください。

```tsx:head.tsx

import * as React from 'react'
import Head from 'next/head'
import info from '../../package.json'

const defaultOGURL = ''
const defaultOGImage = ''

interface Props {
  title: string,
  description?: string,
  url?: string,
  ogImage?: string,
}

const head: React.FC<Props> = props => {
  return (
    <Head>
      <meta charSet="UTF-8" />
      <title>{props.title || ''}</title>
      <meta name="description" content={props.description || info.description} />
      <meta name="viewport" content="width=device-width, initial-scale=1" />
      <meta property="og:url" content={props.url || defaultOGURL} />
      <meta property="og:title" content={props.title || ''} />
      <meta
        property="og:description"
        content={props.description || info.description}
      />
      <meta name="twitter:site" content={props.url || defaultOGURL} />
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:image" content={props.ogImage || defaultOGImage} />
      <meta property="og:image" content={props.ogImage || defaultOGImage} />
      <meta property="og:image:width" content="1200" />
      <meta property="og:image:height" content="630" />

      <link rel="icon" sizes="192x192" href="/static/touch-icon.png" />
      <link rel="apple-touch-icon" href="/static/touch-icon.png" />
      <link rel="mask-icon" href="/static/favicon-mask.svg" color="#49B882" />
      <link rel="icon" href="/static/favicon.ico" />
  </Head>
  )
};

export default head
```

### pagesの準備
次にpagesコンポーネントの準備です。

```tsx:pages/index.tsx

import * as React from 'react';
import Head from '../components/templates/head'
import Navigation from '../components/templates/navigation'

const Index: React.FC = () => {
  return (
    <div>
      <Head title="Index page" />
      <Navigation />
      <p>Hello world</p>
      <p>Index</p>
    </div>
  )
}
export default Index

```

```tsx:pages/todo.tsx

import * as React from "react";
import Head from '../components/templates/head'
import Navigation from '../components/templates/navigation'

const Todo = () => {

  return (
    <div>
      <Head title="todo" />
      <Navigation />
      <h2>Todo with amplify</h2>
    </div>
  )
};

export default Todo;
```

この状態で `npm run dev`のコマンドを実行し、`http://localhost:3000/`にアクセスしてみましょう。以下のような画面にが表示されたら成功です。試しに`/todo`にもアクセスしてみてtodoページが表示されるかもみてみましょう。

<img width="278" alt="スクリーンショット 2019-12-15 18.17.02.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/ef75609e-ec78-cde8-8e31-1ee7be5a8081.png">

Next.jsの準備は一旦以上です。

## AWS Amplifyのセットアップ

### Amplifyのインストールと初期セットアップ

なにはともあれ、amplify/cliのグローバルインストールをします。

```sh
npm install -g @aws-amplify/cli
```

インストールが完了したら、`amplify -v`でバージョンを確認しましょう。以下のような表記になっていたらインストール完了です。

```
Scanning for plugins...
Plugin scan successful
3.17.0
```

次にAWSアカウントの紐付けを行います。コンソールにて

```
amplify configure
```

とコマンドを打つとIAMユーザーを作成するためにブラウザが立ち上がります。アカウントがあればログイン、なければ新規作成を行いましょう。

<img width="558" alt="aws_account.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/bb934dd0-5b57-6343-b126-a24844dbe039.png">

<img width="405" alt="スクリーンショット 2019-11-09 3.09.57.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/9e0d58fb-abef-c08d-e925-eacf196f6528.png">

`accessKeyId`と`secretAccessKeyId`が順番に出るので、それをIAMユーザーの作成画面に貼ります。

<img width="709" alt="スクリーンショット 2019-12-15 18.37.54.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/768d2e48-3745-8566-b401-11c1a8f1e9c9.png">

画面を進めて行くとIAMユーザー作成画面側にアクセスキーIDが表示されるので、コンソール側に貼り付けてEnterを押します。Profile nameを決めた後、Enterを押し`Successfully set up the new user.`のメッセージが出たら完了です。


### Amplifyをプロジェクトで扱えるようにする

ここからは実際にNext.js上でamplifyを使っていく流れを説明していきます。
amplifyバックエンドの様々なサービスを扱えるようにするため、各種リソースをプロジェクトフォルダ内に作成します。
以下コマンドを打ってみましょう。

```
amplify init
```

以下のように、使用言語やフレームワーク、ディレクトリ情報などをインタラクティブ形式で進めていきます。

<img width="806" alt="スクリーンショット 2019-11-09 3.47.46.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/07b06aaa-76be-8649-70ee-1dae9dd8b4fc.png">


` Initializing project in the cloud...`のメッセージが出るとバックエンドの資材を初期化＆準備し始めるので待ちましょう。
`Your project has been successfully initialized and connected to the cloud!`が出たら準備が整う合図になります。

### バックエンドAPI（Graphql）の準備
次にAPIの準備をするために、以下コマンドを打ちます。

```
amplify add api
```

そうすると、プロジェクト内で扱うapiの種類や名前、スキーマの設定をインタラクティブに決めていきます。今回はGraphqlを使用していきます。
以下、質問例になります

<img width="906" alt="スクリーンショット 2019-12-15 18.49.16.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/dd1183ec-072b-bff0-7707-af73f7ceb6fc.png">


<img width="784" alt="スクリーンショット 2019-12-15 18.52.35.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/59066391-daf1-5616-78c1-5ed6eac28df4.png">

`GraphQL schema compiled successfully`のメッセージが出たら、Graphqlでのapi実行に必要なファイル等が自動生成されます。そのあと、schemaファイルに変更をかけたいならば自分で編集します。今回は以下の形式のスキーマにします。

```graphql
type Todo @model {
  id: ID!
  description: String
  isDone: Boolean
}
```

次に以下コマンドでデプロイをします。

```
amplify push
```

ここでも以下のようにインタラクティブに質問を進めていきます。

<img width="763" alt="スクリーンショット 2019-12-15 15.40.57.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/e7aa4056-d665-770e-1bc3-78f96277e698.png">


デプロイが完了すると、バックエンドのリソースが自動生成されます。

### Next.jsとamplifyの結合

必要なバックエンドリソースが揃ったら、いよいよNext.jsの実装に入っていきます。
まずは必要なnpmモジュールをインストールします。

```
npm install --save @aws-amplify/api @aws-amplify/core @aws-amplify/pubsub
```

次にNext.jsのセットアップの時に作った`pages/todo.tsx`を編集します。

基本的にamplifyモジュールのインポートとconfigの処理をかけます。amplifyの処理を書きたいときは基本的にpagesで以下の処理をかけておきます。

```ts:pages/todo.tsx
import Amplify from '@aws-amplify/core';
import PubSub from '@aws-amplify/pubsub';
import API, { graphqlOperation } from '@aws-amplify/api';

import awsmobile from '../aws-exports';

Amplify.configure(awsmobile);
API.configure(awsmobile);
PubSub.configure(awsmobile);
```

次に使用したバックエンドapiを使えるようにします。graphqlで作っているので、そこからquery、mutations, subscriptionsをimportします。ここでは全てインポートしてますが、実際には使う分だけで大丈夫です。

```ts:pages/todo.tsx
import { createTodo, deleteTodo, updateTodo } from '../graphql/mutations';
import { getTodo, listTodos } from '../graphql/queries';
import { onCreateTodo, onUpdateTodo, onDeleteTodo } from '../graphql/subscriptions';
```

少しだけ説明すると、QueryとMutaionsは従来のCRUDに対応させると以下のようになります。

|昨日|CRUD|graphql|
|-----|----|-------|
|作成|CREATE|Mutation|
|取得|READ|Query|
|更新|UPDATE|Mutation|
|削除|DELETE|Mutation|

加えてgraphqlではSubscription（購読）というものがあります。これは、端的にいうとサーバー側からのPushのようなものです。バックエンド側のデータに変更がかかった場合などに検知をして値を知らせてくれるものです。

#### Queryの使い方

```ts:query.ts
export const getTodo = `query GetTodo($id: ID!) {
  getTodo(id: $id) {
    id
    description
    isDone
  }
}
`;
```

importしたlistTodosのクエリを以下のように`API.graphql(graphqlOperation)`を使ってアクセスします。通信が成功するとdataに取得した値が入ってくるのでそれをpropsで渡したり、リストレンダリング用のローカルステートに渡してあげるとReact側でレンダリングすることが出来ます。

```ts
Todo.getInitialProps = async (props) => {
  const data = await API.graphql(graphqlOperation(listTodos));
  return {...props, ...data};
};
```

#### Mutationsの使い方

```ts:mutations.ts

export const createTodo = `mutation CreateTodo($input: CreateTodoInput!) {
  createTodo(input: $input) {
    id
    description
    isDone
  }
}
`;
```

こちらは登録用のファンクションです。`API.graphql(graphqlOperation(createTodo, inputData));`にて第一引数にMutatio、で第二引数で登録するデータを入れます。

```ts:pages/todo.ts
const submitTodo = async (list: Array<string>, todo: string) => {
  const id = Math.floor(Math.random() * Math.floor(1000))
	const inputData = {
    input: {
      id,
      description: todo,
      isDone: false
    }
  }
  
	try {
    await API.graphql(graphqlOperation(createTodo, inputData));
  } catch (e) {
    console.log(e);
  }
};

```

#### Subscriptionの使い方

```ts:subscription.ts
export const onCreateTodo = `subscription OnCreateTodo {
  onCreateTodo {
    id
    description
    isDone
  }
}
`;
```

```ts:pages/todo.tsx
API.graphql(graphqlOperation(onCreateTodo)).subscribe({
	next: e => {
		// 購読する値の取得
		const todo = e.value.data.onCreateTodo
		
		// 値をセットする処理を書く
		...
	}
})
```


## Todoアプリの実装

以下に今回の実装例を載せておきます。

```tsx:pages/todo.tsx
import * as React from "react";
import { useState } from 'react';
import Amplify from '@aws-amplify/core';
import PubSub from '@aws-amplify/pubsub';
import API, { graphqlOperation } from '@aws-amplify/api';

import awsmobile from '../aws-exports';
import {
  createTodo,
  deleteTodo,
  updateTodo
} from '../graphql/mutations';
import { getTodo, listTodos } from '../graphql/queries';
import { onCreateTodo, onUpdateTodo, onDeleteTodo } from '../graphql/subscriptions';

import Head from '../components/templates/head'
import Navigation from '../components/templates/navigation'

Amplify.configure(awsmobile);
API.configure(awsmobile);
PubSub.configure(awsmobile);

interface TodoType {
  id: number,
  description: string
  isDone: boolean
}

interface DataProp {
  data: {
    listTodos?: {
      items: Array<TodoType>
    }
  }
}

const Todo = (props: DataProp) => {
  const { items: todoItems } = props.data.listTodos;

  const [todo, setTodo] = useState('');
  const [list, setList] = useState([]);

  // 新規追加でTodoを追加する
  const submitTodo = async (list: Array<string>, todo: string) => {
    const id = Math.floor(Math.random() * Math.floor(1000))
    const inputData = {
      input: {
        id,
        description: todo,
        isDone: false
      }
    }
    try {
      await API.graphql(graphqlOperation(createTodo, inputData));
    } catch (e) {
      console.log(e);
    }
  };

  // 既存のTodoを削除する
  const deleteItem = async (id) => {

    const deleteData = {
      input: {
        id
      }
    }

    try {
      await API.graphql(graphqlOperation(deleteTodo, deleteData));
    } catch (e) {
      console.log(e);
    }
  };

  return (
    <div>
      <Head title="todo" />
      <Navigation />
      <h2>Todo with amplify</h2>
      <input style={{
        border: 'solid 1px #ddd',
        padding: 10,
        borderRadius: 4,
        fontSize: 18,
        WebkitAppearance: 'none',
        color: '#333'
      }} value={todo} type="text" placeholder="please write todo" onChange={e => setTodo(e.target.value)} />
      <button style={{
        padding: 10,
        background: '#F06292',
        color: '#eee',
        borderRadius: 4,
        fontSize: 18,
        WebkitAppearance: 'none'
      }} onClick={() => submitTodo(list, todo)}>add Todo</button>
      <ul className="ListContainer">{
        todoItems.map( item => (
          <li key={item.id} className="ListItem">
            <span className="title">{item.description}</span>
            <span>{item.isDone}</span>
            <input type="button" value="delete" onClick={() => deleteItem(item.id)} />
          </li>
        ))
      }</ul>
    </div>
  )
};

Todo.getInitialProps = async (props) => {

  const data = await API.graphql(graphqlOperation(listTodos));

  try {
    const client = API.graphql(graphqlOperation(onCreateTodo));
    if ("subscribe" in client) {
      client.subscribe({
        next: e => {
          console.log(e);
        }
      });
    }
  } catch (e) {
    console.error(e);
  }

  return {...props, ...data};
};

export default Todo;

```

試しにあらかじめ以下のようにデータをセットしておきます。（このデータ自体はGraphqlでポストしておいたデータになります。）
<img width="530" alt="スクリーンショット 2019-12-15 20.52.26.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/f8d8e84b-4679-e64d-9283-010cb6dca1e6.png">


その状態で、`npm run dev`でサーバーを起動して`/todo`にアクセスしてみてTodoが表示されていれば成功です。

<img width="510" alt="スクリーンショット 2019-12-15 20.50.52.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/60ac8a96-3d58-e1a1-75f6-89d402d7cc20.png">


## まとめ&感想

ここまで最後ま目を通していただいてありがとうございます。

巷ではサーバーレスWebアプリケーション開発はおそらく非常に人気で、Nuxt.js + Firebaseの情報が非常に多く見受けられます。非常に便利で僕も好きな技術の一つですが、逆にNext.jsがあまり無くバージョンアップにより魅力的な機能が増えてきています。また、React製なのでVueに比べると `壊しやすい`と感じていまして、それでいうとNext.jsもかなりアリかなと思っております。
AWS Amplify自体も元は一つ一つのAWSサービスから成り立っているため使いやすさだけでなくスケーリングやSLAの面でも非常におすすめかと思います。

しかしながらどの技術選定においても、**仮説検証を高速に回し、フィードバックをUXに還元する**ことが大事かなと思っています。その選択肢の一つとして非常に魅力的なので今後少しづつ使っていければと思います。

## 参考
https://aws-amplify.github.io/docs/js/api
https://qiita.com/G-awa/items/a5b2cc7017b1eceeb002
