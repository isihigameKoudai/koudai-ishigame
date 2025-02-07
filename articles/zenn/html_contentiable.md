---
title: "漢は黙ってhtml contenteditable"
emoji: "🍣"
type: "tech"
topics:
  - "html"
  - "javascript"
  - "editor"
published: true
published_at: "2025-02-07 22:38"
---

## はじめに
ブラウザで動くHTML1ライナーテキストエディタ作ったのでコピペ＆ブクマして使ってみてください
## コード

```html
data:text/html,<html contenteditable><script>window.addEventListener("beforeunload",(e)=>{document.body.innerText.length>0&&(e.preventDefault(),e.returnValue="")})</script></html>
```
