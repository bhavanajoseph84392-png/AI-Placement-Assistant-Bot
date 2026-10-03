# 📚 AI Placement Assistant Bot

An AI-powered Placement Assistant built with **Python, Streamlit, LangChain, Hugging Face and FAISS**.

The application allows users to upload one or more placement-related PDF documents, build a searchable knowledge base from those documents, and ask questions through a chat interface.

The assistant uses a **Retrieval-Augmented Generation (RAG)** pipeline to retrieve relevant information from the uploaded documents and generate answers using an LLM.

---

## 🚀 Features

- 📄 Upload one or multiple PDF files
- 🧠 Build a knowledge base from uploaded documents
- ✂️ Split PDF content into smaller text chunks
- 🔢 Generate vector embeddings for document chunks
- 🔎 Perform similarity-based document retrieval
- 🗂️ Store document vectors using FAISS
- 🤖 Generate answers using a Hugging Face hosted LLM
- 💬 Interactive Streamlit chat interface
- 📑 Display retrieved source documents and page numbers
- 🔐 Use environment variables for API credentials
- 🧹 Clear uploaded documents and chat session

---

# 🏗️ System Architecture

```mermaid
flowchart TD

    A[User] --> B[Streamlit Web Interface]

    B --> C{Upload PDF Files}

    C --> D[PyPDFLoader]

    D --> E[Extract Text from PDFs]

    E --> F[Recursive Character Text Splitter]

    F --> G[Text Chunks]

    G --> H[HuggingFace Embeddings]
    H --> I[all-MiniLM-L6-v2]

    I --> J[FAISS Vector Store]

    J --> K[Similarity Retriever]
    
    B --> L[User Question]

    L --> K

    K --> M[Retrieve Top 4 Relevant Chunks]

    M --> N[Build Document Context]

    N --> O[Prompt Construction]

    O --> P[Hugging Face InferenceClient]

    P --> Q[LLM]

    Q --> R[Generated Answer]

    R --> S[Streamlit Chat Interface]

    M --> T[Retrieved Sources]

    T --> S
