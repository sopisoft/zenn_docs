---
title: "目次の今読んでいる項目をハイライトする"
emoji: "👀"
type: "tech"
topics: [nextjs, react, web]
published: true
---

# 最初に

Zenn の目次は、今読んでいる部分が強調されて、長い記事でも読みやすいですよね。  
この記事では、今読んでいる部分がハイライトされる目次を作ろうと思います。

私は、はじめに 2 つの実装方法を考えました。

1. 要素の高さを取得して比較する
2. Intersection Observer API を使う

https://developer.mozilla.org/ja/docs/Web/API/Intersection_Observer_API

まずは、どちらで実装するか考えます。

:::message
今回は、目次のリストに `.toc` クラスが、アンカーに `.header-anchor-link` クラスが設定されています。
また、`.toc .active` で目次がハイライトされます。
:::

## 1. 要素の高さを取得して比較する

```typescript
const anchorsArray = Array.from(
  document.querySelectorAll(".header-anchor-link")
);
const tocArray = Array.from(document.querySelectorAll(".toc"));

//各要素の高さを取得
//各要素の位置=前の要素の高さの総和

window.addEventListener("scroll", () => {
  const currentScrollY = window.scrollY;
  for (let i = 0; i < tocArray.length; i++) {
    // 処理
  }
});
```

こんな感じになるでしょうね。

一見良さそうに見えますが `window.scrollY` などの処理をスクロールイベントごとに実行するのはあまり良い考えとは言えないかもしれません。これらの処理は同期的で、大量のイベントが発火してしまうスクロールイベント内で実行すると、スクロールを阻害してしまいますし、パフォーマンスが低下します。
また、この方法では `<detail>` 要素などを使う場合は、それらのチェンジイベントも取得する必要があるため、面倒です。

## 2. Intersection Observer API を使う

```typescript
const anchorsArray = Array.from(
  document.querySelectorAll(".header-anchor-link")
);
const tocArray = Array.from(document.querySelectorAll(".toc"));

const observerCallback = (entries: any) => {
  // 処理
};

const observer = new IntersectionObserver(observerCallback);

anchorArray.forEach((item: Element) => {
  observer.observe(item);
});
```

Intersection Observer API を使うと、上記のような形で実装できます。

この方法ではスクロールイベントを用いないので、前述の方法に比べてパフォーマンスを期待できます。
Intersection Observer API は、 IE や Safari などのブラウザでは動作しませんが、今回は見なかったことにします。

# 実装してみる

今回は 2 つ目の Intersection Observer API を使う方法で実装しました。

![](/images/highlight-the-toc/result.gif)

## 完成品

```typescript
const router = useRouter();

React.useEffect(() => {
  if (document !== undefined) {
    const highlightToc = () => {
      const anchorsArray = Array.from(
        document.querySelectorAll(".header-anchor-link")
      );
      const tocArray = Array.from(document.querySelectorAll(".toc"));

      const options = {
        root: null,
        rootMargin: "0% 0px -80% 0px",
        threshold: 1,
      };

      const observerCallback = (entries: any) => {
        const entry = entries.find(
          (entry: { isIntersecting: any }) => entry.isIntersecting
        );
        if (entry) {
          const index = anchorsArray.indexOf(entry.target);
          tocArray.forEach((item, i) => {
            i === index
              ? item.classList.add("active")
              : item.classList.remove("active");
          });
        }
      };

      const observer = new IntersectionObserver(observerCallback, options);

      anchorsArray.forEach((item: Element) => {
        observer.observe(item);
      });
    };

    router.events.on("routeChangeComplete", () => {
      highlightToc();
    });

    highlightToc();
  }
}, []);
```

## API のオプション

```typescript
const options = {
  root: null,
  rootMargin: "0% 0px -80% 0px",
  threshold: 1,
};
```

`rootMargin: "0% 0px -80% 0px"` は、アンカーを下からの 80%の位置に入った時発火する設定しています。下図ではオレンジで囲まれた部分です。

![](/images/highlight-the-toc/eventArea.png)

`threshold: 1`は、アンカーが完全に入った時発火する設定です。普通は 1 に設定しませんが、今回はターゲットの要素が小さいので、1 に設定しています。

## ページ遷移を取得する

ページ遷移を取得しようとして、個人的につまずいた点です。Next.js のルーティングについてよくわかっていませんでした。
Next.js では `<Link>` を使ったルーティングでは、遷移先で `window.addEventListener(("DOMLoaded", () => {...}))` や `window.onload = () => {...}` など、ページを読み込んですぐ発火するイベントが使えないので `router.events.on("routeChangeComplete", () => {...})` を代わりに使う必要があります。これは `<Link>` を使ったルーティングでルートが完全に変更されたときに発火するイベントです。

https://nextjs.org/docs/api-reference/next/router

# 最後に

### 参考

- [Intersection Observer API - Web API | MDN](https://developer.mozilla.org/ja/docs/Web/API/Intersection_Observer_API)
- [next/router | Next.js](https://nextjs.org/docs/api-reference/next/router)
