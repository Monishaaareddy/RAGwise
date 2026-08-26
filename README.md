# PDF RAG Chatbot

A Retrieval-Augmented Generation (RAG) chatbot that answers questions grounded in a PDF document. The project runs locally and does not require external API keys.

## Overview

The pipeline follows these stages:

1. PDF loading and text extraction
2. Text cleaning
3. Text chunking
4. Embedding generation
5. Vector storage using ChromaDB
6. Semantic retrieval
7. Prompt construction
8. Answer generation using Llama 3.2:3B
9. Interactive Q&A chatbot
10. Optional evaluation

The complete implementation is provided as a Jupyter Notebook and should be run from top to bottom. The notebook uses page-aware chunks so that generated answers can cite the source page. fileciteturn0file0L8-L13

## Tech Stack

| Component | Technology |
|---|---|
| Language | Python |
| Environment | Jupyter Notebook |
| PDF extraction | pdfplumber |
| Text splitting | LangChain RecursiveCharacterTextSplitter |
| Embeddings | Sentence Transformers — `all-MiniLM-L6-v2` |
| Vector database | ChromaDB |
| Local LLM | Ollama — `llama3.2:3b` |
| Retrieval | ChromaDB semantic search |

## Architecture

```text
                PDF Document
                     |
                     v
              Text Extraction
                     |
                     v
                Text Cleaning
                     |
                     v
                  Chunking
                     |
                     v
                 Embeddings
                     |
                     v
                ChromaDB
              Vector Storage
                     |
          +----------+----------+
          |                     |
          v                     |
     User Question              |
          |                     |
          v                     |
       Embedding                |
          |                     |
          v                     |
    Similarity Search <----------+
          |
          v
   Relevant PDF Chunks
          |
          v
   Prompt Construction
          |
          v
     Llama 3.2:3B
          |
          v
      Final Answer
```

## Setup

### 1. Install Ollama

Install Ollama from:

https://ollama.com

Then download the local Llama model:

```bash
ollama pull llama3.2:3b
```

The notebook specifically uses `llama3.2:3b` as its generation model. fileciteturn0file0L25-L28

### 2. Install Python dependencies

Run:

```bash
pip install pdfplumber sentence-transformers chromadb ollama langchain-text-splitters
```

### 3. Configure the PDF

Update the PDF path in the notebook:

```python
PDF_PATH = "your_document.pdf"
```

The main configuration also includes chunk size, chunk overlap, embedding model, ChromaDB location, retrieval parameters, and the LLM model. fileciteturn0file0L67-L89

## How RAG Works in This Project

### 1. Extract

`pdfplumber` extracts text from every PDF page while retaining the page number.

### 2. Clean

The extracted text is cleaned by:

- Removing hyphenated line breaks
- Normalizing whitespace
- Reducing excessive blank lines

### 3. Chunk

The document is split into smaller pieces using `RecursiveCharacterTextSplitter`.

Current configuration:

```text
Chunk size: 500
Chunk overlap: 50
```

Each chunk stores:

- Chunk ID
- Text
- Page number

### 4. Embed

Each chunk is converted into a numerical vector using:

```text
all-MiniLM-L6-v2
```

These embeddings allow the system to perform semantic similarity search.

### 5. Store

The embeddings and document chunks are stored in a persistent local ChromaDB collection.

```python
client = chromadb.PersistentClient(path=DB_PATH)
```

### 6. Retrieve

When a user asks a question:

1. The question is converted into an embedding.
2. ChromaDB searches for the most relevant chunks.
3. The top 4 chunks are retrieved by default.
4. A similarity threshold is used to determine whether the retrieved information is relevant enough.

The current configuration uses:

```text
TOP_K = 4
MIN_SIMILARITY_THRESHOLD = 0.25
```

### 7. Generate

The retrieved chunks are inserted into a prompt and sent to:

```text
Llama 3.2:3B
```

The prompt instructs the model to answer using only the retrieved context and to say that it does not know when the answer is not contained in the document. It also asks the model to cite page numbers. fileciteturn0file0L298-L315

## Chatbot

The project includes an interactive Q&A loop:

```text
You: <question>

Assistant: <answer>
```

The user can continue asking questions until entering:

```text
exit
```

or:

```text
quit
```

This interactive loop is what turns the underlying RAG pipeline into a simple chatbot. fileciteturn0file0L338-L350

## Evaluation

The notebook includes an optional evaluation section using a `GOLD_SET` containing questions and expected keywords.

The evaluation reports:

```text
PASS / FAIL
Accuracy
```

The provided test cases cover:

- Basic factual retrieval
- Nuance and grounding
- Cross-document/plant discrimination
- Paraphrased questions
- Out-of-scope questions and hallucination prevention

The out-of-scope tests are designed to verify that the chatbot refuses to invent answers when information is not present in the document. fileciteturn0file0L388-L431

## Example Questions

The provided evaluation examples include:

```text
How often should I water a Snake Plant?
Is the Pothos toxic to pets?
How often should houseplants typically be repotted?
What causes yellow leaves on houseplants?
Which plant tolerates low light best?
Compare the light needs of a Snake Plant and a Fiddle Leaf Fig
```

The test set also includes deliberately out-of-scope questions such as:

```text
What's the capital of Japan?
How do I care for a bonsai tree?
What's the best fertilizer brand to buy?
```

For unsupported questions, the expected behavior is to avoid hallucinating an answer. fileciteturn0file0L390-L427

## API Keys

This project does **not require API keys**.

The components run locally:

| Typical Cloud Alternative | This Project |
|---|---|
| OpenAI/Cohere embeddings | Sentence Transformers |
| OpenAI/Anthropic/other hosted LLM | Ollama + Llama 3.2:3B |
| Pinecone/Weaviate Cloud | Local ChromaDB |

Once the required models and packages are installed, the pipeline can run locally without an `OPENAI_API_KEY` or `.env` file. fileciteturn0file0L437-L450

## Project Structure

Recommended GitHub structure:

```text
pdf-rag-chatbot/
│
├── PDF_RAG_Chatbot.ipynb
├── README.md
├── .gitignore
└── your_document.pdf
```

If the PDF contains sensitive or proprietary information, do not commit it to a public repository.

The ChromaDB directory and Python virtual environment should also be excluded from Git using `.gitignore`. fileciteturn0file0L517-L520

## Running the Project

1. Install Ollama.
2. Pull `llama3.2:3b`.
3. Install the Python dependencies.
4. Place your PDF in the project directory.
5. Update `PDF_PATH`.
6. Open the Jupyter Notebook.
7. Run the cells from top to bottom.
8. Ask questions through the interactive chatbot.
9. Type `exit` or `quit` to stop.

## Limitations

The current implementation is a **stateless chatbot**. Each question is processed independently; previous questions are not retained as conversation memory. fileciteturn0file0L495-L497

Other practical limitations include:

- Retrieval quality depends on chunking and embedding quality.
- A small local LLM may produce lower-quality answers than larger hosted models.
- Scanned/image-only PDFs may require OCR before text extraction.
- The similarity threshold may need tuning for different documents.
- The current evaluation is keyword-based rather than a comprehensive RAG evaluation framework.

## Future Improvements

Possible extensions include:

- Conversation memory
- Hybrid search
- Reranking
- Better citation handling
- OCR support for scanned PDFs
- More robust RAG evaluation
- Streaming responses
- Web or UI interface
- Larger local or hosted LLMs
- Metadata filtering
- Document upload through a frontend

## Learning Outcomes

This project demonstrates an end-to-end RAG workflow and provides hands-on experience with:

- LLMs
- Embeddings
- Vector databases
- Semantic search
- Prompt engineering
- Retrieval-Augmented Generation
- Hallucination control
- Local model deployment
- Basic RAG evaluation

## License

This project is intended for learning and demonstration purposes.
