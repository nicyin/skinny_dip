# Export Feature Plan

How to turn the card diagram (sources, annotations, connections) into formats
useful for writing, synthesis, and saving state.

---

## What data exists at export time

When in card view the following is available in memory:

| Data | Description |
|------|-------------|
| `activePool.label` | The pool's (renameable) name |
| `activePool.members` | Source card chunks — `{ id, text, source }` |
| `annotations` | User-written notes — `{ text, attachType, attachCardId, attachConnId }` |
| `connections` | Edges between cards — `{ sourceId, targetId, annotId }` |
| `cardPositions` | Spatial x/y of every card (needed for JSON state-save only) |

The **relational graph** (who is connected to whom, what the annotation says
about that connection) is the most valuable thing to preserve for an LLM.
Raw text alone is just raw text; connections encode the user's thinking.

---

## Formats

### 1. `.md` — writing & synthesis (highest priority)

Best for passing to Claude or another LLM to write essays, proposals, summaries.
Structure:

```markdown
# [Pool Name]

## [Source Title] (from filename.pdf)
[text — full or truncated]

> My note: [annotation attached to this card, if any]

---

## Connection: [Card A title] ↔ [Card B title]
> [annotation riding on the connection line, if any]

---

## Loose notes
> [free-floating annotations not attached to any card]
```

**Why it works for LLMs**: source content and user commentary appear side by
side, so the model sees both *what was said* and *what the user thought about it*.

---

### 2. `.json` — state save / re-import

Faithful graph dump. Not great for direct LLM prompting (noisy), but useful
for "I want to come back to this later" and future re-import functionality.

```json
{
  "pool": "Pool Name",
  "exportedAt": "2026-07-29T…",
  "cards": [
    { "id": "…", "text": "…", "source": "filename.pdf" }
  ],
  "annotations": [
    { "id": "…", "text": "…", "attachType": "card", "attachCardId": "…" }
  ],
  "connections": [
    { "sourceId": "…", "targetId": "…", "annotId": "…" }
  ],
  "positions": {
    "cardId": { "x": 120, "y": -340 }
  }
}
```

---

### 3. `.txt` — bare prompt fodder

Same content as the Markdown export but stripped of all `#`, `>`, `---`
formatting symbols. Good for pasting directly into a system prompt where
Markdown might be misread.

---

## Optional: "Intent" selector before export

Before downloading, prompt the user to pick a use case. The first three prepend
a ready-to-paste instruction so the export is immediately usable without the
user having to craft their own prompt:

| Intent option | Prepended instruction |
|---------------|----------------------|
| *Synthesize into an essay* | "I've been researching [pool name] and organized my notes into connected cards. Help me write a 500-word essay from this material." |
| *Find patterns across cards* | "…Identify recurring themes and surprising connections across these cards." |
| *Summarize for a proposal* | "…Summarize the key ideas and their relationships into a concise proposal brief." |
| *Just save my notes* | (no instruction prepended) |

The intent selector only affects `.md` and `.txt` exports, not `.json`.

---

## Design decisions to settle at implementation time

1. **Scope**: Current pool only, or all pools? Default to current pool; "all
   pools" export would produce one section per pool in `.md`.

2. **Source text length**: Full chunk text can be very long. Consider a
   `truncate(text, 400)` option for the `.md` export (better for LLM context
   windows) vs. full text for the `.json` state-save.

3. **Are.na blocks**: Treat identically to PDF chunks; use `source` field
   (`Are.na / channel-name`) for attribution.

4. **Free annotations**: Collect any annotation with `attachType === 'free'`
   or whose `attachCardId` is not found and put them in a "Loose notes" section
   at the bottom of the `.md`.

5. **File naming**: `[pool-name]-[date].md` e.g. `design-thinking-2026-07-29.md`.

---

## Implementation sketch

```js
// Triggered by exportBtn click → show format/intent picker → then:
function exportPool(format, intent) {
  const content = buildExportContent(format, intent);
  const slug    = activePool.label.toLowerCase().replace(/\s+/g, '-');
  const date    = new Date().toISOString().slice(0, 10);
  const filename = `${slug}-${date}.${format}`;
  downloadFile(filename, content);
}

function downloadFile(name, text) {
  const a = document.createElement('a');
  a.href = URL.createObjectURL(new Blob([text], { type: 'text/plain' }));
  a.download = name;
  a.click();
  URL.revokeObjectURL(a.href);
}
```

No server round-trip needed — everything is in memory in the browser.

---

## Related files

- `js/ARENA_EMBEDDING.md` — plan for integrating Are.na blocks into the
  embedding pipeline (relevant: Are.na blocks should be included in exports).
