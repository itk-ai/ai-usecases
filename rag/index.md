# LLM RAG Systems

## Overview & Structure

- Part 1: Data preparation & ingestion (how your knowledge gets into the system)
- Part 2: Retrieval & context engineering (how you get the right info to the LLM)
- Each part: general design first, then choices & trade-offs, with mermaid flows.


## Information Types & Sources — High-Level Overview

A RAG ingestion system must handle a wide variety of formats and content origins.

```mermaid
flowchart TB
  A[Desktop-Published Documents
PDF, DOCX, PPTX, ODT, EPUB] --> Ingest[Ingestion System]
  B[Plain Text & Markup
TXT, Markdown, HTML, LaTeX, JSON, XML, YAML] --> Ingest
  C[Web Sources
Web pages, APIs, CMS exports, sitemaps, RSS feeds] --> Ingest
  D[Databases & Enterprise Systems
SQL dumps, NoSQL exports, CRM/ERP data, Graph DBs] --> Ingest
  E[Code & Technical Assets
source code repos, notebooks, configuration files] --> Ingest
  F[Multimedia
Audio recordings, podcasts, video transcripts, images with OCR] --> Ingest
  G[Specialized Formats
spreadsheets, CAD files, scientific articles (PubMed), logs] --> Ingest
  Ingest --> Unified[Unified Preprocessing Pipeline]
```

This overview highlights the diversity of real-world formats a complete RAG ingestion system may need to support, from documents to APIs to multimedia and enterprise data.


## Part 1 — Framework-based ingestion (Ragflow & Langflow)

How no-code / low-code RAG builders guide ingestion choices.
        
- **Source connectors**: drag‑and‑drop blocks for files, URLs, APIs, databases.
- **Automatic cleaning**: HTML stripping, PDF parsing, OCR add-ons.
- **Configurable chunking**: fixed-size, semantic, overlap tuning.
- **Embeddings**: choose model, batching size, error handling.
- **Vector store integrations**: Pinecone, Weaviate, Qdrant, Chroma.
- **Workflow orchestration**: ingestion flows that run manually or on schedule.


### Ragflow ingestion options

- **Document loaders**: PDFs, Office docs, HTML, Markdown, images (OCR), audio transcripts.
- **Web crawling**: sitemap ingest, depth control, rate limiting, auto-cleaning.
- **Database ingestion**: SQL query blocks, auto-pagination, incremental loads.
- **Preprocessing blocks**: deduplication, text cleaning, language detection.
- **Chunking panel**: semantic or token-based, adjustable overlap, preview before embedding.
- **Embedding model selector** pick local/cloud models; show cost estimates.
- **Vector DB block**: plug into Pinecone, Qdrant, Milvus; configure metadata fields.

### Langflow ingestion options

- **Loader nodes**: URL loader, text loader, directory loader, S3 loader.
- **Transform nodes**: text splitters (recursive, character, Markdown-aware).
- **Cleaning nodes**: regex cleaning, HTML removal, custom Python transforms.
- **Embedding nodes**: support for OpenAI, HuggingFace, local models.
- **Vector store nodes**: Chroma, FAISS, Pinecone, Weaviate; configurable indices.
- **Scheduling**: build flows that re-run ingestion on a schedule via Langflow API.

### Framework-guided ingestion flow

```mermaid
flowchart LR
  Loader[Source Loader Blocks
(Ragflow / Langflow)] --> Clean[Cleaning Blocks
(HTML strip, OCR, regex)]
  Clean --> Split[Chunking Blocks
(fixed, semantic, overlap)]
  Split --> Embed[Embedding Block
(model choice)]
  Embed --> VDB[Vector DB Block
(metadata mapping)]
  VDB --> Done[Indexed Knowledge Base]
```

## Part 1 — General design: Data preparation & ingestion
        <p class="big">Goal: turn raw knowledge into clean, searchable pieces that a retriever can use.</p>
        <ul>
          <li>Collect sources (documents, databases, web pages, user files)</li>
          <li>Clean & normalize (formatting, deduplication, OCR where needed)</li>
          <li>Chunking — split large content into bite-sized, meaningful pieces</li>
          <li>Embed & index — create dense representations and store them in a vector index</li>
          <li>Metadata — attach context (author, date, source, type) to improve retrieval</li>
        </ul>
      </section>

      <!-- Mermaid flow for part 1 -->
      <section>
        <h3>Information flow — Data preparation (high level)</h3>
        <div class="diagram">
<pre class="mermaid">flowchart LR
  A[Raw sources\n(files, DB, web, email)] --> B[Ingestion pipeline\n(fetch & normalize)]
  B --> C[Cleaning & deduplication]
  C --> D[Chunking & segmentation]
  D --> E[Embedding service\n(vector generation)]
  E --> F[Vector index\n(store & metadata)]
  F --> G[Retriever\n(ANN search / filters)]
  G --> H[LLM prompt assembler\n(context + query]
  style A fill:#f9f,stroke:#333,stroke-width:1px
</pre>
        </div>
        <p class="notes">This flow shows how raw data becomes vectors and then gets used at inference time.</p>
      </section>

      <!-- Part 1 choices -->
      <section>
        <h2>Part 1 — Key choices & trade-offs</h2>
        <h4>1) Source selection</h4>
        <ul>
          <li><strong>Wide:</strong> include everything (comprehensive but noisy)</li>
          <li><strong>Curated:</strong> only high-value sources (cleaner but risks blind spots)</li>
        </ul>
        <h4>2) Cleaning & normalization</h4>
        <ul>
          <li>Strip HTML, correct OCR errors, unify date formats</li>
          <li>Trade-off: more cleaning = better quality but higher prep time</li>
        </ul>
        <h4>3) Chunking strategy</h4>
        <ul>
          <li>Fixed-size chunks (simple, consistent)</li>
          <li>Semantic chunks (paragraphs, sections — keeps meaning but needs parsing)</li>
          <li>Hybrid (limit token size but prefer semantic boundaries)</li>
        </ul>
        <h4>4) Embedding model</h4>
        <ul>
          <li>General-purpose vs. domain-specific — domain models often help for niche vocabulary</li>
          <li>Compute & cost trade-offs</li>
        </ul>
        <h4>5) Index & storage</h4>
        <ul>
          <li>Cloud vector DB (managed, easy) vs self-hosted (flexible, control)</li>
          <li>Indexing strategy: flat vs. approximate (ANN) — ANN gives speed at slight recall cost</li>
        </ul>
      </section>

      <!-- Part 1 implementation patterns -->
      <section>
        <h2>Part 1 — Implementation patterns (non-technical)</h2>
        <ul>
          <li><strong>Batch pipelines:</strong> periodic full loads (good for stable data)</li>
          <li><strong>Streaming / event-driven:</strong> ingest changes in real time (good for fresh content)</li>
          <li><strong>Hybrid:</strong> daily full sync + stream for recent changes</li>
        </ul>
        <p class="notes">Choose based on how often your sources change and how fresh results must be.</p>
      </section>

      <!-- Part 2 general design -->
      <section>
        <h2>Part 2 — General design: Retrieval & Context Engineering</h2>
        <p class="big">Goal: select the best pieces of knowledge and assemble them into a prompt the LLM can use effectively.</p>
        <ul>
          <li>Retriever selects candidate chunks from the vector index</li>
          <li>Reranker (optional) improves ordering and precision</li>
          <li>Filter & apply business rules (date ranges, trust levels, permissions)</li>
          <li>Prompt assembly — decide what to include, how to format it, and where to put the user's question</li>
          <li>Safety & hallucination mitigation — attribution, citations, fallbacks</li>
        </ul>
      </section>

      <!-- Mermaid flow for part 2 -->
      <section>
        <h3>Information flow — Retrieval to LLM</h3>
        <div class="diagram">
<pre class="mermaid">flowchart LR
  Q[User query]
  Q --> R[Retriever: ANN search\n(top-K candidates)]
  R --> S[Reranker (optional)\n(re-score & order)]
  S --> T[Filter rules\n(trust, recency, permissions)]
  T --> U[Context assembler\n(select & format snippets)]
  U --> V[LLM + prompt\n(includes context + question)]
  V --> W[Response post-processing\n(cite, check hallucination)]
  style Q fill:#fffae6,stroke:#333
</pre>
        </div>
        <p class="notes">This flow focuses on selection, ordering, and assembly of context for the LLM.</p>
      </section>

      <!-- Part 2 choices -->
      <section>
        <h2>Part 2 — Design choices & trade-offs</h2>
        <h4>1) Retriever type</h4>
        <ul>
          <li><strong>Dense vector search:</strong> good for semantic matches, handles synonyms well</li>
          <li><strong>BM25 / lexical search:</strong> exact term matches — useful for short, specific queries</li>
          <li><strong>Hybrid:</strong> combine both for robust recall</li>
        </ul>

        <h4>2) K (how many candidates to retrieve)</h4>
        <ul>
          <li>Small K (e.g., 3–5): smaller prompt, faster, less noise</li>
          <li>Large K (e.g., 20+): more coverage but larger prompt — increases cost and risk of irrelevant context</li>
        </ul>

        <h4>3) Reranking</h4>
        <ul>
          <li>Use a lightweight model or signal-based scoring (recency, source trust) to reorder candidates</li>
        </ul>

        <h4>4) Context assembly strategies</h4>
        <ul>
          <li><strong>Concatenate:</strong> simple — paste snippets in order (may confuse the model if noisy)</li>
          <li><strong>Template + citation:</strong> summarize each snippet and link to its source — helps traceability</li>
          <li><strong>Socratic/Q&A framing:</strong> turn snippets into Q&A pairs the model can use directly</li>
        </ul>

        <h4>5) Prompt framing</h4>
        <ul>
          <li>Instruction-first (tell the model what to do, then provide context)</li>
          <li>Context-first (give context, then ask the question)</li>
          <li>Few-shot examples can guide answer style and guardrails</li>
        </ul>
      </section>

      <!-- Examples slide -->
      <section>
        <h2>Practical patterns & examples</h2>
        <h4>Low-cost FAQ assistant</h4>
        <ul>
          <li>Curate top FAQs, chunk by question, embed and index.</li>
          <li>Retriever: dense vectors; K=3; concatenate snippets into a short prompt.</li>
        </ul>
        <h4>Fresh news assistant</h4>
        <ul>
          <li>Stream ingestion; add timestamps; heavy recency filtering; reranker that prioritizes recent sources.</li>
        </ul>
        <h4>Compliance-aware enterprise assistant</h4>
        <ul>
          <li>Strong metadata filters (sensitivity labels), strict source whitelists, human-in-the-loop review for uncertain answers.</li>
        </ul>
      </section>

      <!-- Mermaid: hybrid retrieval example -->
      <section>
        <h3>Hybrid retrieval example (mermaid)</h3>
        <div class="diagram">
<pre class="mermaid">flowchart LR
  UserQ --> Dense[Dense retriever (vectors)]
  UserQ --> Lexical[BM25 / lexical]
  Dense --> Combine[Combine & dedupe]
  Lexical --> Combine
  Combine --> Rerank[Rerank (signals: recency, trust)]
  Rerank --> Assemble[Assemble context + citations]
  Assemble --> LLM
</pre>
        </div>
      </section>

      <!-- Mitigations & best practices -->
      <section>
        <h2>Mitigations & Best Practices</h2>
        <ul>
          <li>Monitor retrieval quality (A/B test different K and rerankers)</li>
          <li>Log sources used for each answer — builds trust and enables audits</li>
          <li>Limit token budgets — trim long snippets, prioritize salient parts</li>
          <li>Provide explicit fallback messages when confidence is low</li>
          <li>Keep a human review path for sensitive domains</li>
        </ul>
      </section>

      <!-- Quick checklist -->
      <section>
        <h2>Quick checklist to get started</h2>
        <ol>
          <li>Inventory your sources and decide curation strategy</li>
          <li>Choose chunking rules (semantic + token limits)</li>
          <li>Pick an embedding model and vector store</li>
          <li>Design retriever + reranker + assembly pipeline</li>
          <li>Deploy monitoring, logging, and review processes</li>
        </ol>
      </section>

      <!-- Closing slide -->
      <section>
        <h2>Summary</h2>
        <p class="big">Two pillars of RAG: prepare your data carefully, then design retrieval to present concise, trustworthy context to the LLM.</p>
        <p>Decisions are trade-offs: accuracy vs. latency vs. cost vs. freshness. Start simple and iterate.</p>
        <p class="notes">Questions? Tell me which slide you'd like to expand or convert into handouts.</p>
      </section>