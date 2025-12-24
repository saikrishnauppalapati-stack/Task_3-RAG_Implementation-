# MCP Knowledge Base Agent (RAG + LangGraph)

An **end-to-end agentic Retrieval-Augmented Generation (RAG) system** built using **MCP (Model Context Protocol)**, **LangGraph**, and **Groq LLM**.

This project demonstrates how to:

* Build an **MCP Knowledge Base Server** with vector search (Chroma)
* Expose KB operations as **MCP tools**
* Create a **custom MCP client** (JSON-RPC over STDIO)
* Dynamically convert MCP tools into **LangChain tools** using **Pydantic schemas**
* Orchestrate reasoning + tool usage using **LangGraph**

---

## 🚀 What This Project Does

✔ Implements a **strict Knowledge-Base-only AI agent**
✔ Uses **RAG (Embeddings + Vector DB + Retrieval)**
✔ Prevents hallucinations by enforcing tool-only answers
✔ Dynamically discovers tools from the MCP server
✔ Maintains conversation memory using LangGraph checkpointing
✔ Fully local embeddings (no embedding API cost)

---

## 🧠 High-Level Architecture

```
User
 │
 ▼
LangGraph Agent (Groq LLM)
 │  - reasoning
 │  - tool selection
 │
 ▼
LangChain Tools (dynamic, Pydantic-based)
 │
 ▼
MCP Client (JSON-RPC over STDIO)
 │
 ▼
MCP Knowledge Base Server
 │  - Chroma Vector DB
 │  - HuggingFace Embeddings
 │  - KB Tools
```

---

## 📂 Project Structure

```
Mcp-server/
│
├── agent.py              # LangGraph agent + MCP client
├── mcp_kb_server.py      # MCP Knowledge Base server (RAG)
├── requirements.txt
├── .env                  # API keys (not committed)
├── README.md
```

---

## 🛠️ MCP Knowledge Base Tools

### 🔍 `search_knowledge_base`

Searches the vector database for relevant chunks.

**Input**

```json
{ "query": "expense reimbursement policy" }
```

---

### 📄 `list_documents`

Lists all available documents in the knowledge base.

---

### 📘 `read_document`

Reads the full content of a specific document.

**Input**

```json
{ "source": "finance_policy.txt" }
```

---

## 📚 Knowledge Base Design

* Documents are split using `RecursiveCharacterTextSplitter`
* Chunk size: `500` with overlap `50`
* Embeddings: `all-MiniLM-L6-v2` (local, fast, free)
* Vector Store: **Chroma (in-memory)**

This ensures **semantic search accuracy** and **fast retrieval**.

---

## ⚙️ Prerequisites

* Python **3.10+**
* Groq API Key

---

## 🔐 Environment Variables

Create a `.env` file:

```
GROQ_API_KEY=your_groq_api_key
```

---

## 📦 Installation

```bash
git clone https://github.com/your-username/mcp-kb-agent.git
cd mcp-kb-agent

python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

---

## ▶️ Running the Agent

```bash
python agent.py
```

What happens internally:

1. Agent starts the MCP KB server as a subprocess
2. Initializes MCP protocol
3. Builds the vector store & embeddings
4. Discovers tools dynamically
5. Compiles LangGraph workflow

---

## 💬 Example Interaction

```
You: What is the expense reimbursement deadline?
Agent: Expense reimbursements must be submitted by the 25th of each month.

You: List all available documents
Agent: Available Documents:
- benefits_guide.txt
- engineering_handbook.txt
- finance_policy.txt
- holiday_schedule.txt
- it_faq.txt
```

---

## 🧩 Dynamic Tool Generation (Key Feature)

MCP tool schemas are converted into **LangChain tools at runtime** using Pydantic:

```python
ArgsModel = create_model("Args", **fields)
return tool(args_schema=ArgsModel)(tool_func)
```

This enables:

* Strict argument validation
* Auto-generated tool schemas
* Zero hardcoding in the agent

---

## 🔁 LangGraph Workflow

* **agent node** → LLM reasoning
* **tools node** → MCP tool execution
* Conditional routing based on tool calls
* Memory persistence via `MemorySaver`

```python
workflow.add_node("agent", call_model)
workflow.add_node("tools", ToolNode(tools))
workflow.set_entry_point("agent")
```

---

## 🛑 Anti-Hallucination Design

The system prompt enforces:

* ❌ No internal knowledge
* ✅ Tool-only answers
* ❌ No guessing
* ✅ Explicit "not in database" responses

This makes the agent **enterprise-safe**.

---

## 🧪 Testing MCP Server

You can test `mcp_kb_server.py` using **MCP Inspector**:

* Transport: STDIO
* Server file: `mcp_kb_server.py`

---

## 🚧 Future Enhancements

* Persistent Chroma storage
* PDF / Directory loaders
* HTTP MCP transport
* Streaming responses
* Access control

