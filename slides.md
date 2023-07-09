---
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
transition: slide-left
title: Next.jsのApp RouterでReact Server Componentsに触れてみる
---

<style>
.slidev-layout.cover h1 {
    font-size: 2.5rem!important;
}

.slidev-layout h2 {
    margin-bottom: 20px;
}
h1 {
  font-weight: bold;
    
}
</style>



# Next.jsのApp RouterでReact Server Componentsに触れてみる

---

## React Server Componentsとは

React Server Components（RSC)は、Reactのサーバーサイドレンダリングを高速化するために開発された技術。
React.jsのバージョンはv18からサポートされる予定の機能で、現在は試験的な機能として提供されている。

コンポーネントの一部をサーバー側でレンダリングして結果をブラウザに返し、それをクライアント側で表示することで、アプリケーションのパフォーマンスを向上させることができる。




<div>

<p class="mb-5">■コンポーネントツリーのイメージ図</p>
<a href="/pages/images/react-server-components.png" target="_blank">
<img
    src="/pages/images/react-server-components.png" alt=""
    class="w-1/3 mx-auto"
/>
</a>
</div>

<!--
あああ
-->

---

## サーバサイドレンダリングとの違いは？

### SSR
- SSRはサーバー側で生のHTMLのDOMを生成する
- その後、クライアントで仮想DOMを生成し、差分を検知した場合はクライアント側で再レンダリングする

### RSC
- RSCはサーバーのみで実行される
- RSCはサーバー側でReactNodeを生成する
- その後、クライアント側でレンダリングを行う
- **クライアント側で再レンダリングは行われない**


---

## RSCのメリット

- パフォーマンスの向上
  - ReactNodeの生成処理をサーバー側で行うことで、クライアント側でのレンダリング時間を短縮できる
- バンドルサイズの削減
  - サーバーコンポーネントはサーバーのみで実行されて結果を返すため、クライアントで実行されるJSファイルにはバンドルされない。
    - ブラウザにはクライアントコンポーネントのチャンクファイルのみがブラウザに返される
- データフェッチにかかる時間の減少
  - サーバーコンポーネントはサーバー側で実行されるため、データフェッチにかかる時間を短縮できる
  - APIを経由せずにサーバーコンポーネント側で直接DBにアクセスすることも可能
  - ファイルに直接アクセスすることも可能

---

## イメージ図

<div class="mt-5">

<a href="/pages/images/rsc.png" target="_blank">
<img
    src="/pages/images/rsc.png" alt=""
    class="w-1/2 mx-auto"
/>
</a>
</div>

青で囲まれた部分がサーバーサイドで仮想DOMを生成してブラウザに返す 

赤枠部分はクライアントサイドで仮想DOMを生成する。

青枠部分のチャンクファイルはブラウザには返さずに赤枠部分のチャンクファイルのみがブラウザに返される。

---

## RSCのコンポーネントの分類

### Server Components
- サーバー側でのみ実行されるコンポーネント
- サーバー側でのみ実行されるため、クライアント側でのレンダリングは行われない

Server Componentsには以下のような制約がある

- stateを持つことができないので、useStateやuseReducerは使用できない
- rerenderが走らないので、useEffectなどのライフサイクルフックは使用できない
- インタラクションある実装（onClickなど）は使用できない
- サーバー上で実行されるため、useContextやuseRefなどのクライアント側でのみ実行されるフックは使用できない
- サーバー上で実行されるため、windowやdocumentなどのブラウザのグローバルオブジェクトにアクセスすることはできない
- サーバー上で実行されるため、ブラウザでのみ利用可能なAPI（DOMやWeb API）は利用できない


---

### Client Components
- クライアント側でのみ実行されるコンポーネント
- 通常のReactコンポーネントと同じ
- インタラクションが必要な場合はクライアントコンポーネントを使用する

Client Componentsには以下のような制約がある

- サーバーコンポーネントをマウントすることはできない
  - ただし、childrenとしてサーバーコンポーネントを渡すことは可能
  - Vueでいうところの`<slot>`のようなもの


---

### Shared Components
- サーバー側とクライアント側の両方で実行されるコンポーネント
  - Server Componentsで呼ばれた場合はサーバー側で実行され、Client Componentsで呼ばれた場合はクライアント側で実行される

Shared ComponentsにはServer ComponentsとClient Componentsの両方の制約を受ける

---

## SSRとの共存

RSCはSSRとの共存が可能。

ページランディング時はSSRでページを表示し、その後にRSCを使用してページを更新することで、SSRとRSCを組み合わせて使用することができる。

どちらもサーバーサイドで実行されるため、ページランディング時のパフォーマンスが向上しつつ、バンドルサイズの削減も可能となる。

SEO観点でも有利となる。

---

## コンポーネントの使い分け

1. まずはパフォーマンス面で有利なサーバーコンポーネントを使うことを考える。
2. その後に、イベントハンドラーを使用したり、状態を管理する必要が出てきた場合にはクライアントコンポーネントを使用する。
3. 共通コンポーネントはShared Componentsを使用する。

といったアプローチでコンポーネントを作る方が良さそう。




---
layout: center
class: text-center
---

# Next.jsで実際に使ってみる

---

## Next.jsでApp Routerを使う

Next.js v13.4からApp Routerが正式版としてリリースされた。

`/app`ディレクトリにページコンポーネントを作成することでApp Routerを使用できる。

App Router配下ではコンポーネントはデフォルトでサーバーコンポーネントが適用される。

<div class="text-xs">
※RSC自体はまだReactの実験的な機能とした位置なので、ライブラリ単体で使用するのは非推奨となっているが、Next.jsなどのフレームワーク側で仕様変更の差分を吸収することで安定して使用できるようになっている。
</div>

<div class="mt-20">
簡単に作ったデモはこちら

- デプロイURL：[https://next-rsc-pokemon.vercel.app/](https://next-rsc-pokemon.vercel.app/)
- ソースコード：[https://github.com/chieeeeno/next-rsc-pokemon](https://github.com/chieeeeno/next-rsc-pokemon)

</div>




---
layout: center
class: text-center
---

# まとめ

---

## まとめ

- React Server Components（RSC）は恐らく今後React界隈では主流になっていくと思われる。
  - 現在は試験的な機能だが、Next.jsが正式に機能をサポートしたことで、今後は安定して使用できるようになると思われる。
- RSCを採用すると今までのコンポーネント設計の思想から大きく変わることになる。
  - サーバー側で実行されるか、クライアント側で実行されるか、両方からじっこうされるか、といったことを意識してコンポーネントを設計する必要がある。
    - それぞれのコンポーネントの制約を理解して、適切に使い分ける必要がある。
  - 今までのコンポーネント設計の思想を捨てて、RSCの思想に沿ったコンポーネント設計を行う必要がある。
