---
title: Vue.js 入門 （htmlベースで基本機能を知ろう編）
tags: JavaScript Vue.js HTML フロントエンド CSS
author: isihigameKoudai
slide: false
---
## 概要
昨今のフロントエンド界隈ではReact.jsの勢いが凄まじく有名なサービスでの導入事例が目立っています。そんな中、Vue.jsもReactに負けず劣らずの勢いで伸びています。自分自身もReact + Reduxを勉強していましたが~~半泣き状態だったこともあり~~、浮気がてらVue.jsを触ってみたら俺でもイケんじゃん！となり一人でも多くの人に自分でもできる！これイイじゃん！となってもらえたらイイなと思ったので今回の記事を書きました。最後まで読んで試していただけたら嬉しいです。

## Vue.jsとは・特徴
・javascriptで動くフレームワークの一つ。
・先進的なのでプログレッシブフレームワークとも呼ばれている。
・Vue.js自体はMVCフレームワークで言う所の、viewの部分を担っています。したがって、アプリケーションの状態やロジックの複雑な管理までは完全に考慮は出来ません。この部分はVueの他のエコシステムを使って管理します。
・Vue.jsで一からプロダクトを生成することもできますし、既存のサービスの一部分にだけVue.jsを適用させることもできるので規模に応じて扱うことができます。
・VuexやVue-Routeなどのエコシステムが公式から出ているのでシステムを組むときに迷わない
・あとは以下の通り↓
Approachable(親しみやすい)
Veratile(融通が効く)
Performant(高性能)
Maintainable(メンテしやすい）
Testable(テストしやすい)

### どんな環境で動くか
・CDN(Contents Delivery Network)を使ってHTML内にVueを取り入れたり、jsファイルに分割したりできます。jQueryを扱うような感覚で、Vuejsも手軽に扱うことができます。
・vue-cliやwebpack環境など最終的にバンドル（ひとまとめに）するようなモダンフロントエンド環境でも動きます。

## Vue入門編
今回は入門編と言うことでhtmlファイルベースでVueを簡単に説明していきます。
### 1,下準備
CDNで試すので以下のようにhtmlファイルを準備してください。
headタグ内のscriptタグでvue.jsを読み込むことでhtmlファイル内で手軽に扱えます。

```html

<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">     
    <meta http-equiv="Content-Type" content="text/html"/>
  </head>
  <body>
  
  </body>
  <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  <script>

  </script>
</html>
```

### 2 Instance, data

bodyタグの中に以下の記述を追加してください。

```html

<div id="app">
  <p>{{message}}</p>
</div>
```
bodyの下のscriptタグ内に以下の記述を追加してください。

```js
var app = new Vue({
  el: "#app",
  data() {
    return {
      message: "Hello Vue.js"
    }
  }
});
```
画面内に「Hello Vue.js」の文字が出れば成功です。
Vue.jsではVueインスタンスと呼ばれるインスタンスを生成し、elプロパティで指定したid要素（ここでは#app）の中にデータを流し込みます。#appの中の {{message}} がVueのmessageのdataを表示しています。

ここで下記のようにブラウザのコンソール画面で「app.message="Good!"」と打ってみると、ブラウザがロードせずとも文字が変わるのが確認できると思います。
Vueのdataがリアクティブになっているので、値が変更されれば即時反映されるようになっているのがVueの特徴でもあります。


### 3 v-model
先ほどの#app内にinputタグを以下のように記載してみてください。すると画面内に、messageの内容が反映されたテキストボックスが出てきます。
inputタグに「v-model="message"」との記載がありますが、こはv-modelを記載したタグとv-modelで指定を受けたプロパティ同士で双方向なデータのやりとり（双方向データバインディング）をしています。これによりテキストボックスの入力内容が即時にpタグに反映され、何かの拍子にmessageの内容が変わればテキストボックスの内容も変わります。

```html
<div id="app">
  <input v-model="message" type="text" />
  <p>{{message}}</p>
</div>
```

### 4 v-ifによる条件つきレンダリング
以下のコードを記述してもらうとおそらくh1要素が条件によって描写されます。v-if,v-eles-if,v-elseが条件によって要素の出し分けを行ってくれます。この時も条件はリアクティブになっているのでコンソールから「app.num=」で使われている値を変えてみるとリアルタイムで要素が切り替わります。

html側に以下のソースを記述します。

```html
<h1 v-if="num >= 10">１０以上</h1>
<h2 v-else-if="num >= 5">５以上</h2>
<h3 v-else>４未満</h3>
```

script側のdataには以下のコードを加えます。

```js
num: 10,
```

### 5 v-forによるリストレンダリング
v-forでは配列データを使ってタグを繰り返しレンダリングを行うことができます。繰り返す要素に対してv-forの記述をし表示したい配列のデータで繰り返します。その中の要素で取り出したデータをパースして表示することができます。非同期で取ってきたデータなどを反映させたり、リストを追加したり削除したりするときにも便利です。

html側に以下のソースを記述します。

```html

<ul>
  <li v-for="todo in todos">
    <p>{{todo.id}} {{todo.text}}</p>
  </li>
</ul>
```

script側のdataには以下のコードを加えます。（textはご自由に）

```js

todos: [
  {id:1, text:"油そばを食べる"},
  {id:2, text:"家の掃除をする"},
  {id:3, text:"食料を買って帰る"}
]
```

### 5 v-on,method
Vue.jsではdataだけでなくメソッドも追加することができ、viewの中のUIからイベントを発火させることができます。
v-on:clickなどの記載ができますが@clickと省略も可能です。イベントは複数登録が可能です。

html側に以下のソースを記述します

```html

<input 
  type="button" 
  value="PUSH"
  v-on:click="tapButton()" 
/>
```

script側のmethodにtapButtonと言うメソッドを加えます。

```js

methods: {
  tapButton() {
    console.log("tapped");
  }
}
```
コンソールにtappedの表記が出ていればOKです。

### 6 v-bindでのバインディング
Vue.jsではhtmlの属性をバインディングさせることができます。srcやvalue、class、styleなどをバインディングさせることでデータの変更があった場合に簡単に属性の変更を反映することができます。
「v-bind:calss=""」のように属性の前につけることで、Vueインスタンスのプロパティと結びつけることができます。ちなみに「v-bind:class=""」は「:class=""」と省略可能です。

クラスバインディングの例

```html

<input 
  type="button" 
  value="BUTTON"
  class="btn-toggle"
  @click="tapButton()"
  :class="{'is-active': toggleFlag}"
/>
```
script

```js

toggleFlag: false, // dataに追加

tapButton() {
  this.toggleFlag = !this.toggleFlag;
}
```

style

```css

.btn-toggle {
  widows: auto;
  height: auto;
  padding: 10px;
  color: #fff;
  background-color: #333;
  border: none;
  border-radius: 10px;
  font-size: 20px;
  font-weight: thin;
  box-shadow: #000 1px 1px 1px;
}

.btn-toggle.is-active {
  background-color: brown;
}
```

## まとめ
ここまで基礎的な部分を説明してきましたが、Vue.jsの世界はこれだけではありません。しかし、今回の部分だけでも簡単なアプリが出来そうなくらい強力なのがVue.jsの魅力でもあると思います。まだまだ可能性を秘めているのでどんどんキャッチアップしていきましょう！
Please keep enjoy hacking!!
