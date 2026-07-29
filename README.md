# skinny dip proto
---
## Project Structure

```
skinny_dip_proto/
│
├── docs/              ← drop your PDFs here
│
└── js/                ← JavaScript version (Express + ChromaDB)
    ├── README.md
    ├── package.json
    ├── rag.js         ← CLI
    └── rag_web.js     ← web UI  →  http://localhost:6601
```

---

## Stack

|  | JavaScript |
|---|---|
| Language |  Node.js 18+ |
| Web server |  Express |
| Embeddings |  Ollama (`nomic-embed-text`) |
| LLM |  Ollama (`qwen2.5:7b`) |
| Vector store |  ChromaDB |
| PDF parsing |  pdf-parse |
| Web port |  6601 |

---

## Quick Start
### 0. Clone the repo
 
`git clone https://github.com/computationalmama/skinny_dip_proto.git`

### 1. Install Ollama and pull models

Download from [ollama.com/download](https://ollama.com/download), then:

```bash
ollama pull nomic-embed-text
ollama pull qwen2.5:7b
```

### 2. Add your PDFs

Copy PDF files into the `docs/` folder.

### 3. Install README link

- **JavaScript** → see [`js/README.md`](js/README.md)

---

## Notes
You can check out more info about the embedding viz in the doc: [VISUALIZE](js/VISUALIZE.md)

---

## Design Updates

29/07/26
- Updates to [`visualize-d3.html`](visualize-d3.html): current version shows LLM-generated pools based on uploaded docs, and lets you annotate and connect chunks in the Cards view.
- Style/visual changes:
    - Colors updated
    - Card styles updated: Source and Annotation
- Interactions:
    - Click on Pool to zoom into Card view. Zoom in not implemented yet, too erratic
- Added annotation flow in Cards view:
    - Card interactions: annotate, create source
    - Double clicking in space adds annotation
    - Both card types should be draggable
- Other features:
    - Double clicking title in Card View can update title, saves it in Pool View as well
- Test/beta features:
    - Are.na flow to add **text** blocks. Does not save/update at the moment, only for testing in Cards view
- Next steps:
    - Making your own pools from blank
    - Adding, deleting cards
    - Image and links as sources
    - See [`/plans'](/plans) for possible paths for embedding other Are.na media and also for Export flow