---
title: AMP（Accelerated Mobile Pages）入門
tags: HTML JavaScript フロントエンド AMP 入門
author: isihigameKoudai
slide: false
---
## 概要
こんにちは、都内でフロントエンドエンジニアをしている[かめぽん](https://twitter.com/kamepon_fe)です。最近、Next.jsのバージョンアップにて対応したことで一層話題になっているAMPですが、僕自身AMPのことをそこまで注目していなくてたまたま調べてみたら割とおもしろうそうだったので、インプットした内容をまとめてみました。サンプルコードなども載せていますので、AMPの基礎的な情報の参考にしていただければと思います。

## 本題
### AMPとは、メリット
AMPとはAccelerated Mobile Pagesの略で、GoogleとTwitterが発足したモバイルページを高速に表示させるためのプロジェクト、またはAMP HTMLというフレームワークのことを指します。

AMP自体は以下の構成要素で成り立っています。

- AMP HTML
- Custom Styling(CSS)
- AMP js
- AMP Cache(CDN)

静的なHTML, CSS, jsは普通のwebと同じですがAMP HTMLはAMP独自の仕様が定められており、その仕様どおりに沿ってモバイルサイトを作成することで表示速度を高速にすることができます。
加えて、AMPで作られたサイトはクローラに取り込まれた後AMP Cacheと呼ばれるCDNに保存されます。つまり、従来のWebと異なるのは、実際にURL先にリソースを取りに行くのではなくCDN先にリソースを読みに行くため通常アクセスよりも速く表示できることが特徴です。

AMPの基本的なスタンスとしては「読み込みに時間がかかるようなことはさせない」ので高速化を進めていく代わりに、現段階でjsを非同期でしか読み込むことができないという制約があり、インタラクティブで動的なサイトまたはそのようなアプリはまだ不向きなようです。

## 実践
ここからは実際のソースコードでやっていきたいと思います。

### How to use AMP

#### HTML宣言
最初にHTML宣言ですが以下のように記載します。雷マークがampを意味します。`<html amp lang="ja">`でも意味は同じです。

```html

<!doctype html>
<html ⚡="" lang="ja">
```

ampはutf-8のみ対応そしてviewportの設定が必須なので記載します。

```html

<link rel="canonical" href="index.html">
<meta name="viewport" content="width=device-width,minimum-scale=1,initial-scale=1">
```

#### application/ld+jsonの設定
ampの設定の中でも大事なのが、LD+jsonです。こちらは、本サイトのamp構造を正しく伝えるための情報になります。Goolgeがschema.orgとLD+jsonで記述することを推奨しています。

```html

<script type="application/ld+json">
{
  "@context": "",
  "@type": "Article",
  "headline": "Welcom to AMP SAMPLE",
  "image": [
    "sample.jpeg"
  ],
  "datePublished": "2019-07-28T08:00:00+08:00"
}
</script>
```

### スタイルの読み込み

ampはインラインスタイルしか読めないので、cssファイルを用意するのではなくamp-boilerplateとして登録しておきます。

```html

<style amp-boilerplate>a,abbr,acronym,address,applet,article,aside,audio,b,big,blockquote,body,canvas,caption,center,cite,code,dd,del,details,dfn,div,dl,dt,em,embed,fieldset,figcaption,figure,footer,form,h1,h2,h3,h4,h5,h6,header,hgroup,html,i,iframe,img,ins,kbd,label,legend,li,mark,menu,nav,object,ol,output,p,pre,q,ruby,s,samp,section,small,span,strike,strong,sub,summary,sup,table,tbody,td,tfoot,th,thead,time,tr,tt,u,ul,var,video{margin:0;padding:0;border:0;font-size:100%;font:inherit;vertical-align:baseline}article,aside,details,figcaption,figure,footer,header,hgroup,menu,nav,section{display:block}body{line-height:1}ol,ul{list-style:none}blockquote,q{quotes:none}blockquote:after,blockquote:before,q:after,q:before{content:'';content:none}table{border-collapse:collapse;border-spacing:0}</style>
```

#### javascriptの読み込み

ampでは、jsは非同期で読み込むことを推奨しているので、async属性をつけましょう。

```html

<script async src="https://cdn.ampproject.org/v0.js"></script>
<script async src="./script.js"></script>
```

あとはいつも通りテンプレートを書けば最低限の対応は大丈夫です。LD+jsonに関してはもっと書くべきですがここでは割愛します。以下、ここまでのサンプルを貼っておきます。

```html

<!doctype html>
<html ⚡="" lang="ja">
  <head>
    <meta charset="utf-8">
    <title>AMP SAMPLE</title>
    <link rel="canonical" href="index.html">
    <meta name="viewport" content="width=device-width,minimum-scale=1,initial-scale=1">
    <script type="application/ld+json">
    {
      "@context": "http://schema.org",
      "@type": "Article",
      "headline": "Welcom tao AMP SAMPLE",
      "image": [
        ""
      ],
      "datePublished": "2015-02-05T08:00:00+08:00"
    }
    </script>
    <style amp-boilerplate="">body{-webkit-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-moz-animation:-amp-start 8s steps(1,end) 0s 1 normal both;-ms-animation:-amp-start 8s steps(1,end) 0s 1 normal both;animation:-amp-start 8s steps(1,end) 0s 1 normal both}@-webkit-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-moz-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-ms-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@-o-keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}@keyframes -amp-start{from{visibility:hidden}to{visibility:visible}}</style><noscript><style amp-boilerplate="">body{-webkit-animation:none;-moz-animation:none;-ms-animation:none;animation:none}</style></noscript>
    <style amp-custom="">a,abbr,acronym,address,applet,article,aside,audio,b,big,blockquote,body,canvas,caption,center,cite,code,dd,del,details,dfn,div,dl,dt,em,embed,fieldset,figcaption,figure,footer,form,h1,h2,h3,h4,h5,h6,header,hgroup,html,i,iframe,img,ins,kbd,label,legend,li,mark,menu,nav,object,ol,output,p,pre,q,ruby,s,samp,section,small,span,strike,strong,sub,summary,sup,table,tbody,td,tfoot,th,thead,time,tr,tt,u,ul,var,video{margin:0;padding:0;border:0;font-size:100%;font:inherit;vertical-align:baseline}article,aside,details,figcaption,figure,footer,header,hgroup,menu,nav,section{display:block}body{line-height:1}ol,ul{list-style:none}blockquote,q{quotes:none}blockquote:after,blockquote:before,q:after,q:before{content:'';content:none}table{border-collapse:collapse;border-spacing:0}</style>
    <style>html{font-family:ヒラギノ明朝 ProN W6,HiraMinProN-W6,HG明朝E,ＭＳ Ｐ明朝,MS PMincho,MS 明朝,serif}.wrap{position:relative;width:auto;text-align:center;margin-top:80px}.title{position:relative;font-size:36px;color:#402c2c;font-family:Cambria,Georgia,serif;font-style:normal;-webkit-font-feature-settings:normal;font-feature-settings:normal;font-variant:normal;font-weight:500;line-height:39.6px;letter-spacing:2px}.title::after{position:relative;content:"";display:block;bottom:0;width:50px;height:3px;top:18px;background:#402c2c;margin-left:auto;margin-right:auto}.sub-title{position:relative;font-size:32px;color:#402c2c;font-family:HG明朝E,ＭＳ Ｐ明朝,MS PMincho,MS 明朝,serif;font-style:normal;-webkit-font-feature-settings:normal;font-feature-settings:normal;font-variant:normal;font-weight:500;line-height:36.6px;letter-spacing:2.5px;margin-top:80px}.flex-wrap{width:970px;height:auto;margin-left:auto;margin-right:auto;display:flex;flex-wrap:wrap;justify-content:space-between;margin-top:60px}.box{width:300px;height:300px;margin-top:40px;-o-object-fit:cover;object-fit:cover;border:none;background:0 0;overflow:hidden;background-position:50%;background-size:cover;background-repeat:no-repeat;transition:all .2s ease;background-image:url(img/sample.jpg)}.box:hover{filter:drop-shadow(3px 3px 4px #909090f0);transform:translate(-1px,-1px)}@media screen and (max-width:768px){.flex-wrap{width:100%}.box{width:33vw;height:33vw;margin:5px}}</style>
    <script async src="https://cdn.ampproject.org/v0.js"></script>
  </head>
  <body>
    <div id="app">
        <div class="wrap">
            <h1 class="title">Portfolio</h1>
            <p class="sub-title">Creative</p>
        </div>
        <div class="flex-wrap">
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
            <div class="box"></div>
        </div>
    </div>
  </body>
</html>

```


また、今回作ったページとGithubのリンクも貼っておきます。ぜひ、参考にしていただけたらと思います。

[サンプルページ](https://reverent-kare-0d5ca3.netlify.com/)
[Github](https://github.com/isihigameKoudai/amp-sample)

最後にページが出来上がったらGoogleが提供しているツールでAMPページとして有効かどうかチェックしてくれますので、コードでも良いですしURLを貼り付けて調べることができます。

[Google Search Console](https://search.google.com/test/amp?hl=ja)

### まとめ
- AMPとはモバイルのページを高速に表示させる技術
- AMPに対応はAMP用のhtmlが必要
- AMPはAMP独特の制約がある

静的なデータでAMP対応のサイトができました。AMP対応により、サイトがかなり高速に表示されるのはかなりのアドバンテージになると思います。特に、メディアやCMS、ニュースサイトやECサイトなどサイトの速さが命のページにとってはかなり強みになると思います。逆に、javascriptの取り扱いに制約を感じることがあるのでWebアプリケーションとしてAMPの導入はもう少し様子見かなという印象です。Webアプリに関してだとPWAという選択肢もあるので、サービスの特徴によって活用していきたい所存ではあります。また、フロントエンド領域に置いてキャッシュの取り扱いやCDNの知識が必須になってきているので、その部分のキャッチアップもしていきたいと思いました。

### 参考
https://search.google.com/test/amp?hl=ja
https://amp.dev/ja/
