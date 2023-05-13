---
title: "WebAudioAPIを使ったユーティリティを作る（オーディオ編）"
emoji: "📢"
type: "tech"
topics:
  - "typescript"
  - "audio"
  - "visualizer"
published: true
published_at: "2022-08-13 19:11"
---

# 概要
こんにちは、ビジュアルディベロッパー＆フロントエンドエンジニアをしています[かめぽん](https://twitter.com/kamepon_fe)です。今までビジネス色の強い管理画面系のアプリケーション開発をしていたのですが、ビジュアルに関わるお仕事に関わるにあたって音声やインタラクティブなコンテンツを作りたいと思うようになり、ビジュアライザーを作っていこうということで、オーディオを使った開発をしています。

が、オーディオの取り扱いをやるにあたってやることが少しばかりあるのと外部依存しないユーティリティがあればいいなと思い、汎用的な便利関数的なものを作ってみたので、web上でオーディオを使う際にご参考にしてもらえればと思いました。（もし指摘などあればコメントも受け付けています🙏）

今回、オーディオビジュアライザーを作るまでやってみたいのですが一つにまとめてしまうと少しばかり量が多くなってしまうので、記事を以下の二本立てでやっていきたいと思います。
- [WebAudioAPIを使ったユーティリティを作る（オーディオ編）](https://zenn.dev/koudaiishigame/articles/228aa7b7524ee6) (本記事)
- [WebAudioAPIを使ったユーティリティを作る（ビジュアライザー編）](https://zenn.dev/koudaiishigame/articles/fc38b0ea18581d)


# 本題
## ユーティリティの構成・機能

本ユーティリティは後のビジュアライザーの実装で拡張しやすいようにするために、機能は以下のようにシンプルにまとめてみました。
- オーディオのセット
- マイクオーディオのセット
- 再生/ストップ/ポーズ

WebAudioAPI自体の説明については以下を参照してみてください。
https://developer.mozilla.org/ja/docs/Web/API/Web_Audio_API

### オーディオのセット
まず、オーディをセットする部分のコードになります。
ここでは、[File API](https://developer.mozilla.org/ja/docs/Web/API/File)で参照した音声ファイルをArrayBuffer型に変換して取得することを想定しています。

```ts
export const createAudioContext = (): AudioContext =>
  new AudioContext() ||
  new (window.AudioContext || window.webkitAudioContext)();
  
...
async setAudio(arrayBuffer: ArrayBuffer) {
  this._context = createAudioContext();
  this._audioSource = this._context.createBufferSource();
  this._audioSource.buffer = await this._context.decodeAudioData(arrayBuffer);
}
```

オーディオの取り扱いに関しては、AudioContextというクラスを使っていきます。インスタンス化することでオーディオに関する様々な機能を使うことができます。
https://developer.mozilla.org/ja/docs/Web/API/AudioContext

createBufferSourceでは実際にみなさんが触っていく音声データの本巻のようなものと考えてもらって大丈夫だと思います。再生/停止などの機能もここに含まれています。
`  this._audioSource.buffer = await this._context.decodeAudioData(arrayBuffer);` の行では、AudioContextで設定されたレートに基づいてデコードしたものをバッファーに保存します。Promiseで返却されるので、async awaitで行います。（もっとちゃんとやるのではれば例外処理もちゃんとやると良いですね💦）

### マイクオーディオのセット(optional)
ここでは、必須ではないですがマイクで拾った音も使っていきたいので、getUserMediaを使ってマイクの収録機能も作りたいと思います。

```ts
async setDeviceAudio(constraints = { audio: true }) {
  try {
    const stream = await navigator.mediaDevices.getUserMedia(constraints);
    this._context = createAudioContext();
    this._mediaSource = this._context.createMediaStreamSource(stream);
  } catch (e) {
    console.error(e);
  }
}
```
マイク音声の取得は、`navigator.mediaDevices.getUserMedia`の非同期関数で取得することができます。詳しくは以下を参照してみてください。
https://developer.mozilla.org/ja/docs/Web/API/MediaDevices/getUserMedia
getUserMediaではconstraintsでマイクやビデオカメラの設定をすることができますが、今回はマイクオーディの取得ができれば良いのでデフォルトで `{ audio: true }`を設定することでマイクオーディオを標準で拾うようにします。

取得したマイクオーディオは[MediaStream](https://developer.mozilla.org/ja/docs/Web/API/MediaStream)という形式で保存されます。そのままではビジュアライザーで使うには扱いづらいので、MediaStreamAudioSourceNode形式で保存するために`_context.createMediaStreamSource(stream);`で変換して保存します。

### 再生/停止/一時停止
#### 再生
音声の再生に関しては音声ファイルとマイクオーディオで異なりますが、マイクオーディオに関してはstreamを取得した時点でマイクがONになるのでここでは音声ファイルの制御がメインになります。

```ts
play() {
  if (this._mediaSource) {
    return;
  }

  if (this._audioSource) {
    this._audioSource.disconnect();
  }

  this._audioSource?.connect(this._context.destination);
  this._audioSource?.start(0);
  this.isPlaying = true;
}
```

音声の再生には、オーディオ自体とアウトプット先の接続が必要になります。ギターとアンプorスピーカーをつなぐようなイメージで良いと思います。そうすることで、音声と接続されたコンテキストを使用することでビジュアライザーと連携することができますが、それについてはビジュアライザー編で説明します。
https://developer.mozilla.org/en-US/docs/Web/API/AudioNode/connect

音声とアウトプット先の接続が完了したら、`  this._audioSource?.start();`で再生します。
https://developer.mozilla.org/ja/docs/Web/API/AudioBufferSourceNode/start

#### 停止
停止については音声ファイルとマイクで方法が少し異なります。
音声ファイルに関しては、オーディオの停止（stop）と接続解除があります。接続解除はconnectにて繋いだ音声データとアウトプット先のコネクションを解除します。

```ts
// 音声ファイルの停止
stopAudio() {
  if (!this._audioSource) {
    return;
  }

  this._audioSource.stop();
  this._audioSource.disconnect();
  this._audioSource.buffer = null;
  this.isPlaying = false;
}
```
次にマイクオーディオの停止ですが、マイクは停止という概念はなく接続解除が実質の停止のようなものになります。MediaStreamAudioSourceNodeにはAudioNodeクラスを継承しておりdisconnectで解除できます。
```ts
// マイクオーディオの停止
stopDeviceAudio() {
  if (!this._mediaSource) {
    return;
  }
  this._mediaSource.disconnect();
  this._mediaSource = null;
  this.isPlaying = false;
}
```

#### 一時停止
ここまで記事を読んだ方であれば、stopが一時停止じゃないのかと思ったと思いますが、考え方的にはstopは完全なる停止で、suspend/resumeが一時停止の制御になります。MDNにも

> 音声ハードウェアへのアクセスを一時的に停止し、処理に必要だったCPU/バッテリーの使用を減らすことが出来ます

とありますので、一時停止は再生中かどうかを判断してsuspend/resumeを使い分けることでパフォーマンス的にも良いことがわかります。再生中の判断はここでは自前でやっていますが、AudioContext.stateでも判断できます。
https://developer.mozilla.org/ja/docs/Web/API/AudioContext/suspend

```ts
pause() {
  if (this.isPlaying) {
    this._context.suspend();
    this.isPlaying = false;
  } else {
    this._context.resume();
    this.isPlaying = true;
  }
}
```

# まとめ

いかがだったでしょうか、音声の取り扱いは一見難しそうに見えますが機能をシンプルにしてみるとやることはそこまで多くない印象だったと思います。ここで紹介したサンプルコードは以下のGithubで公開しているので参考にしてみてください。
次は、ここで作成したオーディオクラスを使って汎用的に使えるビジュアライザーユーティルを作ってみたいと思います。

https://github.com/isihigameKoudai/util-packages/blob/main/packages/audio.ts