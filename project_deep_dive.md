# PathWise — Skill Gap to Job Matching Agent
## Complete Project Deep-Dive (Final Round Edition)

---

## 1. The Problem — Why This Exists

### Who Is Losing Time & Money?
India produces **~1.5 million engineering graduates per year** (NASSCOM, 2023). A significant portion apply to jobs they are *almost* qualified for — they have 70–80% of the required skills, but no structured way to know what the 20–30% gap costs them or how quickly they can close it.

**The person suffering today:**
A 2024 B.Tech graduate from Pune applies to 40–50 companies.  
- They spend **3–5 days manually comparing** their resume to job descriptions.  
- They get generic advice like "learn Python" — with no signal on *which specific skill* unlocks the most jobs.  
- They waste ₹10,000–₹20,000 on random online courses that may not match the market demand.  
- **Average time from graduation to first relevant job in India: 6–18 months** (TeamLease, 2022).

### Why Existing Tools Fall Short
| Tool | Problem |
|------|---------|
| LinkedIn / Naukri | Shows jobs — does NOT tell you what you're missing |
| Generic skill assessment apps | Not tied to real, live job market demand |
| HR platforms | Employer-side; candidate is passive |
| Career counselors | Expensive, slow, not data-driven |

**PathWise is the first tool that connects: your current skills → real job market gaps → specific courses → quantified ROI → "how many weeks until you're hireable."**

---

## 2. What PathWise Does — One-Sentence Pitch

> **PathWise is an AI-powered multi-agent platform that takes a candidate's resume, matches it against real job postings, identifies the exact skill gaps, and tells the candidate precisely which course to take to unlock the most job opportunities — measured in weeks.**

---

## 3. High-Level Architecture

```
┌─────────────────────────────────────────────────────┐
│                    FRONTEND (React + Vite)           │
│  Resume Upload │ Text Input │ Voice Input │ Chat     │
└──────────────────────────┬──────────────────────────┘
                           │ REST / SSE (EventSource)
┌──────────────────────────▼──────────────────────────┐
│               BACKEND (FastAPI + Python)             │
│                                                     │
│  ┌──────────────────────────────────────────────┐   │
│  │       LangGraph Orchestration Pipeline       │   │
│  │                                              │   │
│  │  [1] Profile Parsing Node                   │   │
│  │  [2] Job Search Node ──► Jooble API          │   │
│  │       └─ Fallback: 50 curated demo jobs      │   │
│  │  [3] RAG Retrieval Node ──► FAISS/TF-IDF     │   │
│  │       └─ Fallback: Deterministic continue    │   │
│  │  [4] Skill Matching Node ──► SentenceTransf. │   │
│  │  [5] Gap Analysis Node ──► deterministic     │   │
│  │  [6] Time-to-Ready Node ──► courses.json     │   │
│  │  [7] Gemini Reasoning Node ──► Gemini API    │   │
│  │       └─ Fallback: deterministic explanation │   │
│  │  [8] Training Recommendation Node            │   │
│  └──────────────────────────────────────────────┘   │
│                                                     │
│  Supporting Services:                               │
│  • Career Simulator (what-if analysis)              │
│  • Skill Combination Optimizer                      │
│  • Resume Chat (AI assistant)                       │
│  • Voice Transcription (ElevenLabs)                │
│  • Progress Streaming (SSE)                         │
└─────────────────────────────────────────────────────┘
```

---

## 4. Technology Stack (Every Library, Every Choice)

### Backend
| Technology | Version / Details | Why This Choice |
|-----------|-------------------|----------------|
| **Python 3.11** | `.python-version` file locks it | Async support + latest typing features |
| **FastAPI** | Latest | Async-native, automatic OpenAPI docs, Pydantic integration |
| **LangGraph** | `langgraph` | Explicit node-by-node stateful workflow; NOT a black-box chain. Each node is a named Python function we fully control |
| **Google Gemini API** | `google-generativeai` + `langchain-google-genai` | `gemini-2.0-flash` for tool calling; separate key for gap analysis |
| **Sentence Transformers** | `all-MiniLM-L6-v2` | 384-dimension embeddings; CPU-friendly, fast, 22MB model |
| **FAISS** | CPU build | Vector similarity search for RAG; falls back to TF-IDF cosine if unavailable |
| **PyMuPDF / pdfminer** | `document_extractor.py` | PDF text extraction |
| **Pillow** | Image resume parsing | JPG/PNG resumes via Gemini Vision |
| **httpx** | Async HTTP client | Jooble API calls with 8-second timeout |
| **Pydantic v2** | Schema validation | `UserProfile`, `JobPosting`, `JobMatchResult` — all validated |
| **uvicorn** | ASGI server | Production-ready async server |
| **ElevenLabs Scribe v1** | Speech-to-Text API | Voice resume input via WebM/WAV recording |
| **numpy** | Cosine similarity matrix | Used in `matching_service.py` for semantic scoring |

### Frontend
| Technology | Why |
|-----------|-----|
| **React 18** | Component-based UI; hooks for state |
| **Vite** | Fast HMR dev server; ES module builds |
| **lucide-react** | Icon library |
| **jsPDF** | Client-side PDF report export |
| Vanilla CSS | `style.css`, `dashboard.css` — no Tailwind dependency |

### Infrastructure
| Service | Purpose |
|---------|---------|
| **Render.com** | Backend deployment (`render.yaml`) |
| **Vercel** | Frontend deployment (`vercel.json`) |
| **Jooble API** | Real-time job search (POST to `https://jooble.org/api/{key}`) |

---

## 5. Complete Step-by-Step Workflow Walkthrough

### STEP 0 — User Inputs Resume (Three Pathways)

**Pathway A — File Upload (PDF/DOC/DOCX/TXT):**
```
User uploads file → FastAPI reads bytes → 
document_extractor.py extracts raw text (PyMuPDF for PDF) → 
parse_resume_with_gemini(text) → structured JSON
```

**Pathway B — Image Upload (JPG/PNG/WEBP):**
```
User uploads image → FastAPI detects IMAGE_EXTENSIONS →
parse_resume_image_with_gemini(bytes, filename) →
Gemini Vision API reads image as base64 →
returns structured JSON
```

**Pathway C — Voice Input:**
```
Browser records via MediaRecorder API → sends WebM audio →
POST /api/voice/transcribe → ElevenLabs Scribe v1 STT →
returns transcript text → UI auto-fills text box →
then goes through Pathway A text route
```

---

### STEP 1 — Profile Parsing (Gemini API)

**File:** [`backend/profile_parsing/gemini_client.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/profile_parsing/gemini_client.py)

The raw resume text is sent to Gemini with a strict JSON-schema prompt. Gemini extracts:
- `name`, `email`, `phone`, `location`
- `education[]` — degree, field, institution, graduation year
- `skills[]` — **normalized** (e.g., `"reactjs"` → `"React"`, `"nodejs"` → `"Node.js"`)
- `experience[]` — company, role, duration, responsibilities
- `projects[]` — name, description, technologies
- `certifications[]`
- `interests[]`
- `target_role` (inferred from resume)

**Skill Normalization Table (hardcoded + regex):**
```python
SKILL_NORMALIZATION = {
    "react js": "React", "reactjs": "React",
    "node js": "Node.js", "nodejs": "Node.js",
    "mongo db": "MongoDB", "mongodb": "MongoDB",
    "postgres sql": "PostgreSQL", "postgresql": "PostgreSQL",
    "natural language processing": "NLP",
    "rest api": "REST API",
    ...
}
```
Extra whitespace is collapsed with `re.sub(r"\s+", " ", ...)` before lookup.

**Output:** `UserProfile` Pydantic model. Validated before anything else runs.

**Multi-key fallback:** The system tries up to 3 Gemini API keys in sequence (`GEMINI_API_KEY_PROFILE_PARSING`, `GEMINI_API_KEY_PARSER`, `GEMINI_API_KEY_1`).

---

### STEP 2 — LangGraph Pipeline Begins

**File:** [`backend/orchestration/skill_gap_graph.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/orchestration/skill_gap_graph.py)

The HTTP endpoint `POST /api/skill-gap/analyze/run` creates a **run_id** via `ProgressEventManager` and launches the full LangGraph pipeline in a background async task. The frontend polls `GET /api/agent/progress/{run_id}` via **Server-Sent Events (SSE)** to show a live progress bar.

**The Graph State: `SkillGapState` (TypedDict)**  
A single immutable-ish dictionary passed through every node. Each node *writes only the keys it owns*.  
Key fields:
```
user_profile, jobs, job_source, user_skills,
matching_results, gap_analyses, current_jobs,
opportunity_analysis, opportunity_discovery,
training_recommendations, time_to_ready,
retrieved_jobs, retrieved_courses, retrieved_sources,
rag_available, ai_reasoning, errors, warnings,
node_status, status, messages, tool_calls, tool_trace,
run_id, free_only
```

**Graph Compilation:**
```python
workflow = StateGraph(SkillGapState)
# ... add_node() for each of 16 nodes
workflow.set_entry_point("profile_parsing")
skill_gap_graph = workflow.compile()  # singleton; reused across requests
```

---

### GRAPH NODE 1 — `profile_parsing_node`

**What it does:**  
Validates the incoming `profile_input` dict into a `UserProfile` Pydantic model using `model_validate()`. If this fails (e.g., malformed JSON), it sets `profile_valid = False` and routes to `error_handler`.

**Conditional Edge:**
```python
def route_after_profile(state):
    return "job_search" if state["profile_valid"] else "error_handler"
```

---

### GRAPH NODE 2 — `job_search_node`

**File:** [`backend/skill-match-agent/services/jooble_service.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/skill-match-agent/services/jooble_service.py)

Calls `JobSourceService.get_jobs()` which internally:

1. Hits `https://jooble.org/api/{JOOBLE_API_KEY}` via `httpx.AsyncClient` POST with:
   ```json
   {"keywords": "Python Developer", "location": "Pune", "page": 1}
   ```
   - 8-second timeout (`JOOBLE_REQUEST_TIMEOUT = 8.0`)
   - 10-minute in-memory cache (`CACHE_TTL_SECONDS = 600`)
   
2. If Jooble returns empty / errors / no API key → **falls back to 50 curated demo jobs** from `demo_jobs.py`

3. All jobs are normalized via `job_normalizer.py` into `JobPosting` Pydantic models with:
   - `job_id` (UUID generated if missing)
   - `title`, `company`, `location`, `description`
   - `required_skills[]` — extracted from job description using `skill_extractor.py`

**Conditional Edge:**
```python
def route_after_job_search(state):
    return "rag_retrieval" if state["jobs"] else "job_search_fallback"
```
If absolutely no jobs → `job_search_fallback_node` loads the 50 demo jobs directly.

---

### GRAPH NODE 3 — `rag_retrieval_node`

**Files:** [`backend/orchestration/rag/retriever.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/orchestration/rag/retriever.py), [`vector_store.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/orchestration/rag/vector_store.py)

**What is RAG here?**  
Retrieval-Augmented Generation — we index the job postings and course catalog into a vector store so that when Gemini generates explanations, it's grounded in *our actual data*, not hallucinated information.

**Process:**
1. `document_preparation.py` converts jobs and courses into text documents
2. `vector_store.py` indexes them using:
   - **FAISS** (if `faiss-cpu` installed) — L2 similarity on dense embeddings
   - **TF-IDF cosine similarity** fallback (scikit-learn) — no GPU needed
3. `retriever.py` queries the index with the candidate's missing skills + profile as query string
4. Returns `retrieved_jobs[]`, `retrieved_courses[]`, `retrieved_sources[]`

**`rag_available` flag:**  
Set to `True` if retrieval succeeds. If it fails for any reason → `rag_fallback_node` → continues deterministically with empty retrieved sources.

**Conditional Edge:**
```python
def route_after_rag(state):
    return "skill_matching" if state["rag_available"] else "rag_fallback"
```

---

### GRAPH NODE 4 — `skill_matching_node`

**File:** [`backend/skill-match-agent/services/matching_service.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/skill-match-agent/services/matching_service.py)

**The Hybrid Scoring Formula:**
```
final_score = (0.70 × exact_score) + (0.30 × semantic_score)
```

**Exact Score (70% weight):**
- Normalize both candidate skills and job required skills with the `normalizer.py` skill map
- Count how many normalized job skills appear in normalized candidate skills
- `exact_score = (matched_count / total_required) × 100`

**Semantic Score (30% weight):**
- Load `all-MiniLM-L6-v2` (SentenceTransformer, 22MB)
- Encode **candidate skills** as a matrix — done ONCE per request for efficiency
- Encode each **unmatched job skill** separately
- Compute **cosine similarity matrix** using numpy
- If `max_similarity >= 0.5` (threshold) → skill counts as semantically matched
- `semantic_score = (sum(similarity_scores) / total_required) × 100`

**Why 70/30 split?**  
Exact keyword match is highly reliable. Semantic match catches variants like "React" ↔ "React.js development" but must be weighted lower to avoid false positives.

**Output:** `List[JobMatchResult]` sorted descending by `match_score`. Each result has:
- `matched_skills[]` (exact + semantic matches)
- `missing_skills[]`
- `match_score`, `explicit_score`, `semantic_score`

---

### GRAPH NODE 5 — `gap_analysis_node`

**File:** [`backend/gap-analysis-agent/services/gap_analysis_service.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/gap-analysis-agent/services/gap_analysis_service.py)

Takes the **top 10 matched jobs** (capped to bound compute) and runs `analyze_multiple_jobs()`:

For each job:
- Cross-references the job's `required_skills` against `user_skills`
- Produces a structured gap object:
```json
{
  "job_id": "...",
  "job_title": "Python Developer",
  "matched_skills": ["Python", "FastAPI"],
  "missing_skills": ["PostgreSQL", "Docker"],
  "match_score": 72.5
}
```

This is **100% deterministic** — no AI involved here, just set operations on skill lists with normalization.

---

### GRAPH NODE 6 — `time_to_ready_node`

**Files:** [`backend/training-agent/time_to_ready.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/training-agent/time_to_ready.py), [`backend/orchestration/opportunity_analysis.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/orchestration/opportunity_analysis.py)

**This node does TWO things:**

**6a — Opportunity Discovery:**
```python
analyze_opportunities(
    candidate_skills=user_skills,
    missing_skills=all_missing_skills,
    jobs=jobs,
    matching_service=matching_service
)
```
For each missing skill, it re-runs `rank_jobs()` with the skill **added** to the candidate's skill set and counts how many additional jobs become fully compatible (zero missing skills). This answers: *"If I learn PostgreSQL, how many more jobs can I apply for?"*

Uses `itertools.combinations()` up to `MAX_COMBINATION_SKILLS=5` to find **skill combos** that unlock even more opportunities.

**6b — Time-to-Ready per Job:**
```python
calculate_per_job_time_to_ready(analyses, courses, free_only)
```
For each job's `missing_skills`:
1. Look up each skill in `build_skill_to_courses_map()` (from `courses.json`)
2. Find the shortest matching course
3. Calculate:
   - `total_weeks_sequential` = sum of all individual course durations
   - `total_weeks_parallel` = max(individual course durations) — assuming simultaneous study
   - `total_cost_inr` = sum of course prices (₹0 if `free_only=True`)
4. Returns per-job breakdown

**Course Catalog (courses.json — 20+ courses):**  
Real providers mapped:
- Infosys Springboard (free)
- NPTEL (free)
- Skill India / NSQF (free)
- Coursera (₹2,999)
- Udemy (₹3,499)
- GeeksforGeeks (₹5,999)
- Great Learning

---

### GRAPH NODE 7 — `gemini_reasoning_node`

**File:** [`backend/gap-analysis-agent/services/gemini_service.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/gap-analysis-agent/services/gemini_service.py)

**What it does:**  
Calls Gemini to generate a *human-readable narrative* of the gap analysis. Only runs Gemini for the **top 1 job** (by match score) — all others get deterministic fallbacks instantly. This bounds API costs while keeping the most important result AI-enriched.

**Prompt strategy:**  
The prompt is constructed with all deterministic data already computed:
- User profile summary
- Job requirements
- Matched skills, missing skills
- Retrieved job/course evidence from RAG

Gemini returns structured JSON with:
```json
{
  "summary": "...",
  "strengths": ["Python", "FastAPI"],
  "priority_gaps": [{"skill": "PostgreSQL", "reason": "Required for 8/10 matching jobs"}],
  "learning_focus": ["Build working knowledge of PostgreSQL"],
  "source": "gemini"
}
```

**Hallucination protection:**  
The service validates that `strengths` only contains skills from `matched_skills` and `priority_gaps` only contains skills from `missing_skills`. If Gemini invents a skill, it's stripped.

**Citation attachment:**  
`citation_validator.py` → `attach_default_citations()` → links RAG source IDs to the reasoning output for provenance.

**Fallback:**  
If Gemini API fails (timeout, rate limit, no key), the node generates the same structured output deterministically using Python string formatting — the frontend never sees a difference.

---

### GRAPH NODE 8 — `training_recommendation_node`

**File:** [`backend/training-agent/agent.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/training-agent/agent.py)

**The ROI Formula:**
```python
roi = (jobs_unlocked / duration_weeks) * 10
```
If `jobs_unlocked = 8` and `duration_weeks = 4` → ROI = 20.0

**Process:**
1. Collect all `missing_skills` from gap analyses
2. Sort by `jobs_unlocked` (descending) — highest-impact skill first
3. For each of the top 3 missing skills, call `recommend_training()`:
   - Filter `courses.json` by `free_only` flag if set
   - Score each matching course: `total_score = roi + (match_count × 50)`
   - Return the top-scoring course
4. Deduplicate by course name
5. Enrich with RAG-retrieved course evidence (if available)

**Output per recommendation:**
```json
{
  "course_name": "Database Management Systems & PostgreSQL",
  "provider": "NPTEL",
  "duration_weeks": 8,
  "jobs_unlocked": 8,
  "learning_impact": 1.0,
  "reasoning": "Mastering PostgreSQL via this course closes your primary market gap, unlocking +8 jobs in 8 weeks",
  "url": "https://nptel.ac.in/courses",
  "price_inr": 0,
  "is_free": true
}
```

---

### SUPPLEMENTARY: Career Simulator

**File:** [`backend/orchestration/career_simulator.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/orchestration/career_simulator.py)

**Before/After What-If Simulation:**
1. Runs `rank_jobs()` with *current skills* → `current_eligible` jobs
2. Adds up to 4 top missing skills → `simulated_skills`
3. Runs `rank_jobs()` again → `simulated_eligible` jobs
4. `jobs_unlocked = len(simulated_ids - current_ids)` — set subtraction

Returns:
```json
{
  "current": {"matching_jobs": 3, "average_match": 68.5},
  "simulated": {"skills_added": ["PostgreSQL", "Docker"], "matching_jobs": 11, "average_match": 82.1},
  "change": {"jobs_unlocked": 8, "average_match_change": 13.6},
  "learning": {"courses": [...], "total_weeks": 12, "total_cost_inr": 0},
  "constraints": {"within_time_limit": true, "within_budget": true}
}
```
This is auto-invoked at the end of every full pipeline run with the top 4 missing skills.

---

### SUPPLEMENTARY: Skill Combination Optimizer

**File:** [`backend/orchestration/skill_combination_optimizer.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/orchestration/skill_combination_optimizer.py)

Given user constraints (`max_weeks`, `budget_inr`), this evaluates all combinations of missing skills (up to C(n,5)) and ranks them by jobs unlocked within the budget/time limit. Uses the same `rank_jobs()` as everything else — no new logic invented.

---

### SUPPLEMENTARY: Resume Chat

**Endpoint:** `POST /api/profile/resume-chat`

Keeps a rolling 8-message history. Sends Gemini a context-rich prompt with:
- Parsed profile
- Matched jobs, skill gaps, training recommendations

Temperature: 0.3 (low — factual, not creative). Max tokens: 700.  
System instruction: *"Answer using ONLY the supplied resume context. Never invent experience or skills."*

---

## 6. Real-Time Progress Streaming (SSE Architecture)

**File:** [`backend/progress_events.py`](file:///c:/xampp/htdocs/Projects/Skill-Gap-to-Job-Matching-Agent/backend/progress_events.py)

**Why SSE instead of WebSockets?**  
SSE is unidirectional (server → client), much simpler to implement and deploy on Render.com. WebSockets would require a separate WS upgrade and persistent connection management.

**How it works:**
1. `POST /api/skill-gap/analyze/run` → returns `{"run_id": "run_abc123def"}` immediately (HTTP 202)
2. Frontend opens `GET /api/agent/progress/{run_id}` as `EventSource`
3. Each LangGraph node is wrapped by `_tracked_node()` decorator which calls `progress_manager.publish()` twice — once at node start ("running") and once at end ("completed")
4. `ProgressEventManager` uses `asyncio.Queue` per subscriber — thread-safe with `threading.RLock`
5. Events are formatted as:
   ```
   event: progress
   data: {"node":"skill_matching","label":"Semantic Matching","status":"completed","message":"47 candidate matches ranked.","timestamp":"..."}
   ```
6. On workflow completion, a final `event: complete` event is published containing the full result JSON

**Pipeline nodes tracked:**  
`profile_parsing` → `job_search` → `rag_retrieval` → `skill_matching` → `gap_analysis` → `time_to_ready` → `training_recommendations`

---

## 7. Error Handling & Fault Tolerance

Every node follows the same pattern — **never crash the pipeline**:

| Failure Mode | Recovery |
|-------------|---------|
| Jooble API timeout (8s) | Automatically use 50 curated demo jobs |
| RAG/FAISS unavailable | `rag_fallback_node` → continue without evidence |
| Gemini API key missing / rate limit | Deterministic fallback text (same JSON shape) |
| Empty job results | `job_search_fallback_node` → demo jobs |
| Invalid user profile | `error_handler_node` → clean error state returned |
| Course not found for a skill | Skip that recommendation; don't crash |

**`_tracked_node()` decorator:**  
Every node is wrapped — sets `node_status["node_name"] = "completed"` or `"failed"`, emits SSE events. Judges can ask "what node ran?" — the answer is always tracked.

---

## 8. API Endpoints Reference

| Method | Endpoint | Purpose |
|--------|---------|---------|
| `GET` | `/health` | Health check |
| `POST` | `/api/profile/parse` | File upload resume parsing |
| `POST` | `/api/profile/parse-text` | Raw text resume parsing |
| `POST` | `/api/profile/resume-chat` | AI chat about profile |
| `POST` | `/api/voice/transcribe` | Voice → text transcription |
| `POST` | `/api/training/recommend` | Direct training recommendation |
| `POST` | `/api/skill-gap/analyze` | Synchronous full pipeline |
| `POST` | `/api/skill-gap/analyze/run` | Async pipeline (returns run_id) |
| `GET` | `/api/agent/progress/{run_id}` | SSE progress stream |
| `GET` | `/api/agent/runs/{run_id}` | Get final result by run_id |
| `POST` | `/api/skill-combination-optimizer` | Skill combo optimizer |
| `POST` | `/api/career-simulator/simulate` | Career what-if simulation |

---

## 9. Design Decisions & Tradeoffs (For Cross-Questions)

### Q: Why LangGraph instead of LangChain LCEL or a simple function chain?
**A:** LangGraph gives us a **named, inspectable, stateful directed graph**. Each node is a Python function we wrote — we know exactly what runs, in what order, and what each node's inputs/outputs are. With LCEL you lose that per-node observability. With a simple function chain you can't add conditional edges (e.g., "go to fallback if no jobs found"). LangGraph also lets us add `ToolNode` for Gemini native function calling without rewriting the pipeline.

### Q: Why Sentence Transformers (`all-MiniLM-L6-v2`) instead of OpenAI embeddings?
**A:** Three reasons: (1) **Cost** — `all-MiniLM-L6-v2` runs locally at zero API cost; (2) **Speed** — 22MB model, embeds 100 skills in < 100ms on CPU; (3) **Privacy** — candidate skills never leave our server for the matching step. We chose L6-v2 over larger models because the semantic similarity threshold of 0.5 is sufficient for skill matching (short phrases, not paragraphs).

### Q: Why Jooble instead of LinkedIn/Indeed APIs?
**A:** LinkedIn and Indeed APIs are not publicly accessible (they require enterprise partnerships). Jooble has a free tier API key that returns real job postings. Our 10-minute cache means repeated searches for the same role+location don't spam the API.

### Q: What's mocked vs. real?
**Fully real:** Profile parsing (Gemini API), job retrieval (Jooble or demo), semantic matching (real SentenceTransformer), gap analysis (deterministic set operations), course data (real course URLs/providers), training ROI (real formula), Gemini reasoning (real API call with fallback).  
**Simplified:** The course catalog is a static `courses.json` — not a live Coursera/NPTEL API pull. In a production system this would be a live crawler or partnership API. The Gemini tool agent (`gemini_tool_agent_node`) is included but not in the main execution path of the simplified graph build.

### Q: How do you prevent Gemini from hallucinating skills?
**A:** After Gemini returns its JSON, `citation_validator.py` validates that:
- Every skill in `strengths` is in the deterministically computed `matched_skills`
- Every skill in `priority_gaps` is in the deterministically computed `missing_skills`
- Any skill Gemini invents is stripped from the output

The deterministic gap analysis runs **before** Gemini — Gemini only adds narrative explanation, never creates the gap data.

### Q: Why is the ROI formula `(jobs_unlocked / duration_weeks) × 10`?
**A:** It measures **opportunity density** — how many job doors does each week of study unlock? A course that unlocks 8 jobs in 4 weeks (ROI=20) beats one that unlocks 10 jobs in 12 weeks (ROI=8.3). The ×10 is a simple scaler to make the number more readable. Match boost (`match_count × 50`) heavily favors courses that directly address the missing skill over tangentially related ones.

### Q: What happens if two API keys hit rate limits simultaneously?
**A:** `config.py` defines separate keys for profile parsing (`GEMINI_API_KEY_PROFILE_PARSING`) and gap analysis (`GEMINI_API_KEY_GAP_ANALYSIS`). If both fail, the deterministic fallback in `gemini_reasoning_node` generates the same JSON shape so the frontend always receives a valid response.

---

## 10. What Judges Will See in the Live Demo

### Happy Path Flow:
1. Upload a PDF resume → watch the progress bar tick through 7 stages
2. See matched jobs with exact `match_score` (e.g., 78.5%)
3. See `missing_skills` for each job
4. See the recommended course with jobs_unlocked count
5. See the career simulator before/after (current: 3 jobs → after learning: 11 jobs)
6. Ask the resume chatbot "What should I learn first?" — gets a contextual answer

### Edge Cases You Should Know:
| Input | What Happens |
|------|-------------|
| Paste resume as text | Goes through `parse-text` endpoint → same Gemini parsing |
| No Jooble API key in `.env` | Auto-uses 50 demo jobs; `job_source: "demo"` shown |
| Skills with no matching courses | Training recommendation returns empty for that skill, not a crash |
| `free_only: true` flag | All courses filtered to `is_free: true`; cost shown as ₹0 |
| Image resume (JPG) | Gemini Vision reads it; no text extraction needed |

---

## 11. Numbers That Demonstrate Real Impact

- **50 curated demo jobs** covering 15+ roles (Backend Engineer, Data Analyst, DevOps, ML Engineer, etc.)
- **20+ courses** mapped to 30+ skills across Infosys Springboard, NPTEL, Skill India, Coursera, Udemy, GeeksforGeeks
- **Matching engine processes all 50 jobs in < 1 second** (batch embedding with pre-computed candidate matrix)
- **Time-to-ready formula** answers: "Can I be job-ready in 4 weeks?" — with a real yes/no plus specific courses
- **LangGraph pipeline completes in 3–8 seconds** end-to-end (dominated by Gemini API latency on the first job)

---

## 12. File-by-File Quick Reference

```
backend/
├── main.py                          ← FastAPI app, all endpoints registered
├── config.py                        ← API keys, Gemini model, server config
├── graph.py                         ← Early prototype graph (not main pipeline)
├── progress_events.py               ← SSE run tracking (ProgressEventManager)
├── profile_parsing/
│   ├── gemini_client.py             ← Gemini resume parser + skill normalizer
│   ├── document_extractor.py        ← PDF/DOC/TXT text extraction
│   ├── pdf_extractor.py             ← PyMuPDF wrapper
│   └── schemas.py                   ← UserProfile, ResumeProfile Pydantic models
├── skill-match-agent/
│   ├── services/
│   │   ├── matching_service.py      ← 70/30 hybrid scoring + rank_jobs()
│   │   ├── embedding_service.py     ← all-MiniLM-L6-v2 singleton
│   │   ├── jooble_service.py        ← Jooble API client + 10min cache
│   │   ├── job_source_service.py    ← Jooble → normalizer → demo fallback
│   │   ├── job_normalizer.py        ← Raw Jooble JSON → JobPosting models
│   │   ├── skill_extractor.py       ← Extract skills from job description text
│   │   └── normalizer.py            ← Skill normalization for matching
│   └── schemas.py                   ← JobPosting, JobMatchResult
├── gap-analysis-agent/
│   ├── services/
│   │   ├── gap_analysis_service.py  ← analyze_multiple_jobs(), dedupe_skill_list()
│   │   └── gemini_service.py        ← GeminiService.analyze_gap_reasoning()
│   └── schemas.py                   ← GapAnalysis, AiReasoning schemas
├── training-agent/
│   ├── agent.py                     ← recommend_training(), ROI formula
│   ├── time_to_ready.py             ← calculate_time_to_ready(), per-job plans
│   ├── courses.json                 ← 20+ course catalog (static)
│   └── local_course_mcp.py          ← MCP-compatible course tool interface
└── orchestration/
    ├── __init__.py                  ← Exports: run_skill_gap_workflow, simulate_career_path
    ├── skill_gap_graph.py           ← 1406 lines: ALL 16 nodes + graph compilation
    ├── opportunity_analysis.py      ← analyze_opportunities(), skill combos
    ├── career_simulator.py          ← simulate_career_path() before/after
    ├── skill_combination_optimizer.py ← budget/time constrained optimizer
    ├── chart_data.py                ← Build chart-ready career intelligence data
    └── rag/
        ├── retriever.py             ← RAGRetriever.retrieve() main entry
        ├── vector_store.py          ← FAISS + TF-IDF fallback vector store
        ├── document_preparation.py  ← Convert jobs/courses to text documents
        └── citation_validator.py    ← Attach + validate RAG citations on AI output
```

---

## 13. One-Paragraph Demo Script (Memorize This)

*"Today, a fresh graduate in India spends weeks applying to jobs they're almost qualified for, wasting time and money on the wrong courses. PathWise solves this in under 10 seconds. Watch: I upload my resume — Gemini parses it into a structured profile. Our LangGraph pipeline fetches real job postings from Jooble, runs semantic matching using Sentence Transformers, identifies exactly which skills I'm missing, and calculates that learning PostgreSQL via a free NPTEL course will unlock 8 additional job opportunities in 8 weeks — at zero cost. This isn't a story; here are the numbers, here is the course link, and here is the career simulator showing my job count going from 3 to 11 after those skills. That's what PathWise does."*
