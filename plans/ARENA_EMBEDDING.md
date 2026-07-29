# Are.na → Embedding Pipeline

How to graduate Are.na blocks from "transient visual cards" into first-class
RAG chunks that participate in clustering, search, and chat.

---

## Current state (as of this writing)

When you pull an Are.na channel through the ⇄ CONNECT panel:

- Text blocks are fetched directly from `api.are.na/v2/channels/{slug}/contents`.
- Each block is turned into a lightweight chunk object `{ id, text, source, embedding: null }`.
- It is pushed onto `activePool.members` so it survives re-entry to that pool's
  card view within the current browser session.
- **It has no embedding.** It is never written to ChromaDB. After a page reload
  (or a `chroma run` restart) the block is gone.

---

## What needs to change to make it persistent

### 1. Compute an embedding for each block

The project already uses `nomic-embed-text` running in Ollama. The same call
used in `js/rag.js` / `js/rag_web.js` can be reused:

```js
// existing pattern in rag_web.js
async function embed(text) {
  const res = await fetch('http://localhost:11434/api/embeddings', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ model: 'nomic-embed-text', prompt: text }),
  });
  const { embedding } = await res.json();
  return embedding; // 768-float array
}
```

### 2. Upsert into ChromaDB

The JS Chroma client (`chromadb` npm package, version used by this project) talks
to the Chroma HTTP server on `:8000`. Adding documents looks like:

```js
const { ChromaClient } = require('chromadb');
const client = new ChromaClient({ path: 'http://localhost:8000' });
const collection = await client.getOrCreateCollection({ name: 'docs' });

await collection.upsert({
  ids:        ['arena-12345678'],
  documents:  ['The full text of the block…'],
  embeddings: [[0.023, -0.118, … /* 768 floats */]],
  metadatas:  [{ source: 'Are.na / my-channel', arenaId: 12345678, arenaChannel: 'my-channel' }],
});
```

`upsert` is idempotent — re-pulling the same channel will not create duplicate
chunks.

### 3. Add a server endpoint

`rag_web.js` should expose a new route that the browser can POST to:

```
POST /arena-import
Body: { slug: "my-channel-name" }
```

The server handler would:
1. Fetch the Are.na channel (same API call, but now from Node so no CORS issue).
2. Filter to text blocks.
3. Compute embeddings via Ollama for each block.
4. Upsert into ChromaDB.
5. Return `{ added: N, skipped: M }`.

The browser-side `fetchArena()` function in `visualize-d3.html` would call
`/arena-import` instead of `api.are.na` directly, and then still inject the
returned chunks as cards.

### 4. Make `/embeddings` return the new chunks

`rag_web.js`'s `/embeddings` endpoint already queries the whole collection:

```js
const results = await collection.get({ include: ['documents','embeddings','metadatas'] });
```

Are.na chunks upserted with `source: 'Are.na / …'` in their metadata will
automatically appear here after the next page load. No changes needed to the
visualizer's clustering or RAG logic.

### 5. Participation in k-means clustering

Because Are.na blocks now have a real 768-d embedding, `kmeans()` will assign
them to a pool just like any PDF chunk. They may land in an existing pool or
cause a new one to emerge (depending on `k`). The pool view and chat will work
with them seamlessly.

---

## Migration path (step by step)

| Step | File | What to do |
|------|------|------------|
| 1 | `js/rag_web.js` | Add `POST /arena-import` route |
| 2 | `js/rag_web.js` | Reuse `embed()` and the Chroma `upsert` call |
| 3 | `js/visualize-d3.html` | Change `fetchArena()` to call `/arena-import` instead of `api.are.na` directly; still inject returned chunks as cards |
| 4 | — | Restart the Chroma server and reload the page; Are.na blocks now appear in pools |

---

## Notes & caveats

- **Pagination**: The current browser-side fetch uses `?per=100`. Channels with
  > 100 blocks will be truncated. The server-side handler should loop through
  pages using `?page=N&per=100` until `contents` is empty.
- **Rate limits**: Are.na's public API is unauthenticated and lightly rate-limited.
  For large channels, add a small delay between embedding requests to avoid
  hammering Ollama.
- **Block updates**: Are.na blocks can be edited. Re-pulling the channel and
  calling `upsert` again will update the stored text and recompute the embedding.
- **Deletion**: ChromaDB's `collection.delete({ ids: ['arena-…'] })` can remove
  specific blocks if a channel is "disconnected".
- **Private channels**: Would require an Are.na personal access token stored
  server-side (never in the browser). Add it as `ARENA_TOKEN` in a `.env` file
  and pass it as `Authorization: Bearer <token>` in the server fetch.
