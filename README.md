# doc2rag-forge

Local-first document-to-RAG forge for precise extraction, OCR, indexing, Obsidian, and graph memory workflows.

`doc2rag-forge` is a planned document intelligence pipeline for turning human-readable material into a structured, searchable, and reusable knowledge base for local AI agents and models.

The goal is not just to upload files into a vector database. The goal is to build a controlled document operating pipeline that preserves source evidence, page references, extraction quality, structured metadata, Markdown notes, vector indexes, and graph memory exports.

## Plans

- [English plan]([docs/EN/doc2rag-forge_EN.md](https://github.com/chrsdme/doc2rag-forge/blob/main/docs/en/doc2rag-forge_EN.md)
- [English PDF]([docs/EN/doc2rag-forge_EN.pdf)](https://github.com/chrsdme/doc2rag-forge/blob/main/docs/en/doc2rag-forge_EN.pdf)
- [Romanian plan]([docs/RO/doc2rag-forge_RO.md)](https://github.com/chrsdme/doc2rag-forge/blob/main/docs/ro/doc2rag-forge_RO.md)
- [Romanian PDF]([docs/RO/doc2rag-forge_RO.pdf)](https://github.com/chrsdme/doc2rag-forge/blob/main/docs/ro/doc2rag-forge_RO.pdf)

## Purpose

The project is designed to ingest and process:

* PDF books
* scanned documents
* university study material
* magazines
* RSS feeds
* web articles
* images
* technical notes
* other human-readable material

The processed output can then be used for:

* RAG retrieval
* local agent memory
* Obsidian knowledge bases
* Graphify graph memory
* Markdown archives
* structured JSON records
* search and citation workflows

## Core idea

The pipeline follows this model:

```text
Raw source
  -> source registry
  -> document router
  -> extraction
  -> OCR / vision fallback
  -> validation
  -> Markdown + JSON artefacts
  -> chunking
  -> embeddings
  -> hybrid search index
  -> Obsidian export
  -> Graphify export
  -> agent memory promotion
```

## Design principles

### 1. Provenance first

Every extracted answer should be traceable back to:

* source file
* page number
* chunk ID
* extraction method
* processing timestamp
* confidence score

### 2. Precision over speed

The system should prefer accurate extraction and verifiable citations over fast but unreliable ingestion.

### 3. Hybrid retrieval

Vector search alone is not enough. The project should combine:

* keyword search
* full-text search
* vector search
* metadata filters
* reranking
* page-level citations

### 4. Local-first

The system is designed to run locally first, using local models and local storage where possible.

### 5. Automatic but controlled

The user should only need to provide the data. The system should decide how to process it, but each stage should be logged, validated, and reversible.

## Planned architecture

### Source registry

Stores metadata about every input source:

* filename
* source type
* hash
* language
* document category
* processing state
* update status

### Document router

Classifies files before processing.

Example categories:

* digital PDF
* scanned PDF
* image-heavy PDF
* academic paper
* book
* magazine
* RSS article
* web article
* image
* Office document

### Extraction layer

Extracts text, tables, structure, headings, images, and layout data.

Candidate tools:

* Python
* PyMuPDF
* pdfplumber
* Docling
* Marker
* MinerU

### OCR and vision fallback

Used only when normal extraction is weak, for example:

* scanned pages
* broken layout
* missing text
* corrupted Romanian or English text
* image-only pages
* diagrams
* screenshots
* tables
* formulas

Candidate tools and models:

* PaddleOCR-VL
* olmOCR
* Qwen VL models
* other local vision-language models

### Validation layer

Each document and page should receive quality metadata.

Example:

```json
{
  "page": 12,
  "text_confidence": 0.94,
  "layout_confidence": 0.88,
  "table_confidence": 0.72,
  "ocr_used": true,
  "needs_review": false
}
```

### Chunking layer

Chunks should preserve structure and page provenance.

Chunk types may include:

* definition
* explanation
* table
* figure
* quote
* procedure
* command
* code block
* summary

### Indexing layer

The project should support hybrid retrieval using:

* SQLite metadata
* SQLite FTS5 or another full-text search engine
* Qdrant or another vector database
* embeddings
* reranking

### Obsidian export

The system should generate Markdown notes suitable for Obsidian.

Generated notes may include:

* document summaries
* topic notes
* concept notes
* source notes
* study notes
* reference notes

### Graphify export

The system should generate graph-ready nodes and relationships.

Example node types:

* document
* topic
* concept
* entity
* module
* source
* author
* method
* tool
* claim

Example relationships:

* mentions
* explains
* contradicts
* depends_on
* belongs_to
* derived_from
* supports
* references

### Memory promotion

Not everything should become long-term memory.

Promotion should be controlled by levels:

```text
L0 raw source
L1 extracted text
L2 indexed chunks
L3 curated Obsidian notes
L4 graph memory
L5 stable agent memory
```

## Hardware target

The initial target hardware is a single local AI-PC.

Minimal recommended hardware:

* 8-core class CPU
* 32 GB RAM
* NVIDIA GPU with 12 GB VRAM or better
* SSD storage for indexes and artefacts
* larger secondary storage for raw documents

Current target hardware:

* Intel i7-10750 class CPU
* 32 GB DDR4 RAM
* NVIDIA RTX 5060 Ti 16 GB
* local storage under `/srv`

Laptops are treated as endpoints only for viewing, uploading, and controlling the system.

## Planned storage layout

```text
/srv/ai-workdesk/kb/
  raw/
  registry/
  artefacts/
  indexes/
  obsidian/
  graphify/
  reports/
  eval/
```

## Planned MVP stages

### MVP 1: Read-only ingestion

* watch folder
* source registry
* file hashing
* document router
* Markdown extraction
* JSON extraction
* extraction report

### MVP 2: Searchable RAG base

* chunking
* embeddings
* vector database
* full-text search
* hybrid retrieval
* citation-aware answer output

### MVP 3: Obsidian and Graphify export

* Markdown note generation
* topic extraction
* graph node generation
* graph edge generation
* promotion rules

### MVP 4: Update loop

* RSS monitoring
* file watcher
* changed-source detection
* changed-page detection
* partial re-indexing
* update reports

### MVP 5: Evaluation harness

* golden document set
* extraction scoring
* OCR comparison
* retrieval scoring
* citation checks
* regression reports

## Example retrieval goal

A user asks:

```text
What does my COM4010 material say about authentication protocols?
```

The system should answer with:

* concise explanation
* source document names
* page numbers
* chunk IDs
* confidence score
* related Obsidian notes
* related graph nodes

## Status

Planning stage.

The current focus is defining the architecture, repository structure, processing strategy, and local-first implementation path.

## License

MIT License.
