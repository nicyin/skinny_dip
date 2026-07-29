# skinny dip proto
---
## Project Structure

```
skinny_dip/
│
├── docs/                    ← drop your PDFs here
│
├── js/                      ← Express + ChromaDB app
│   ├── README.md
│   ├── VISUALIZE.md         ← explainer for the visualizer pages
│   ├── package.json
│   ├── rag.js               ← CLI (build / ask / stats)
│   ├── rag_web.js           ← web server  →  http://localhost:6601
│   ├── visualize.html       ← pools viz (p5.js + canvas)
│   └── visualize-d3.html    ← pools viz (D3.js + SVG)
│
├── plans/                   ← feature planning docs
│   ├── ARENA_EMBEDDING.md
│   └── EXPORT.md
│
├── rag_database/            ← Chroma's on-disk store (gitignored)
├── venv/                    ← Python venv for Chroma server (gitignored)
└── AGENTS.md                ← instructions for AI coding agents
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
 
`git clone https://github.com/nicyin/skinny_dip.git`

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
- Updates to [`visualize-d3.html`](visualize-d3.html):
    - This version uses LLM-generated pools based on uploaded docs (Ambika's work), and lets you annotate and connect chunks in the Cards view.
    - I used [Small](https://drive.google.com/file/d/1N7HonT_Zf0YAtnLImIT-6cnFydsW-x7G/view) + [Esoteric](https://drive.google.com/file/d/1yYgO60wMiA2n8X0lKjZLhWdiGcTODtuc/view) + [Ancestral](https://drive.google.com/file/d/1K00R6yrfa-KBJXho8u8icm6mu8UAzxI3/view) AI zines as my test material but they were too big to include on github. Save them and add them to the /docs folder.
- Style/visual:
    - Added some AIxD-inspired color!
    - Card styles updated: Source and Annotation
- Interactions:
    - Click on Pool to zoom into Card view (zooming in not implemented yet, too erratic)
    - Double clicking title in Card View to update title, saves it in Pool View as well
    - Annotation flow in Cards view:
        - Card interactions: annotate, create source (TBD)
        - Double click in blank space to add annotation
        - Both card types should be draggable
- Test/beta features:
    - Are.na flow to add **text** blocks. Does not save/update at the moment, only for testing in Cards view
- Next steps:
    - Making your own pools from blank
    - Adding, deleting cards
    - Image and links as sources
    - See [/plans](/plans) for possible paths for embedding other Are.na media and also for Export flow