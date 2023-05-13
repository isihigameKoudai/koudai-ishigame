---
title: Typescript + Express + Webpackで作る、node.jsサーバー環境構築入門
tags: TypeScript Express Node.js JavaScript フロントエンド
author: isihigameKoudai
slide: false
---
## 概要
こんにちは、都内でフロントエンドエンジニアをやっております、[かめぽん](https://twitter.com/kamepon_fe)です。最近、typescriptの猛威がすごいわけですが、フロントエンドでは一般的になってきてて、そもそも型で品質担保ができることってサーバーサイドこそじゃね？と思いました。そこで、Typescript + Expressでサーバーを作るという僕にはあまり考えてなかった技術だったことと、typescriptってフロントエンドの印象が強かったこともあり、Typescript + Express + Webpackで環境を作ってみようと思います。
フロントエンドをやっていく中でサーバーサイドに関わることをちょくちょくあると思うので、この記事を読んでちょっとでもサーバーフレンドリーになれたら嬉しいなと思います。

## ターゲット

- フロントエンドだけどサーバーサイドをちょっとやってみたい人
- Webpack環境でExpressサーバーを作ってみたい人
- nodeサーバーの開発環境を作ってみたい人

## メリット

- サーバーサイドにおけるWeboackとTypescriptの環境構築ができる
- Expressの立ち上げができるようになる
- サーバーフレンドリーになる

## 本題
### 今回作るサーバーのディレクトリ構成

今回の成果物のディレクトリの内容は以下のようになっています。tsのトランスパイル後の吐き出し先の`dist`、実際に僕たちが記述するソースがある`src`、コンパイル設定などを司る`webpackの３つのディレクトリ`に分かれます。

```
├─dist
│ └─server.js
├─src
│ └─index.ts
└─webpack
│ ├─base.config.js
│ ├─dev.config.js
│ └─prod.config.js
├─package.json
├─nodemon.json
└─tsconfig.json
```

### 基本セットアップ

なにはともあれ、以下コマンドでサクッとnpm環境まで作ってしまいましょう！

```
// ディレクトリの作成と移動
mkdir ts-server && cd ts-server 

// npm初期化&pakcage.jsonの作成
npm init -y

// 吐き出し用ディレクトリの作成
mkdir dist

// ソースディレクトリの作成
mkdir src

// webpack用ディレクトリの作成
mkdir webpack

```

### Typescriptの導入

tsの導入をします。ここではTypescriptだけではなく `ts-node`と呼ばれる、tsをnode場で動かせる実行環境(厳密にはREPL)も一緒にインストールします。これにより、開発中にわざわざtsをトランスパイルして実行しなくても手軽にtsをnode上で実行することができます。また、ts-nodeと一緒に`nodemon`と呼ばれる実行環境を導入します。これにより、nodeサーバーを手動ではなくファイルに変更を加えた時点で自動でプロセスを再起動し変更分含めて反映します。

```
npm install --save-dev typescript ts-node nodemon
```

次にtsの設定をしてきます。

```
./node_modules/.bin/tsc --init
```

`tsconfig.json`というファイルができることを確認してください。tsconfigはtsの変換ルールについて記載がありプロジェクトによってどういうルールーにするかは個々に委ねられるので、ここでは以下のように設定しておきます。

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "esModuleInterop": true,
    "target": "es6",
    "noImplicitAny": true,
    "moduleResolution": "node",
    "sourceMap": true,
    "outDir": "dist",
    "baseUrl": "src",
    "lib": ["es2015"],
    "paths": {
      "*": ["node_modules/*", "src/types/*"]
    }
  },
  "include": ["src/**/*"]
}

```

ひとまずtsの設定は以上になります。

### Lintの設定
次にLintの設定に入っていきます。以下コマンドでLinter系のモジュールをインストールします。

```
npm isntall --save-dev eslint eslint-config-prettier eslint-plugin-prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser prettier
```

そうしましたら、`.eslintrc`を作りLintの設定をしていきましょう。

```
touch .eslintrc
```

今回は以下のように設定したら、eslintの設定は終了です。

```json

{
  "parser": "@typescript-eslint/parser",
  "extends": [
    "plugin:@typescript-eslint/recommended",
    "plugin:prettier/recommended",
    "prettier/@typescript-eslint"
  ]
}

```


### Expressの導入
次にサーバー本体を実装するためにexpressとそれに関連するモジュールを追加していきまさす。

```
npm install --save express
npm install --save-dev @types/express body-parser
```

次に、いよいよサーバー本体の実装に入ります。

```
touch src/index.ts
```

```ts:index.ts
import express from "express";
import * as bodyParser from "body-parser";

const app = express();
const router = express.Router();

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

router.get(
  "/",
  async (req: express.Request, res: express.Response): Promise<void> => {
    res.send("hello world");
  }
);

app.use("/", router);

app.listen(3000, () => {
  console.log("listening on port 3000");
});

export default app;

```


### npm scriptsとnodemonの設定
ここまできたら、今度は実際に動かす準備をしていきます。
最初にnodemonの設定をしていきます。以下コマンドで、`nodemon.json`を作成してください。script無いでnodemonコマンドを使うとこのファイルが呼び出され設定された通りにプロセスが起動します。

```
touch nodemon.json
```

```json:nodemon.json
{
  "restartable": "rs",
    "env": {
    "NODE_ENV": "development" // developmentモードとして作動します。
  },
  "watch": ["src"], // 監視する対象ディレクトリ
  "ignore": ["tests/**/*.ts"],
  "exec": "ts-node ./src/index.ts" // nodemonを起動したら実行されるスクリプト
}

```

次に実際に実行されるコマンドを編集します。package.jsonのscriptsに以下のコアマンどを追加します。

```json
"dev": "nodemon -L"
```

そのまま `npm run dev`をターミナルで実行し、ローカルホストにアクセスした時にブラウザ上に `hello world`が出れば成功です。



<img width="275" alt="スクリーンショット 2019-06-16 19.34.35.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/bc853ab9-636b-c1d6-5912-0e9b97e927e1.png">

### Webpack環境設定
development環境での
tsサーバーが動いたところで、今度はproductionを含めたバンドル環境を整えていきます。まずは、webpack系モジュールとts-loaderをインストールしていきます。

```
npm install --save-dev webpack webpack-cli webpack-merge webpack-node-externals ts-loader
```

そうしましたら、webpakディレクトリを作成していただき、その中に３つのwebpackのconfigファイルを作っていきます。

```
touch webpack/base.config.js //共通して使うwebpackの設定ファイル
touch webpack/dev.config.js // developmentモードで使う設定ファイル
touch webpack/prod.config.js // productionモードで使う設定ファイル
```

ここからはどんどん設定ファイルを作っていきます。
このファイルは開発用でも本番用でも共通で読み込まれるファイルです。

```js:base.config.js

const webpack = require('webpack');
const nodeExternals = require("webpack-node-externals");
const path = require("path");

const BUILD_ROOT = path.join(__dirname, "../dist");
const SRC_ROOT = path.join(__dirname, "../src");

module.exports = {
  context: SRC_ROOT,
  entry: path.resolve("src", "index.ts"),
  externals: [nodeExternals()],
  output: {
    filename: "server.js",
    path: BUILD_ROOT
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        exclude: /node_modules/,
        loader: "ts-loader",
        options: {
          configFile: "tsconfig.json"
        }
      }
    ]
  },
  resolve: {
    extensions: [".ts", ".js", ".json"],
    alias: {
      "@": path.join(__dirname, "/src/")
    }
  }
};

```


こちらは開発用の設定ファイルです。今回は直接使いませんが、本番と開発用で扱いたい時に分割することができます。`webpack-merge`と呼ばれるwebpackのファイルをマージしてくれるモジュールを使っていて、`dev.config.js`が`base.config.jsを取り込み`開発用として動作するようにしています。

```js:dev.config.js
const baseConfig = require("./base.config.js");
const merge = require("webpack-merge");

const config = merge(baseConfig, {
  mode: "development",
  devtool: "inline-source-map",
  devServer: {
    contentBase: "dist",
    host: "0.0.0.0",
    port: 3000
  }
});

module.exports = config;

```

こちらは本番用のconfigファイルです。動作は、`dev.config.js`と同じです。

```js:prod.config.js
const baseConfig = require("./base.config.js");
const merge = require("webpack-merge");

const config = merge(baseConfig, {
  mode: "production"
});

module.exports = config;

```

ここまで、きたら残るはnpm scriptの編集です。以下の内容を`package.json`のscriptsに追記してください。

```
"build": "webpack --config ./webpack/prod.config.js",
"start": "npm run build && node dist/server.js",
```

そうしましたら、`npm run build`でsrc内のファイルがdist配下に`server.js`として出力されることを確認してみてください。`prod.config.js`を読んでいるので、`server.js`がミニファイされていることがわかると思います。

今度は、`npm run start`でwebpackのビルドが実行されつつ、そのファイルを元にnodeサーバーが立ち上がることが確認できると思います。これで、開発＆本番用のWebpack環境ができたかと思います。ここからは各々のプロジェクトに応じてカスタマイズできると思います。

## まとめ
いかがだったでしょうか？Webpackを使ったts + expressサーバーの開発環境ができたかと思います。最近はBFF層なども出てきていてフロントエンドでもサーバーとの接点は割と多いと思います。その中で自分たちの使ってる技術でフロントエンド以外の一定の領域にも足を踏み入れることができるので、どんどん使っていきたいなと思います。

## 引用、参考など

https://qiita.com/sadnessOjisan/items/ea5590efa3f55ef56edd
https://aknow2.hatenablog.com/entry/2018/04/09/150006
