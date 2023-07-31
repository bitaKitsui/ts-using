---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
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
title: Welcome to Slidev
---

# TypeScript 5.2 using

---

# 自己紹介

## 橘井 貴輝（PD9U）

- https://open.spotify.com/intl-ja/album/6aHKT82AHUSJgkqKybC2b3?si=TFC8w1WbTDmQ4xb4HN_hSQ
- bitaラジオに曲を提供してます🎸
- バンドメンバー募集中（轟音トリプルギター + 儚い歌モノ）
- スケボーの動画にハマってます
  - Jahmir Brown

---

# Agenda

- 簡単な using 紹介
- JavaScript のメモリー管理
- 結局 using とは？
- まとめ

---

# using とは

- TypeScript 5.2 で追加される宣言方法
- ECMAScript の Stage3 Proposal の内容を先行で実装したもの
- ES2015で追加された `let` や `const` のような変数宣言

```ts
using x = expr1;
using y = expr2, z = expr4;
```

---

# tc39 の内容

- ECMAScript Explicit Resource Management (明示的リソース管理)
  - 様々なリソース（メモリ、I/Oなど）の寿命と管理に関する、ソフトウェア開発における一般的なパターンに対処することを意図している
  - リソースの割り当てと、重要なリソースを明示的に解放する機能が含まれる

https://github.com/tc39/proposal-explicit-resource-management

---

# JavaScript のメモリー管理

- プリミティブなりオブジェクトなり関数なりを宣言するとメモリを使用する
- JavaScript のメモリー管理は自動で行われる
- Cのような低水準言語には `malloc()` や `free()` のようなメモリー管理する仕組みが存在している

---

# JavaScript のメモリー管理

- JavaScript では、オブジェクトを作成するときにメモリーを自動的に確保し、使用しなくなったら解放する仕組みがある
  - ガベージコレクション
    - コンピュータプログラムが動的に確保したメモリ領域のうち、不要になった領域を自動的に解放する機能
  - 自動で行われる ≠ メモリー管理について心配する必要がない

---

# メモリーライフサイクル

どのプログラミング言語でもライフサイクルは変わらない

1. 必要なメモリーを割り当てる
2. 割り当てられたメモリーを使用する（読み書き）
3. 必要なくなったら、割り当てられたメモリーを解放する

2は全ての言語で明示的に行う<br>
1,3は低水準言語では明示的に、 JavaScript では暗黙的に行う

---

# 必要なメモリーを割り当てる

値を宣言すると同時にメモリーが割り当てられる

```js
const n = 123; // 数値を格納するメモリーが割り当てられる
const s = "azerty"; // 文字列を格納するメモリーが割り当てられる

const o = {
  a: 1,
  b: null,
}; // オブジェクトとそれに含まれる値を格納するためのメモリーが割り当てられる

function f(a) {
  return a + 2;
} // 関数を格納するメモリーが割り当てられる
```

---

# 割り当てられたメモリーを使用する（読み書き）

割り当てられたメモリーを使用する = 値を使用する

- 変数やオブジェクトの値を読み書きする
- 引数を関数に渡す

---

# 必要なくなったら、割り当てられたメモリーを解放する

このタイミングの判断がとても難しい！

- ガベージコレクションのアルゴリズムでは、特定のメモリーがまだ必要であるかどうかを判断することはできない
- `using` で明示的に記述することでこの問題を回避することができる

---

# 参照カウントのガベージコレクション

- とあるガベージコレクションアルゴリズムの例
- 「あるオブジェクトが必要なくなった」 => 「あるオブジェクトがその他のオブジェクトから参照されていない」ことと定義している
- https://developer.mozilla.org/ja/docs/Web/JavaScript/Memory_management#%E5%8F%82%E7%85%A7%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88%E3%81%AE%E3%82%AC%E3%83%99%E3%83%BC%E3%82%B8%E3%82%B3%E3%83%AC%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3

---

# JavaScript におけるメモリリークの例

- メモリリークとは
  - アプリケーションが過去に使用し、もはや必要とされていない空き領域のメモリプールや、OSに返されていないメモリーの断片のこと

```js
const elements = {
    button: document.getElementById('button'),
    image: document.getElementById('image')
};
function doStuff() {
    elements.image.src = 'http://example.com/image_name.png';
}
function removeImage() {
    document.body.removeChild(document.getElementById('image'));

    // この時点ではグローバルな要素オブジェクトである #button への参照がまだ残っている
    // = button 要素もまだGCによって回収されずメモリに存在することになる
}
```

---

# using

- `Symbol.dispose` 関数で指定したものがスコープから出たときに、そのものを処分することができる変数宣言
- データベース接続などのリソース管理に役立つと考えられている🤔

```ts
{
  const getResource = () => {
    return {
      [Symbol.dispose]: () => {
        console.log('廃棄します')
      }
    }
  }
  using resource = getResource();
} // '廃棄します'
```

---

# `Symbol.dispose` とは

- JavaScript の新しいグローバルシンボルのこと
  - `Symbol.dispose` に割り当てられた関数を持つものは全て「リソース」とみなされる
  - 「リソース」 = 「特定のライフタイムを持つオブジェクト」
  - 割り当てられたものは `using` キーワードを使用できる
  - `Symbol.dispose` が使用されていない場合に `using` キーワードを使うと TypeError がスローされる

---

# `await using`

- `Symbol.asyncDispose` と `await` を使って非同期に廃棄するリソースを処理することもできる

```ts
const getResource = () => ({
  [Symbol.asyncDispose]: async () => {
    await someAsyncFunc();
  },
});
{
  await using resource = getResource();
}
```

---

# using の使用例

- ファイルハンドル
  - node のファイルハンドラを使ったファイルシステムへのアクセスがより簡単になる

Before

```ts
import { open } from "node:fs/promises";
let filehandle;
try {
  filehandle = await open("thefile.txt", "r");
} finally {
  await filehandle?.close();
}
```

---

# using の使用例

- ファイルハンドル
  - node のファイルハンドラを使ったファイルシステムへのアクセスがより簡単になる

After

```ts
import { open } from "node:fs/promises";
const getFileHandle = async (path: string) => {
  const filehandle = await open(path, "r");
  return {
    filehandle,
    [Symbol.asyncDispose]: async () => {
      await filehandle.close();
    },
  };
};
{
  await using file = await getFileHandle("thefile.txt");
  // Do stuff with file.filehandle
} // Automatically disposed!
```

---

# using の使用例

- データベース接続

Before

```ts
const connection = await getDb();
try {
  // Do stuff with connection
} finally {
  await connection.close();
}
```

---

# using の使用例

- データベース接続

After

```ts
const getConnection = async () => {
  const connection = await getDb();
  return {
    connection,
    [Symbol.asyncDispose]: async () => {
      await connection.close();
    },
  };
};
{
  await using db = await getConnection();
  // Do stuff with db.connection
} // Automatically closed!
```

---

# まとめ

- メモリーの解放は一筋縄ではいかない
- 複数のメモリーの同時管理したい場合とかが大変
- その辺りを `using` で解決！
- https://devblogs.microsoft.com/typescript/announcing-typescript-5-2-beta/