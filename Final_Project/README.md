# AI-Powered Study Assistant (RAG System)

A Retrieval-Augmented Generation (RAG) system that lets students upload their own study
materials (PDF or TXT) and ask questions about them in plain English. The assistant
retrieves the most relevant parts of the material and uses an open-source language model
to generate an answer grounded in that content, instead of relying on general knowledge.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Key Features](#key-features)
- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Setup and Usage](#setup-and-usage)
- [Evaluation](#evaluation)
- [Limitations](#limitations)
- [Future Improvements](#future-improvements)

---

## Project Overview

Traditional chatbots answer questions using only what they learned during training, which
means they can give outdated or made-up answers, especially about material they were never
trained on (like a student's personal lecture notes). This project solves that problem
using **Retrieval-Augmented Generation (RAG)**.

Instead of answering from memory, the assistant:
1. Reads the student's uploaded notes.
2. Breaks the notes into small, searchable chunks.
3. Finds the chunks most relevant to the question asked.
4. Feeds those chunks to a language model along with the question, so the answer is based
   only on the actual uploaded material.

This makes the assistant reliable for studying, since answers are traceable back to the
source document instead of being guessed.

---

## Key Features

- **Document upload**: Supports PDF and TXT files, with support for multiple files at once.
- **Smart text chunking**: Splits documents into overlapping chunks so context isn't lost
  at chunk boundaries.
- **Semantic search with FAISS**: Uses sentence embeddings (`BAAI/bge-base-en-v1.5`) and a
  FAISS vector index to find the most relevant chunks for any question.
- **Cross-encoder re-ranking**: A second-stage re-ranker improves the precision of retrieved
  chunks before they are sent to the language model.
- **Grounded answer generation**: Uses `microsoft/Phi-3-mini-4k-instruct` (4-bit quantized)
  to generate answers strictly from the retrieved context.
- **Refusal on out-of-scope questions**: If the answer isn't in the uploaded material, the
  assistant says so instead of making something up.
- **Persistent vector store**: The FAISS index is saved to disk so it doesn't need to be
  rebuilt every time.
- **Built-in evaluation**: Automatically tests the system on sample questions and reports
  retrieval accuracy, groundedness, and refusal accuracy.

---

## How It Works

```
Upload Document (PDF/TXT)
        |
        v
  Extract & Clean Text
        |
        v
   Split into Chunks
        |
        v
  Generate Embeddings  --->  Store in FAISS Index
        |
        v
   User Asks a Question
        |
        v
  Retrieve Top Matching Chunks (FAISS + Re-ranker)
        |
        v
  Build Prompt (Question + Retrieved Context)
        |
        v
  Language Model Generates Answer
        |
        v
     Answer + Sources
```

---

## Tech Stack

| Component            | Tool / Model                              |
|-----------------------|--------------------------------------------|
| Language              | Python (Google Colab, T4 GPU)              |
| PDF/Text parsing      | `pypdf`                                    |
| Text chunking         | `langchain-text-splitters`                 |
| Embedding model       | `BAAI/bge-base-en-v1.5` (Sentence-Transformers) |
| Vector search          | FAISS (`IndexFlatIP`)                      |
| Re-ranking model      | `cross-encoder/ms-marco-MiniLM-L-6-v2`     |
| Language model (LLM)  | `microsoft/Phi-3-mini-4k-instruct` (4-bit) |
| Model loading          | Hugging Face `transformers`, `bitsandbytes` |

---

## Project Structure

```
├── Study_Assistant_RAG.ipynb   # Main notebook: full pipeline (upload -> answer)
└── README.md                   # Project documentation (this file)
```

The notebook is organized into clear steps, each with its own markdown explanation:

1. Install dependencies
2. Import libraries and check GPU
3. Upload study materials
4. Extract text from documents
5. Chunk the text
6. Generate embeddings and build the FAISS index
7. Retrieve relevant chunks for a question
8. Load the language model
9. Build the prompt and generate answers
10. Ask a question (demo)
11. Interactive Q&A loop (optional)
12. Evaluate the system

---

## Setup and Usage

1. Open `Study_Assistant_RAG.ipynb` in **Google Colab**.
2. Go to `Runtime > Change runtime type` and select **T4 GPU**.
3. Run the cells from top to bottom.
4. When prompted, upload one or more PDF or TXT files (your study notes).
5. Once the index and model are loaded, run the "Ask a Question" cell and type your question.
6. Optionally, run the interactive Q&A cell to keep asking questions in a loop.

No API key is required — all models used are open-source and run locally within the Colab
session.

---

## Evaluation

The notebook includes a built-in evaluation step that tests the assistant on two types of
questions:

- **In-scope questions** (factual, summarization, synthesis, and detail-based) — scored on:
  - **Retrieval similarity**: how relevant the retrieved chunks are to the question.
  - **Lexical groundedness**: how much of the answer's wording overlaps with the source text.
  - **Semantic groundedness**: how close the answer's meaning is to the source text, even if
    the wording is different (this fairly rewards paraphrasing).
  - **Latency**: how long the assistant takes to respond.

- **Refusal check** (out-of-scope and trick questions) — checks whether the assistant
  correctly says it cannot find the answer, instead of making one up. This is important
  because it shows the assistant is not hallucinating when material isn't available.

---

## Limitations

- Answer quality depends on how well the uploaded document is structured; scanned PDFs
  with poor text layers may extract text incorrectly.
- The language model (Phi-3-mini) is small enough to run on a free T4 GPU, so answer
  quality is not as strong as larger commercial models.
- Response time (around 5–20 seconds per answer) may feel slow for very short questions.
- The system works best on one topic/document set at a time; mixing unrelated documents
  can reduce retrieval accuracy.

---

## Future Improvements

- Semantic chunking (splitting text based on meaning instead of fixed character length).
- Support for voice input and voice output.
- A simple web interface instead of running everything inside a notebook.
- Option to switch to a larger language model if more GPU memory is available.
- Tracking which topics a student asks about most, to suggest review material.
