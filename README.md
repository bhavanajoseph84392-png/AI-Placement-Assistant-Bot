🎯 Problem Statement

Placement information is often distributed across multiple PDF documents such as:

Placement notifications
Job descriptions
Eligibility criteria
Company information
Interview instructions
Exam details
Salary information
Selection processes

Searching through these documents manually can be time-consuming.

💡 Solution

The AI Placement Assistant Bot allows users to upload these documents and ask questions in natural language.

Instead of manually searching through PDFs, the application:

Upload PDF → Process Document → Retrieve Relevant Information → Generate Answer

🚀 Features
Feature	Description
📄 PDF Upload	Upload one or multiple placement PDFs
✂️ Text Chunking	Split documents into smaller chunks
🔢 Embeddings	Convert text chunks into numerical vectors
🗂️ FAISS	Store and search document embeddings
🔎 Similarity Search	Retrieve relevant document chunks
🤖 LLM	Generate answers using Hugging Face
💬 Chat Interface	Interactive Streamlit interface
📑 Sources	Display source documents and page numbers
🔐 Environment Variables	Secure API credential management
🔄 Retry Handling	Retry temporary LLM API failures
🏗️ System Architecture
🔄 Overall Architecture
🧠 RAG Workflow

The application has two major phases.

Phase 1 — Document Ingestion
┌─────────────────────────┐
│     Upload PDF Files    │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│      PyPDFLoader        │
│     Extract PDF Text    │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│      Text Chunking      │
│   chunk_size = 500      │
│   overlap = 100         │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ HuggingFace Embeddings  │
│  all-MiniLM-L6-v2       │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│    FAISS Vector Store   │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│   Similarity Retriever  │
│        Top 4            │
└─────────────────────────┘
Phase 2 — Question Answering
┌─────────────────────────┐
│      User Question      │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│    Similarity Search    │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│   Top 4 Relevant Chunks │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│   Build Document        │
│       Context           │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│    Prompt Construction  │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ Hugging Face            │
│ InferenceClient         │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│          LLM            │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│     Generated Answer    │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│   Streamlit Chat UI     │
│   + Retrieved Sources   │
└─────────────────────────┘
🧩 How the Application Works
1️⃣ Upload Placement Documents

The user uploads one or more placement-related PDF documents through the Streamlit interface.

User
 ↓
Upload PDF Files
 ↓
Streamlit
2️⃣ Extract Text from PDFs

The uploaded PDFs are processed using:

PyPDFLoader

It loads the PDF pages and extracts their text.

The application also maintains metadata such as:

Source filename
Page number
Flow
PDF
 ↓
PyPDFLoader
 ↓
Pages + Text + Metadata
3️⃣ Split Documents into Chunks

Large documents are divided into smaller pieces using:

RecursiveCharacterTextSplitter
Configuration
chunk_size   = 500
chunk_overlap = 100

The overlap helps maintain contextual continuity between neighboring chunks.

Document
    ↓
Extracted Text
    ↓
┌────────────┐
│  Chunk 1   │
└────────────┘

┌────────────┐
│  Chunk 2   │
└────────────┘

┌────────────┐
│  Chunk 3   │
└────────────┘
4️⃣ Generate Embeddings

Each text chunk is converted into a numerical vector using:

HuggingFaceEmbeddings
Embedding Model
sentence-transformers/all-MiniLM-L6-v2
Flow
Text Chunk
    ↓
Embedding Model
    ↓
Numerical Vector

These vectors allow the application to compare the semantic similarity between the user's question and document chunks.

5️⃣ Store Embeddings in FAISS

The generated embeddings are stored in:

FAISS Vector Store

The application creates a similarity-based retriever using:

k = 4

Therefore, the retriever attempts to return the top 4 relevant chunks.

Document Chunks
      ↓
Embeddings
      ↓
FAISS
      ↓
Similarity Retriever
6️⃣ User Asks a Question

After creating the knowledge base, the user can ask a question.

Example
What are the eligibility criteria for this placement drive?
User Question
      ↓
Retriever
7️⃣ Retrieve Relevant Information

The retriever performs similarity search against the FAISS vector store.

User Question
      ↓
Similarity Search
      ↓
Top 4 Relevant Chunks

The application also maintains source information for the retrieved chunks.

Source Filename
       +
Page Number
8️⃣ Build the Context

The retrieved chunks are combined into a document context.

Example:

Source: placement.pdf
Page: 3

[Relevant document content]


Source: placement.pdf
Page: 7

[Relevant document content]

Then:

Retrieved Chunks
       +
User Question
       ↓
Document Context
9️⃣ Generate the Answer

The application uses:

Hugging Face InferenceClient

to communicate with the hosted LLM.

Default Model
openai/gpt-oss-120b
Configuration
Temperature        : 0.2
Maximum response  : 350 tokens
Retry attempts     : 3

The model name can be configured using:

MODEL_NAME
🔟 Grounded Answer Generation

The system prompt instructs the LLM to:

Answer only from the supplied document context
Avoid outside knowledge
Give concise answers
Respond in a beginner-friendly manner

If the required information is not available in the retrieved context, the assistant is instructed to respond:

I don't know based on the uploaded documents.
Grounding Flow
Retrieved Context
       ↓
System Instructions
       ↓
User Question
       ↓
LLM
       ↓
Grounded Answer
1️⃣1️⃣ Display Answer and Sources

The generated answer is displayed in the Streamlit chat interface.

The application also displays retrieved source information.

Retrieved Context
       ↓
      LLM
       ↓
Generated Answer
       ↓
Streamlit Chat
       ↓
Retrieved Sources
🔄 Complete End-to-End Flow
┌───────────────────────┐
│         USER          │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│    Upload PDF Files   │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│     PyPDFLoader       │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│    Extract Text       │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│      Chunking         │
│   500 / 100 overlap   │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ HuggingFace Embedding │
│   all-MiniLM-L6-v2    │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│    FAISS Vector DB    │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ Similarity Retriever  │
│       Top 4           │
└───────────┬───────────┘
            │
            │
            │     ┌──────────────────┐
            └────►│  User Question   │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Relevant Chunks  │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Context Building │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Prompt Creation  │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Hugging Face     │
                  │ InferenceClient  │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │       LLM        │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Generated Answer │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Streamlit Chat   │
                  │  + Sources       │
                  └──────────────────┘
🛠️ Technology Stack
Technology	Purpose
🐍 Python	Application development
🎈 Streamlit	Web interface and chat UI
🔗 LangChain	Document processing and retrieval components
📄 PyPDFLoader	PDF text extraction
✂️ RecursiveCharacterTextSplitter	Text chunking
🧠 HuggingFaceEmbeddings	Generate text embeddings
🔢 all-MiniLM-L6-v2	Embedding model
🗂️ FAISS	Vector storage and similarity search
🤖 Hugging Face InferenceClient	LLM API communication
🔐 python-dotenv	Environment variable management
📁 Project Structure
AI-Placement-Assistant-Bot/
│
├── 📄 app.py
├── 📦 requirements.txt
├── 🔒 .gitignore
└── 📖 README.md
📄 app.py

The main application contains:

Streamlit Interface
Environment Configuration
Hugging Face Client
Embedding Model
PDF Processing
Text Chunking
FAISS Vector Store
Retriever
RAG Pipeline
LLM Response Generation
Chat History
Source Display
📦 requirements.txt

Contains all Python dependencies required to run the project.

Main dependencies include:

streamlit
langchain
langchain-community
langchain-huggingface
sentence-transformers
huggingface-hub
faiss-cpu
pypdf
python-dotenv
⚙️ Installation
1. Clone the Repository
git clone https://github.com/bhavanajoseph84392-png/AI-Placement-Assistant-Bot.git
2. Open the Project
cd AI-Placement-Assistant-Bot
3. Create Virtual Environment
python -m venv venv
4. Activate Virtual Environment
Windows
venv\Scripts\activate
5. Install Dependencies
pip install -r requirements.txt
🔑 Environment Configuration

Create a .env file in the project directory.

HF_TOKEN=your_huggingface_token
MODEL_NAME=openai/gpt-oss-120b

The application reads the Hugging Face token from the environment or Streamlit secrets.

⚠️ Important

Never commit your actual API token to GitHub.

Keep your .env file inside .gitignore.

▶️ Run the Application

Run:

streamlit run app.py

The application will open in your browser.
