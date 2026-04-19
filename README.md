# Prostate Cancer Biomarker Analysis — Transcriptomics + Knowledge Graph

> **Status: Work in Progress** — Steps 1–7 are functional. The knowledge graph (Step 8) and downstream querying interface are incomplete.

End-to-end pipeline integrating GEO transcriptomics data with PubMed literature for prostate cancer biomarker analysis. Differentially expressed genes are extracted, merged with relevant publications, chunked and embedded into a vector store, and queried via a RAG system. The final step builds a Neo4j knowledge graph from the merged data.

---

## Pipeline

```
notebooks/
├── step1_pubmed_extraction.ipynb          # Fetches prostate cancer papers from PubMed via E-utilities and
│                                          # Biopython Entrez; downloads and parses GEO dataset GSE45016
│                                          # using GEOparse; extracts expression matrix to CSV
│
├── step2_GEO_DEG_analysis.ipynb           # Loads GSE45016 expression data; filters differentially expressed
│                                          # genes by logFC (≥1 upregulated, ≤-1 downregulated) and p-value
│                                          # cutoff (p < 0.05); plots logFC distribution; saves DEG CSVs
│
├── step4_convert_to_JSON.ipynb            # Loads fuzzy-merged GEO + PubMed CSV; converts each row into a
│                                          # structured JSON record with gene symbol, title, regulation
│                                          # status, and associated paper metadata; saves as JSONL
│
├── step5_chunking.ipynb                   # Applies sliding window sentence chunking (150 words, stride 75)
│                                          # to JSONL records; formats each chunk with gene context prepended;
│                                          # outputs chunked JSONL for embedding
│
├── step6_vectorise_embed_chromadb.ipynb   # Generates sentence-transformer embeddings for all text chunks;
│                                          # stores embeddings and texts in a persistent ChromaDB collection
│
├── step7_RAG_system.ipynb                 # Retrieves top-k relevant chunks from ChromaDB using OpenAI
│                                          # text-embedding-3-small; passes context to GPT for answer
│                                          # generation; interactive Q&A loop (incomplete)
│
└── step8_knowledge_graph_neo4j.ipynb      # Connects to Neo4j AuraDB; creates Gene nodes and
                                           # INVOLVED_IN relationships from merged DEG + paper data
                                           # (incomplete — graph schema partially defined)
```

> Note: Step 3 (fuzzy merging of GEO expression data with PubMed papers) was run as a standalone script; output is `GEO_Papers_Merged_Fuzzy.csv`.

## Stack

Python, PubMed E-utilities API, Biopython, GEOparse, pandas, sentence-transformers, ChromaDB, OpenAI API, Neo4j (AuraDB)

## Data

- `GSE45016` — prostate cancer gene expression dataset from GEO (Affymetrix microarray)
- PubMed abstracts queried via E-utilities API
- Intermediate outputs (merged CSV, embeddings pickle) not included due to file size
