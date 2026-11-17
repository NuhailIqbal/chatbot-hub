# ChatBot Hub — RAG Knowledge Chatbot (n8n)

A RAG-based support chatbot built in n8n: a **retrieval pipeline** that answers customer questions from a Pinecone vector store, and an **ingestion pipeline** that loads a knowledge base into that store.

Built by [Nuhail Iqbal](https://www.linkedin.com/in/nuhail-iqbal-a93b1935b/) — AI Agent & Automation Engineer (n8n · VAPI · RAG · WhatsApp Chatbots · Python).

## How it works

### 1. Retrieval pipeline — answering questions

```
Webhook (POST) → Edit Fields → AI Agent → Respond to Webhook
```

- **Webhook** receives a POST with `{ "text": "<customer question>" }`.
- **Edit Fields** maps the incoming text to a `Question` field.
- **AI Agent** (LangChain agent) answers using only the vector store. System prompt: professional, on-topic, and it says plainly when an answer is not in the knowledge base instead of guessing.
- **OpenRouter Chat Model** is the LLM behind the agent.
- **Simple Memory** keeps a 10-message buffer window, so follow-up questions keep context.
- **Pinecone Vector Store** is attached as a tool (`retrieve-as-tool`, index `chatbotdata`) — the agent fetches relevant chunks before answering.
- **Embeddings — HuggingFace Inference** (`sentence-transformers/distiluse-base-multilingual-cased-v2`) embeds the query for retrieval.

### 2. Ingestion pipeline — loading the knowledge base

```
Manual Trigger → Edit Fields → Pinecone Vector Store (insert)
```

- **Edit Fields** holds the knowledge-base text (see `docs/sample-knowledge-base.md`).
- **Default Data Loader** splits it into documents.
- **Embeddings — HuggingFace Inference** embeds each chunk.
- **Pinecone Vector Store** (insert mode, same `chatbotdata` index) stores the vectors.

Run ingestion once (or whenever the knowledge base changes), then the retrieval pipeline can answer from it.

## Try it

### Prerequisites

- n8n (self-hosted or cloud)
- A Pinecone account with an index named `chatbotdata`
- An OpenRouter API key
- A Hugging Face API token (inference API)

### Setup

1. Import `workflows/ChatBot_Hub.json` into n8n (Workflows → Import from file).
2. Create credentials in n8n and connect them to the matching nodes:
   - OpenRouter account → **OpenRouter Chat Model**
   - Pinecone account → both **Pinecone Vector Store** nodes
   - Hugging Face account → both **Embeddings** nodes
3. Run the ingestion branch once (click **Execute workflow** on the manual trigger) to load the sample knowledge base into Pinecone.
4. Activate the workflow and send a question:

```bash
curl -X POST "https://YOUR-N8N-HOST/webhook/chatbot-hub" \
  -H "Content-Type: application/json" \
  -d '{"text": "What are the support hours?"}'
```

Example answer: live chat runs Monday to Friday, 9:00–18:00 PKT; email is answered within one business day.

## What this demonstrates

- RAG in production form: retrieval as an agent tool, not a blind context dump
- Grounded answers — the agent is instructed to stay on-topic and admit when an answer is not in the knowledge base
- Conversation memory with a bounded 10-message window
- A clean split between ingestion and retrieval, so the knowledge base can be refreshed without touching the answering flow

## Files

| File | What it is |
| --- | --- |
| `workflows/ChatBot_Hub.json` | The importable n8n workflow (13 nodes, 2 pipelines) |
| `docs/sample-knowledge-base.md` | The sample knowledge base loaded by the ingestion pipeline |

## Notes

- The workflow ships with `active: false` and no credentials — add your own in n8n after import.
- The knowledge base in this repo is fictional sample data (Acme Helpdesk), written for the demo. Swap in your own docs to make it answer about your product.

## License

MIT — see [LICENSE](LICENSE).
