# HR Hiring Bot — OPTIMIZED ATS SYSTEM

---

##  Project Title

**HR Hiring Bot** — AI‑Powered Resume Screening, JD Understanding, and Candidate Ranking System

---

##  Overview

The **HR Hiring Bot** automates end‑to‑end resume evaluation against job descriptions using **LLMs + Embeddings + ATS‑style keyword matching**. It ensures unbiased, consistent, explainable and scalable hiring assistance.

The system:

* Converts **Job Descriptions (JD)** into structured machine‑understandable JSON
* Parses **PDF resumes** into structured JSON with projects, metrics, skills & domain tags
* Scores each candidate based on **Project relevance, Keyword match and Semantic understanding**
* Produces **ranked candidate lists** with complete explainability
* Provides an easy‑to‑use **Streamlit UI** for HR teams

---

##  Core Objective

| Goal              | Implementation                                                                         |
| ----------------- | -------------------------------------------------------------------------------------- |
| Understand JD     | LLM‑based parser (JDGpt.py) → `JD.json` (role, seniority, domain tags, weighting etc.) |
| Understand Resume | LLM‑based parser (GptJson.py) → detailed resume JSON                                   |
| Compare Fairly    | Hybrid Ranking = Project + Semantic + Keyword                                          |
| Explainability    | Provenance spans + section‑level semantic matches                                      |
| HR Usability      | Streamlit UI + Downloadable rankings                                                   |

---

##  System Features

🔹 JD → JSON (no information loss, adds inferred domain, seniority, HR notes)

🔹 Resume → JSON (projects with metrics, canonical skills, inferred skills, domain tags)

🔹 Multi‑stage scoring:

* **KeywordComparitor** → ATS‑style weighted matching
* **ProjectProcess** → project‑depth & domain‑match scoring
* **SemanticComparitor** → OpenAI embedding deep‑match

🔹 **FinalRanking** — ensures **no candidate is skipped unless all scores = 0**

🔹 **RAM‑safe UI ranking** — shown even if cloud host blocks file write

🔹 **PDF handling fallback system** — PyMuPDF → PyPDF2 → regex fallback

---

##  Tech Stack

| Layer             | Tools / Libraries                                    |
| ----------------- | ---------------------------------------------------- |
| UI                | Streamlit                                            |
| LLM Parsing       | OpenAI GPT‑4o‑mini (function calling)                |
| Semantic Matching | OpenAI Embeddings — `text‑embedding‑3‑small`         |
| Caching           | Pickle cache for embeddings                          |
| PDF Extraction    | PyMuPDF (fitz), PyPDF2 with fallback router          |
| Data Processing   | Python, NumPy, JSON                                  |
| Orchestration     | runpy (no subprocess) for Streamlit execution safety |

>  Heavy libraries like `torch` / sentence‑transformers were removed for deployment compatibility.

---

##  Repository Structure

```
HR_Hiring_Bot/
│ main.py                        → Streamlit UI + pipeline orchestration
│ requirements.txt
│ README.md
│
├─ InputThread/
│   ├─ JD/
│   │   ├─ JD.txt              → Raw uploaded JD
│   │   ├─ JD.json             → Structured normalized JD
│   ├─ AI Processing/
│   │   ├─ JDGpt.py            → JD → JSON using LLM
│   │   ├─ GptJson.py          → Resume → JSON using LLM
│   ├─ file_router.py          → Picks best PDF extraction method
│   ├─ extract_pdf.py          → PDF extraction engines
│
├─ ResumeProcessor/
│   ├─ ProjectProcess.py       → Project depth + metrics scoring
│   ├─ KeywordComparitor.py    → ATS weighted keyword matching
│   ├─ SemanticComparitor.py   → Embedding similarity matching
│   └─ Ranker/
│       ├─ FinalRanking.py     → Final ranking aggregation & RAM export
│
├─ Processed-TXT/              → Extracted raw PDF text
├─ ProcessedJson/              → LLM generated structured resume JSON
├─ Ranking/
│   ├─ Scores.json             → Consolidated scores across all modules
│   ├─ Final_Ranking.json      → Sorted ranking with final scores
│   ├─ DisplayRanks.txt        → HR readable list (Name | Score)
│   └─ Skipped.json            → Only resumes with **all scores = 0**
```

---

##  Complete Candidate Evaluation Pipeline

### **🔹 Step 1: Upload JD → Run JDGpt.py**

* Accepts PDF or pasted text
* Extracted → `JD.txt`
* LLM produces `JD.json` containing:

```
role_title, seniority_level, domain_tags, weighting,
required_skills, preferred_skills,
embedding_hints, hr_notes, hr_points, meta
```

> `domain_tags` is later used by resume parser for **domain‑relevant scoring**.

### **🔹 Step 2: Upload Resumes → route_pdf() → GptJson.py**

* Raw text saved to `/Processed-TXT`
* LLM produces structured `/ProcessedJson/*.json` with:

```
name, canonical_skills, inferred_skills,
projects (with metrics, tech, responsibilities),
experience_entries, profile_keywords_line,
domain_tags, ats_boost_line, provenance
```

### **🔹 Step 3: ProjectProcess.py → project_aggregate**

* Evaluates each project for:

  * technical complexity
  * execution quality
  * outcome/metrics
  * domain alignment with JD

### **🔹 Step 4: KeywordComparitor.py → Keyword_Score**

* Weighted ATS‑style matching using:

```
required_skills
preferred_skills
weighted_keywords
experience_keywords
certifications
education
```

* Handles resume errors safely → if issue → assign `Keyword_Score = 0` and continue

### **🔹 Step 5: SemanticComparitor.py → Semantic_Score**

* OpenAI embeddings cache → fast re‑runs
* Section‑level similarity for:

```
profile, skills, projects, responsibilities, education, overall
```

* Normalizes scores across all candidates

### **🔹 Step 6: FinalRanking.py → Final_Ranking.json**

Ranking Logic:

| Case           | Rule                        |
| -------------- | --------------------------- |
| All scores = 0 | skip                        |
| Only 1 score   | apply small decay → include |
| 2 or 3 scores  | weighted normalized ranking |

Also exposes:

```
run_ranking() → returns ranking in RAM for UI
RANKING_RAM   → cached ranking
```

---

##  Streamlit User Flow

| Step | Action                                      |
| ---- | ------------------------------------------- |
| 1    | Upload JD (PDF or text)                     |
| 2    | Click **“Process JD”**                      |
| 3    | Upload multiple candidate resumes (PDF)     |
| 4    | Click **“Process & Rank Resumes”**          |
| 5    | Navigate to **🏆 Rankings** tab             |
| 6    | Download ranking list or use refresh button |

Even if file‑write fails on cloud hosting:
✔ Rankings still display via `RANKING_RAM`

---

##  Output Examples

### **DisplayRanks.txt** (HR friendly)

```
1. Neeraj Jain | 0.931
2. Abhiraj Kumar Singh | 0.751
3. Mohit Patil | 0.719
...
```

### **Final_Ranking.json** (machine consumable)

```
[
  {"name": "Neeraj Jain", "Final_Score": 0.931, ...},
  {"name": "Abhiraj Kumar Singh", "Final_Score": 0.751, ...},
  ...
]
```

---

##  Requirements

```
python >= 3.10
streamlit
openai
pandas
numpy
PyPDF2
pymupdf
```

Run UI:

```
streamlit run main.py
```

---

##  Future Enhancements

* OCR support for scanned resumes
* CSV / Excel export for ranking
* ATS integrations (Greenhouse / Lever API)
* Interview scheduling automation
* Explainability dashboard (match heatmaps)

---

## 👤 Author

| Detail   | Info                                                                                         |
| -------- | -------------------------------------------------------------------------------------------- |
| Name     | **Harsh Chinchakar**                                                                         |
| Email    | **[harshchinchakar33@gmail.com](mailto:harshchinchakar33@gmail.com)**                        |
| LinkedIn | [https://www.linkedin.com/in/harshchinchakar/](https://www.linkedin.com/in/harshchinchakar/) |

---

##  License

This project is released under the **MIT License**.

---


