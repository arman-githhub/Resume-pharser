# 📄 AI Resume Parser & Matcher (resume-parser)

An AI-powered, full-stack web application designed to automatically extract, parse, clean, evaluate, and rank candidate resumes against Job Descriptions (JDs). It leverages Natural Language Processing (NLP) with **SpaCy**, semantic text embeddings with **Sentence-Transformers (SBERT)**, and Optical Character Recognition (OCR) via **Tesseract / RapidOCR**.

---

## ⚙️ How It Works (Step-by-Step System Flow)

```
┌─────────────────┐       ┌─────────────────┐       ┌────────────────────────┐
│  User / Recruiter│ ───► │ Upload JD &     │ ───► │  FastAPI Backend       │
│  (React UI)     │       │ Resumes (Files) │       │  Processing Queue      │
└─────────────────┘       └─────────────────┘       └───────────┬────────────┘
                                                                │
                                                                ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                       AI & Extraction Pipeline                             │
├────────────────────────────────────────────────────────────────────────────┤
│ 1. Text Extraction : PyMuPDF / python-docx / OCR (Tesseract / RapidOCR)   │
│ 2. Text Normalization: Cleaning, lemmatization & tokenization              │
│ 3. NLP Parsing     : SpaCy entity recognition (Skills, Roles, Experience)  │
│ 4. Vector Embedding: Sentence-Transformers (SBERT all-MiniLM-L6-v2)        │
│ 5. Hybrid Scoring  : (Keyword Overlap × 40%) + (SBERT Cosine Similarity × 60%)│
└────────────────────────────────────────────────────────────────────────────┘
                                                                │
                                                                ▼
┌─────────────────┐       ┌─────────────────┐       ┌────────────────────────┐
│ MongoDB Atlas   │ ◄───  │ Decision Engine │ ───►  │ Ranked Dashboard       │
│ (Persistent DB) │       │ (Accepted/Reject)│       │ (Results & CSV Export) │
└─────────────────┘       └─────────────────┘       └────────────────────────┘
```

### Detailed Workflow Explanation

1. **User Authentication & Session Management**:
   - The recruiter logs in or signs up via the React Frontend.
   - The backend validates credentials using hashed passwords (`bcrypt`) and returns an encrypted JWT Bearer Token stored in browser storage.

2. **Job Description & Resume Ingestion**:
   - The user inputs the Job Title, Job Description text, and uploads candidate resumes (`.pdf`, `.docx`, `.txt`, `.png`, `.jpg`).
   - The frontend sends a multipart request to `/api/score/batch`.

3. **Multi-Engine Document Parsing**:
   - **PDFs & Word Files**: Extracted using **PyMuPDF (`fitz`)** and **`python-docx`**.
   - **Image Resumes / Scanned PDFs**: Automatically routed to Optical Character Recognition (**Tesseract OCR**) or **RapidOCR**.

4. **NLP Processing & Keyword Extraction**:
   - **SpaCy (`en_core_web_sm`)** processes both the Job Description and cleaned resume text to identify key entities: candidate names, technical skills, job titles, and experience keywords.

5. **AI Hybrid Scoring Algorithm**:
   - **Keyword Overlap Score (TF-IDF Weight)**: Measures exact skill and domain keyword matches.
   - **Semantic Similarity Score (SBERT)**: Encodes text into 384-dimensional dense vector embeddings using `all-MiniLM-L6-v2` and computes Cosine Similarity between JD and resume.
   - **Final Match Score Formula**:
     $$\text{Final Score} = (\text{Keyword Match} \times 0.40) + (\text{SBERT Semantic Similarity} \times 0.60)$$

6. **Candidate Ranking & Storage**:
   - Resumes with a score $\ge 0.45$ (configurable via `MIN_ACCEPT_SCORE`) are classified as **ACCEPTED**, while lower-scoring resumes are marked as **REJECTED**.
   - Evaluation batch results and extracted metrics are saved to **MongoDB Atlas**.
   - Live progress is reported via polling, and final results are rendered on an interactive dashboard with CSV export support.

---

## 🌟 Key Features

- 🔐 **User Authentication**: Secure JWT-based Signup and Login system with password hashing (`passlib` + `bcrypt`).
- 📝 **Multi-Format Resume Support**: Parses `.pdf`, `.docx`, `.txt`, and image files (`.png`, `.jpg`, `.jpeg`).
- 🔍 **Hybrid Text & OCR Extraction**:
  - Direct text extraction using **PyMuPDF** (`fitz`) & **python-docx**.
  - Automatic OCR fallback via **Tesseract** (`pytesseract`) or **RapidOCR** (`rapidocr-onnxruntime`) for scanned resumes.
- 🧠 **NLP & Semantic Scoring**:
  - Entity & keyword extraction using **SpaCy** (`en_core_web_sm`).
  - Semantic vector similarity calculated with **SBERT** (`all-MiniLM-L6-v2`).
  - Dual scoring: TF-IDF / Keyword overlap score + SBERT Semantic Embedding similarity score.
- 📊 **Detailed Match Analytics**: Categorizes resumes into **Accepted** or **Rejected** based on configurable threshold scores.
- 📁 **History & Export**: Tracks past evaluation batches and supports exporting candidate results to CSV.
- ⚡ **Modern UI**: Clean, responsive frontend built using **React**, **Vite**, and **Tailwind CSS**.

---

## 🏗️ Project Architecture & Tech Stack

```text
resume-parser/
├── backend/            # FastAPI REST API & NLP pipeline
│   ├── api/            # Route handlers (Auth, History, Scoring)
│   ├── core/           # Configuration, security, MongoDB connection
│   ├── services/       # Text extraction, NLP entity extraction, SBERT scoring
│   ├── schemas.py      # Pydantic data schemas & validations
│   └── app.py          # FastAPI application entry point
└── frontend/           # React + Vite Single Page Application
    ├── src/
    │   ├── pages/      # Login, Dashboard, Progress, Results, History
    │   ├── api.js      # Axios instance & API wrapper methods
    │   └── App.jsx     # Main React Router setup
```

### Tech Stack

- **Backend**: Python 3.9+, FastAPI, Uvicorn, Pydantic, PyMongo
- **AI & NLP**: SpaCy (`en_core_web_sm`), Sentence-Transformers (`all-MiniLM-L6-v2`), PyMuPDF, PyTesseract, RapidOCR
- **Database**: MongoDB (Local or Cloud MongoDB Atlas)
- **Frontend**: React 18, Vite, Tailwind CSS, Axios, Lucide React

---

## 🚀 Setup & Installation Guide

### Prerequisites

- **Python 3.9 - 3.12** *(Python 3.10+ recommended)*
- **Node.js 18+** & **npm**
- **MongoDB**: Local MongoDB instance or a [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) URI.
- *(Optional)* **Tesseract OCR**: For optimal image OCR extraction (`brew install tesseract` on macOS).

---

### Step 1: Clone & Configure Environment Variables

#### Backend Environment (`backend/.env`)
Create a `.env` file inside the `backend/` directory:

```env
# MongoDB Atlas or Local MongoDB connection string
MONGO_URI=mongodb+srv://<username>:<password>@cluster0.xxx.mongodb.net/?appName=Cluster0
MONGO_DB_NAME=resume_pharser

# Security Key
JWT_SECRET_KEY=dev-secret-change-me
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=720

# Scoring Settings
MIN_ACCEPT_SCORE=0.45
PROCESS_CONCURRENCY=4
```

#### Frontend Environment (`frontend/.env`)
Create/update `.env` inside the `frontend/` directory:

```env
VITE_API_URL=http://localhost:8000
```

---

### Step 2: Backend Setup & Execution

Open a terminal window and execute:

```bash
# 1. Navigate to backend
cd backend

# 2. Create and activate Python virtual environment
python3 -m venv .venv
source .venv/bin/activate    # On Windows: .venv\Scripts\activate

# 3. Upgrade pip and install backend dependencies
./.venv/bin/pip install --upgrade pip
./.venv/bin/pip install -r requirements.txt

# 4. Download SpaCy English language model
./.venv/bin/python -m spacy download en_core_web_sm

# 5. Run FastAPI backend server
./.venv/bin/uvicorn app:app --reload --port 8000
```

> 💡 **Backend API Docs**: Once started, view interactive API documentation at [http://localhost:8000/docs](http://localhost:8000/docs).

---

### Step 3: Frontend Setup & Execution

Open a **second terminal window** and execute:

```bash
# 1. Navigate to frontend
cd frontend

# 2. Install Node dependencies
npm install

# 3. Start Vite development server
npm run dev
```

> 💡 **Application UI**: Open your browser at [http://localhost:5173](http://localhost:5173).

---

## 📡 API Endpoints Summary

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/auth/signup` | Register a new user |
| `POST` | `/api/auth/login` | Log in and receive JWT bearer token |
| `POST` | `/api/score/batch` | Upload Job Description & resumes to evaluate |
| `GET` | `/api/score/batch/{batch_id}` | Retrieve evaluation status & live progress |
| `GET` | `/api/history` | List all historical evaluation batches for user |
| `GET` | `/api/history/{batch_id}` | Detailed results for a specific batch |
| `DELETE` | `/api/history/{batch_id}` | Delete a batch from history |

---

## 📜 License

This project is licensed under the MIT License.
