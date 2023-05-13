---
title: 俺のNuxtがこんなところで躓くはずがない
tags: Vue.js JavaScript HTML フロントエンド Nuxt
author: isihigameKoudai
slide: false
---
## 概要

こんにちは、アジアクエスト株式会社でフロントエンドエンジニアをやってます、「結局最後はエモさの」
[かめぽん](https://twitter.com/kamepon_fe)です。
Nuxt.jsで開発することが多くてQiita記事で盛大にNuxtをもてはやしてますが、僕も無駄に詰んだり変なところで時間食ったりなど、ありますのでそれらを供養できればと思い書かせていただきました。
皆様の大切な時間を守る記事に少しでもなっていただければ嬉しいなと思ってます。

##　本題
### その１：エラーページ実は１撃で終わる問題
問題というかこれはNuxtの良い点の一つなんですが、404や500番代エラーのページなどを実装しようと思ってて、自前でページのエラー処理を書こうと意気込んでた矢先のこと、いつもお世話になってる`layouts/default.vue`と同じディレクトリ内に`error.vue`を置くだけ。
これだけで、ページ周りのエラー時に自動でerror.vueが表示されます。
errorページにはいくつか特殊な項目があるので、以下に記載します。

- page周りでエラーが起こると`error.vue`に飛ぶ
- エラーページには`<nuxt/>`タグを含めてはならない
- `props:['error']`で取得後、`error.statusCode`で404か500系なのか条件分岐することができます

単一ファイルコンポーネントを読んだ時点で、popsに値が入りその中のstatusCodeによってメッセージによって表示内容を変えるだけで、簡単にエラーページを実装することが出来ます。
<img width="405" alt="スクリーンショット 2018-12-15 21.20.51.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/76053a70-cc90-4e2b-a928-fe6a79dca4ec.png">


### その２ sketchアニメーションが死ぬ問題

processingでアニメーションを実装するために以下のコードを書きました。

```html

<div class="visual">
  <canvas data-processing-sources="pde/sketch.pde"></canvas>
</div>

<script>
export default {
  head() {
    return {
      script: [{ src: "script/processing.min.js" }]
    };
  },
...
}
</script>

```

これ自体ページは表示時にアニメーションがしっかりと表示されます。
以下が、表示された際のDOMの状況です。
<img width="451" alt="スクリーンショット 2018-12-15 20.47.06.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/e843a13e-6c91-d973-cd96-b5d14e6163f4.png">


しかし、別ページからnuxt-linkでやってくると、なんとアニメーションが動かないんですよね〜
そしてその時のDOMの状況です。アニメーションが表示されていた時とは違って、widthとheightの設定がなく実態がない状態になってますね。

<img width="444" alt="スクリーンショット 2018-12-15 20.47.36.png" src="https://qiita-image-store.s3.amazonaws.com/0/105049/554ef64c-131f-11cb-2625-b1c8878d8a2c.png">



ページリロードがかからないといけないので、nuxt-linkではなくaタグを用いて画面遷移することで解決します。

### その３　nuxt generateで動的ルーティングが効かなくて焦った話
Nuxtには静的サイトジェネレータという最終的な成果物をHTML/CSS/jsの静的なページとして出力してくれる機能があります。
pages配下にファイルと置くだけで、ジェネレート時にも勝手にファイルを生成してくれるのですが、これには一つ落とし穴があって、`/user/:id`のように、`/user/10`とか/hoge/foo/3など動的ルーティングの場合は反映されません。
ですので、nuxt.config.jsのgenerate設定で、動的ルーティングにおける**全てのパターン**のを記述しなければいけません。

```js

module.exports = {
  generate: {
    routes: [
      '/foo/1',
      '/foo/2',
      '/foo/3',
      ...
    ]
  }
}
```

しかしながら、ヘッドレスCMSなど外部APIなどによってページ毎に内容が変わる場合はその全パターンもgenerateで設定する必要があります。


```js
export default {
  generate: {
    routes() {
      return axios.get('[apiのurl]')
      .then((res) => {
        return res.data.map((v) => {
          return '/foo/' + v.id
        })
      })
    }
  }
}
```

ここでお分かりだとは思うのですが、動的な内容のジェネレートはAPIで受け取る内容の量がかなり多い場合に非常に時間がかかってしまいます。また、コンテンツの使用頻度がかなり頻繁だとジェネレートがそもそもその運用方法に敵視しているのかなど出てくるので、ジェネレートが良いのかSSRが良いのか適切に判断していきましょう。

## まとめ
ここまで、僕がNuxtの開発で若干積んだ部分をご紹介しました。
Nuxt.jsは非常に融通がきくFWですが、普通のHTKML/CSSコーディングと同じ感覚で行くと若干やけどしちゃうので、公式リファレンス片手に（ここ大事）ガンガン開発していきましょう！
では、エモくいきましょう！
