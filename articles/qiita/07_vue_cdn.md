---
title: Vue.js(CDN)を使ったhtmlコーディング最小構成
tags: HTML CSS JavaScript コーディング
author: isihigameKoudai
slide: false
---
## 背景
みなさんこんにちは、都内でフロントエンドエンジニアをしています、かめぽんです。
最近、大きく環境が変わりましてVue.js+Atomic Designでかなーりかなーりがっつり開発していまして非常にエキサイティングな日々を過ごしております。

しかしながら、世の中そんな楽しい世界ばかりではなく思いっきりVueが使えない、案件的にレガシーな部分が多い、そもそもフロントエンジニアリングじゃなくてコーディングメインでだけどモダンなことしてみたい！という人もいると思います。
また、自分自身もコーディング作業をやってほしい！みたいなこと言われるんですが、Vueに取り憑かれてしまった以上jQueryを読み込みたく無いんですよね〜。
ということで、VueのCDNを読み込んでコーディングするための環境を１分くらいで考案したので、サクッとみてやってくださいm(_ _)m

## ターゲット
- Vueメッッチャきになるけどhtmlそんなにやってない人
- コーダーさん
- Vue使ってみたいけどWebpack環境までやら無い方
- 脱jQueryしたい人

## 概要
なんの変哲も無いhtmlコーディング用の環境です。

## ディレクトリ内容

```
|--.editorconfig
|--.gitignore
|--package.json
|--css
|  |--reset.css
|  |--style.css
|--js
|  |--script.js
|  |--vue.js
|  |--vue.min.js
|--static
|  |--img
|  |--svg
|--template
|  |--hoge.html
|  |-- ・・・
|--index.html
```

html/css/javascriptについては省きます。jsディレクトリ内のvue.js(開発用)とvue.min.js（プロダクション用）は[こちら](https://jp.vuejs.org/v2/guide/installation.html#lt-script-gt-%E7%9B%B4%E6%8E%A5%E7%B5%84%E3%81%BF%E8%BE%BC%E3%81%BF)で入手可能です。

### .editorconfig
こちらはコードのインデントや記述ルールに関するファイルです。[こちら](https://editorconfig.org/)のサイトからコピペで使えます。複数人でコードを共有するときタブとかスペースとかで喧嘩しないように仲良く開発しましょう。
下に例を貼っておきます。

```
| root = true |
|:--|
|  |
|  |
| [*] |
|  |
| indent_style = space |
| indent_size = 2 |
|  |
| end_of_line = lf |
| charset = 'utf-8' |
| trim_trailing_whitespace = true |
| insert_final_newline = true |
|  |
| [*.md] |
| trim_trailing_whitespace = false |
```

### package.json
npmパッケージを管理するためのファイルで、どんなパッケージをインストールしたかが書いてあり、他の人も使えるようにするため何を導入すればいいかが記述してあるファイルです。そのファイルと同じディレクトリで```npm install```や``` yarn install ```を叩くと記述されたものや依存関係にあるものを全てコマンド一発で導入してくれます。こんかいのパっケージではシンプルなサーバーを使うため```http-server```というものを使います。こちらはグローバルインストールするパッケージなので、事前に```npm install -g http-server```を打っておくことをお勧めします。そしてそのあとにpackage.jsonのscript内に```"dev": "http-server"```を追記します。こうすることで、ディレクトリ内で```npm run dev```と打つとサーバーが立ち上がりhtmlファイル達をブラウザでそのまま参照することができます。package.json自体は```npm init```のコマンドを作業ディレクトリをうち質問に答えると生成されます。

```
{
  "name": "html-starter",
  "version": "1.0.0",
  "description": "",
  "main": "index.html",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "http-server" // <<= この部分を追記しましょう。
  },
  "repository": {
    "type": "git",
    "url": "
  },
  "author": "",
  "license": "MIT",
  "bugs": {
    "url": "https://hoge"
  },
  "homepage": "https://hoge"
}

```

コーディング現場でもモダンフロントエンド開発っぽくできますので、じゃんじゃん使っていただけると嬉しいです。今回の内容のリポジトリは[こちら](https://github.com/isihigameKoudai/FrontendEnv/tree/html-starter)の```html-starter```にあります。
