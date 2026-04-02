# 🧠 RAG Knowledge Base Agent

> Full RAG (Retrieval-Augmented Generation) pipeline — automatically extracts PDFs from Google Drive, converts them to vector embeddings, stores them in Pinecone, and answers questions accurately from indexed documents.

---

## 🧠 System Overview

This n8n workflow implements a complete RAG system in two parts:
1. **Ingestion Pipeline** — Reads PDFs from Google Drive, splits text into chunks, creates embeddings, and stores them in Pinecone with per-file namespacing
2. **Query Pipeline** — Accepts chat questions, retrieves the top-k most relevant chunks from Pinecone, and generates grounded answers using GPT-4.1-mini

---

## ⚙️ Tech Stack

| Tool | Role |
|------|------|
| **n8n** | Workflow automation engine |
| **Google Drive** | PDF document source storage |
| **Google Sheets** | Document tracking & deduplication |
| **OpenAI Embeddings** | `text-embedding-ada-002` vector creation |
| **Pinecone** | Vector store with namespace-per-file isolation |
| **OpenAI GPT-4.1-mini** | Grounded answer generation |

---

## 🔄 Workflow Logic

```
── INGESTION PIPELINE ─────────────────────────────────────────
Schedule Trigger (or Manual)
  └─> Google Drive: List files in monitored folder
  └─> Google Sheets: Get already-indexed file list
        └─> Merge + JavaScript: Find NEW files only
              └─> Append to Sheets (mark as indexed)
              └─> Loop over new files
                    └─> Download PDF from Drive
                    └─> Recursive Text Splitter → chunks
                    └─> OpenAI Embeddings → vectors
                    └─> Pinecone (namespace = file_name): Store

── QUERY PIPELINE ─────────────────────────────────────────────
Chat Trigger (user sends question)
  └─> AI Agent (GPT-4.1-mini)
        └─> Tool: Vector Store Search
              └─> Pinecone: Retrieve top-10 relevant chunks
              └─> OpenAI Embeddings (query vector)
        └─> Generate grounded answer from retrieved context
```

---

## 🌟 Key Features

- **Auto-deduplication** — Google Sheets tracks indexed files; only new files are processed
- **Namespace isolation** — Each file stored in its own Pinecone namespace for clean retrieval
- **Recursive text splitting** — Smart chunking preserves sentence/paragraph context
- **Grounded answers only** — AI Agent responds strictly from indexed documents; admits when information is missing
- **Scalable** — Works with any number of PDFs; loop-based processing handles large batches
- **Dual trigger** — Runs on schedule or manually triggered for on-demand ingestion

---

## 📋 Google Sheets Schema Required

**Sheet: Indexed Files**

| Column | Type | Description |
|--------|------|-------------|
| `NAME` | Text | File name from Google Drive |
| `ID` | Text | Google Drive file ID |
| `Indexed_At` | DateTime | When the file was processed |
| `Namespace` | Text | Pinecone namespace (same as file name) |
| `Status` | Text | `indexed` / `failed` |

---

## 🚀 Setup Instructions

### 1. Prerequisites
- n8n instance (self-hosted or cloud)
- Google Drive folder with PDF documents
- Google Sheets for file tracking
- Pinecone account with an active index (dimension: 1536 for `text-embedding-ada-002`)
- OpenAI API key

### 2. Import Workflow
1. Download `workflow.json`
2. In n8n: **Settings → Import from file**
3. Upload `workflow.json`

### 3. Configure Credentials
Update the following in n8n credentials:
- `Google Drive OAuth2` — Your Google account
- `Google Sheets OAuth2` — Same Google account
- `OpenAI API` — Your OpenAI API key
- `Pinecone API` — Your Pinecone API key

### 4. Update Placeholders in Workflow
Search and replace:
- `YOUR_PINECONE_INDEX_NAME` → Your Pinecone index name
- `YOUR_GOOGLE_DRIVE_FOLDER_ID` → Your PDF folder ID in Google Drive
- `YOUR_GOOGLE_SHEETS_ID` → Your tracking spreadsheet ID
- `YOUR_GOOGLE_DRIVE_CREDENTIAL_ID` → n8n credential ID
- `YOUR_OPENAI_CREDENTIAL_ID` → n8n credential ID
- `YOUR_PINECONE_CREDENTIAL_ID` → n8n credential ID

### 5. Configure the AI Agent System Prompt
In the `AI Agent` node, update the system prompt to match your knowledge domain. The default prompt is domain-agnostic — the AI answers only from indexed documents and states when information is unavailable.

### 6. Activate
- Run the ingestion workflow manually first to index your documents
- Enable the **Schedule Trigger** for automated periodic ingestion
- Test the chat via n8n's built-in chat interface or connect to your preferred frontend

---

## 📌 Pinecone Index Configuration

| Setting | Value |
|---------|-------|
| Dimensions | `1536` (text-embedding-ada-002) |
| Metric | `cosine` |
| Cloud | `aws` or `gcp` (your choice) |
| Namespaces | Auto-created per file name |

---

## 📌 Important Notes

- All PDFs must be accessible in the monitored Google Drive folder
- The deduplication logic uses Google Sheets to track `file NAME + ID` combinations
- Pinecone namespace per file enables targeted retrieval and easy cleanup
- All credentials are stored in n8n credential store, never in the workflow file
- The AI Agent is configured to never hallucinate — it only answers from retrieved context

---

## 👤 Author

**Mustapha Taleb** — AI Automation Engineer  
📍 Agadir, Morocco  
🔗 [LinkedIn](https://www.linkedin.com/in/talebmustapha/) | [GitHub](https://github.com/mustaphatb1975)

---

## 📄 License

MIT License — Free to use and modify.
