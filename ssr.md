---
marp: true
---

# SSR Streaming と React Server Components

---

https://github.com/reactwg/react-18/discussions/37
![h:200px](./CleanShot%202022-09-30%20at%2015.12.18.png)

https://github.com/josephsavona/rfcs/blob/server-components/text/0000-server-components.md
![h:200px](./CleanShot%202022-09-30%20at%2015.15.24.png)

---

例としてこのコンポーネントを使ってます

```tsx
const App = () => {
  const [count, setCount] = useState(0)
  const increment = useCallback(() => setCount((c) => c + 1), [])
  return (
    <div>
      <button onClick={increment}>increment</button>
      <p>count: {count}</p>
    </div>
  )
}
```

---

# CSR

初めは 1 つの div だけ送信した後、JS で DOM 操作して UI を作るやり方

```html
<html>
  <body>
    <div id="button_container"></div>
    //JS
    <script src="like-button.js"></script>
  </body>
</html>
```

---

![](./CleanShot%202022-09-30%20at%2016.00.42.png)

---

# SSR

```html
<html>
  <body>
    <div id="button_container">
      // serverで生成したhtml
      <div data-reactroot="">
        <button>increment</button>
        <p>
          count:
          <!-- -->0
        </p>
      </div>
      //
    </div>

    //JS
    <script src="like-button.js"></script>
  </body>
</html>
```

---

### 初期描画 →hydrate→ 最終描画

![h:600](./CleanShot%202022-09-30%20at%2016.08.22.png)
![bg right h:300px](./CleanShot%202022-09-30%20at%2016.11.26.png)

---

# 従来の SSR の課題

- HTML を返すためにすべての component を renderToString しないといけない
- hydrate するためにすべての JS を読み込まないといけない

etc...

レスポンスの遅い API を叩いていたり、複雑な JS 処理を(カルーセルなど)もつ component があるとブロッカーとなりやすい

---

# SSR Streaming

HTML 生成が終わったところから、順次 client に返却していく

![h:400](./CleanShot%202022-09-30%20at%2016.16.21.png)

---

# SSR Streaming

読み込んだ JS から順次 hydrate していく

![h:400](./CleanShot%202022-09-30%20at%2016.18.01.png)

---

# SSR Streaming

| HTML が全て揃う前に hydrate 開始                       | 緊急性によって hydrate する順番を入れ替える            |
| ------------------------------------------------------ | ------------------------------------------------------ |
| ![h:300](./CleanShot%202022-09-30%20at%2016.19.58.png) | ![h:300](./CleanShot%202022-09-30%20at%2016.21.14.png) |

---

# 巨大化する JS

```tsx
import marked from 'marked'; // 35.9K (11.2K gzipped)
import sanitizeHtml from 'sanitize-html'; // 206K (63.3K gzipped)

function NoteWithMarkdown({text}) {
  const html = sanitizeHtml(marked(text));
  return (/* render */);
}
```

必要なのは数 byte の処理なのに 200kbyte を読み込まないといけない。。

---

# React Server Components

- Server Components(`***.server.js`)
- Client Components(`***.client.js`)
  - Client Component の中では、Server Component を読み込めない

---

# React Server Components

server 側でやる処理

native dom(div など)か Client Component に到達するまで、Server Component をレンダーする

---

こんなのがclientに送られるイメージ
```json
{
  '$$typeof': Symbol(react.element),
  type: 'ClientComponent', // ClientComponent
  key: null,
  ref: null,
  props: { children: [
    {
  '$$typeof': Symbol(react.element),
  type: 'div', // div
  key: null,
  ref: null,
  _owner: null,
  _store: {}

    }
   ] },
  _owner: null,
  _store: {}
}
```

---

# SSRとServer Componentsの比較

||  SSR  |  Server Components  |
|---| ---- | ---- |
|やり取りするもの|  HTML string  |  React Object(JSON)  |

