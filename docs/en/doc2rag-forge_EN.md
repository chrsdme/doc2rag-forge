---
title: "Document operating system for RAG"
subtitle: "Local-first dynamic knowledge base for documents, RAG, Obsidian, Graphify, and agent memory"
author: "Cristian"
date: "2026-07-08"
---

# Executive summary

This plan designs a local-first **Document operating system for RAG**. The purpose is to ingest books, magazines, PDFs, scanned documents, university material, RSS feeds, web pages, screenshots, and other human-readable sources, then turn them into a precise, searchable, source-linked knowledge base.

The target is not a simple vector database. A vector database is only one storage layer. The correct target is a controlled document pipeline that keeps raw evidence, extracted text, Markdown, JSON, page images, tables, figures, chunks, embeddings, full-text indexes, graph entities, Obsidian notes, and reviewed memory entries as separate layers.

The system should work like this:

```text
source material
  -> source registry
  -> document router
  -> extraction pipeline
  -> selective OCR / vision validation
  -> quality scoring
  -> structured Markdown and JSON
  -> chunking with page provenance
  -> hybrid search index
  -> RAG answer service with citations
  -> reviewed Obsidian / Graphify / memory promotion
  -> update loop for changing sources
```

The main rule is: **never lose provenance**. Every answer produced from the knowledge base should be traceable back to a source file, page, chunk, extraction method, timestamp, and confidence score.

# Corrected hardware assumptions

The active processing machine is one AI-PC with:

- one active GPU: **RTX 5060 Ti 16 GB**
- CPU: **Intel i7-10750 class**
- RAM: **32 GB DDR4**
- laptops: endpoints only for viewing, uploading, checking, and controlling jobs

Do not design for unused GPUs. Do not assume multi-GPU scheduling. The system must run with one GPU worker queue and one CPU/disk extraction queue.

## Minimal hardware spec

This is the minimum practical specification for the first usable version:

- 6-core / 12-thread CPU or better
- 32 GB RAM
- NVIDIA GPU with 12 GB VRAM or better
- 1 TB SSD for working artefacts
- separate large storage for raw documents and generated outputs
- Linux host with Docker, Python, and stable NVIDIA drivers

Your current AI-PC fits this class, mainly because of the 16 GB VRAM GPU. The 32 GB RAM is workable, but batching must be controlled.

## Recommended hardware spec

This is the recommended target if the system becomes large:

- 8 to 12 CPU cores or better
- 64 GB RAM
- NVIDIA GPU with 16 GB VRAM or better
- 2 TB NVMe for active processing
- 4 TB+ SSD/HDD for raw library, page images, and artefact archives
- daily backup target
- UPS if running long OCR/index jobs

The first upgrade I would consider is **RAM to 64 GB**, not another GPU. Large PDFs, page rendering, OCR workers, embedding batches, Qdrant, and dashboard services can pressure 32 GB before they fully saturate the GPU.

# Design principles

## 1. Router first, OCR second

Do not OCR every PDF. That is slow and can damage precision when the embedded text is already good.

Every file should first go through a document router that decides:

- is it a digital PDF?
- is it a scanned PDF?
- does it contain tables?
- does it contain formulas?
- is it multi-column?
- does it contain charts, figures, or screenshots?
- is it Romanian, English, or mixed?
- is it a university file, book, article, RSS item, or web capture?
- does it require a fallback tool?

Then the system chooses the cheapest reliable extraction path.

## 2. Store layers, not just chunks

The system should not only store vector chunks. It should preserve:

- raw file
- source metadata
- extracted plain text
- extracted Markdown
- page images
- OCR output
- layout JSON
- table JSON / CSV
- figure captions
- chunks
- embeddings
- full-text search rows
- graph nodes and edges
- Obsidian notes
- memory promotion decisions

Vectors are disposable. Raw sources and structured extraction artefacts are not.

## 3. Hybrid retrieval, not vector-only retrieval

Vector search is useful for semantic similarity, but it is weak with exact facts, module codes, legal wording, command names, page references, table labels, and uncommon Romanian or technical terms.

Use hybrid retrieval:

```text
query
  -> keyword / BM25 / SQLite FTS
  -> dense vector search
  -> optional sparse vector search
  -> metadata filters
  -> reranker
  -> answer with source citations
```

## 4. Memory promotion must be gated

Do not automatically push every extracted sentence into Obsidian, Graphify, or long-term agent memory.

Use promotion levels:

- L0 raw evidence
- L1 extracted document artefacts
- L2 indexed chunks
- L3 curated Obsidian notes
- L4 Graphify concepts and relationships
- L5 stable agent memory

Only L3 to L5 should be treated as long-term knowledge. Everything else is evidence and retrieval material.

## 5. Precision beats automation speed

The system should prefer slower reliable processing over fast broken extraction. Every extraction pass must produce a confidence score and a failure report.

# Target architecture

## Main services

Use a small number of clear services:

- **ingest-api**: receives files, URLs, RSS subscriptions, and manual jobs
- **registry-db**: SQLite database for source records, documents, pages, jobs, chunks, and quality scores
- **worker-extract**: CPU-heavy parser for digital documents
- **worker-ocr-vlm**: GPU-gated OCR and vision worker
- **worker-embed**: embedding worker
- **qdrant**: vector database
- **fts-index**: SQLite FTS5 full-text index
- **rag-api**: retrieval and answer service
- **exporter-obsidian**: Markdown note generator
- **exporter-graphify**: graph node and edge generator
- **review-ui**: low-confidence and promotion review queue
- **scheduler**: systemd timer jobs for RSS, changed files, and re-indexing

## Folder layout

```text
/srv/ai-workdesk/kb/
  raw/
    books/
    magazines/
    university/
    rss/
    web/
    images/
  registry/
    kb.sqlite
  artefacts/
    extracted_text/
    extracted_markdown/
    page_images/
    ocr_json/
    layout_json/
    tables/
    figures/
    chunks/
    reports/
  indexes/
    qdrant/
    sqlite_fts/
  exports/
    obsidian/
    graphify/
    memory_candidates/
  eval/
    golden_docs/
    extraction_scores/
    retrieval_scores/
```

## Job state machine

Every source should move through explicit states:

```text
REGISTERED
  -> ROUTED
  -> EXTRACTED
  -> VALIDATED
  -> CHUNKED
  -> EMBEDDED
  -> INDEXED
  -> REVIEWED
  -> PROMOTED
  -> AVAILABLE_FOR_RAG
```

Failed jobs should not disappear. They should move to:

```text
NEEDS_RETRY
NEEDS_SECONDARY_OCR
NEEDS_MANUAL_REVIEW
BLOCKED_UNSUPPORTED_FORMAT
```

# Source registry

The registry is the control plane. It prevents duplicate work and allows reliable updates.

Recommended tables:

## sources

```text
source_id
source_kind: file | rss | url | folder_watch | manual_note
original_location
canonical_location
sha256
size_bytes
created_at
registered_at
last_checked_at
licence_status
privacy_class
status
```

## documents

```text
doc_id
source_id
title
author
language
content_type
page_count
parser_selected
ocr_required
created_at
updated_at
```

## pages

```text
page_id
doc_id
page_number
page_hash_text
page_hash_image
text_confidence
layout_confidence
table_confidence
ocr_used
needs_review
```

## chunks

```text
chunk_id
doc_id
page_start
page_end
section_title
chunk_type
text
summary
keywords
entities
embedding_model
source_hash
created_at
```

## jobs

```text
job_id
source_id
doc_id
job_type
state
priority
started_at
finished_at
error_message
retry_count
```

# Document routing

The router should choose the extraction strategy before any expensive processing.

## Routing rules

| Condition | First action | Fallback |
|---|---|---|
| Digital PDF with good embedded text | PyMuPDF / pdfplumber / Docling | Marker or MinerU |
| Scanned PDF | OCR pipeline | VLM page validation |
| Technical PDF with formulas | MinerU / Marker | VLM validation for failed pages |
| Magazine or complex layout | Docling / Marker | OCR/VLM on low-confidence pages |
| Image or screenshot | OCR + VLM caption | manual review if low confidence |
| RSS article | feed parser + content extraction | browser snapshot if page is dynamic |
| Web page | trafilatura/readability/browser capture | screenshot + OCR if blocked |
| DOCX/PPTX/XLSX | Docling / Marker / MinerU | LibreOffice conversion as fallback |

## Router output

```json
{
  "doc_id": "...",
  "route": "digital_pdf_structured",
  "language": "en_ro_mixed",
  "parser_chain": ["docling", "marker_fallback"],
  "ocr_policy": "selective",
  "vlm_policy": "low_confidence_pages_only",
  "expected_risk": "tables_and_two_column_layout"
}
```

# Extraction pipeline

## Pass 1: normal extraction

For digital PDFs, start with normal extraction:

- embedded text
- headings
- page numbers
- tables
- figures
- captions
- links
- references
- reading order
- formulas where available

Candidate tools:

- PyMuPDF for fast text and page metadata
- pdfplumber for text and table inspection
- Docling for structured document conversion
- Marker for Markdown/JSON/chunk conversion
- MinerU for technical and scientific documents

## Pass 2: selective OCR and vision

Run OCR only where needed:

- page has no embedded text
- embedded text is corrupted
- Romanian diacritics or English text is broken
- table extraction score is low
- formula extraction is weak
- page has screenshots, diagrams, charts, or handwritten content
- reading order looks wrong

Candidate tools/models:

- PaddleOCR / PaddleOCR-VL for multilingual OCR and document parsing
- olmOCR for clean natural reading order from PDF/image documents
- Qwen2.5-VL or Qwen3-VL class model for selected page validation

Because the AI-PC has one 16 GB GPU, all OCR/VLM jobs should use a **GPU lock**. Only one heavy GPU job should run at a time.

## Pass 3: extraction validation

Each page gets a score:

```json
{
  "page_number": 12,
  "text_confidence": 0.94,
  "layout_confidence": 0.88,
  "table_confidence": 0.72,
  "ocr_used": true,
  "needs_review": false,
  "warnings": ["possible_table_loss"]
}
```

Validation checks:

- empty or near-empty page detection
- repeated header/footer removal
- wrong reading order detection
- broken words across line wraps
- table row/column count sanity
- formula preservation check
- image/figure presence check
- language mismatch check
- duplicate page detection

# OCR and VLM strategy on one GPU

The 16 GB GPU is enough for a strong local-first pipeline, but only if the jobs are queued.

Recommended rule:

```text
CPU extraction can run in parallel.
GPU OCR/VLM runs one job at a time.
Embedding batches run when OCR/VLM is idle.
The chat/retrieval model gets priority over background processing.
```

## Practical scheduling

Use three queues:

- **fast queue**: small digital PDFs, RSS, webpages
- **batch queue**: books, magazines, long university packs
- **heavy queue**: scanned PDFs, OCR, VLM validation, table repair

Example policy:

```text
08:00-22:00: light ingestion and retrieval priority
22:00-07:00: batch OCR, embeddings, index rebuilds
```

If the system is used during study or coding, pause heavy OCR.

# Model and tool strategy

## Local reasoning and agent control

Recommended candidates:

- Qwen3 14B quantized through Ollama for main local reasoning
- Qwen3 8B for faster classification and routing
- Qwen2.5-Coder 14B or similar for code repair and pipeline scripts

Use the main LLM for decisions, not raw extraction. The parser/OCR tools should do extraction. The LLM should decide routing, validate summaries, create metadata, and promote memory candidates.

## Embeddings

Primary recommendation:

- **BGE-M3** for multilingual retrieval, especially English/Romanian mixed material

Why:

- multilingual
- dense retrieval
- sparse retrieval
- multi-vector retrieval
- supports longer input granularity than many small embedding models

Future candidate:

- **Qwen3 Embedding 0.6B / 4B / 8B**, tested after the first pipeline works

Do not change embedding models casually. Store the embedding model name and version per chunk.

## Reranking

Add reranking after basic hybrid retrieval works.

Candidate rerankers:

- BGE reranker family
- Qwen3 reranker family

Reranking should be optional because it adds latency. Use it for final answers, not for every exploratory search.

## Vision/document models

Candidate tools/models:

- PaddleOCR / PaddleOCR-VL for OCR and multilingual document parsing
- olmOCR for PDF/image documents where reading order matters
- Qwen2.5-VL or Qwen3-VL class model for low-confidence visual review

Do not ask a VLM to process entire books page by page unless the normal parser failed. VLMs are expensive and slower.

# Chunking strategy

Chunking should preserve meaning and page evidence.

## Chunk types

Use different chunk types:

- definition
- explanation
- procedure
- table
- figure
- code
- quote
- formula
- summary
- reference
- question-answer
- assignment/coursework note

## Chunk size rules

| Source type | Chunking rule |
|---|---|
| University notes | heading-based, 400-900 words |
| Books | section-based, 700-1200 words |
| Magazines | article/section chunks |
| Tables | one table = one chunk plus summary |
| Code/commands | keep intact |
| Diagrams | OCR text + caption + linked page image |
| RSS/web | article sections with canonical URL |
| Definitions | separate atomic chunks |

## Parent-child structure

Use a tree:

```text
document
  chapter
    section
      chunk
        table
        figure
        quote
```

This allows the RAG system to retrieve a small chunk but expand to the parent section when more context is needed.

# Retrieval and answer service

## Retrieval pipeline

```text
user query
  -> query language detection
  -> query expansion / normalisation
  -> metadata filters
  -> SQLite FTS / BM25 search
  -> dense vector search
  -> optional sparse vector search
  -> result fusion
  -> reranking
  -> context pack
  -> answer with citations
```

## Metadata filters

Support filters such as:

- source type: university, book, RSS, web, image
- module code
- author
- year
- language
- confidence threshold
- date ingested
- reviewed only
- include or exclude OCR-only pages
- private/public classification

## Answer rule

The answer service should never say something came from the KB unless it can cite the source.

Every generated answer should include:

```text
Sources:
- document title
- page number or article URL
- chunk ID
- extraction confidence
```

If retrieval is weak, it should say:

```text
I found related material, but I cannot confirm the answer from the indexed sources.
```

# Obsidian and Graphify export

## Obsidian export

Obsidian should receive curated notes, not raw dumps.

Recommended note types:

- source notes
- concept notes
- module notes
- topic maps
- definition notes
- assignment support notes
- weekly university summaries
- reading summaries
- unresolved questions

Example frontmatter:

```yaml
title: OAuth 2.0 Authorization Code Flow
type: concept
source_count: 3
confidence: reviewed
created: 2026-07-08
updated: 2026-07-08
tags:
  - cybersecurity
  - authentication
  - university
```

Example note body:

```markdown
# OAuth 2.0 Authorization Code Flow

## Summary
Short human-readable summary.

## Key points
- Point 1
- Point 2

## Source evidence
- COM4010 Lecture 04, page 22, chunk abc123
- Security textbook, page 118, chunk def456

## Related
- [[OpenID Connect]]
- [[Access Token]]
- [[Refresh Token]]
```

## Graphify export

Graphify should receive entities and relationships:

```json
{
  "node_type": "concept",
  "name": "OAuth 2.0 Authorization Code Flow",
  "aliases": ["auth code flow"],
  "sources": [
    {"doc_id": "...", "page": 22, "chunk_id": "abc123"}
  ],
  "relations": [
    {"type": "used_in", "target": "Web authentication"},
    {"type": "contrasts_with", "target": "Implicit Flow"}
  ]
}
```

## Memory promotion

Only promote stable, reusable knowledge into long-term agent memory:

- project rules
- recurring preferences
- stable definitions
- course/module summaries
- validated workflows
- system architecture decisions

Do not promote:

- unverified OCR
- temporary RSS headlines
- duplicate chunks
- noisy summaries
- low-confidence extraction
- full copyrighted book content

# Dynamic update loop

## Source update detection

| Source | Detection method |
|---|---|
| RSS | ETag, Last-Modified, URL hash |
| Web page | canonical URL + content hash |
| PDF file | file hash + page hash |
| University folder | file watcher + hash check |
| Book | usually static, no recurring check |
| GitHub repo | commit hash |
| Obsidian note | frontmatter hash |

## Update flow

```text
scheduled check
  -> detect changed source
  -> compare hashes
  -> extract changed pages/articles only
  -> rebuild affected chunks
  -> delete stale vectors for changed chunks
  -> insert new vectors
  -> update FTS index
  -> update Graphify candidate changes
  -> update Obsidian candidate notes
  -> generate report
```

Use systemd timers first. n8n can be added later for visual workflows, approvals, and notifications, but it should not be the core processing engine.

# Dashboard and endpoint workflow

Laptops should be endpoints only.

Recommended user flow:

1. Open dashboard from laptop.
2. Drop file, folder, RSS feed, or URL.
3. Select source category if needed.
4. System registers the source and starts routing.
5. Dashboard shows extraction status.
6. Low-confidence pages appear in review queue.
7. User approves or rejects promotion candidates.
8. Approved notes go to Obsidian and Graphify.
9. RAG endpoint can answer with citations.

Dashboard cards:

- Ingestion queue
- OCR queue
- Failed pages
- Low-confidence tables
- Recently indexed sources
- Obsidian export candidates
- Graphify export candidates
- Retrieval test results
- Storage usage
- GPU queue status

# Quality gates

## Extraction score

Every document should receive:

- text score
- layout score
- table score
- figure score
- citation/provenance score
- OCR confidence
- language confidence
- final pass/fail status

## Required reports

For every batch:

```text
extraction_report.md
low_confidence_pages.md
failed_tables.json
changed_sources.json
retrieval_eval.json
promotion_candidates.md
```

## Golden test set

Create a small benchmark library:

- 10 digital PDFs
- 10 scanned PDFs
- 5 table-heavy documents
- 5 Romanian documents
- 5 English documents
- 5 mixed Romanian/English documents
- 5 university PDFs
- 5 pages with diagrams or screenshots
- 5 RSS/web sources

Manual checks:

- page count correct
- text not missing
- reading order correct
- table rows and columns preserved
- figures detected
- formulas acceptable
- search returns correct source in top 5
- answer cites correct page

# Security and privacy

Treat all documents as untrusted input.

Rules:

- process files in containers where possible
- disable macro execution
- do not auto-run embedded scripts
- preserve raw files as read-only after registration
- keep private documents local
- keep logs free of sensitive full text where possible
- classify sources as private, course, project, public, or unknown
- never send private documents to cloud services without explicit approval
- keep backups encrypted if they include personal or university material

# Implementation roadmap

## Phase 1: Foundation

Goal: controlled ingestion with no memory writes.

Build:

- folder structure
- SQLite registry
- file hashing
- source registration API
- upload folder
- basic dashboard status
- job state machine
- reports folder

Exit criteria:

- files can be added
- duplicates are detected
- jobs are tracked
- raw files remain immutable

## Phase 2: Document routing and extraction

Build:

- document router
- digital PDF parser
- Markdown/JSON output
- page image rendering
- table extraction attempt
- extraction report

Exit criteria:

- digital PDFs produce Markdown and JSON
- every page has page-level metadata
- failed pages are visible

## Phase 3: OCR and visual validation

Build:

- OCR fallback
- low-confidence page detection
- OCR/VLM GPU lock
- table repair queue
- scanned PDF path

Exit criteria:

- scanned PDFs can be processed
- low-confidence pages are routed correctly
- OCR does not block normal retrieval

## Phase 4: Indexing and RAG

Build:

- chunker
- BGE-M3 embedding worker
- Qdrant collection
- SQLite FTS5 index
- hybrid retrieval endpoint
- answer endpoint with source citations

Exit criteria:

- user can ask questions
- answers cite document/page/chunk
- weak retrieval is reported honestly

## Phase 5: Obsidian and Graphify export

Build:

- Obsidian note templates
- Graphify node/edge JSON export
- promotion queue
- review UI

Exit criteria:

- no automatic brain dumping
- reviewed notes can be exported
- graph relationships cite sources

## Phase 6: Dynamic updates

Build:

- RSS checker
- folder watcher
- URL checker
- changed-page detector
- incremental re-index
- update reports

Exit criteria:

- only changed material is reprocessed
- stale vectors are removed
- reports show what changed

## Phase 7: Evaluation harness

Build:

- golden document set
- extraction scoring
- retrieval scoring
- model comparison reports
- regression tests

Exit criteria:

- changes to parsers/models can be measured
- broken extraction is detected before promotion

# Recommended first build stack

Start with:

```text
Python
SQLite
PyMuPDF
pdfplumber
Docling
Marker
MinerU
PaddleOCR / PaddleOCR-VL
olmOCR
BGE-M3 via FlagEmbedding
Qdrant
SQLite FTS5
Ollama for local LLM control
systemd timers
Obsidian Markdown export
Graphify JSON export
```

Add later:

```text
n8n for visual approvals and notifications
Qwen3 Embedding / reranker after baseline retrieval works
Qwen-VL page validation after OCR routing is stable
Telegram / WhatsApp notifications
```

# Working strategy for the AI-PC

Recommended operating mode:

- one GPU-heavy job at a time
- background OCR mainly overnight
- embedding batches after extraction
- live RAG/chat gets priority
- keep batch size small until memory use is measured
- never process a large book without batch checkpoints
- write every intermediate artefact to disk
- keep all hashes and model versions

Good default policy:

```text
Small files: process immediately
Books: process in batches
Scanned files: queue for OCR window
RSS: process on schedule
Web pages: process with hash checks
University material: process immediately, then create review candidates
```

# Final target

The final system should let you drop in a PDF, scanned document, RSS feed, book, image, or university file and have the AI-PC automatically:

1. register it
2. classify it
3. extract it
4. validate it
5. OCR only where needed
6. create Markdown and JSON
7. chunk it with provenance
8. embed it
9. index it for hybrid retrieval
10. create reviewable Obsidian/Graphify candidates
11. update changed sources later
12. answer questions with citations

The core idea is simple:

> Build a document operating system first, then build RAG on top of it.

A vector database is only one part of the stack. The valuable part is the controlled pipeline that turns messy human-readable material into trustworthy, source-linked knowledge.

## Sources checked

Checked on 2026-07-08. Use these as starting references, not as permanent truth. Re-check versions before installing.

- **Docling** - Document parsing and conversion, including PDF understanding, OCR, tables, formulas, and reading order.  
  https://www.docling.ai/
- **Marker** - Converts PDF, images, Office files, EPUB, HTML to Markdown, JSON, chunks, and HTML.  
  https://github.com/datalab-to/marker
- **MinerU** - Document parsing for PDF, image, DOCX, PPTX, XLSX to Markdown and JSON, useful for scientific/technical documents.  
  https://github.com/opendatalab/MinerU
- **PaddleOCR / PaddleOCR-VL** - OCR and document parsing toolkit for PDFs/images, including multilingual document parsing.  
  https://github.com/PaddlePaddle/PaddleOCR
- **olmOCR** - Toolkit for converting PDFs and image-based documents into clean text in natural reading order.  
  https://github.com/allenai/olmocr
- **Qdrant hybrid queries** - Hybrid retrieval combining dense and sparse representations.  
  https://qdrant.tech/documentation/search/hybrid-queries/
- **SQLite FTS5** - SQLite full-text search virtual table module.  
  https://www.sqlite.org/fts5.html
- **FlagEmbedding / BGE-M3** - BGE-M3 multilingual dense, sparse, and multi-vector retrieval.  
  https://github.com/FlagOpen/FlagEmbedding
- **Qwen3 Embedding** - Embedding and reranking model family for multilingual retrieval.  
  https://github.com/QwenLM/Qwen3-Embedding
- **Ollama Qwen3 14B** - Qwen3 14B local model packaging information for Ollama.  
  https://ollama.com/library/qwen3:14b
- **systemd timers** - Timer-based activation for recurring local jobs.  
  https://man.archlinux.org/man/systemd.timer.5.en
