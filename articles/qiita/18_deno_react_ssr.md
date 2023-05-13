---
title: DenoでReact Server Side Renderingした話
tags: deno TypeScript React JavaScript フロントエンド
author: isihigameKoudai
slide: false
---
## 概要
最近何かと注目が集まってるDenoですが、なんとDenoでjsxが動くみたいなのでDenoでReactが動くかやってみました。

## Denoってなんぞや
これについては、今Deno Advent Calendarにて@kt3kさんが [Deno ってなんだっけ?](https://qiita.com/kt3k/items/e1647683ad08ff6b6e95)と言う記事をあげているので、そちらを参考にしてみてください。

## Denoのインストール
何はともあれ、Denoを導入してみましょう！

```
brew install deno
```
インストールが完了したら、`deno -v`で以下のような画面が出てきたらインストール完了です。
<img width="400" alt="スクリーンショット 2019-12-03 23.26.35.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/105049/9367b03d-df3f-447f-2c12-0c97f0229388.png">

## コードを書いてみよう
早速コードを書いていきたいわけですが、今回二つファイルを作ります。

- index.tsx
- App.tsx

### App.tsx

こちらはいつも通りのreactコンポーネントです。一つ違うのはDenoはnode_modulesではなく直接ダウンロードする方式ですね。後のコードはいつも通りです。

```tsx:Apptsx
import React from 'https://dev.jspm.io/react';

const App = () => <div>Hello Deno React</div>;
export default App;
```

### index.tsx

次にindexファイルですが、

```tsx:index.tsx
import { createRouter } from 'https://denopkg.com/keroxp/servest/router.ts';
import React from 'https://dev.jspm.io/react';
import ReactDOMServer from 'https://dev.jspm.io/react-dom/server';

import App from './app.tsx';

const router = createRouter();
router.handle('/', async req => {
  await req.respond({
    headers: new Headers({
      'content-type': 'text/html; charset=UTF-8',
      status: 200,
    }),
    body: ReactDOMServer.renderToString(
      <html>
	<head>
          <title>deno react ssr</title>
	</head>
	<body>
          <App />
	</body>
      </html>
    )
  })
});

router.listen(':8000');
```

ファイルの準備は以上になります。
ここで、以下コマンドを実行した上でhttp:/localhost:8000/にアクセスしてみましょう！

```
deno index.tsx --allow-net // denoの起動
```

これで画面上に `Hello Deno React`が出れば成功です！
Deno、未来ありますよね！では！

