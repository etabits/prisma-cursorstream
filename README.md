# prisma-cursorstream

Prisma Stream Client Extension (Cursor-based Implementation)

## Features

- Clean API (`for await`)
- Fully typed <img src="https://www.typescriptlang.org/favicon-32x32.png" height="16" width="16" alt="" />
- Minimal implementation. The [source](https://github.com/etabits/prisma-cursorstream/blob/main/src/index.ts) itself is well below 100 LOCs.

## Installation & Activation

```sh
npm install prisma-cursorstream
```

Then, extend your prisma client like this:

```js
import { PrismaClient } from "@prisma/client";
import cursorStream from "prisma-cursorstream";

const db = new PrismaClient().$extends(cursorStream);
```

<details>
<summary>Using CommonJS `require`?</summary>

### CommonJS `require`

```js
const cursorStream = require("prisma-cursorstream").default;

const db = new PrismaClient().$extends(cursorStream);
```

Refer to [#1](https://github.com/etabits/prisma-cursorstream/issues/1).

</details>

## Usage

```js
const stream = db.post.cursorStream({
  // Your usual findMany args
  select: {
    id: true,
    title: true,
  },
  where: {
    published: true,
  },
});
for await (const post of stream) {
  console.log(post); // {id: 1, title: 'Hello!'}
}
```

## Advanced API

The following options (passed as a second argument) provide additional controls over the streaming process.

- **Batch size**: `batchSize` specifies how many rows are fetched each batch. Defaults to `100`
- **Pre-fill size**: Since the streaming happens asynchronously, it may be useful to fetch more rows well before current batch is completely consumed. `prefill` specifies the internal `highWaterMark` of the `Readable` stream. Defaults to double the batch size (usually `200`).
- **Batch transformation**: Sometimes you may need to pre-process the resulting rows in batches before they are consumed. This is where the `batchTransformer` comes in. If provided, it receives an array of the last fetched batch of rows, can operate on them asynchronously, and then return an array of transformed rows to be consumed instead. Types are properly handled.

```js
const stream = db.post.cursorStream(
  {
    where: {
      published: true,
    },
  },
  {
    batchSize: 42, // batches of 42 posts at a time
    prefill: 69, // prefetch up to 69+ posts
    async batchTransformer(posts) {
      const translations = await translate(posts.map((p) => p.title));
      return posts.map((p, i) => ({
        ...p,
        translatedTitle: translations[i],
      }));
    },
  }
);

for await (const translatedPost of stream) {
  console.log(translatedPost); // {..., translatedTitle: "مرحبا!"}
}
```
