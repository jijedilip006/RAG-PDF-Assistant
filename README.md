# RAG Document Assistant

A full-stack Retrieval-Augmented Generation (RAG) assistant designed to answer user queries based strictly on internal company documents (e.g., user guides, employee handbooks). 

The system is designed to provide zero-hallucination responses by grounding the LLM entirely on pre-indexed document chunks, utilizing semantic and hybrid search.

## Architecture Overview

The system is intentionally decoupled into two distinct pipelines to separate the heavy, resource-intensive document parsing from the fast, lightweight web application.

### 1. Offline Ingestion Pipeline (Admin / Python)
Since company documents are updated periodically, parsing and indexing happen offline rather than during user interactions.
* **Parser:** [Docling](https://github.com/DS4SD/docling) (Python) - Extracts highly structured Markdown from PDFs, preserving tables, headers, and section hierarchies.
* **Embeddings:** Google GenAI (`text-embedding-004`).
* **Storage:** [Supabase](https://supabase.com/) (`pgvector`) - Stores document text chunks, metadata (page numbers, section titles), and vector embeddings.

### 2. Live Web Application (User Facing / Next.js)
A lightweight Next.js application that handles user chat interactions, deployed entirely on Vercel.
* **Frontend:** Next.js (App Router), React, and Tailwind CSS.
* **Backend API:** Next.js Route Handlers (`/api/chat`).
* **Retrieval Engine:** Supabase Vector Search (Cosine Similarity).
* **LLM Engine:** Google GenAI (`gemini-2.0-flash`) with strict grounding prompts and token streaming via Server-Sent Events (SSE).

## Project Structure

This is a monorepo containing both the offline ingestion tools and the production web application.

```text
rag-company-assistant/
├── README.md
├── .gitignore
│
├── ingestion/                          
│   ├── .env                            
│   ├── requirements.txt                
│   ├── data/                           
│   └── ingest.py                       
│
└── web/                                
    ├── package.json
    ├── .env.local                      
    ├── next.config.ts
    ├── src/
    │   ├── app/
    │   │   ├── api/chat/route.ts       
    │   │   ├── layout.tsx
    │   │   └── page.tsx                
    │   ├── components/                 
    │   ├── hooks/                      
    │   └── lib/                        
```

## Directory & File Guide
### Root
* ```README.md``` — Project documentation, architecture overview, and setup guides.

* ```.gitignore``` — Specifies files and folders untracked by Git (e.g., virtual environments, local environment variables, cache).

```/ingestion``` **(Offline Processing)**
* ```data/``` — Local directory where source PDFs, manuals, and documents are placed before indexing.

* ```ingest.py``` — Python script that reads documents via Docling, splits them into semantic chunks, generates vector embeddings, and writes the records to Supabase.

* ```requirements.txt``` — Python dependencies needed only for ingestion (Docling, Google GenAI SDK, Supabase client).

* ```.env``` — Local environment variables for ingestion (Supabase Service Role Key and Gemini API Key).

```/web``` **(Production Application)**
* ```src/app/page.tsx``` — The main chat user interface layout.

* ```src/app/layout.tsx``` — Root application wrapper containing font definitions, metadata, and global HTML headers.

* ```src/app/api/chat/route.ts``` — Server-side API endpoint handling query rewriting, vector retrieval from Supabase, and streaming the grounded response from Gemini.

* ```src/components/``` — Modular React UI components including message displays, input boxes, loading skeletons, and citation badges.

* ```src/hooks/``` — Custom React hooks for consuming Server-Sent Events streams and persisting chat sessions in browser local storage.

* ```src/lib/``` — Shared server utilities, database client initializations, TypeScript type definitions, and system prompts.

* ```.env.local``` — Web app secrets and public client keys for Vercel and local development.

* ```package.json``` — JavaScript/TypeScript dependencies and build scripts.

* ```next.config.ts``` — Next.js runtime configuration.