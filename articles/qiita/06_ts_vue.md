---
title: Typescriptで動くVue.js開発環境を作った話
tags: JavaScript TypeScript es6 フロントエンド
author: isihigameKoudai
slide: false
---
## 概要

こんにちは、都内でフロントエンドエンジニアをしています、かめぽんです。
先日、[プログラミング言語別エンジニア年収ランキング発表](http://news.livedoor.com/article/detail/15125851/) と言う記事が出ていて
目にした人も多いのではないでしょうか？

「javascriptは何位だろうな〜？」と胸を躍らせながらページを開いたら、、、そう、

そこにはjavascript(es6とか)の文字はなかったのです、、、（まあそりゃそうか）

だいたい、サーバーサイドの言語が並んでいる中フロントエンドでもガンガンやってけそうな言語ってないのかな〜とみてると、、、


「Typescript」なるものがありました。（神！

Angularでも採用してるし、型が導入されてていいことありそうだなーと思いつつ、今もっとも勢いのあるjavascriptフレームワークのvueの開発環境をtsで作ってみたので、みなさんの参考にしていただければとても嬉しいです。Typescriptも追ってみたいし、Vueも気になってるんだよね！と言う方にぜひ読んでいただければと思います！[Github](https://github.com/isihigameKoudai/VueOnTypescript)にも今回の記事のコードを挙げていますので、参考にしてみてください！

## Typescriptとは
Typescript（以下、ts）はマイクロソフトが開発した言語で、列挙型やクラスベースなオブジェクト指向などありますが大きな特徴はやはり「静的型付け」です。明示的に型を指定することができ、異なる型の変数の代入などをコンパイル時にエラーとして教えてくれます。tsはブラウザで直接認識はされないので、コンパイルしてjavascriptに変換された後に使用されます。フロントエンドでは定番のwebpackなどでコンパイル設定などを行います。

## Typescriptのキホン
開発環境に入る前のtsの基本的な機能について紹介したいと思います。
### 型の宣言（変数）
tsでは以下の型が用意されています。

```ts

var hoge: number = 100; //数値（整数だけでなく実数も扱える）
var fuga: string = "sample"; // 文字列
var piyo: boolean = true; //　true/false
var maru: string[] = 100; // 配列(number[]もいける)
```
そのほかにも以下のような型があります。

- any (従来のjsと同様の扱いが出来る)
- void（値として存在しない）
- null（値として存在しない）
- undefined（値として存在しない）


### 型の宣言（関数）
tsでは関数に置ける型の指定も出来るようになっています。これは良さげですね！

```
let add = (a: number, b: number): number => {
  return a + b
}
```


### クラス
tsではjavaやc#などでおなじみのclassや継承などが使え、クラスベースでのオブジェクト指向な実装が可能になっています。getMessageに置いて引数やreturn値も型指定をするので、型safeな世界になりますね！

```ts
class Messenger {
    private message: string;

    getMessage(message: string): string {
        this.message = "hello " + message;
        return this.message;
    }
}

let msg = new Messenger('world!!'); // 'hello world!!'が返却される
console.log(msg); // 'hello world!!'が表示される

```

### インターフェース
インターフェースとは、中に実装やmethod的な振る舞いを持たず型や接合部分の定義をもつものになります。使うメリットとしては、そのインターフェースを使っているクラスや変数等は同じメンバーが在ることを保証されること、各所の返却値などがオブジェクトの場合に共通化することで実装が楽になることが挙げられます。

classで扱う場合

```ts

//　インターフェースの定義
interface Person {
    name:string;
    age: number;
}

// 定義したインターフェースをclassに付与する
class Hoge implements Person {
    name = 'foo bar';
    age = 24;
}
```

変数の型として使う場合

```
interface Person {
    name:string;
    age: number;
}

let who: Person = {};
who.name = 'foo bar';
who.age = 30;
```


## 開発環境構築
ここまでtsの基礎的な内容に触れてきたので、今度はそれを実行できる且つ今もっとも勢いの在るフレームワークのVue.jsが動く環境構築の設定を紹介します。ちなみにここでは、webpack環境構築等の説明はそこまでしないので、各々vue-cliやスクラッチでの構築を事前にしておくことをお勧めします。環境構築に関しては、私自身の[記事](https://qiita.com/isihigameKoudai/items/520c1cb6540e0641a00c)もありますので参考にしていただけると幸いです。

### Typescriptのインストール
なにはともあれnpm/yarnでインストール！
パッケージを追加したらpackage.jsonに追加されたか確認してみましょう。

```
npm install --save-dev typescript ts-loader
```
### webpack設定ファイルを編集
webpack設定のミソは、entryでのindex.tsファイルの指定、rulesにてts-loaderの設定, resolveでts/tsxの名前解決をすることです。

```js

const VueLoaderPlugin = require("vue-loader/lib/plugin");

const path = require('path');
const webpack = require('webpack');

const BUILD_ROOT = path.join(__dirname, '../dest');
const SRC_ROOT = path.join(__dirname, '../src');

module.exports = {
  context: SRC_ROOT,
  entry: ["babel-polyfill", path.resolve("src", "index.ts")],  // ファイルの拡張しの変更
  output: {
    filename: "bundle.js",
    path: BUILD_ROOT
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx)$/,            //ts-loaderの追加
        loader: "ts-loader",
        exclude: "/node_modules/",
        options: {
          appendTsSuffixTo: [/\.vue$/]
        }
      },
      {
        test: /\.vue$/,
        exclude: /node_modules/,
        loader: "vue-loader"
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader"
      },
      {
        test: /\.(css)$/,
        exclude: /node_modules/,
        use: [
          "vue-style-loader",
          "css-loader"
        ]
      },
      {
        test: /\.(scss|sass)$/,
        exclude: /node_modules/,
        loader: 'style-loader!css-loader!sass-loader'
      },
      {
        test: /\.(jpg|png|json|svg)$/,
        loaders: "url-loader"
      }
    ]
  },
  resolve: {
    extensions: [".ts", ".tsx", ".vue", ".js", ".jsx", ".json"],
    alias: {
      vue$: "vue/dist/vue.esm.js",
      "@": path.resolve("src")
    }
  },
  plugins: [new VueLoaderPlugin()]
};
```

### package.jsonのmainを編集
package.jsonのmainプロパティを```index.js```から```index.ts```に変更しましょう。

### .tslintの追加と編集
とりあえずモジュール追加しましょ！

```
npm install --save-dev tslint tslint-loader tslint-config-airbnb tslint-eslint-rules tslint-microsoft-contrib tslint-plugin-prettier tsutils
```
次に.tsllint.jsonの設定をします

```json
{
  "extends": ["tslint-config-airbnb","tslint-config-prettier"],
  "rules": {
    "max-line-length": [true, 120],
    "object-shorthand-properties-first": false,
    "radix": false,
    "ter-arrow-parens": false
  }
}
```

### .tsconfigの設定
以下、コマンドでtsconfig.jsonファイルを生成します。

```
./node_modules/.bin/tsc --init
```

すると、ある程度設定されたtsconfig.jsonが生成されます。あとは任意に設定していきます。
 
```json

{
  "compilerOptions": {
    "target": "es5",
    "module": "es2015",
    "outDir": "./dest",
    "strict": true,
    "strictNullChecks": true,
    "noImplicitThis": true,
    "moduleResolution": "node",
    "baseUrl": "./src",
    "paths": {
      "@": [
        "src"
      ]
    },                      
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "experimentalDecorators": true
   },
      "include": [
        "./src/**/*"
      ]
}
```

### index.tsの追加
エントリーポイントのファイルはjsになっていると思うので、これをtsに変更します。内容はいつものindex.jsとほぼ同じです。

```ts
import 'babel-polyfill';
import Vue from 'vue';
import App from "./App.vue"

new Vue({
	el: '#app',
	render: h => h(App),
});
```

### vueファイルの追加 
vueファイルでtsを使うときは、scssやsassを使う時と一緒でscriptタグに```lang="ts"```
を追加します。

### .d.tsファイルの追加

```ts

declare module '*.vue' {
    import Vue from 'vue'
    export default Vue
}
```

### npmでスタートする
以下コマンドで、いつものvueファイルの中身は動いていれば成功です！

```
npm rund dev
```

## 所感
型が使えるので型セーフな世界になるな〜というのが所感ですが、anyという便利型があとで地雷踏みそうな感じがするのでそこは気をつけたいなと感じてます。心配だった文法もES2015以降の流れを汲み取っているので、結構使えそうだなと思ってます。年収あげるため、、、でなくともtsはいじっておきたいですね！！
ちなみに、今回のコードは[Github](https://github.com/isihigameKoudai/VueOnTypescript)に置いていますので、ぜひ参考にしてください。
