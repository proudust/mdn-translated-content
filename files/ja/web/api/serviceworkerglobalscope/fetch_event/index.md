---
title: "ServiceWorkerGlobalScope: fetch イベント"
slug: Web/API/ServiceWorkerGlobalScope/fetch_event
l10n:
  sourceCommit: 6f2391b536a4db607ea94e4ce07396b2cead6a88
---

{{APIRef("Service Workers API")}}

**`fetch`** イベントはメインスレッドがネットワークリクエストを行うときに発生します。これによりサービスワーカーがリクエストをカスタマイズされたレスポンス（ローカルキャッシュなど）に差し替えることができます。

このイベントはキャンセル不可で、バブリングしません。

## 構文

このイベント名を {{domxref("EventTarget.addEventListener", "addEventListener()")}} 等のメソッドで使用するか、イベントハンドラープロパティを設定するかしてください。

```js
addEventListener("fetch", (event) => {});

onfetch = (event) => {};
```

## 解説

この `fetch` イベントは、メインスレッドがネットワークリクエストを行う際に、サービスワーカーのグローバルスコープで発生します。これはメインスレッドからの明示的な {{domxref("fetch()")}} 呼び出しだけではなく、ページナビゲーションや JavaScript、CSS、画像などのリソースの取得による暗黙的なリクエストでも発生します。

イベントハンドラは {{domxref("FetchEvent")}} オブジェクトを受け取り、{{domxref("Request")}} インスタンスを通じてリクエストにアクセスできます。

`FetchEvent` はパラメータに {{domxref("Response")}} 、または `Response` で解決する `Promise` を受け取る {{domxref("FetchEvent.respondWith()", "respondWith()")}} メソッドを持っています。これにより、サービスワーカーのイベントハンドラはメインスレッドでリクエストに返されるレスポンスを差し替えることができます。

例えばサービスワーカーは以下のような値へ差し替えることができます。

- {{domxref("Cache")}} インターフェースから取得したレスポンスのローカルキャッシュ
- {{domxref("Response.json()")}} や {{domxref("Response.Response()", "Response()")}} コンストラクタなどのメソッドでサービスワーカーが合成したレスポンス。
- {{domxref("Response.error_static()", "Response.error()")}} による ネットワークエラー。この場合 `fetch()` のプロミスは拒否されます。

`respondWith()` メソッドは、一つのリクエストに対して一度だけ呼び出すことができます。複数の `fetch` イベントリスナーが設定された場合、`respondWith()` が呼び出されるまで、登録された順に呼び出されます。

`respondWith()` メソッドは同期的に呼び出す必要があります。つまり `then` ハンドラから呼び出すことはできません。

通常、`fetch` イベントハンドラは URL などのリクエスト特徴に応じて異なる戦略をとります。

```js
function strategy1() {
  return fetch("picnic.jpg");
}

function strategy2() {
  return Response.error();
}

const pattern1 = /^\/salamander/;
const pattern2 = /^\/lizard/;

self.addEventListener("fetch", (event) => {
  const url = new URL(event.request.url);
  if (pattern1.test(url.pathname)) {
    event.respondWith(strategy1());
  } else if (pattern2.test(url.pathname)) {
    event.respondWith(strategy2());
  }
});
```

ハンドラ内で `respondWith()` が呼び出されなかった場合、ユーザーエージェントは自動的に元のネットワークリクエストを行います。例えば上記のコードでは `pattern1` または `pattern2` にマッチしないリクエストはすべて、サービスワーカーが存在しなかったかのように振る舞います。

## イベント型

{{domxref("FetchEvent")}} です。

## 例

### キャッシュからネットワークへのフォールバック

この `fetch` イベントハンドラは、まずキャッシュ済みのレスポンスを探します。レスポンスが見つかった場合はキャッシュ済みのレスポンスを返します。そうでない場合はネットワークからリソースを取得しようとします。

```js
async function cacheThenNetwork(request) {
  const cachedResponse = await caches.match(request);
  if (cachedResponse) {
    console.log("Found response in cache:", cachedResponse);
    return cachedResponse;
  }
  console.log("Falling back to network");
  return fetch(request);
}

self.addEventListener("fetch", (event) => {
  console.log(`Handling fetch event for ${event.request.url}`);
  event.respondWith(cacheThenNetwork(event.request));
});
```

### キャッシュのみ

この `fetch` イベントハンドラは、スクリプトとスタイルシートに対して「キャッシュのみ」ポリシーを実装します。リクエストの {{domxref("Request.destination", "destination")}} が `"script"` または `"style"` である場合、ハンドラはキャッシュだけを探し、レスポンスが見つからない場合はエラーを返します。その他のリクエストはすべてネットワークを通して行われます。

```js
async function cacheOnly(request) {
  const cachedResponse = await caches.match(request);
  if (cachedResponse) {
    console.log("Found response in cache:", cachedResponse);
    return cachedResponse;
  }
  return Response.error();
}

self.addEventListener("fetch", (event) => {
  if (
    event.request.destination === "script" ||
    event.request.destination === "style"
  ) {
    event.respondWith(cacheOnly(event.request));
  }
});
```

## 仕様書

{{Specifications}}

## ブラウザーの互換性

{{Compat}}

## 関連情報

- [サービスワーカーの使用](/ja/docs/Web/API/Service_Worker_API/Using_Service_Workers)
- [サービスワーカーの基本的なコード例](https://github.com/mdn/dom-examples/tree/main/service-worker/simple-service-worker)
- {{domxref("fetch()")}} メソッド
- {{domxref("Request")}} インターフェース
- {{domxref("Response")}} インターフェース
