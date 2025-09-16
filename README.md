# AI RAG Assistant Using LangChain

A simple PDF Question-Answering app that uses Retrieval Augmented Generation (RAG) with IBM watsonx.ai for LLM and embeddings, LangChain for orchestration, and Chroma as the vector store. A Gradio UI lets you upload a PDF and ask questions answered from the document.

## Features
- Upload a single PDF and ask natural language questions
- Automatic chunking and embedding of document text
- Retrieval with Chroma vector store
- Generation via IBM watsonx.ai Mixtral instruct model
- Clean, single-file app (`qabot.py`)

## Architecture
- Loader: `PyPDFLoader` extracts text from your PDF
- Splitter: `RecursiveCharacterTextSplitter` creates 100-char chunks (20 overlap)
- Embeddings: `WatsonxEmbeddings` (Mixtral embedding model)
- Vector DB: `Chroma` from `langchain_community`
- LLM: `WatsonxLLM` (Mixtral 8x7B Instruct)
- Chain: `RetrievalQA` (stuff) combines retriever + LLM
- UI: `gr.Interface` to upload a file and enter a query

## Prerequisites
- Windows 10/11 with PowerShell
- Python 3.10 or 3.11 recommended
- IBM Cloud account with watsonx.ai access and a project ID
- watsonx.ai credentials configured (see below)

## Setup (Windows PowerShell)
1. Clone or download this repo, then open it in VS Code or a terminal.
2. Create and activate a virtual environment:
   ```powershell
   py -3.11 -m venv .venv; .\.venv\Scripts\Activate.ps1
   ```
3. Install dependencies:
   ```powershell
   pip install -r requirements.txt
   ```

## Configure IBM watsonx.ai
The LangChain IBM integrations (`langchain_ibm`) use the IBM Watsonx SDK credentials under the hood. Set the following environment variables in your PowerShell session before running the app:

```powershell
# Your IBM Cloud API Key
$env:IBM_CLOUD_API_KEY = "<your_api_key>"

# watsonx.ai region URL (example below uses Dallas US-South)
$env:WATSONX_URL = "https://us-south.ml.cloud.ibm.com"

# Project ID that has access to the foundation models
$env:WATSONX_PROJECT_ID = "<your_project_id>"
```

The code defaults to `url="https://us-south.ml.cloud.ibm.com"` and `project_id="skills-network"`. You can either:
- Set your own env vars and then adapt `qabot.py` to read them (recommended; see Configuration), or
- Replace the hardcoded values in `qabot.py` with your actual URL and project ID.

## Run the app
```powershell
python .\qabot.py
```
Then open http://localhost:7860 in your browser. Upload a PDF and ask a question.

## Configuration
Key knobs you can change inside `qabot.py`:
- LLM model: `model_id = 'mistralai/mixtral-8x7b-instruct-v01'`
- LLM params: temperature and max new tokens in `get_llm`
- Splitter: `chunk_size=100`, `chunk_overlap=20`
- Embedding model: `'mistralai/mixtral-8x7b-embedding-v01'`
- Server port: `rag_application.launch(server_port=7860)`

Environment variable support (optional edit): you can modify `get_llm()` and `watsonx_embedding()` to read values from env:
```python
import os
url = os.getenv("WATSONX_URL", "https://us-south.ml.cloud.ibm.com")
project_id = os.getenv("WATSONX_PROJECT_ID", "skills-network")
```

## Troubleshooting
- Module not found errors: ensure the venv is active and `pip install -r requirements.txt` completed successfully.
- IBM auth/403 errors: double-check `$env:IBM_CLOUD_API_KEY`, region URL, and Project ID have access to the selected models.
- PDF parse errors: try a simpler PDF or ensure the file isn’t encrypted; `pypdf` is used under the hood.
- TypeError: tuple instead of dict for embed params:
  - Resolved in this repo by fixing `embed_params` to be a dict (no trailing comma).
- Gradio file path issues:
  - The app uses `type="filepath"` and handles both string and file-object inputs.

## Project structure
```
AI-RAG-Assistant-Using-LangChain/
├─ qabot.py          # Main app
├─ README.md         # This file
├─ requirements.txt  # Python dependencies
```

## Next steps
- Cache the vector store to disk to avoid re-embedding on every question
- Add support for multiple PDFs and document selection
- Expose model and splitter settings in the UI
- Show source snippets used to answer

