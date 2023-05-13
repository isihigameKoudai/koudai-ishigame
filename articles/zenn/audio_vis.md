---
title: "WebAudioAPIを使ったユーティリティを作る（ビジュアライザー編）"
emoji: "📉"
type: "tech"
topics:
  - "typescript"
  - "webgl"
  - "audio"
  - "visualizer"
published: true
published_at: "2022-08-14 00:51"
---

# 概要
こんにちは、ビジュアルディベロッパー＆フロントエンドエンジニアをしています[かめぽん](https://twitter.com/kamepon_fe)です。[前作の記事](https://zenn.dev/koudaiishigame/articles/228aa7b7524ee6)に引き続き、WebAudioAPIを使ったユーティリティ開発ということで、前回の記事で作ったユーティリティを使って今回はオーディオビジュアライザーのユーティリティを作ります。

- [WebAudioAPIを使ったユーティリティを作る（オーディオ編）](https://zenn.dev/koudaiishigame/articles/228aa7b7524ee6) 
- [WebAudioAPIを使ったユーティリティを作る（ビジュアライザー編）](https://zenn.dev/koudaiishigame/articles/fc38b0ea18581d)(本記事)

# 本題
## ユーティリティの構成・機能
オーディオビジュアライザーの作成を作る上で大体キモになってくるのが、前述で出てきたAudioContextと今回メインになってくるAnalyserNodeです。詳細は以下のリンクをチェックしてもらえたらと思いますが、ざっくりいうと時間的に変化する周波数データを離散的に取り出せるインターフェースになります。
そのインスタンスをオーディオと接続し、インスタンス経由で離散的な波形データを取得するというのがこのユーティリティの役割です。
https://developer.mozilla.org/ja/docs/Web/API/AnalyserNode

それを踏まえて本ユーティリティでは以下の機能を持たせます。

- オーディオクラスの継承
- オーディオの接続とアナライザーの設定
- 再起的なレンダリング
- ストップ

### オーディオクラスの継承&オーディオとの接続
よくオーディオビジュアライザーを調べるとAudioContextを使ったサンプルが出ますが、以前作ったので継承してヴィジュアライズする部分とつなぎます。オーディオ部分をシンプルに作っていたのはこのためでもあります。

少し行数は長いですがstart関数の中でやっていることは、以下の内容です。
- 音声の再生
- analyserNodeの作成と接続
- canvasの設定
- レンダリングの開始

まず、`super.play()`で音声の再生を始めます。この時点でAudioContextの設定は終えています。
その後、`this.context.createAnalyser()`でヴィジュアライザーのコアとなるAnalyserNodeを作成します。作成したNodeは各オーディオのNodeとアウトプット先の`this.context.destination`とconnectします。
smoothingTimeConstantやfftSizeなどフーリエ変換や周波数分析に使うプロパティも外部から設定できるようにしています。
その後は、描画先のcanvasを保存しレンダリングします。

```ts
import Audio from '...';
...
export default class Visualizer extends Audio {
...
start(
  renderCallBack: RenderCallBack,
  {
    $canvas,
    canvasWidth = window.innerWidth,
    canvasHeight = window.innerHeight,
    smoothingTimeConstant = 0.5,
    fftSize = 2048,
  }: RenderOptions
) {
  super.play();
  this.analyzer = this.context.createAnalyser();
  this.times = new Uint8Array(this.analyzer.frequencyBinCount);
  if (this._audioSource) {
    this._audioSource.connect(this.analyzer);
  }

  if (this._mediaSource) {
    this._mediaSource.connect(this.analyzer);
  }

  this.analyzer.connect(this.context.destination);
  
  $canvas.width = canvasWidth;
  $canvas.height = canvasHeight;
  this.$canvas = $canvas;
  this.analyzer.smoothingTimeConstant = smoothingTimeConstant;
  this.analyzer.fftSize = fftSize;

  this.render(renderCallBack);
}
```

### 再起的なレンダリング

レンダリング関数では、主にrequestAnimationFrameでブラウザの更新タイミングで再起的なレンダリングをするようにしています。これで時間的な音声をリアルタイムで反映するようにしています。
また、使用者側で自由に描写内容を決めれるようにrenderCallBackをpropsとして受け取るようにしています。
レンダリングするごとに、`this.analyzer.getByteTimeDomainData(this.times);`でtimesの配列データが更新されるので、それを受け取って描写内容などを変化させることができます。renderCallBackに渡すプロパティは適宜増やしてもらっても良いと思います。

```ts
render(renderCallBack: RenderCallBack) {
  if (!this.analyzer) {
    throw new Error("analyzer is null");
  }

  this.analyzer.getByteTimeDomainData(this.times);

  renderCallBack({
    $canvas: this.$canvas!,
    frequencyBinCount: this.analyzer.frequencyBinCount,
    times: this.times,
  });

  this.requestAnimationFrameId = window.requestAnimationFrame(
    this.render.bind(this, renderCallBack)
  );
}
```

### 停止
停止はとても簡単で、音声の停止・アナライザーの接続解除・requestAnimationFrameの削除になります。

```ts
stop() {
  super.stop();
  this.analyzer?.disconnect();
  window.cancelAnimationFrame(this.requestAnimationFrameId);
}
```

## サンプルのビジュアライザーを作ってみる
これまでの機能を使って実際にどういったように使うかreactのサンプルを作りながらやってみます。

```tsx
import Visualizer from '...';
...
function App() {
  const visualizer = new Visualizer();
  const $canvas = useRef<HTMLCanvasElement>(null);
  const onOpenAudio = useCallback(async () => {
    const { files } = await fetchAudio()
    const buffer = await files[0].arrayBuffer()
    visualizer.setAudio(buffer); // arrayBufferをセット
  },[]);
  
  const onPlayAudio = useCallback(() => {
    visualizer.start(({ $canvas, times, frequencyBinCount}) => {
      const $gl = $canvas.getContext('2d')
      
      const w = window.innerWidth;
      const h = window.innerHeight;
      const barWidth = w / frequencyBinCount;

    $gl!.fillStyle = "rgba(0, 0, 0, 1)";
    $gl!.fillRect(0, 0, w, h);

    for (let i = 0; i < frequencyBinCount; ++i) {
      const time = times[i];
      const percent = time / 255;
      const height = h * percent;
      const offset = h - height;
      $gl!.fillStyle = "#eee";
      $gl!.fillRect(i * barWidth, offset, barWidth, 2);
    }
      },{
        $canvas: $canvas.current!
      });
    },[]);
  
  return <div>
    ...
    <canvas id="canvas" ref={$canvas}></canvas>
  </div>
}
```

するとこんな感じの動きになると思います。start関数の中ではcanvasが帰ってきているで、シェーダーをかますなりcanvasに直接色をしてするなりよしなに実装することができます。

![](https://storage.googleapis.com/zenn-user-upload/6e4390c76317-20220814.gif)


# まとめ
どうだったでしょうか？オーディオビジュアライザーは派手で面白いのですが、その手前の準備はなかなかめんどくさいものではあります。しかし、１回ユーティリティを作ってみると内部で何をやっているか整理できるのでとてもおすすめです。実際に作ったサンプルのユーティリティを貼っておきますので、ぜひ参考にしてみてください。
では！
https://github.com/isihigameKoudai/util-packages/blob/main/packages/Visualizer.ts