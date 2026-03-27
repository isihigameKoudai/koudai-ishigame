<!-- この記事ドラフトはこのまま投稿せず、骨組みの参考程度にすること -->
---
title: "Vite 7→8 移行ガイド：Rolldown統合でビルド53%高速化した話"
emoji: "⚡"
type: "tech"
topics:
  - "vite"
  - "rolldown"
  - "フロントエンド"
  - "パフォーマンス"
  - "migration"
published: false
---

## 初めに

こんにちは、[かめぽん](https://twitter.com/kamepon_fe)です。

Vite 8 がリリースされ、内部バンドラが esbuild + Rollup から **Rolldown（Rust製）** に統一されました。

https://vite.dev/blog/announcing-vite8

「ビルドが速くなるらしい」という噂は聞いていたものの、実際にどのくらい変わるのか、移行はスムーズにいくのかが気になったので、自分のプロジェクトで Vite 7 → 8 のアップグレードを実施し、ベンチマークを取ってみました。

結論から言うと、**Production Build は約53%高速化、バンドルサイズも約20%縮小**という大きな改善が得られました。一方で、移行中にいくつかハマりポイントがあったので、その解決策も含めて共有します。

この記事は割とボリュームが多いですが、Vite 8 への移行を考えている方の参考になれば幸いです。

---

## 計測環境

まず、ベンチマークの計測条件を明確にしておきます。

### 対象プロジェクト

今回ベンチマークを取ったのは、**React 19 + TypeScript 5.9** で構成された再利用可能なユーティリティと実験的フィーチャーのサンドボックスプロジェクトです。TS/TSX ファイル **229本**、合計 **約13,000行** の中小規模コードベースで、テストファイルは **17本**（Vitest）を含みます。Three.js や @mediapipe、TensorFlow.js といった比較的重めのライブラリも依存に含まれているので、バンドルサイズやビルド時間には一定のボリュームが出る構成です。

### マシン・ツールバージョン

| 項目 | 値 |
|------|-----|
| Machine | darwin-arm64（Apple Silicon） |
| Node.js | v25.2.1 |
| Vite 7 | 7.1.9 |
| Vite 8 | 8.0.2 |
| 計測日 | 2026-03-23 |
| 計測ルール | 各指標3回計測・中央値採用、`dist/` + `node_modules/.vite` 毎回削除 |

キャッシュの影響を排除するため、毎回ビルド成果物と Vite のキャッシュディレクトリを削除してから計測しています。

---

## ベンチマーク結果

### サマリー

| 指標 | Vite 7.1.9 | Vite 8.0.2 | 変化 |
|------|----------:|----------:|-----:|
| **Production Build Time**（中央値） | 11,586 ms | 5,401 ms | **-53.4%** |
| **Bundle Size**（total） | 4.4 MB | 3.5 MB | **-20.5%** |
| **Bundle Size**（JS） | 4.4 MB | 3.5 MB | **-20.5%** |
| **Bundle Size**（CSS） | 4.0 KB | 4.0 KB | ±0% |
| **Dev Server Cold Start**（中央値） | 373 ms | 402 ms | +7.8% |

Production Build で **約2倍の高速化** が得られています。バンドルサイズも JS が約 0.9MB 減っており、ユーザーへの配信サイズの改善にもつながります。

### Production Build Time（詳細）

| Run | Vite 7 | Vite 8 |
|----:|-------:|-------:|
| 1 | 12,406 ms | 5,434 ms |
| 2 | 11,582 ms | 5,397 ms |
| 3 | 11,586 ms | 5,401 ms |
| **中央値** | **11,586 ms** | **5,401 ms** |

3回の計測でばらつきも小さく、安定して高速化されていることがわかります。

### Bundle Size

| 内訳 | Vite 7 | Vite 8 |
|------|-------:|-------:|
| Total | 4.4 MB | 3.5 MB |
| JS | 4.4 MB | 3.5 MB |
| CSS | 4.0 KB | 4.0 KB |

CSS はほぼ変わらず、JS のバンドルサイズが約20%縮小しました。これは Rolldown + Oxc ミニファイによる tree-shaking の改善によるものです。

### Dev Server Cold Start

| Run | Vite 7 | Vite 8（internal） | Vite 8（port ready） |
|----:|-------:|-----------------:|--------------------:|
| 1 | 376 ms | 402 ms | 903 ms |
| 2 | 373 ms | 396 ms | 848 ms |
| 3 | 373 ms | 407 ms | 862 ms |
| **中央値** | **373 ms** | **402 ms** | **862 ms** |

Dev Server に関しては約30ms の微増です。Rolldown の依存関係プリバンドルが esbuild より若干遅いことが原因ですが、体感としてはほぼ変わりません。

---

## 移行中のつまづきと解決策

ここからが本記事の肝です。ベンチマーク結果は華々しいですが、移行は一筋縄ではいきませんでした。

### ESM でも CJS でもないパッケージがビルドエラーになる

#### 前提知識: JavaScript のモジュール形式

この問題を理解するために、npm パッケージの配布形式について少し整理しておきます。

npm で配布されているパッケージには、主に以下のモジュール形式があります。

| 形式 | 特徴 | 例 |
|------|------|-----|
| **ESM** | `import`/`export` を使う現代的な形式 | React, Vue など主要ライブラリ |
| **CJS** | `require()`/`module.exports` を使う Node.js 由来の形式 | 古めのパッケージ全般 |
| **IIFE** | 即時実行関数。`this` やグローバル変数にプロパティを登録する | 一部の Google 系ライブラリ |

ほとんどのパッケージは ESM か CJS で配布されており、Vite はどちらも問題なく扱えます。しかし、**IIFE 形式**のパッケージは少し特殊です。`import { Hands } from '@mediapipe/hands'` のように named import しても、パッケージ側にはそもそも `export` 文が存在しないため、バンドラが解決できないのです。

#### 症状

`npm run build` で以下のエラーが発生しました。

```
[MISSING_EXPORT] "Hands" is not exported by "node_modules/@mediapipe/hands/hands.js"
[MISSING_EXPORT] "FaceDetection" is not exported by "node_modules/@mediapipe/face_detection/face_detection.js"
```

今回問題になったのは Google の `@mediapipe/*` パッケージです。これは Google Closure Compiler で生成された IIFE 形式で配布されており、内部的には `this["Hands"] = ...` のようにプロパティを登録する仕組みになっています。

#### なぜ Vite 7 では動いていたのか

Vite 7 が Production Build に使っていた esbuild は、IIFE 形式のコードでもある程度「よしなに」解釈して named import を解決してくれていました。しかし Vite 8 で導入された Rolldown は、**モジュール形式の判定がより厳密**です。`export` 文が存在しないファイルからの named import は明確にエラーとして弾かれます。

つまり、「Vite 7 で動いていたから大丈夫」とは限らないわけです。

#### 試したこと（うまくいかなかったもの含む）

| # | 試みた方法 | 結果 | なぜダメだったか |
|--:|-----------|------|-----------------|
| 1 | `legacy.inconsistentCjsInterop: true` | ❌ | これは CJS の interop 設定。IIFE は CJS ですらないので無関係 |
| 2 | `optimizeDeps.include` に追加 | ❌ | `optimizeDeps` は dev server 用の設定で、production build には影響しない |
| 3 | `build.rolldownOptions` で設定 | ❌ | Rolldown には IIFE → ESM の自動変換機能がそもそもない |

#### 解決策: カスタム Vite プラグインで IIFE → ESM に変換

最終的に、IIFE 形式のコードを ESM に変換するカスタム Vite プラグインを書いて解決しました。

```typescript
// vite.config.ts に mediapipeEsmPlugin() を追加

// プラグインの仕組み（概要）:
// 1. resolveId: @mediapipe/* の import を仮想モジュール ID に書き換え
// 2. load: 元の IIFE コードを読み込み、ESM ラッパーを生成

// 生成されるコード:
var __mediapipe_exports__ = {};
(function () {
  /* 元の IIFE コード（this にプロパティを登録する） */
}).call(__mediapipe_exports__);
export var Hands = __mediapipe_exports__['Hands'];
```

ポイントは `.call(__mediapipe_exports__)` の部分です。IIFE 内の `this` が空のオブジェクトを指すようにすることで、`this["Hands"] = ...` で登録されたプロパティを捕捉し、それを ESM の `export` として再公開しています。

Vite のプラグインフック `resolveId` と `load` を組み合わせることで、**任意のモジュール形式を ESM に変換できる**のが Vite プラグインシステムの強みです。

### 本番ビルドを直しても dev server で同じエラーが出る

#### 症状

上記のカスタムプラグインで `npm run build` は通るようになりましたが、`npm run dev` で dev server を起動すると同じ MISSING_EXPORT エラーが再発しました。

#### 原因

Vite の dev server と production build では、**依存関係の処理パイプラインが別**になっています。

| | 依存関係の処理 | 使う仕組み |
|---|---|---|
| **dev server** | 依存関係プリバンドル | `optimizeDeps`（起動時に依存を事前バンドル） |
| **production build** | フルバンドル | Rolldown + プラグイン |

production build ではカスタムプラグインが IIFE → ESM 変換を行いますが、dev server は起動時の `optimizeDeps` フェーズで Rolldown が `@mediapipe/*` を**プラグインより先に**直接プリバンドルしようとします。この時点で同じ IIFE 問題に遭遇するわけです。

#### 解決策

`optimizeDeps.exclude` に該当パッケージを追加して、プリバンドル対象から除外しました。

```typescript
// vite.config.ts
optimizeDeps: {
  exclude: [
    '@mediapipe/hands',
    '@mediapipe/face_detection',
    '@mediapipe/face_mesh',
    '@mediapipe/pose',
  ],
},
```

こうすることで dev server は @mediapipe のプリバンドルをスキップし、カスタムプラグイン経由で処理するようになります。

:::message
**学び**: Vite の production build と dev server は依存関係の解決パスが異なります。片方で直しただけでは不十分な場合があるので、移行時は**両方をテスト**することが必須です。
:::

---

## Vite 7→8 移行チェックリスト

移行で気をつけるべきことをまとめます。

1. **パッケージのモジュール形式を実際に確認する** — `package.json` の `main`/`module` フィールドだけでなく、ファイルの中身を見る。CJS / ESM / IIFE / UMD で対処法が異なる
2. **Rolldown は esbuild より厳密** — Vite 7 で暗黙的に動いていた interop が壊れる可能性がある
3. **production build と dev server の両方をテストする** — 依存関係の処理パスが異なる
4. **Warning を無視しない** — Error にならなくても実行時に壊れることがある
5. **カスタムプラグインは最終手段だが、確実** — `resolveId` + `load` フックで任意のモジュール形式を変換できる

---

## まとめ

いかがだったでしょうか。Vite 7 → 8 への移行で、Production Build が **53%高速化**、バンドルサイズが **20%縮小** という大きな改善が得られました。Rolldown の統合によるパフォーマンス向上は数字として明確に表れており、移行する価値は十分にあると思います。

一方で、Rolldown は esbuild よりモジュール形式の判定が厳密になっています。ESM でも CJS でもない形式（IIFE など）で配布されているパッケージがあると、Vite 7 では動いていたのに Vite 8 でエラーになるケースがあります。その場合はカスタムプラグインで対処できますが、**dev server と production build の両方で動作確認する**ことを忘れないようにしましょう。

この記事が Vite 8 への移行を検討している方の参考になれば幸いです。最後まで読んでいただきありがとうございます！

## 参考

https://vite.dev/
https://rolldown.rs/
https://oxc.rs/
