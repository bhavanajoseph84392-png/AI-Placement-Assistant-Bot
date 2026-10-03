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
                     DOCUMENT INGESTION
                        │
                        ▼
              ┌──────────────────┐
              │   Upload PDFs    │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │   PyPDFLoader    │
              │ Extract PDF text │
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────────┐
              │ Text Chunking        │
              │ chunk_size = 500     │
              │ overlap = 100        │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ HuggingFace         │
              │ Embedding Model     │
              │ all-MiniLM-L6-v2    │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │    FAISS Vector DB   │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ Similarity Retriever │
              │ Top 4 chunks         │
              └──────────┬───────────┘
                         │
                         │
              USER QUESTION
                         │
                         ▼
              ┌──────────────────────┐
              │ Retrieve Relevant    │
              │ Document Chunks      │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ Build Context        │
              │ + User Question      │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ Hugging Face         │
              │ InferenceClient      │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │       LLM            │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ Generated Answer     │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │ Streamlit Chat UI    │
              └──────────────────────┘
              🧠 How the Application Works
1. Upload Placement Documents

The user uploads one or more PDF files through the Streamlit interface.

The application accepts multiple PDF files dynamically.

User
  ↓
Upload PDF files
  ↓
Streamlit

The application checks whether PDF files are available before creating the knowledge base.

2. Extract Text from PDFs

The uploaded PDF files are temporarily stored and processed using:

PyPDFLoader

PyPDFLoader loads the PDF pages and extracts their text.

Each page also keeps metadata such as the source filename and page number.

PDF
 ↓
PyPDFLoader
 ↓
Pages + Text + Metadata
3. Split the Documents into Chunks

Large documents are divided into smaller chunks using:

RecursiveCharacterTextSplitter

The project uses:

chunk_size = 500
chunk_overlap = 100

The overlap helps maintain some contextual continuity between neighboring chunks.

Document
     ↓
Text
     ↓
Chunk 1
Chunk 2
Chunk 3
...
4. Generate Embeddings

Each text chunk is converted into a numerical vector using:

HuggingFaceEmbeddings

The embedding model used is:

sentence-transformers/all-MiniLM-L6-v2

The embeddings are normalized before being used by the vector store.

Conceptually:

Text Chunk
    ↓
Embedding Model
    ↓
Numerical Vector

These vectors allow the system to compare the semantic similarity between a user's question and document chunks.

5. Store Embeddings in FAISS

The generated embeddings are stored in:

FAISS

The application creates the vector store from the document chunks and embeddings.

It then creates a similarity-based retriever with:

k = 4

This means the retriever attempts to return the 4 most relevant chunks for a user's question.

Document Chunks
      ↓
Embeddings
      ↓
FAISS Vector Store
      ↓
Similarity Retriever
6. User Asks a Question

After the knowledge base is created, the user can enter a question in the Streamlit chat interface.

Example:

"What are the eligibility criteria for the placement drive?"

The question is passed to the retrieval function.

User Question
      ↓
Retriever
7. Retrieve Relevant Information

The retriever searches the FAISS vector store and returns the most relevant document chunks.

The application retrieves up to 4 relevant chunks using similarity search.

User Question
      ↓
Similarity Search
      ↓
Top 4 Relevant Chunks

The application also keeps the source filename and page number for each retrieved chunk.

8. Build the Context

The retrieved chunks are combined into a context that is passed to the LLM.

The generated context contains information such as:

Source 1: placement.pdf, page 3

[relevant document content]

Source 2: placement.pdf, page 7

[relevant document content]

This allows the application to tell the model where the retrieved information came from.

9. Generate the Answer

The retrieved document context and the user's question are placed into a prompt.

The application uses:

Hugging Face InferenceClient

to communicate with the hosted LLM.

The model name can be configured through the environment variable:

MODEL_NAME

The default configured model is:

openai/gpt-oss-120b

The application uses a low temperature of:

0.2

and a maximum response length of:

350 tokens

The application also retries the LLM request up to three times if a temporary API error occurs.

10. Grounded Answer Generation

The system prompt instructs the model to:

Answer only from the supplied document context
Avoid outside knowledge
Give concise and meaningful answers
Respond in a beginner-friendly manner
Say:
I don't know based on the uploaded documents.

when the answer is not available in the retrieved context.

This helps keep the generated response grounded in the uploaded placement documents.

11. Display the Answer

The generated response is added to the chat history and displayed through the Streamlit chat interface.

The application also displays the retrieved source filenames and page numbers.

Retrieved Context
       ↓
      LLM
       ↓
Generated Answer
       ↓
Streamlit Chat
       ↓
Retrieved Sources
🧩 Complete End-to-End Flow
User
 │
 │ Uploads placement PDFs
 ▼
Streamlit Interface
 │
 ▼
PyPDFLoader
 │
 │ Extract PDF pages
 ▼
Document Text
 │
 ▼
RecursiveCharacterTextSplitter
 │
 │ chunk_size = 500
 │ overlap = 100
 ▼
Text Chunks
 │
 ▼
HuggingFace Embeddings
 │
 │ all-MiniLM-L6-v2
 ▼
Vector Embeddings
 │
 ▼
FAISS Vector Store
 │
 ▼
Similarity Retriever
 │
 │
 │ User asks question
 │
 ▼
Question Embedding / Similarity Search
 │
 ▼
Top 4 Relevant Chunks
 │
 ▼
Context Construction
 │
 ▼
Prompt
 │
 ▼
Hugging Face InferenceClient
 │
 ▼
LLM
 │
 ▼
Generated Answer
 │
 ├──────────────► Chat Response
 │
 └──────────────► Retrieved Sources
🛠️ Technology Stack
Technology	Purpose
Python	Application development
Streamlit	Web interface
LangChain	Document processing and retrieval components
PyPDFLoader	PDF text extraction
RecursiveCharacterTextSplitter	Document chunking
HuggingFace Embeddings	Text embeddings
all-MiniLM-L6-v2	Embedding model
FAISS	Vector storage and similarity search
Hugging Face InferenceClient	LLM API communication
python-dotenv	Environment variable management

The project's requirements.txt includes Streamlit, LangChain, LangChain Community, LangChain HuggingFace, sentence-transformers, Hugging Face Hub, FAISS CPU, PyPDF and python-dotenv dependencies.

📁 Project Structure
AI-Placement-Assistant-Bot/
│
├── app.py
├── requirements.txt
├── .gitignore
└── README.md
app.py

Main application containing:

Streamlit interface
Environment configuration
Hugging Face client
Embedding model
PDF processing
Text chunking
FAISS vector store
Retriever
RAG pipeline
LLM response generation
Chat history
Source display
requirements.txt

Contains the Python dependencies required to run the application.

⚙️ Installation
1. Clone the repository
git clone https://github.com/bhavanajoseph84392-png/AI-Placement-Assistant-Bot.git
cd AI-Placement-Assistant-Bot
2. Create a virtual environment
python -m venv venv

Activate it on Windows:

venv\Scripts\activate
3. Install dependencies
pip install -r requirements.txt
🔑 Environment Configuration

Create a .env file in the project directory.

HF_TOKEN=your_huggingface_token
MODEL_NAME=openai/gpt-oss-120b

The application reads the Hugging Face token from the environment or Streamlit secrets.

Never commit your actual API token to GitHub.

▶️ Run the Application

Start the Streamlit application using:

streamlit run app.py

The application will open in your browser.
