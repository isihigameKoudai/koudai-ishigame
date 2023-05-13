---
title: 今から始めても遅くない、Vue.jsをより効果的に学ぶための記事まとめ
tags: JavaScript Vue.js 入門 HTML フロントエンド
author: isihigameKoudai
slide: false
---

## 概要
こんにちは、都内でフロントエンドエンジニアをしてます[かめぽん](https://twitter.com/kamepon_fe)です。フロントエンドですっかりお馴染みになりました**Vue.js**ですが、サーバーサイドエンジニアやデザイナーなど非フロントエンド領域の方々も触っている方がかなり増えて来たのでは、という印象を受けます。また、Vue.jsを使ってFirebaseやAWS、Netlifyなど様々なサービスと組み合わせて爆速で開発されているかたもかなりいるのでは無いかと思います。さらに、フロントエンドとしてモダンな開発をしたいとなると大概Vue.jsやReact.jsなどモダンなフレームワークやライブラリの習得が必須となっていることが多いです。

そんな中で、「**Vue.jsを勉強したいけど正直全然さわれてない。。。**」とか「**フロントエンド覚えること多いから結局何すればいいのかわからない**」とか「**勉強してるけど何がわからないかわからない・・・**」など、いち早くキャッチアップしたいのになかなか出来ていないと感じている方はいると思います。   
結論から言いますと、いち早くキャッチアップをするなら**いち早く情報のゲシュタルトを作る**ことが大事です。つまり、**大枠を掴む**ということですね。

今回その大枠を掴むのにぴったりな、概念的な部分のものから実際のVue.jsを使った開発で役立つような記事や書籍などをまとめてみました。   
※・・・概念的な記事にはReactやReduxが出てくることもあります。

## ターゲット
- これからVue.jsを効果的に勉強して行きたい方
- 何をインプットすればわからない方
- HTML, CSS, javascriptで簡単な実装ができる方

## 記事まとめ
基本的に以下の構成で進めて行きます。

- [導入編](#input)・・・概念的な部分のインプット
- [入門・基礎編](#basic)・・・文法やお作法をみてみて雰囲気を掴む
- [実践編（CLI環境、環境構築）](#develop)・・・入門・基礎編でやったことを使って実際の開発にかなり近い環境で開発してみる
- [ガイドライン](#guideline)・・・実践のクオリティを高めるためのインプット

<a id="input"></a>
### <a href="#input">導入編</a>
Vue.jsに入る前に、SPA(Single Page Application)や、仮想DOM、Fluxについてインプットすると良いと思います。
Vueを使うと何がおいしいのか、Vue使った設計はどんなようになるか、そもそもSPAって何をしたいのかなどサラッと読んでおくと実際の開発でイメージがつきやすいです。

- [SPA(Single Page Application)ってなに？](https://digitalidentity.co.jp/blog/creative/about-single-page-application.html)
- [なぜ仮想DOMという概念が俺達の魂を震えさせるのか](https://qiita.com/mizchi/items/4d25bc26def1719d52e6)
- [Fluxという設計思想](https://app.codegrid.net/entry/react-ex-1)
- [Fluxとはなんなのか](https://qiita.com/knhr__/items/5fec7571dab80e2dcd92)

<a id="basic"></a>
### <a href="#basic">入門・基礎編</a>
基本的にVueの世界に飛び込むためわかりやすい記事があるので、実際に手を動かしながら文法などについて学んでみると良いです。
エディタでも良いですし[CodeSandBox](https://codesandbox.io/)などですぐ試すのもおすすめです。書籍や公式では非常によくまとまっているのでいろんなコンテンツがあるのですが、一旦Vue独特の文法について触れる程度で十分かと思います。

- [基礎から学ぶ Vue.js（書籍）](https://www.amazon.co.jp/%E5%9F%BA%E7%A4%8E%E3%81%8B%E3%82%89%E5%AD%A6%E3%81%B6-Vue-js-mio/dp/4863542453/ref=pd_lpo_sbs_14_t_0?_encoding=UTF8&psc=1&refRID=GD023QRE0D3FQJCJJ9RZ)
- [jQuery から Vue.js へのステップアップ](https://qiita.com/mio3io/items/e7b2596d06b8005e8e6f)
- [Vue.js公式](https://jp.vuejs.org/v2/guide/)

<a id="develop"></a>
### <a href="#develop">実践編(CLI環境)</a>
入門・基礎編では主に文法に触れましたが、今回はSFC(Single File Component)やコンポーネントを使った開発、Vuexを使ったStore構築、Vue-Routerを使ったルーティングなどかなり実践的なものとなっています。

- [Vue-CLI](https://cli.vuejs.org/guide/#components-of-the-system)
- [Vuex](https://vuex.vuejs.org/ja/)
- [Vue-Router](https://router.vuejs.org/ja/)
- [Vue.js入門 基礎から実践アプリケーション開発まで](https://www.amazon.co.jp/Vue-js%E5%85%A5%E9%96%80-%E5%9F%BA%E7%A4%8E%E3%81%8B%E3%82%89%E5%AE%9F%E8%B7%B5%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E9%96%8B%E7%99%BA%E3%81%BE%E3%81%A7-%E5%B7%9D%E5%8F%A3-%E5%92%8C%E4%B9%9F/dp/4297100916)

### 実践編（環境構築）
CLI環境編ではVueが動く前提で開発出来ていたと思いますが、ここではVueが動く環境を実際に構築します。   
基本的にはVueもReactもWebpack+Babel+ESLint+Prettierの構成で開発環境を作ることが多いです。そこに、lodaerをなどを組み込みVueファイルを読めるようにしていくとVueで開発ができるようになって行きます。

- [Webpack](https://webpack.js.org/guides/getting-started)
- [Babel](https://babeljs.io/setup#browser)
- [ESLint](https://eslint.org/docs/user-guide/getting-started)
- [ESLint 最初の一歩](https://qiita.com/mysticatea/items/f523dab04a25f617c87d)
- [Prettier](https://prettier.io/docs/en/install.html)
- [Prettier 入門 ～ESLintとの違いを理解して併用する～](https://qiita.com/soarflat/items/06377f3b96964964a65d)

<a id="guideline"></a>
### <a href="#guideline">ガイドライン</a>
ここの記事は必須項目ではありませんが、読んでおくとかなり役立つページになっています。   
特にスタイルガイドは実際の開発でも読み直してみるとお互いに共通認識を持ちながらメンテナンサブルな開発を行って行くことが出来ます。

また実践的なコードなども乗っているので、読むだけでかなり役立つかなと思います。
- [Vue.jsスタイルガイド](https://jp.vuejs.org/v2/style-guide/index.html)
- [Vue.jsコツ＆ベストプラクティス](https://012-jp.vuejs.org/guide/best-practices.html)
- [Vue Patterns](https://learn-vuejs.github.io/vue-patterns/patterns/#component-declaration)


## まとめ
いかがだったでしょうか、概念的な部分の導入からガイドラインなどの実践的な記事まで紹介してみました。自分も学習を進めていく上で助けになったものも多かったので、一通り舐めていくと良いかと思います。コンテンツの量的には少し多めな気もしますが、最初は一つ一つを厳密に追っていくのではなくあくまで**雰囲気を掴む**ことが大事なので、最初は本当にサラッと読んでみると良いです。
サラッと読む → 導入部分のインプット → 少し手を動かして体感する → ゲシュタルトが出来てくるのサイクルができればどんどん覚えていけると思います。
ぜひ、楽しいVueライフを送って行って欲しいです。
