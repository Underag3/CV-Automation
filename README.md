  # 🚀 Automated CV Screening

An end-to-end automated screening pipeline built with Python and Google Colab to streamline candidate evaluation, administrative validation, CV downloading, text extraction, and matrix-based matching for Recruitment.

---

## 📌 Table of Contents
- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Key Features](#-key-features)
- [Folder Structure](#-folder-structure)
- [Tech Stack & Dependencies](#-tech-stack--dependencies)
- [Getting Started](#-getting-started)
- [Pipeline Workflow](#-pipeline-workflow)
- [Administrative Flags & Exception Handling](#-administrative-flags--exception-handling)
- [Future Improvements](#-future-improvements)
- [Author & Acknowledgments](#-author--acknowledgments)

---

## 📖 Overview

Recruiting at scale requires fast, accurate, and unbiased candidate pre-screening. This project automates the entire recruitment workflow—from ingesting live form responses via Google Sheets, validating compliance rules, downloading PDF CVs, extracting text using high-performance PDF parsers (`PyMuPDF`), to generating structured screening reports in Excel.

Designed for **Human-in-the-loop Automation**, the system automatically flags edge cases (e.g., folder links instead of direct file links, missing portfolios, locked Drive permissions) for manual recruiter review while automatically passing fully compliant candidates.

---

## 🏗 System Architecture

```
┌─────────────────────────┐
│ Google Sheets Responses │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐     ┌───────────────────────────┐
│ 1. Data Ingestion &     │ ◄── │ Reference Matrices        │
│    Column Mapping       │     │ - Skill Matrix            │
└────────────┬────────────┘     │ - Academic Matrix         │
             │                  │ - Division Keywords       │
             ▼                  └───────────────────────────┘
┌─────────────────────────┐
│ 2. Data Cleaning &      │
│    Text Normalization   │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ 3. Administrative Check │ ──► [ Flags: Invalid Link, Missing Data, etc. ]
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ 4. Batch CV Download    │
│    & PyMuPDF Extraction │ ──► [ Output: Local .pdf & .txt files ]
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ 5. Profile Aggregation  │
│    & Matrix Scoring     │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ 6. Output Generation    │ ──► [ Final Excel Report (.xlsx) ]
└─────────────────────────┘
```

---

## ✨ Key Features

- **Live Google Sheets Integration**: Direct data fetching using `gspread` and Google Drive Auth to prevent string truncation or data type corruption (e.g., phone numbers).
- **Flexible Column Mapping & Normalization**: Regex-driven column detection handling variations in form field names, extra spaces, line breaks, and raw HTML URLs.
- **Rules-Based Administrative Screening**:
  - Validates mandatory fields (Name, Contact, Education, Division Choice, CV Link).
  - Enforces portfolio submission rules specifically for Creative roles (*Graphic Design*, *Content Creator*).
  - Differentiates direct Google Drive file URLs (`/file/d/`) from Drive Folder URLs (`/folders/`).
- **Batch CV Download & Text Extraction**:
  - Downloads candidate PDFs using `gdown`.
  - Extracts text via `PyMuPDF` (`fitz`) into local `.txt` files for complete auditability.
  - Detects short or unreadable text layers (e.g., image-based scans).
- **Consolidated Candidate Matching Profiles**: Combines submission form answers and extracted CV text into a unified searchable context for keyword and matrix matching.
- **Audit-Ready Excel Output**: Exported directly into standardized multi-column screening templates.

---

## 📂 Folder Structure

```text
CV Screening Automation/
├── 01_input/                  # Reference files & input matrices
│   ├── template_screening_cv.xlsx
│   ├── skill_matrix.xlsx
│   ├── academic_matrix.xlsx
│   └── division_keywords.xlsx
├── 02_output/                 # Timestamped Excel screening reports
│   └── hasil_screening_YYYYMMDD_HHMMSS.xlsx
├── 03_archive/                # Historical records and backups
├── 04_temp_cv_downloads/      # Downloaded PDF CV files (001_Name.pdf, etc.)
├── 05_cv_text/                # Extracted text files (001_Name.txt, etc.)
└── Screening_CV_SOKO.ipynb    # Main Python/Jupyter Notebook pipeline
```

---

## 🛠 Tech Stack & Dependencies

- **Language**: Python 3.x
- **Environment**: Google Colab / Jupyter Notebook
- **Data Manipulation**: `pandas`, `numpy`
- **Excel Handling**: `openpyxl`
- **Cloud & Google API**: `gspread`, `google-auth`, `gdown`
- **PDF Extraction**: `PyMuPDF` (`fitz`)
- **Text Processing**: `re` (Regular Expressions)

---

## 🚀 Getting Started

### 1. Prerequisites
Ensure you have access to Google Colab or a local Python 3.10+ environment with access to Google Drive / GCP authentication.

### 2. Environment Setup
Install required Python libraries:
```bash
pip install pandas openpyxl gspread PyMuPDF gdown
```

### 3. Folder & File Preparation
1. Mount Google Drive and set the base directory path:
   ```python
   BASE_DIR = Path('/content/drive/Shareddrives/CV Screening Automation')
   ```
2. Upload required reference matrices to `01_input/`:
   - `template_screening_cv.xlsx`
   - `skill_matrix.xlsx`
   - `academic_matrix.xlsx`
   - `division_keywords.xlsx`

### 4. Running the Pipeline
Open `Screening_CV_SOKO.ipynb` in Google Colab, set your Google Sheets response link in `SPREADSHEET_URL`, and execute cells sequentially:
- **Part 1**: Project setup & directory creation
- **Part 2**: Ingest data & load matrices
- **Part 3**: Clean responses & normalize columns
- **Part 4**: Perform administrative validation & flagging
- **Part 5**: Batch download PDF CVs & extract text
- **Part 6**: Generate consolidated profiles & score candidates

---

## 🚩 Administrative Flags & Exception Handling

The pipeline uses explicit status indicators to prevent false rejections:

| Status Flag | Meaning | Trigger Condition |
| :--- | :--- | :--- |
| `Administrasi Lengkap` | Full Compliance | All required data, valid CV file link, and required portfolio provided. |
| `CV_LINK_IS_FOLDER` | Manual Review Needed | Candidate pasted a Google Drive folder link instead of a direct file link. |
| `MISSING_PORTFOLIO_FOR_CREATIVE` | Action Required | Candidate applied for *Graphic Design* or *Content Creator* without a portfolio link. |
| `CV_DOWNLOAD_FAILED` | Access Issue | File permission restricted ("Anyone with the link" disabled) or broken URL. |
| `TEXT_TOO_SHORT` | OCR Candidate | Extracted CV text < 300 characters (likely an image-based/scanned CV). |
| `MISSING_ADMIN_PROOF` | Incomplete | Candidate missed required recruitment proof fields. |

---

## 🔮 Future Improvements

- [ ] **Google Drive API v3 Integration**: Automatically traverse inside Drive folders when a candidate provides a folder link instead of a direct file URL.
- [ ] **OCR Fallback Support**: Integrate `pytesseract` and `pdf2image` to perform Optical Character Recognition on scanned PDF CVs.
- [ ] **Automated Email Notifications**: Trigger automated status update emails to applicants based on their final screening score.
- [ ] **NLP-Based Semantic Matching**: Incorporate embedding models (e.g., Sentence-BERT) for contextual matching beyond exact keyword hits.

---

## 👤 Author & Acknowledgments

- **Developed By**: Recruitment & Automation Team @ SOKO Financial
- **License**: MIT License
