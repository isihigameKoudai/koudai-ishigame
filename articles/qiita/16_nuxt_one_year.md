---
title: 1年間サーバーレスで運用したNuxt.js製サイトをTypescript移行した話とそこで学んだこと
tags: JAMstack TypeScript JavaScript Vue.js Nuxt
author: isihigameKoudai
slide: false
---
## 概要・前置き
どうも、都内でフロントエンドエンジニアをしてます、[かめぽん](https://twitter.com/kamepon_fe)です。以前、[Nuxt.js + Netlifyで爆速構築するサーバーレス開発入門](https://qiita.com/isihigameKoudai/items/e3b136e9964f1d30d73d)という記事を書きまして、その記事と同じNuxt.js + Netlifyのシンプルな構成で作った、以下のサイトを１年間と３ヶ月ほど運用してきました。撮影からデザイン、インタラクションの実装からデプロイまで一気通貫でやってみました。

https://www.brightanddizain.com/<img width="1106" alt="スクリーンショット 2019-12-03 22.45.40.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/b080a79d-9ff6-5f92-a17f-5ddbcb11c546.png">


Lighthouseの評価もよくサイトがきっかけでお仕事もちょくちょくいただいたりと、おかげさまでここまでくることができました。（PWA対応もしてます）

<img width="935" alt="スクリーンショット 2019-12-01 1.52.16.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/d7256059-fd01-cfd3-73ba-73f6bea769cd.png">

しかしながら、フロントエンド技術の流れが非常に早く陳腐化してきたことやサステナビリティを高めたいとは思いつつも課題が多かったのでこの機にnuxtのtypescript移行とそれに伴う環境基盤の構築をしました。そこで学んだことや大事なことなどをご紹介し、少しでも技術の移行やフロントエンド基盤構築や設計のお役に立てればと思います。
少々長いですが、最後まで読んでいただけると嬉しいです！

### サイトの概要
以下、サイトの技術スタックです。動的な部分は少なく頻繁にメンテもするわけではないので、静的なコンテンツで管理しています。しかし、DXの向上を目指しつつメンテしやすいことが条件ではあり、かつコンポーネント指向開発は必須なためnuxt.jsを使用しています。

- Nuxt.js（nuxt generate）
- Netlify
- Atomic Design
- slack api

### 移行の目的
現状、SFCのVueコンポーネントでサイトを構成していますが、コンポーネントの管理があまりできておらず将来的にさらに陳腐化が進むことが懸念されます。
また、これからくるであろう技術を出来るだけ**実践的に**試せるプレイグラウンド的な立ち位置にしたく整理をしていきます。さらに、Typescriptがこれからデファクトスタンダートのような立ち位置になるのではないかと予想をふみjavascriptから脱却し、型安全による品質とサステナビリティ向上を目指します。

- コンポーネント資産の管理と品質の向上
- プレイグラウンド的立ち位置
- 陳腐化による技術スタックの刷新

## 実際にやったこと・構成のポイント
ここからは実施したものやリファクタをした内容などを説明していきますが、実際に導入した施策はかなり多かったので、**厳選して** nuxt typescript移行をする際の重要な部分を記載します。また、アーキテクチャを考えるときに重要視している**StoreとService層**について少し説明します。

- typescript導入
- Vuex Storeの型定義
- StoreとSeriviceについて

### typescript導入
まず、nuxtのtypescript移行についてですがこちらは 公式でも出ている[typescript.nuxtjs.org](https://typescript.nuxtjs.org/)に従って移行します。ここでやることとしては以下の通りです。

<img width="755" alt="スクリーンショット 2019-12-03 22.42.24.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/8e293eac-b7a6-90a4-9cdc-f6a24604d9c9.png">

- 必要なモジュールの導入
- config系の編集

#### 必要なモジュールの導入

まずはじめにnuxt tsに移行するために、 `@nuxt/types,` `@nuxt/typescript-build` `@nuxt/typescript-runtime` を導入します。 加えて、`ts-loader` もインストールします。`@nuxt/typescript-runtime`は任意ではありますがnuxt.configなどでtsを使う場合には必要なのでインストールします。なお@nuxt/typesは@nuxt/typescript-buildに含まれているので個別にインストールする必要はありません。

```
npm i -D @nuxt/typescript-build ts-loader
```

`npm i @nuxt/typescript-runtime`はプロダクション環境でも必要になるためdependenciesでインストールします。

```
npm i @nuxt/typescript-runtime
```

#### config系の編集
最初にtsconfig.jsonの準備をしましょう。設定はお好みで良いですが以下に例を載せておきます。typesの欄に`@nuxt/types`を追加しておきましょう。

```json:package.json
{
  "compilerOptions": {
    "target": "es5",
    "module": "esnext",
    "moduleResolution": "node",
    "lib": [
      "esnext",
      "esnext.asynciterable",
      "dom"
    ],
    "typeRoots" : ["./type"],
    "allowSyntheticDefaultImports": true,
    "noImplicitAny": false,
    "esModuleInterop": true,
    "allowJs": true,
    "sourceMap": true,
    "strict": true,
    "noEmit": true,
    "rootDir": "./src/",
    "baseUrl": "./src/",
    "paths": {
      "@*": [
        "./*"
      ],
      "*": [
        "*"
      ]
    },
    "types": [
      "@types/node",
      "@nuxt/types"
    ]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.vue",
    "src/**/*.spec.ts",
    "src/**/*.spec.tsx",
    "src/**/*.test.ts",
    "src/**/*.test.tsx"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

次に nuxt.configの設定ですが、先ほど `@nuxt/typescript-runtime`を導入したので早速 nuxt.config.jsの拡張子を `.ts`に変えます。また、nuxtConfigに型をつけるので@nuxt/typesでtypeをつけます。

```ts
import { Configuration as NuxtConfiguration } from '@nuxt/types'

...

const nuxtConfig: NuxtConfiguration = {
  mode: 'universal',
  ...
  build: {
    extend(config: any, { isDev, isClient }) {
      const tsLoader = {
        loader: 'ts-loader',
        options: {
          appendTsSuffixTo: [/\.vue$/],
          context: __dirname,
          configFile: 'tsconfig.json'
        }
      }

      for (let rule of config.module.rules) {
        if (rule.loader === 'vue-loader') {
          rule.options.loaders = {
            ...rule.options.loaders,
            ts: tsLoader
          }
        }
      }
    }
  },
  buildModules: [
    [
      '@nuxt/typescript-build', // buildMudulesに@nuxt/typescript-buildを追加します。
      {
        typeCheck: true,
        ignoreNotFoundWarnings: true
      }
    ]
  ]
}
export default nuxtConfig
```

次に、npmコマンドからtsを動かせるようにするためにpackage.jsonのscriptsを編集します。今までは `nuxt build`などで動きましたがts環境では `nuxt-ts`を使用するのでnpm scripts内のコマンドを書き換えましょう。

```json:package.json
{
	...
  "scripts": {
    "dev": "nuxt-ts --spa",
    "build": "nuxt-ts build",
    "start": "nuxt-ts start",
    "generate": "nuxt-ts generate"
		...
	}
	...
}
```

ビルド周りの設定は以上なので、`npm run dev`のコマンドを実行し `http://localhost:3000/`にアクセスしてサイトが表示されたら成功です。

### Vuex Storeの型定義
次にStoreのtypescript対応になりますが、Storeの型定義に関しては@takepepeさんの [ts-nuxtjs-express](https://github.com/takefumi-yoshii/ts-nuxtjs-express)を参考にさせていただいてます。圧倒的に感謝です！

NuxtのVuex Storeでやることとしては以下の通りになります。

- typesの準備
- Vuexの型の拡張
- storeの型付け

最初にVuexにおけるtypesの準備をします。Storeはモジュールモードで以下のようなディレクトリ構成にします。

```
├─store/
  ├─ contact/
    ├─ index.ts  // contactのstoreモジュール本体
    └─ type.ts  // Storeの型定義ファイル
```

ここで、nuxtが型定義のファイルまでstore配下のファイルを自動的にstoreに認識してしまうため `.nuxtignore`ファイルを作り自動的にstoreに認識されないようにします。

```:.nuxtignore

store/**/type.ts

```

次にcontact storeの型定義ファイルを作ります。（記事に収めるるため実際のパラメータよりも少なくしています）
interfaceの名前が略称になってますが、それぞれ S(State)、 G(Getters)、 RG(RootGetters)、 M(Mutations)、 RM(RootMutations)、 A(Actions)、 RA(RootActions)の意味になります。

```ts:store/contact/type.ts
export interface S {
  name: string
  tel: string
  message: string
}

export interface G {
  name: string
  tel: string
  message: string
  isErrName: boolean
  isErrTel: boolean
  isErrMessage: boolean
}

export interface RG {
  'contact/name': G['name']
  'contact/tel': G['tel']
  'contact/message': G['message']
  'contact/isErrName': G['isErrName']
  'contact/isEreTel': G['isErrTel']
  'contact/isErrMeassage': G['isErrMessage']
}

export interface M {
  SET_NAME: string
  SET_TEL: string
  SET_MESSAGE: string
}

export interface RM {
  'contact/SET_NAME': M['SET_NAME']
  'contact/SET_TEL': M['SET_TEL']
  'contact/SET_MESSAGE': M['SET_MESSAGE']
}

export interface A {
  setName: string
  setTel: string
  setMessage: string
  resetContacts: void
  sendContacts: void
}

export interface RA {
  'contact/setName': A['setName']
  'contact/setTel': A['setTel']
  'contact/setMessage': A['setMessage']
  'contact/resetContacts': A['resetContacts']
  'contact/sendContacts': A['sendContacts']
}

```

次にVuexの型の拡張に入ります。以下のようにプロジェクトのルートに `types`ディレクトリを作成しVuexの型を拡張していきます。

```
├─types/
  ├─ vuex/
    ├─ index.d.ts  // プロジェクト全体に影響する実際に呼び出されるVuexの型定義
    ├─ root.ts  // プロジェクト固有のstoreで定義したtypesをVuexにつなぎこむ場所
    └─ type.ts  // Storeの型定義ファイル
```

index.d.tsでは主に共通的に使えるtype.tsとプロジェクト固有のルールを含んだroot.tsをインポートします。このプロジェクトでVuexを使う場合このファイルが読み込まれます。

```ts:types/vuex/index.d.ts
import './root'
import './type'
```

こちらはプロジェクト固有のルールを含んだVuexの型拡張になります。

```ts:types/vuex/root.ts
import 'vuex'
import * as Contact from '../../store/contact/type'
import * as View from '../../store/view/type'

declare module 'vuex' {
  type RootState = {
    contact: Contact.S
    viwe: View.S
  }
  type RootGetters = Contact.RG
  type RootMutations = Contact.RM
  type RootActions = Contact.RA
}

```

以下はVuexで共通的に使われる型拡張です。

```ts:types/vuex/type.ts
import 'vuex'

declare module 'vuex' {
  type Getters<S, G> = {
    [K in keyof G]: (
      state: S,
      getters: G,
      rootState: RootState,
      rootGetters: RootGetters
    ) => G[K]
  }

  type Mutations<S, M> = { [K in keyof M]: (state: S, payload: M[K]) => void }
  type ExCommit<M> = <T extends keyof M>(type: T, payload?: M[T]) => void
  type ExDispatch<A> = <T extends keyof A>(type: T, payload?: A[T]) => any
  type ExActionContext<S, A, G, M> = {
    commit: ExCommit<M>
    dispatch: ExDispatch<A>
    state: S
    getters: G
    rootState: RootState
    rootGetters: RootGetters
  }
  type Actions<S, A, G = {}, M = {}> = {
    [K in keyof A]: (ctx: ExActionContext<S, A, G, M>, payload: A[K]) => any
  }

  interface ExStore extends Store<RootState> {
    getters: RootGetters
    commit: ExCommit<RootMutations>
    dispatch: ExDispatch<RootActions>
  }
  type StoreContext = ExActionContext<
    RootState,
    RootActions,
    RootGetters,
    RootMutations
  >
}
```

最後にtsconfig.jsonのfilesにtypes/vuex/index.d.tsをfilesに記述します。基本的に独自で拡張した型ファイルがあれば、随時tsconfigに追加すようにしましょう。

```json:tsconfig.json
"files": [
  "src/types/vuex/index.d.ts"
],
```

ここまで、Storeの型定義をしてきましたが、ここからは実際のStoreに型を付けていきます。先ほど配置した `store/contact/index.ts`に、定義した型を当てていきます。１行目のvuexのimportで、先ほどtypes/vuex/index.d.tsで拡張したGtters、Mutations、Actionsの型を取り込みます。２行目では`store/contact/type.ts`のinterface宣言した型を取り込んでいます。

GettersやActions内にあるserviceディレクトリから取り込んでいるのは、ビジネス要件を含んだ純関数になっています。

```ts:store/contact/index.ts
import { Getters, Mutations, Actions } from 'vuex'
import { S, G, M, A } from './type'

import {
  validName,
  validTel,
  validMessage
} from '../../service/validation'
import submitContact from '../../service/Contact'

export const state = (): S => ({
  name: '',
  tel: '',
  message: ''
})

export const getters: Getters<S, G> = {
  name: state => state.name,
  tel: state => state.tel,
  message: state => state.message,
  isErrName({ name }) {
    return validName(name)
  },
  isErrTel({ tel }) {
    return validTel(tel)
  },
  isErrMessage({ message }) {
    return validMessage(message)
  }
}

export const mutations: Mutations<S, M> = {
  SET_NAME(state, payload) {
    state.name = payload
  },
  SET_TEL(state, payload) {
    state.tel = payload
  },
  SET_MESSAGE(state, payload) {
    state.message = payload
  }
}


export const actions: Actions<S, A, G, M> = {
  setName({ commit }, payload) {
    commit('SET_NAME', payload)
  },
  setTel({ commit }, payload) {
    commit('SET_TEL', payload)
  },
  setMessage({ commit }, payload) {
    commit('SET_MESSAGE', payload)
  },
  resetContacts({ commit }) {
    commit('SET_NAME', '')
    commit('SET_TEL', '')
    commit('SET_EMAIL', '')
    commit('SET_COMPANY', '')
    commit('SET_MESSAGE', '')
  },
  sendContacts({ state }) {
    const { company, name, email, tel, message } = state

    return submitContact({
      name,
      tel,
      message
    })
  }
}

```

Storeの型定義と型付けに関しては以上で、あとはいつも通りVueコンポーネントでmapGettersやmapActionsでStoreの内容を取り込むだけで型安全なStoreを扱うことができます。モジュールモードになっているので、別のStoreを導入し型安全にしていきたい場合は同様の手順を踏んでいくと良いと思います。
ここで書いてあるtypescriptのコードはあくまで一例なのでプロジェクトにあった型定義を模索していければと思います。

### StoreとService層ついて

NuxtでもVueのプロジェクトでもビジネスルールを含んだロジックの記述場所は悩みのタネの一つだと思います。コンポーネント側に寄せるのか、それともStoreに集約されるのか、両方使うならどれくらStoreに寄せるべきかなどなどあると思います。よく、 `Storeの肥大化`なんて言われるかと思います。とはいえ、コンポーネントにロジックが並んでいるのもViewとロジックが同居してしまい混乱の原因になり得ます。では、Storeの肥大化を防ぎながらどうやってビジネスルールを含んだロジックをVuexで管理するか、というところになります。

僕の意見としては、**「ロジックの網羅性」**と**「ビジネス固有の複雑さ」**を役割分担することが大事だとおもってます。
ここでいうと、Storeがロジック（ユースケース）の網羅性を担保し、Service層でビジネス固有の複雑性を含むことです。

例えば先ほどのStoreの型をつけているところでいうと、Gettersを宣言しているisErrNameやisErrTelなどのreturn部分で、validNameやvalidTelなどがあります。これがServiece層の関数になっています。ここでは、Storeはこのロジックの中身を知りません。しかしながら、Storeは画面側で必要なプロパティやActionsをもつべきなので、このようにしています。

```ts
export const getters: Getters<S, G> = {
  name: state => state.name,
  tel: state => state.tel,
  message: state => state.message,
  isErrName({ name }) {
    return validName(name)  // ValidNameがService層
  },
  isErrTel({ tel }) {
    return validTel(tel)
  },
  isErrMessage({ message }) {
    return validMessage(message)
  }
}
```

こちらがそのService層の中身です。ここではあくまでとてもシンプルな例を出してますが、アプリケーション固有のロジックを以下のように値を受け取って出力するだけの純関数にすることで、簡単に使い回せますしVuexのコードを汚すことなく済みます。

```ts
type ValidType = (value: string) => boolean

export const validName: ValidType = value => {
  const isErr: boolean = value.length < 4 // 名前が４文字以内だと弾かれるというルール
  return isErr
}
```
StoreとService層とで分割することで、Vuexのことを気にせずこの関数の中身自体をスケールさせることも分割することも用意です。
また、純関数なので非常にテストがしやすく、Vuex以外の技術に移行しやすくその時のFWなどに依存しずらいというメリットもあります。（そもそもビジネスがFWの流行り廃りに左右されるのは本望ではないかと思います）

Service層が割とutils層みたいな部分と区別がつかないみたいな話もありますが、僕個人の意見としてはutils層は `アプリケーション固有の情報を含まず共通して使えるもの`と認識しております。
例えば、axiosをラップした外部リソース取得用のモジュールなどです。それ自体は、ビジネスルールを含む訳ではないのでutils層におきます。ただそこから、特定のAPIを叩くための関数だったりプロジェクト固有の独自キーを用いたストレージへのアクセスとなるとそれを取り込んだ上でService層で定義します。

## 学んだこと
### サーバーレス&generateサイトを１年間運用してみて
サーバーレス&静的サイトジェネレートということでNuxt + Netlifyで１年間サイトを運用してみて、非常に楽だったなという思いです。動的コンテンツが少なくちょっとしたお問い合わせフォームだけで複雑な実装の画面などはなかったので静的コンテンツの管理だけで済みました。
また、料金面でもドメイン取得料だけだったので総額の出費も**３０００円程度**で済みました。Netlifyに限った話ではなく、Amazon S3にアップするだけでも良いですし、若干レガシー環境だとレンタルサーバーに書き出したファイルを設定するだけでサイト作成がほぼ完了したりします。またNuxtなので、**静的ファイルも書き出せつつコンポーネント志向開発で出来る**という強みは非常に大きかったです。

### 見えてきた課題とアーキテクチャを考える上で大事なこと

という主語がデカめのタイトルになってしまって申し訳ないですが、**「配置すること」**って結構重要だと思っていてディレクトリ構成やなんの技術をどう使うか、という話は後の運用に大きく影響し導入時に大部分が決まる、みたいなとこはあると思います。

実際に、typescript移行は影響範囲が全体でコンポーネントやStore全てに影響が出たので、最初からTSにすればよかったんや！！って思うことがありました。また、それによって必要なモジュールも増えるので決めるべき時にしっかり決めるべきだなと感じました。
その上で設計や技術選定で気をつけたいなと思ったことは「壊しやすいか」と「組織あるいはチームの構造はどんなか」です。

#### 壊しやすさについて
壊しやすさは「依存度を下げる」みたいな意味合いも入っていて、ある特定の技術がなければ作れないあるいは作り直したほうが早いみたいな状況です。

**いかに既存の資産を活かせる形をとれるか**が重要になってくるんですが、先ほどのStoreとService層の話はまさにそれです。
Service層はビジネス固有のルールを含んでいますが、Vuexに対して依存しないようにしています。そうすることによって、最悪VueやVuexが使えない状態になったとしても、そのServiceの純関数群はそのまま他のFWでも扱うことが出来ます。（あくまで「壊しやすい形」のほんの一例です）

おそらく、**フレームワークのライフサイクルよりも事業のライフサイクルの方がおそらく長い（事業による）**かと思います。また、プロダクトやサービスに与える影響の大きさは、フレームワークではなく**マーケットの動きだったりユーザーフィードバックの方が大きい**はずなので、それに合わせていかに**変化（壊しやすく）**出来るかが重要、という感じです。

#### 組織あるいはチームの構造はどんなか
こちらは、コンウェイの法則でもあるんですが 

```
The structure of any system designed by an organization is isomorphic to the structure of the organization.
```

 とあるので、**出来上がるシステムの構造は設計する組織の構造に依存する**、と解釈できます。
 設計する時の組織構造は何か、どんなメンバーがいるのか、どんなチーム構成なのかという部分がシステムやアプリケーションに影響します。加えて、そのサービスやプロダクトが将来的にどういった方向に行きたいのか、どういった形態になりうるのか、未来の組織の様子を踏まえて設計することが必要です。つまり、**未来のチームの姿と現在との差分を定義した上でアーキテクチャを考える**と良さそうだなと、思ってます。

## まとめ

- Nuxt typescrptおすすめ！
- 静的サイトジェネレート＋ホスティングで運用コストを減らそう
- 設計は「壊しやすいか」と「組織あるいはチームの構造はどんなか」を考慮する

ここまで読んでいただき誠に感謝です。
前章で組織構造大事やぞ、と言いつつ今回のNuxtのサイトは僕一人で作ったので、今回に関しては組織構造とかないですが技術に依存し過ぎずでもそれの旨味は味わいつつ、という風にやっていけそうな気はするのでこれからも運用していきます。以下、本記事のまとめになります。

今回実際に刷新したサイトのリポジトリをのせていますのでご参考にしていただければと思います。
https://github.com/isihigameKoudai/bright-and-dizain

## 参考
以下、参考リポジトリや資料になります。ありがとうございます！
- https://typescript.nuxtjs.org/
- https://github.com/takefumi-yoshii/ts-nuxtjs-express
- https://ja.wikipedia.org/wiki/%E3%83%A1%E3%83%AB%E3%83%B4%E3%82%A3%E3%83%B3%E3%83%BB%E3%82%B3%E3%83%B3%E3%82%A6%E3%82%A7%E3%82%A4
