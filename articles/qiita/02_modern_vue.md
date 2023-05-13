---
title: モダンフロントエンド開発環境からVue.js環境までひと通り作ってみた
tags: JavaScript webpack Vue.js babel es6
author: isihigameKoudai
slide: false
---
# 概要
フロントエンド界隈ではReactやVueなどの開発を円滑に進めてくれる便利ツール（WebpackやLint、CLIなどなど）がたくさんあり開発がとてもしやすくなって来ていると思います。
とても便利な世の中になったな〜とCLIとかNext.jsなどを叩きながらサンプルばかり作ってたのは良いのですが、

開発環境そういえばどうやって作るん？？？、
それ管理しろ言われたら環境わかってないと死ぬんじゃないか、
フロントエンド界隈~~カオスすぎて~~結局何すればいいんだろうか？？？

という疑問や不安があったので、一度脳みその整理がてら開発環境をひと通り作ってみました。
開発環境周りの設定ってあまり進んでやりたくないのでスターターキット的な感じにまとめてGitHub ( https://github.com/isihigameKoudai/VuejsStarterKit )にあげてみました。また、最近はVue.jsでガリガリ開発するようになり将来的にはSPAやPWAなども作りたいなーという欲望があったので一緒に導入してみました。

# 今回の環境構築で導入したもの
## ES6環境
ES6はECMAscriptが策定した仕様の新しいjavascriptになっています。import,exportによるモジュール構文やclass構文などが使えるようになっており、reactやvueでは積極的にES6の書き方を使用していきます。
## ESLint
ESLintはjavascriptのための静的な検証ツールで、コード実行前に括弧やインデント・スペースなど書き方を統一したり、明らかなバグを発見したりすることが出来ます。
## Prettier
Perttierはコードフォーマッターと呼ばれるコードを自動であるべき形式に直してくれるツールのことです。自動で直してくれるので、コードを綺麗に保つための労力を割かなくてもいいようになります。とっても楽です。エディタの方でも設定できるようなので、普段使っているエディタの方で動くか調べてみるといいと思います。
## Babel
javascriptの書き方を新しいものから古い書き方に変換するツールのことです。ブラウザの環境によってはES6だと動かない場合があるので、それをES5などの形式に変換するために使います。したがって、ES6で書きたいけどサービスのターゲットブラウザがES6未対応の場合に導入すると良いですが、ES6を使うならとりあえず入れときましょう。
## Webpack
各リソースやアセットなどを一つのファイルにバンドル（まとめる）&コンパイルしてくれる便利ツールです。コンポーネントやモジュールなどのファイルを作っておけば良しなにまとめて吐き出してくれます。

## Vue.js
Vue.jsとはjavascript上で動くProgressive Javascript Frameworkでjsフレームワークの一種になります。主に受け取った情報を元に画面の描写するViewを担うものになります。approachable（親しみ安い）、veratile（融通が効く）、performant（高性能）、Maintainable（メンテしやすい）、Testable（テストしやすい）というと特徴がありシンプルで非常に扱いやすいです。

## Vuex
Vueでは主にMVCで言う所のVの部分を担っているためそれ以外のロジックを管理するのは得意では有りません。小さい規模のサービスであれば特に必要はないのですが、規模が大きくなるに連れてプロップスのバケツリレーが起きたり、一つの状態によって複数の部品が作用しあったりするので、そういった場合に用います。MVCやMVVMとは異なりFluxという概念を取り入れています。

## Vue-Router
Vueで作られたリソースに対してURLでアクセスできるようルーティング機能を提供するVue.js公式のエコシステムになります。

# 環境構築
### 作業ディレクトリ作成
```
mkdir [workspace-name]
cd [workspace-name]
```

### gitとyarn(npmでもOK)の初期化
git initでgitリポジトリとして扱えるようになります。
yarn initではそのプロダクトの情報を聞かれ、最終的にはpackage.jsonと呼ばれるディレクトリ内でのパッケージやコマンドを管理するファイルが生成されます。

```
git init
yarn init
```

### gitignoreとeditorconfigの追加
gitignoreは指定したファイルをgitの管理下から除外するためのものです。
editorconfigはいろんなエディタ間でのルール（インデントやタブ・スペースなど）を統一させるファイルです。

```
touch .gitignore
touch .editorconfig
```

.gitignore

```
node_modules/
build/
temp/
.ds_store
```

.editorconfig

```ini
root = true


[*]

indent_style = tab
indent_size = 2

end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false
```

### ES6とESLintとPrettier

```
yarn add --dev eslint eslint-plugin-import eslint-config-airbnb-base prettier eslint-config-prettier eslint-plugin-prettier
```

こちらも追加で導入しましょう。

```
yarn add --dev eslint-config-airbnb eslint-plugin-flowtype eslint-plugin-import eslint-plugin-jsx-a11y
```

.eslintrc.jsのファイルに以下を記述します

```js
module.exports = {
    "parser": "babel-eslint",
    "extends": [
      "airbnb",
      "prettier",
      "plugin:flowtype/recommended"
    ],
    "plugins": [
      "prettier"
    ],
    "rules": {
      "prettier/prettier": ["error", {
        "singleQuote": true,
        "bracketSpacing": true,
        "jsxBracketSameLine": true
      }]
    }
  };
```
Lint用のショートカットコマンドをpackage.json内のscriptsに追加します。

```
"lint": "eslint \"src/**/*.js\"",
"lint-fix": "eslint \"src/**/*.js\" --fix",
```

### babelの設定
javacsriptの新しい書き方は、ブラウザによって認識されない場合があるので認識できる形にトランスパイルしてくれる便利屋さんがbabel。

```
yarn add --dev babel babel-core babel-eslint babel-loader babel-preset-env babel-preset-stage2
```

### webpackの設定
webpackは指定したディレクトリ内のリソース（jsファイルやimg,cssなど）をひとまとめにし、実行時に単一ファイルとして書き出してくれる。
webpack-dev-serverで開発用サーバーを立ち上げブラウザすぐに確認することもできる。

```
yarn add --dev webpack webpack-dev-server webpack-cli
```

webpack.config.js

```js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: [
    'babel-polyfill',
    path.resolve('src', 'index.js')
  ],
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dest/')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
      },
    ]
  },
  resolve: {
    extensions: ['.js','json','jsx'],
    alias: {
      vue$: 'vue/dist/vue.esm.js',
    },
  },
  devServer: {
    contentBase: 'dest',
  },
};
```

cssやその他ファイルも読めるようにしましょう。

```js
yarn add --dev node-sass css-loader sass-loader style-loader url-loader
```

webpack.config.jsのrulesにloaderを追記しましょう。

```js
{
  test: /\.(css|sass|scss)$/,
  loader: 'sass-loader',
  options: {
    outputStyle: 'expanded',
    sourceMap: true,
  },
},
{
  test: /\.(jpg|png|json|svg)$/,
  loaders: 'url-loader'
},
```

webpackのショートカットコマンドが使えるようにするため、package.jsonのscriptsに以下のコードを追加します。

```
"test": "echo \"Error: no test specified\" && exit 1",
"dev": "webpack-dev-server --config ./webpack.config.js --hot --host 0.0.0.0",
"build": "webpack --config ./webpack.config.js --env.production -p",
"start": "webpack-dev-server --config ./webpack.config.js --env.production --host 0.0.0.0"
```
babel-polyfillのインストール

```
yarn add babel-polyfill
```

vueファイルが読み込めるようにwebpack.conig.jsのmodulesのrulesに以下の内容を記述します。

```
{
  test: /\.vue$/,
  loader: 'vue-loader',
},
```
拡張子の名前解決するため、resolveのextensionsにvueを追加します。

```
[".js", ".vue"]
```

### パッケージの準備
vueとvue系のエコシステムを動かす為にパッケージを導入します。

```
yarn add vue vuex vue-router
```

### ディレクトリの準備
ソース用と公開用ディレクトリの作成

```
mkdir src //ソース用
mkdir dest //公開用
```

### 各ファイルの準備

htmlファイルではVueで生成されるDOMを受け取る用のidの記述と、webpackによってバンドルされたjsファイルを読むだけのシンプルな作りになっています。
/dest/index.html

```vue
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <title>FrontendEnv Sample</title>
  </head>
  <body>
    <div id="app"></div>
    <script src="/bundle.js"></script>
  </body>
</html>
```

/src/App.vue

```vue
<template>
  <div class="g-content">
    <h2 class="title">Welcome to Vue world</h2>
    <router-link to="/">Top</router-link>
    <router-link to="/about">About</router-link>
    <p>RouterView↓↓↓</p>
    <router-view></router-view>
  </div>
</template>
<script>
</script>
<style lang="scss" scoped>
.g-content {
  background-color: #eee;
  .title {
    font-size: 24px;
    font-weight: bold;
  }
}
</style>
```

vueファイルの中はtemplate,script,styleの三部構成になっておりtemplate内ではhtmlやcssがそのまま記述できます。
styleはlangを指定することでsass/scss,lessなどを使うことも出来ます。また、scopedを指定するとグローバル汚染しないcssを書くことも可能です。

/src/index.js

```
import 'babel-polyfill';

import Vue from 'vue';
import Vuex from 'vuex';
import VueRouter from 'vue-router';

import App from './App.vue';
import route from './route';
import store from './store';

Vue.use(Vuex);
Vue.use(VueRouter);

new Vue({
  el:'#app',
  store: store,
  router: route,
  render: h => h(App)
});

```

vuexの中核となるstoreのファイルを記述します。

store/index.js

```js
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

const state = {
  mode: ''
};

//Mutaions is synchronous only methods to change state values.
const mutations = {
  commitModeToDev(state,payload) {
    state.mode = 'devlopment';
  },
  commitModeToPro(state,payload) {
    state.mode = 'production'; 
  }
};

//Actions is methods includes asynchronous processes.
//and commit mutations.
const actions = {
  getSomething({commit},payload) {
    const num1 = 1;
    const num = 2;
  },
  updateModeDev({commit},payload) {
    commit('commitModeToDev',payload);
  },
  updateModePro({commit},payload) {
    commit('commitModeToPro',payload);
  }
};

export default new Vuex.Store({
  state: state,
  mutations: mutations,
  actions: actions
});
```

各リソースに対してURLでアクセスするためルーティングファイルを作成します。
route/index.js

```js
import VueRouter from 'vue-router';
import Top from '../pages/top';
import About from '../pages/about';

const routes = [
  {
    path: '/',
    component: Top
  },
  {
    path: '/about',
    component: About
  },
];

export default new VueRouter({
  routes,
});
```

src/pages/top.vue

```vue
<template>
  <div>
    <h2>Top page</h2>
    <input 
      type="text" 
      v-model="message" 
    />
    <p>input message=> {{message}}</p>
    <input 
      type="button"
      value="DEVELOP"
      @click="commitModeToDev"
    />
    <input 
      type="button"
      value="PRODUCT"
      @click="commitModeToPro"
    />
    <p>mode=>{{mode}}</p>
  </div>
</template>
<script>
import {mapState, mapMutations, mapActions} from 'vuex';

export default {
  data() {
    return {
      title: 'Welcome to Vue world',
      message:''
    }
  },
  computed: mapState(["mode"]),
  methods: mapMutations(["commitModeToDev","commitModeToPro"])
};
</script>
<style scoped>

</style>

```

src/pages/about.vue

```vue
<template>
  <div>
    <h2>About page</h2>
    <p>{{text}}</p>
  </div>
</template>
<script>
export default {
  data() {
    return {
      text: 'Hello About Page'
    }
  }
};
</script>
<style scoped>

</style>
```

### 起動
次のコマンドでブラウザ上でhttp://localhost:8080/ にアクセスしてみて、「Welcome to Vue world」が出ればOK。

```
yarn start
```

# まとめ
・ES6の書き方は慣れるととても書きやすい
・環境構築はちょっとというか割と大変
・Vue.jsは触った方がいい、絶対

ここまで読んでいただき本当にありがとうございます。
一人でも多くフロントエンドの開発者が現れ、あわよくばVue信者も増えて欲しいなと思います。
みなさん、Vue.jsやりましょう！


# 参考URL
https://github.com/masakitm/vue-nocli
https://github.com/nabepon/frontend/tree/env-setup-tutorial
https://ics.media/entry/12140
https://www.kken.io/posts/prettier-eslint/
上記の記事を参考にさせていただきました。ありがとうございます！
