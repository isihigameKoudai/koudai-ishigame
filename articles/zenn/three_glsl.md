---
title: "Three.jsで素早く作る、GLSL実行環境"
emoji: "😇"
type: "tech"
topics:
  - "react"
  - "typescript"
  - "webgl"
  - "glsl"
published: true
published_at: "2022-11-21 23:54"
---

## 初めに
こんにちは、WebGLで飯が食いたい[かめぽん](https://twitter.com/kamepon_fe)です。昨今、１０代の若手がプロ顔負けのCGを生み出すような世の中になり始めて、時代の転換期がきてるな〜と恐れ慄きながらCGだけでなくTouchDesignerやWebGLなどインターネット上（ブラウザ含む）での表現力が格段に上がってきていると思います。ビジネス的なアプリケーション開発だけでなく、エンターテイメントや没入感のあるコンテンツというのはどんどん需要が高まる側面と、かっこいいビジュアルが作れるという

## Three.jsについて
Threejsはみなさんご存知の方も多いと思いますが、WebGLで3Dグラフィックが扱えるjavascriptライブラリです。GLSLやWebGLの専門的な知識がそこまでなくともブラウザで3Dが手軽に扱えることで有名かと思います。
https://threejs.org/

## GLSLとシェーダーについて
GLSLとはOpenGL Shading Languageの略称で、OpenGL（厳密にはOpenGL ARB）によって策定されたC言語をベースとされたシェーダー言語のことです。シェーダーとは3Dオブジェクトがレンダリングされる際に陰影処理（シェーディング）を行うプログラムのことを指します。GLSLで具体的にどんなことができるの？という方は、[vertexshaderart](https://www.vertexshaderart.com/)など見てみるとイメージがつきやすいと思います。
https://www.vertexshaderart.com/

かつては陰影計算や頂点の処理などを固定機能のシェーダーの組み合わせによって行なっていましたが、GPUの進化によりソフトウェア的に処理を行うことができるようになっています。GLSLを含むシェーダー言語はピクセル単位で色や陰性の制御ができるようになっており、より複雑な表現をすることが可能になっています。

シェーダーにはいくつか種類がありますが、頂点単位で色や陰影制御を行うVertex Shaderとフラグメント（ピクセル）単位で陰影制御を行うFragment Shaderがよく登場します。

書いたGLSLをすぐにグラフィックに反映して試せる[the book of shader](https://thebookofshaders.com/edit.php)や[GLSL editor](http://jp.wgld.org/js4kintro/editor/)などのサイトや、[glsl-canvas](https://marketplace.visualstudio.com/items?itemName=circledev.glsl-canvas)というVSCode拡張もあるので試してみてください。

## 今回作るShaderユーティル（GLSL実行環境）について

### Shaderユーティルの概要
今回はGLSLをブラウザで手軽に表示するわけですが、そのためには通常多くの手続きを記述しなければならないのでその部分はThree.jsに任せます。ですので、作りとしてはThree.jsをラップしつつシェーダーが表示するところまでの最低限必要な手付きをまとめるような形になります。

### 使いかた

まず、Three.jsを前提としているので必要なnpmモジュールをインストールします。
```
npm i three
npm i -D @types/three
```

```glsl:vertex.vert
void main(void){
  gl_Position = vec4(position, 1.0);
}
```

その後、vertex shaderとfragment shaderをimportしたのち、Shaderをインスタンス化します。
今回はピクセル単位で陰影などを制御するためvertex shaderは以下の内容にしておきます。
基本的にはどちらのシェーダーもmain関数を必ず使い、その中に伝達すべき内容を記述します。
`gl_Position`はGLSLのビルトイン変数で、頂点データを渡す必要があります。


```glse:roundRing.frag
precision mediump float;
uniform float time;
uniform vec2 resolution;

void main(void){
  vec2 p = (gl_FragCoord.xy * 2.0 - resolution) / min(resolution.x, resolution.y);
  float cs;
  for(float i = 0.0; i < 3.0; i++) {
      float f = 0.01 / abs(length(p) - (0.5 * abs(sin(time + 0.5 * i))));
    cs = cs + f;
  }
  
  gl_FragColor = vec4(vec3(cs), 1.0);
}
```

fragment shaderでは、実際に画面表示されるシェーダーを記述します。
`precision mediump float`はprecision修飾子を使い、どれくらいの精度で計算させるかをlowp,mediump,highpのいずれか一つで設定します。
***uniform変数***は、いわばCPUから伝達されてきた値、つまりjavascript側から渡ってきた値です。これらの内容はユーティル宣言時に任意の値を渡すことができるようにしています。ここでは、時間の流れを表すtimeとディスプレイ解像度を表すresolutionを受け取ることを想定します。
main関数内では、フラグメントシェーダーで実際に目に見えるグラフィックを記述します。

```tsx:ShaderPage.tsx
import React, { useEffect, useRef } from 'react';
import * as THREE from 'three';
import Shader from '../../../packages/Shader';
import vertex from '../../../packages/glsl/vertex.vert?raw';
import roundRing from '../../../packages/glsl/roundRing.frag?raw';

const ShaderPage: React.FC = () => {
  const $shader = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    if(!$shader || $shader === null) return;

    new Shader({
      $target: $shader.current!,
      material: {
        uniforms: {
          time: {
            value: 0
          },
          resolution: {
            value: new THREE.Vector2(window.innerWidth, window.innerHeight)
          }
        },
      },
      vertexShader: vertex,
      fragmentShader: roundRing
    });
  },[]);
  
  return <div id="shader" ref={$shader}></div>
}

export default ShaderPage;
```
シェーダーの準備ができたら、今度は作成したユーティルを使って表示していきます。
シェーダーをimportしたのち、Shaderインスタンスを作成します。
$targetにはターゲットとなるDOMを設定、vertexShaderとfragmentShaderはそれぞれシェーダーを設定します。uniformsは前章で話したGLSLに渡す値を設定します。ここの細かい仕様に関しては触れませんが、最後にvalueというプロパティで実際の値を設定します。


以下のようなグラフィックが出てくれば成功です🎉🎉🎉
![](https://storage.googleapis.com/zenn-user-upload/9647dfea069d-20221121.gif)


## 参考
https://mofu-dev.com/blog/stable-fluids/
https://qiita.com/doxas/items/00567758621bb506e584
https://ics.media/entry/14771/
https://qiita.com/konweb/items/ec8fa8cd3bc33df14933