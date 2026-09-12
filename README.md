# NightDesk

**Source-checked study support for the night before an exam.**

NightDesk is a local-first study workspace that answers questions from a student's own course material and keeps the supporting evidence visible beside every response. Instead of guessing when evidence is missing, it can return `NOT_COVERED`, `PARTIALLY_ANSWERED`, or `LOW_QUALITY_SOURCE`.

## Why NightDesk?

Students often revise from a mix of PDFs, lecture slides, handwritten notes, Markdown files, and text documents. Searching across all of them manually is slow, especially during final revision.

NightDesk brings those sources into one workspace and provides:

- grounded question answering from uploaded study material
- exact source citations with page or slide locations
- visual PDF and PowerPoint evidence previews
- source and format filtering
- handwritten-note ingestion with extraction-quality awareness
- explicit refusal when the available material does not support an answer
- multi-part evidence completeness checks
- revision actions such as explain, compare, quiz, and exam sprint

## Current Demo Corpus

The local demo workspace contains **5 sources and 416 pages / slides / sections**, including:

- `DBMS.pdf`
- `SQL-SESSION1.pptx`
- two handwritten DBMS note images
- one Markdown notes file

The real study corpus is intentionally kept out of Git through `.gitignore`.

## Core Principle

> **Your course material is the authority. Missing evidence stays missing.**

NightDesk is designed to prefer an honest refusal over an unsupported answer.

Possible answer states include:

- `ANSWERED`
- `PARTIALLY_ANSWERED`
- `NOT_COVERED`
- `LOW_QUALITY_SOURCE`
- `ERROR`

## Example Demo Queries

Examples from the current DBMS/SQL workspace:

```text
What is DBMS?
```

Grounded in `SQL-SESSION1.pptx`, slide 2.

```text
What does the PRIMARY KEY clause specify?
```

Grounded in `SQL-SESSION1.pptx`, slide 24.

```text
What are the application areas of DBMS?
```

Grounded in `DBMS.pdf`.

```text
How does PostgreSQL MVCC vacuum freeze transaction IDs?
```

The current corpus does not sufficiently support this question, so NightDesk returns `NOT_COVERED` instead of inventing an answer.

## Architecture

```text
Course Material
    |
    v
Ingestion Pipeline
    |
    +-- PDF text extraction
    +-- PPT/PPTX extraction
    +-- Markdown/Text parsing
    +-- Handwriting/OCR pipeline
    |
    v
Chunk + Metadata Store
    |
    v
Hybrid Retrieval
BM25 + Semantic Retrieval + RRF
    |
    v
Evidence / Sufficiency Gate
    |
    +-- supported --------> grounded answer + citations
    +-- partial ----------> partial answer + missing evidence
    +-- unsupported ------> NOT_COVERED
    +-- uncertain OCR ----> LOW_QUALITY_SOURCE
    |
    v
React Study Workspace
```

## Retrieval

NightDesk uses a hybrid retrieval pipeline combining:

- BM25 lexical retrieval
- semantic retrieval
- Reciprocal Rank Fusion (RRF)

Retrieved passages are then checked against the requested evidence needs before an answer is shown.

This separation between **retrieval** and **answer sufficiency** is important: finding a related passage does not automatically mean that the passage answers the question.

## Evidence and Citations

Every supported answer can expose:

- source filename
- exact page, slide, or section
- extracted source representation
- visual preview where available
- link to the original local source
- extraction quality
- evidence completeness for multi-part questions

PDF pages are rendered directly for preview. PowerPoint slides can also be rendered to images so the original visual evidence can be inspected beside the answer.

## Handwritten Notes

Handwritten images are supported as a separate source type.

NightDesk tracks extraction quality rather than automatically trusting uncertain OCR. When handwriting extraction is too unreliable, the system can return:

```text
LOW_QUALITY_SOURCE
```

and direct the user to inspect the original material.

## Tech Stack

### Frontend

- React
- TypeScript
- Vite
- Tailwind CSS

### Backend

- Python
- FastAPI
- SQLite
- PyMuPDF
- python-pptx
- Pillow

### Retrieval

- BM25
- semantic retrieval
- Reciprocal Rank Fusion

## Project Structure

```text
nightdesk/
├── backend/
│   └── app/
│       ├── main.py
│       ├── ingestion.py
│       ├── reasoning.py
│       ├── retrieval.py
│       └── ...
├── frontend/
│   ├── src/
│   └── vite.config.ts
├── scripts/
│   ├── ingest_corpus.py
│   └── ...
├── eval/
├── corpus/
│   └── private/          # ignored by Git
├── data/                 # generated local data
├── requirements.txt
└── README.md
```

## Local Setup

### 1. Clone the repository

```bash
git clone https://github.com/nidhi500/nightdesk.git
cd nightdesk
```

### 2. Create the Python environment

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

### 3. Add study material

Place personal course material inside:

```text
corpus/private/
```

Supported formats include:

```text
.pdf
.ppt
.pptx
.md
.txt
.jpg
.jpeg
.png
.webp
```

Private course material is excluded from version control.

### 4. Ingest the corpus

```powershell
.\.venv\Scripts\python.exe -X utf8 scripts\ingest_corpus.py
```

### 5. Start the backend

```powershell
.\.venv\Scripts\python.exe -m uvicorn backend.app.main:app --host 127.0.0.1 --port 8001
```

### 6. Start the frontend

Open another terminal:

```powershell
cd frontend
npm install
npm run dev
```

Then open the local Vite URL shown in the terminal.

## Testing

Run backend tests with:

```powershell
.\.venv\Scripts\python.exe -m pytest backend/tests -q
```

Frontend checks can be run from `frontend/` using the available npm scripts.

## Privacy

NightDesk is designed for local study material.

Private corpus files, generated databases, converted slide images, OCR artifacts, caches, and environment-specific files should remain outside Git. The repository contains application code rather than the user's private course documents.

## Limitations

- Extractive mode favors wording present in the source material.
- Retrieval can find related content without that content being sufficient to answer the question.
- OCR quality depends on handwriting clarity and available OCR support.
- Complex teaching transformations or free-form paraphrasing may require an external model/provider configuration.
- Visual PowerPoint previews require locally rendered slide images.
- The current demo corpus is DBMS/SQL-focused and is not intended to answer topics outside those materials.

## Design Goal

NightDesk is not meant to be a general-purpose chatbot.

It is a revision assistant built around one rule:

> **Answer from the student's material, show the evidence, and say when the evidence is not enough.**

---

**NightDesk — Your Notes. Your Night.**
