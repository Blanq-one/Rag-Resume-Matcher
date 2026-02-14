# RAG Resume Matcher

**AI-powered resume analysis using Retrieval-Augmented Generation (RAG) to match resumes against job descriptions with semantic precision.**

Built with Flask, React, Pinecone, Sentence Transformers, and OpenAI GPT-3.5 Turbo.

---

## Overview

RAG Resume Matcher goes beyond simple keyword matching. It uses semantic embeddings to deeply understand resume content and job requirements, then leverages LLM-powered analysis to deliver structured, actionable insights.

**How it works:**

1. **Upload** your resume as a PDF
2. **Paste** any job description
3. **Get** a detailed fit analysis with scores, skill breakdowns, and improvement recommendations

---

## Features

### Resume-JD Fit Analysis
- Semantic similarity scoring between resume content and job descriptions
- Fit verdicts: *Excellent Fit*, *Good Fit*, *Average Fit*, *Weak Fit*, or *No Fit*
- Relevant skills extraction
- Detailed summary explaining the match

### Comprehensive Resume Assessment
- Strengths and weaknesses breakdown
- Missing skills identification
- Actionable improvement recommendations
- Overall competitiveness score (0-10)

### Smart Processing
- PDF text extraction with intelligent chunking
- Quality filtering to remove noise (headers, contact blocks, formatting artifacts)
- Vector embeddings stored in Pinecone for fast semantic search
- Multi-turn conversation support for follow-up queries

### User Experience
- Dark mode / Light mode toggle
- Chat history with localStorage persistence
- Export analysis results as JSON
- Responsive design for desktop and mobile

---

## Tech Stack

| Layer         | Technology                                                  |
|---------------|-------------------------------------------------------------|
| **Frontend**  | React, CSS (custom dark mode support)                       |
| **Backend**   | Flask, Flask-CORS, Gunicorn                                 |
| **Embeddings**| Sentence Transformers (`all-MiniLM-L6-v2`, 384-dim)        |
| **Vector DB** | Pinecone                                                    |
| **LLM**       | OpenAI GPT-3.5 Turbo                                       |
| **PDF Parse** | pdfplumber                                                  |
| **NLP**       | NLTK (sentence tokenization)                                |

---

## Architecture

```
┌─────────────┐     ┌──────────────────────────────────────────────┐
│   React UI  │────▶│              Flask REST API                  │
│  (Frontend) │◀────│                                              │
└─────────────┘     │  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
                    │  │ PDF      │  │ Sentence │  │ Pinecone  │  │
                    │  │ Extractor│─▶│ Embedder │─▶│ Vector DB │  │
                    │  └──────────┘  └──────────┘  └─────┬─────┘  │
                    │                                     │        │
                    │  ┌──────────────────────────────────▼─────┐  │
                    │  │         OpenAI GPT-3.5 Turbo           │  │
                    │  │  (Fit Analysis + Resume Assessment)    │  │
                    │  └────────────────────────────────────────┘  │
                    └──────────────────────────────────────────────┘
```

**RAG Pipeline:**
1. **Retrieval** — Resume chunks are embedded and stored in Pinecone. Job descriptions are embedded at query time and matched via cosine similarity.
2. **Augmentation** — Top matching chunks are retrieved and assembled as context.
3. **Generation** — GPT-3.5 Turbo produces structured JSON analysis using the retrieved context.

---

## API Endpoints

| Method | Endpoint              | Description                                  |
|--------|-----------------------|----------------------------------------------|
| POST   | `/upload-pdf`         | Upload and process PDF resumes               |
| POST   | `/qa`                 | Analyze resume-JD fit (semantic match)       |
| POST   | `/assess-resume`      | Full resume assessment with recommendations  |
| GET    | `/list-uploaded-files`| List all uploaded resume files               |
| GET    | `/count-vectors`      | Get vector count statistics                  |
| POST   | `/clear-index`        | Clear all stored vectors                     |
| POST   | `/export`             | Export analysis results as JSON              |
| GET    | `/health`             | Health check with feature listing            |

---

## Getting Started

### Prerequisites

- Python 3.10+
- Node.js (for the React frontend)
- A [Pinecone](https://www.pinecone.io/) account (free tier works)
- An [OpenAI](https://platform.openai.com/) API key

### 1. Clone the repository

```bash
git clone https://github.com/Blanq-one/RAG-Resume-Matcher.git
cd RAG-Resume-Matcher
```

### 2. Set up environment variables

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_api_key
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX_NAME=rag-to-riches
PINECONE_NAMESPACE=resume
```

### 3. Install backend dependencies

```bash
pip install -r requirements.txt
```

### 4. Run the backend

```bash
python app.py
```

The API will start at `http://localhost:5000`.

### 5. Run the frontend

```bash
cd frontend
npm install
npm start
```

The React app will open at `http://localhost:3000`.

---

## Project Structure

```
RAG-Resume-Matcher/
├── app.py                 # Flask API — routes and request handling
├── embedder.py            # Sentence embedding (MiniLM-L6-v2)
├── llm_qa.py              # OpenAI GPT integration — analysis & assessment
├── pinecone_utils.py      # Pinecone vector DB operations
├── requirements.txt       # Python dependencies
├── runtime.txt            # Python runtime version
├── Procfile               # Gunicorn deployment config
├── frontend/
│   ├── App.js             # Main React component
│   ├── App.css            # Styles with dark mode support
│   ├── index.js           # React entry point
│   ├── index.css          # Global styles
│   └── ResultCard.js      # Result display component
└── README.md
```

---

## Usage Examples

### Upload a resume

```bash
curl -X POST http://localhost:5000/upload-pdf \
  -F "files=@resume.pdf"
```

### Analyze fit against a job description

```bash
curl -X POST http://localhost:5000/qa \
  -H "Content-Type: application/json" \
  -d '{"query": "Looking for a Senior Python Developer with 5+ years experience in Flask, REST APIs, and cloud services..."}'
```

**Response:**

```json
{
  "summary": {
    "Summary": "The candidate demonstrates strong Python and Flask experience...",
    "Verdict": "Good Fit",
    "Score": 7,
    "Relevant Skills": ["Python", "Flask", "REST APIs", "AWS"]
  },
  "chunks_used": [...],
  "metadata": {
    "total_results": 15,
    "filtered_results": 8,
    "average_similarity": 0.642
  }
}
```

### Assess a resume with improvement recommendations

```bash
curl -X POST http://localhost:5000/assess-resume \
  -H "Content-Type: application/json" \
  -d '{"job_description": "Senior Python Developer...", "resume_file": "resume.pdf"}'
```

---

## Deployment

The app is deployment-ready with Gunicorn. The included `Procfile` supports platforms like Render, Railway, or Heroku:

```
web: gunicorn app:app
```

---

## License

This project is open source and available under the [MIT License](LICENSE).

---

## Contributing

Contributions are welcome! Feel free to open an issue or submit a pull request.
