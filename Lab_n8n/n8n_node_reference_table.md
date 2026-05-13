# n8n Node Reference Table
**Workflow: Document Upload RAG Chatbot with Cohere Reranking**

---

## lab_summary.md

The most useful nodes in this workflow are the **Pinecone Vector Store**, **Embeddings**, and **AI Agent** — together they form the backbone of any RAG pipeline. When picking a node, the guiding question is: *what does this node do to the data?* — trigger nodes start flows, loader/splitter nodes prepare documents, embedding nodes convert text to vectors, vector store nodes persist or retrieve them, and agent/LLM nodes reason over the results. My top debugging tip: always check the **Input/Output panel** in n8n's execution view — mismatched data shapes (e.g. flat JSON when an array is expected) account for 80% of node errors, and seeing the actual JSON structure immediately tells you where the chain is broken.

---

## Workflow Overview

```
[Upload Documents Form] 
        │
        ▼
[Store in Pinecone] ◄── [OpenAI Embeddings] ◄── [Document Loader] ◄── [Text Splitter]

[Chat Interface]
        │
        ▼
[RAG Agent] ◄── [OpenAI Chat Model]
        ▲
        └── [Retrieve from Pinecone] ◄── [Query Embeddings]
                                     ◄── [Cohere Reranker]
```

---

## Node Reference Table

| Node | Type | Parameters | Settings | What It Does | JSON Input | JSON Output | Key Transformations |
|------|------|------------|----------|--------------|------------|-------------|---------------------|
| **Upload Documents Form** | Trigger | `formTitle`, `formDescription`, `formFields` (fieldLabel, fieldType, fieldName, acceptFileTypes) | `appendAttribution: false` | Renders a web form for file uploads; starts the ingestion pipeline when user submits | None (trigger) | `{ "documents": <binary file data>, "formId": "..." }` | HTTP multipart form → n8n binary item |
| **Store in Pinecone** | Vector Store | `mode: insert`, `pineconeIndex: "test-n8n"` | Credentials: Pinecone API | Receives embedded document chunks and upserts vectors into the Pinecone index | `{ "documents": [{ "pageContent": "...", "metadata": {} }] }` (from Document Loader via ai_document) | `{ "success": true, "ids": ["doc1#chunk1", ...] }` | LangChain Document objects → Pinecone upsert |
| **OpenAI Embeddings** | Embeddings | `dimensions: 512` | Credentials: OpenAI API | Converts text chunks into 512-dimensional dense vectors used for indexing | `{ "text": "chunk content..." }` | `{ "embedding": [0.023, -0.11, ...] }` (512 floats) | Text → float vector array |
| **Document Loader** | Document Loader | `dataType: binary`, `textSplittingMode: custom` | — | Reads binary file data (PDF/CSV/JSON) and converts it into LangChain Document objects for downstream processing | `{ "documents": <binary> }` (from Form trigger) | `[{ "pageContent": "extracted text", "metadata": { "source": "filename.pdf" } }]` | Binary file → LangChain Document array |
| **Text Splitter** | Text Splitter | `chunkSize` (default 1000), `chunkOverlap` (default 200) | — | Splits long documents into smaller overlapping chunks to fit within embedding model token limits | `[{ "pageContent": "long text..." }]` | `[{ "pageContent": "chunk 1..." }, { "pageContent": "chunk 2..." }]` | Long document → overlapping text chunks |
| **Chat Interface** | Trigger | `public: true`, `initialMessages` | `webhookId` auto-generated | Exposes a public chat UI; triggers the retrieval pipeline when user sends a message | None (trigger) | `{ "chatInput": "user's question", "sessionId": "abc123" }` | User chat message → n8n item |
| **RAG Agent** | AI Agent | `systemMessage` (custom prompt), connects to LLM + tools | `executionOrder: v1` | Orchestrates the full RAG loop: receives user query, calls retrieval tool, passes context to LLM, returns answer | `{ "chatInput": "question", "sessionId": "..." }` | `{ "output": "AI-generated answer citing documents" }` | Query + retrieved context → grounded LLM response |
| **OpenAI Chat Model** | Language Model | `model: gpt-4.1-mini`, `builtInTools: {}` | Credentials: OpenAI API | Provides the LLM reasoning layer for the RAG Agent; generates answers based on retrieved context | `{ "messages": [{ "role": "system", ... }, { "role": "user", ... }] }` | `{ "content": "answer text", "usage": { "tokens": 320 } }` | Prompt + context → generated text |
| **Retrieve from Pinecone** | Vector Store (Retrieve) | `mode: retrieve-as-tool`, `pineconeIndex: "n8n"`, `topK: 5`, `useReranker: true` | Credentials: Pinecone API | Queries Pinecone for the top-K most similar vectors to the user's query; exposed as a tool to the RAG Agent | `{ "query": "user question embedding" }` | `[{ "pageContent": "relevant chunk", "metadata": {}, "score": 0.91 }]` (top 5) | Query vector → ranked relevant document chunks |
| **Query Embeddings** | Embeddings | `dimensions: 512` | Credentials: OpenAI API | Converts the user's query string into a 512-dim vector for similarity search in Pinecone | `{ "text": "user's question" }` | `{ "embedding": [0.045, -0.08, ...] }` (512 floats) | Query text → float vector |
| **Cohere Reranker** | Reranker | *(default settings)* | Credentials: Cohere API | Re-scores the top-K Pinecone results using Cohere's cross-encoder reranking model for higher precision | `[{ "pageContent": "chunk", "score": 0.82 }, ...]` (5 candidates) | `[{ "pageContent": "chunk", "score": 0.97 }, ...]` (reranked) | Bi-encoder scores → cross-encoder reranked scores |

---

## Additional Nodes from Lab Template

| Node | Parameters | Settings | What It Does | JSON Input | JSON Output | Key Transformations |
|------|------------|----------|--------------|------------|-------------|---------------------|
| **Webhook** | Method, Path, Response Mode | Auth, Response Code | Receives HTTP requests and starts a workflow | HTTP request | `{ "body": {}, "headers": {}, "query": {} }` | HTTP request → n8n data format |
| **HTTP Request** | Method, URL, Auth | Response Format, Headers | Makes API calls to external services | n8n data item | API response JSON | n8n → external API → n8n |
| **Set** | Keep Fields, Values | Assignments | Adds, modifies, or removes fields in the data item | Any JSON | Modified JSON with new/changed fields | Field manipulation |
| **Function / Code** | Code | Language (JS/Python) | Executes custom code for complex transformations | Any JSON | Any JSON (custom shape) | Arbitrary code-based transformation |
| **IF** | Condition, Value1, Operation, Value2 | Combine Conditions | Routes items down true or false branch based on condition | Any JSON | Same JSON (routed to branch) | Conditional routing |
| **Switch** | Rules | Mode | Routes items to one of multiple branches | Any JSON | Same JSON (routed to matching branch) | Multi-path routing |
| **Merge** | Mode | Options | Combines two or more data streams into one | Multiple input streams | Merged item array | Data stream combination |
| **Split In Batches** | Batch Size | Options | Breaks a large array into smaller batches for rate-limit-safe processing | Array of items | Batches of N items | Array → batch chunks |
| **Wait** | Amount, Unit | Resume | Pauses workflow execution for a set time | Any JSON | Same JSON (after delay) | Time delay |
| **Notion** | Operation, Resource | Database ID, Page ID | Reads/writes Notion pages and databases | n8n data | Notion API response | n8n ↔ Notion database |

---

## Key Concepts: Data Flow in This Workflow

### Ingestion Pipeline (Upload → Pinecone)
```
Binary file (PDF/CSV/JSON)
  → Document Loader     [binary → pageContent + metadata]
  → Text Splitter       [long text → chunks ~1000 chars]
  → OpenAI Embeddings   [text → 512-dim float vector]
  → Store in Pinecone   [vector + metadata → upserted record]
```

### Retrieval Pipeline (Chat → Answer)
```
User message (string)
  → Query Embeddings    [text → 512-dim float vector]
  → Retrieve Pinecone   [vector → top 5 similar chunks]
  → Cohere Reranker     [5 chunks → reranked by relevance]
  → RAG Agent           [chunks + query → grounded answer]
  → OpenAI Chat Model   [prompt → generated text]
```

---

## Important Notes

- **Two separate Pinecone indexes** are used: `test-n8n` for storing, `n8n` for retrieval — in production these should be the same index.
- **Embedding dimensions must match** between ingestion (OpenAI, 512-dim) and retrieval (OpenAI, 512-dim) — mixing models will cause errors.
- **Cohere Reranker** improves precision after Pinecone's approximate nearest-neighbor search by using a slower but more accurate cross-encoder model.
- **`retrieve-as-tool` mode** exposes the Pinecone node as a callable tool for the AI Agent, enabling the agent to decide *when* to retrieve rather than always retrieving.
- **Document Loader uses binary mode** — it reads the file uploaded via the form trigger directly, so no intermediate HTTP download step is needed.
