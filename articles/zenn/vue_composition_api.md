---
title: "Vue Composition API時代のUI状態管理と設計で気をつけたいこと"
emoji: "😊"
type: "tech"
topics:
  - "typescript"
  - "vue"
  - "compositionapi"
  - "設計"
  - "vue3"
published: true
published_at: "2021-01-03 20:06"
---

## 初めに

こんにちは、アジアクエスト株式会社でフロントエンドエンジニアをしています[かめぽん](https://twitter.com/kamepon_fe)です。初めてZennの記事を書きます！がしかし、なんの記事を書こうかな迷っていまして、最近VueのComposition APIを業務で触ることがあったので、そこでの知見とかちょこっと考察してみたのでそれを書き留められればと思います。この記事で扱う機能はまだ改善の余地が含まれるため、一つの参考として捉えてもらえればと思います。

## Composition APIについて

Vue3にも搭載されたことでも話題のComposition APIですが期待値の高さから見るに、知っている人も聞いたことある人も多いと思います。3.0だけでなく2.x向けのpluginも配布されているので手軽に触ることができると思います。

Vueと言えば、Options APIでSFCを書くことがほとんどかと思いますが、今回用意されたAPIはビジネスロジックとViewに関する部分を分離しうまく再利用しやすくするものとなっています。

カウンターアプリを例にOptions APIとの差をみてみましょう。

### Options APIの場合

```tsx
export default {
  name: "App",
  data: () => ({
    count: 0,
  }),
  methods: {
    increment() {
      this.count++;
    },
    decrement() {
      this.count--;
    },
  },
};
```

こちらはOptions APIの馴染みのあるコードですね。しかし、アプリケーションが大きくなってくると以下の問題が起こり得ます。

- 値がthis依存になっているので、ロジックが切り出しづらくなる
- ロジック部分とViewに関する部分のコードが混ざり合ってしまう
- ビジネスロジックが肥大化してくると、テストがしづらくなる

Options API自体誰が書いてもある程度秩序あるコードをかけるのが旨味ではありますが、大規模になってくるとロジックの分離などが結構難しくなりメンテしずらくなる傾向は否めません。

そんな中で出てきたのがComposition APIになります。

### Composition APIの場合

```tsx
// Counter.vue
// Vue3もしくは、Vue2.x + pluginで動作可能
import { defineComponent, reactive } from "@vue/composition-api";

export default defineComponent({
  setup() {
    const state = reactive({
      count: 0,
    });

    const increment = () => {
      state.count++;
    };

    const decrement = () => {
      state.count--;
    };

    return {
      state,
      increment,
      decrement,
    };
  },
});
```

Composition APIでは大きな特徴として、setup関数の中でプロパティやメソッド・ライフサイクルに紐づく副作用などを設定します。最後にreturnすることでtemplateで値をそのまま使用することでがきます。ここではreactiveを使っていますがこれはComposition APIで新しく追加されたリアクティブプロパティを設定するものでVue.observableとほぼ同じと思ってもらって大丈夫です。他の新しい機能に関しては[公式](https://v3.vuejs.org/guide/composition-api-introduction.html)でチェックしてみてください。

### Composition APIにすると結局のところ何がいいのか

先程のカウンターアプリで、count、increment、decrementがカウンター機能を司るロジック部分だと思います。ReactではこれらをCustom hooksとして外部から流入するような使い方をすることができますが、Composition APIを使うことでVueアプリでも同じような実装をすることが可能です。Vueのrfcでは`Composition Function` と読んでいます。以下にロジック部分をComposition Functionとして分離したコードをみてみましょう。

```tsx
// useCounter.ts
import { ref } from "@vue/composition-api";

export const useCounter = (intial = 0) => {
 const count = ref<number>(initial)
 const increment = () => {
  count.value++
 }

 const decrement = () => {
  count.value--
 }

 return {
  count,
  increment,
  decrement
 }
}

// Counter.vue
import { defineComponent } from '@vue/composition-ap'
import { useCounter } from './useCounter.ts'

export default defineComponent({
  setup() {
    const { count, increment, decerement } = useCounter()

    return {
      state,
      increment,
      decrement,
    };
  },
});
```

 Counter.vue自体のコードがかなりスッキリしました。また、ロジックに関する記述がなくなったと思います。カウンターアプリ程度だとそこまで旨味が出づらいかもですが、アプリケーションがスケールしていくと以下のように多くのメリットを被ることができます。

- ビジネスロジックを分離しやすくできるので、結果として再利用性を高めることができる
- ロジックを含んだComposition Function単体でテストがしやすくなる

今までは、vue-test-utilsを使ってコンポーネントをマウントして中のロジックをテストする必要がありましたが、純粋なスクリプトロジックなのでかなりテスタブルになります。

再利用性が高くなりテストがしやすいというのはかなりアドバンテージではありますが、一方で注意しなければいけないこともあります。

まず、Composition APIのリアクティブプロパティなどは、setup関数内で定義しなければならなりません。したがってtemplateで必要になってくるものはComposition Function含めてsetup関数内で定義・returnすることが必要です。

そして、もう一つ注意しなければいけない点は、`設計力・純粋に綺麗なコードをかけるかが以前よりも必要になってくる`ことです。Options APIよりも自由に書くことができたり分離もしやすくなったりする反面、ロジックが無秩序に散らばってしまったりスパゲッティコードを生み出しやすい懸念もあります。Composition FUnctionもコンポーネントと同様に粒度をどこまでで止めるべきかチーム内で意思疎通を取る必要があると思われます。

## Provide / Inject について

Provide / InjectはReactのContext APIに似たもので、propsやemitのバケツリレーをせずとも特定の値やメソッドを共有することができる機能です。よく、DI（Dependency Injection）の文脈で語られることが多いですが、任意の階層からProvideすることで以下の階層のコンポーネントでInjectすることで依存性を取り込むことがたやすくできます。

![](https://storage.googleapis.com/zenn-user-upload/vbj33tydhvpzzojweufz9e6bm23q)


ここではProvideでリアクティブプロパティを提供することろまでを記述していきます。

```tsx
import { SetupContext, h, defineComponent, provide } from '@vue/composition-api'
import { ref, Ref } from '@vue/composition-api'
import { InjectionKey } from '@vue/composition-api'
import LoadingKey from './Loading/LoadingKey'

// hooks層
export const useLoading = (): TUseLoading => {
  const isLoading = ref<boolean>(false)

  const setIsLoading = (loading: boolean) => {
    isLoading.value = loading
  }

  return {
    isLoading,
    setIsLoading
  }
}

// InjectionKeyの生成
const LoadingKey: InjectionKey<TUseLoading> = Symbol('LoadingStore')

// Provider
export default defineComponent({
  setup (props, context: SetupContext) {
  　provide(LoadingKey, useLoading())
   const childNode = context.slots.default

   return () => h('div', { class: 'app-provider' }, childNode())
  }
})
```

これでProvide側の設定は完了し、あとはtemplateにProviderを配置し任意のコンポーネントでInjectします。機能（プロパティやメソッド）はCompositionFunctionが提供し、Provide/Injectでデリバリーするという感覚でいるとわかりやすいかと思います。

```tsx
// TOPレベル
(() => {
  new Vue({
    ...
    template: `
      <Provider>
        <ChildComponent />
        <ChildFooComponent />
        <ChildBarComponent />
      </Provider>`
  })
})()

//ChildComponent.vue
import { defineComponent, inject, Ref, ref } from '@vue/composition-api'
import { TUseLoading, useLoading } from '@hooks/useLoading'
import LoadingKey from './LoadingKey' // Composition APIのInjectionKeyでProvideされたプロパティを取得する

export default defineComponent({
  setup () {
    // Composition APIを含んだリアクティブプロパティをsetup内でinjectする
    const { isLoading } = inject(LoadingKey, useLoading())
    ...
    return {
      isLoading
    }
  }
})
```

これで、Provider配下のツリーのコンポーネント内でInjectをすることができます。

これらの機能はとても便利ですが、もちろん乱用は好ましくありません。では、いつ導入するべきなのかという問題があると思います。

### Vuexとの使い分けについて

props/emitのバケツリレーを避けるソリューションとしてはVuexがあげられます（というかイベントバスの他にこれしかない気はする）。ですが、そのためだけにVuexを用いるのはオーバーエンジニアリングな気もします。また、Vuex自体は厳密に管理されているとはいグローバルであるため、便利ですがどのコンポーネントからも読めてしまうという問題があります。さらに、コンポーネントのテストもわざわざVuexをモックしてあげないといけないので、少々手間でもあります。

もちろんVuexの使用が適切なパターンもありますが、以下の場合においてはProvide/Injectのようなパターンで検討することも一つの手だと思います。

- コンポーネントのネストが深いが、props/emitのバケツリレーを避けたい
- Provideした値のスコープがある程度限定的

注意点としては、Injectしたコンポーネントは依存度が上がってしまうという懸念もあるので、そのコンポーネントが再利用前提なのかProviderコンポーネントと密な関係なのかを考慮する必要があります。

## これからのVueアプリの設計で何を考えるか

Composition FunctionとProvide/Injectの組み合わせもしくはこれらの概念は、Vueアプリでの状態管理やUIとしてのデータの持たせ方に新しい選択肢を与えます。
特にフロントエンドではAtomic Designでアプリケーションを構築していく場合が少なからずあり、グローバルないしスコープ範囲がある程度ある値をどのコンポーネントを通して提供するかがあると思います。 

そんななかでtakepepeさんが[Atomic Redsign](https://zenn.dev/takepepe/articles/atomic-redesign)という興味深い記事を書いていて、原子の単位をUIから `依存度`で分類することを提唱していました。そう見るとOrganismsでProvideするのが一つの解ではあります。もちろん、エントリーポイントでトップレベルからProvideすることもあり得るので、提供する値がどんなコンテキストなのかを見定める必要があります。

では、それを踏まえてUIの設計で何を考えるか
- コンポーネントあるいは値のスコープと依存度別になんのリアクティブモジュールを使用するか
- 上記のモジュールに応じてコンポーネントの粒度を決める

と行った具合になるのかなと思います。
Composition APIはまだまだ改良が重ねられそうな雰囲気があるので、細いTipsは追いつつも抽象レベルの概念は持っておいて損は無いなと感じました。
最後まで読んでいただきありがとうございます！


## 参考
https://v3.vuejs.org/guide/composition-api-introduction.html
https://speakerdeck.com/potato4d/the-last-architecture-of-the-vue-2-dot-x
https://qiita.com/kahirokunn/items/b2dd95217e48072565f1
https://qiita.com/karamage/items/4bc90f637487d3fcecf0
https://zenn.dev/takepepe/articles/atomic-redesign