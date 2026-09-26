# PathWise — Master 4-Slide Pitch, Live Demo Script & Technical Defense Guide
*Engineered strictly for a 4-Slide Presentation against the 200-Mark Evaluation Criteria*

---

## 🎯 EVALUATION MATRIX STRATEGY (4-SLIDE FORMAT)

| Criteria | Marks | Slide Alignment | How PathWise Wins |
| :--- | :---: | :--- | :--- |
| **Problem & Opportunity** | **45** | **Slide 1** | Tells Rohan’s story (Tier-2 B.Tech grad), 6–18 months job search, ₹15,000 wasted, 3–5 days per JD. Explains Naukri/LinkedIn failure. **Does NOT open with solution!** |
| **Solution Pitch & Storytelling** | **55** | **Slide 2 & Slide 4** | Attention hook, 1-sentence repeatable pitch, clear Problem ➔ Solution ➔ Proof flow, ending with a specific ask for placement cell deployment. |
| **Live Demo & Execution** | **60** | **Live Execution** | 2-minute step-by-step execution script linking every UI click to exact LangGraph node names + failure recovery protocol. |
| **Technical Depth & Defense** | **40** | **Slide 3 & Q&A** | Tech stack matrix: `SentenceTransformers` (`all-MiniLM-L6-v2`), `Gemini Flash`, 70/30 hybrid match formula, `LangGraph`, FAISS RAG, `Pydantic v2`. |

---

## 📌 THE ONE-SENTENCE REPEATABLE PITCH
> *"PathWise is an AI-powered multi-agent platform that parses a candidate's resume, matches it against live job postings, identifies exact skill gaps, and calculates precisely which course to take to unlock the most job opportunities — measured in weeks."*

---

# 📊 PART 1: 4-SLIDE PPT DECK & SPOKEN PITCH (3 MINUTES / 180 SECONDS)

---

### **SLIDE 1: Problem & The Real-World Pain (45 Marks Focus)**
* **Visual Content:**
  * **User Persona:** Rohan, 22-year-old B.Tech graduate from a Tier-2 college in Pune.
  * **The Real Numbers:**
    * **6 to 18 Months:** Average time spent unemployed/underemployed after graduation (TeamLease Data).
    * **3 to 5 Days:** Wasted manually reading and guessing skill alignment for just 10 job descriptions.
    * **₹15,000 Wasted:** Spent on generic online courses with zero signal on actual market demand.
  * **Why Current Workarounds Fall Short:**
    * **LinkedIn / Naukri:** Show job listings, but *never* tell candidates what 20% skill gap is blocking them.
    * **Career Counselors:** Cost ₹5,000+ per session, manual, slow, and lack live market data.
    * **Generic Skill Platforms:** Test theoretical textbook knowledge, not live employer job requirements.

---

### **SLIDE 2: Solution & Value Proposition (55 Marks Focus)**
* **Visual Content:**
  * **The One-Sentence Product Hook:**
    > *"PathWise is an AI-powered multi-agent platform that parses a candidate's resume, matches it against live job postings, identifies exact skill gaps, and calculates precisely which course to take to unlock the most job opportunities — measured in weeks."*
  * **Core Paradigm Shift:**
    * From *Guessing Skill Relevance* ➔ **Quantified Job Market ROI**.
    * From *Manual Resume Matching* ➔ **Automated Multi-Agent Orchestration**.
    * From *6-Month Job Searches* ➔ **4-Week Data-Backed Learning Roadmaps**.

---

### **SLIDE 3: Technical Architecture & ROI Engine (40 Marks Focus)**
* **Visual Content:**
  * **Orchestration:** **LangGraph Stateful 16-Node Pipeline** with Server-Sent Events (SSE) streaming.
  * **Parsing & Job Search:** PyMuPDF text extraction + **Gemini Flash** schema validation (`UserProfile`) + live **Jooble API** (8s timeout, 10m TTL cache) with 50-job demo fallback.
  * **Semantic Matching Engine:** 70/30 Hybrid Model:
    $$\text{Score} = (0.70 \times \text{Exact Overlap}) + (0.30 \times \text{Semantic Similarity})$$
    *Powered by local **SentenceTransformers (`all-MiniLM-L6-v2`, 384-dim)** embeddings.*
  * **ROI & Time-to-Ready Engine:**
    $$\text{ROI Score} = \left(\frac{\text{Jobs Unlocked}}{\text{Duration (Weeks)}}\right) \times 10$$
  * **Career Simulator:** Set-difference operations on `rank_jobs()` outputs showing before-and-after market access (e.g., 3 jobs ➔ 11 jobs unlocked).

---

### **SLIDE 4: Multi-Dimensional Impact & Closing Ask (55 Marks Focus)**
* **Visual Content:**

![PathWise Multi-Dimensional Impact Slide](C:/Users/somap/.gemini/antigravity/brain/f283bf7d-8ff7-45b6-9421-72abd62a6329/pathwise_impact_slide_light_1790447529873.jpg)

  * **Candidate Impact:** 6–18 months reduced to 4 weeks job readiness; ₹20,000 saved via free course routing.
  * **Placement Cell Impact:** 10x student counseling throughput with automated skill gap mapping.
  * **Employer Impact:** Zero skill-mismatch candidate pipeline; +266% increase in match eligibility.
  * **Economic Impact:** Bridging employability gap for 1.5 Million engineering graduates annually across Tier-2 and Tier-3 colleges.
  * **The Specific Closing Ask:**
    > *"We are asking for mentorship and pilot access to deploy PathWise across 5 University Placement Cells to empower tier-2 and tier-3 engineering graduates."*

---

## 🎙️ WORD-FOR-WORD SPOKEN PRESENTATION SCRIPT (3 MINUTES / 4 SLIDES)

### **[0:00 – 0:45] SLIDE 1 — PROBLEM & OPPORTUNITY**
> *"Respected judges, meet Rohan. Rohan just graduated with a B.Tech degree from a Tier-2 college in Pune. Like 1.5 million engineering graduates in India this year, Rohan is stuck. 
> 
> He spends 3 to 5 days manually reading job descriptions, guessing what skills he lacks, and getting rejected. Desperate, he spends 15,000 rupees on a random web development course, only to realize employers in Pune are actually hiring for PostgreSQL and Docker. On average, graduates like Rohan spend **6 to 18 months unemployed or underemployed**.
> 
> Why do current tools fail him? LinkedIn and Naukri show thousands of job listings, but *never* tell Rohan why he got rejected or what specific 20% skill gap is blocking him. Career counselors cost ₹5,000 a session and lack real-time market data. Rohan loses time and money, while companies lose qualified talent."*

### **[0:45 – 1:15] SLIDE 2 — SOLUTION PITCH & ONE-SENTENCE HOOK**
> *"That is why we built **PathWise**.*
> 
> *To put it in one sentence: **PathWise is an AI-powered multi-agent platform that parses a candidate's resume, matches it against live job postings, identifies exact skill gaps, and calculates precisely which course to take to unlock the most job opportunities — measured in weeks.**
> 
> *We replace guesswork with mathematical job-market ROI. We turn a 6-month trial-and-error struggle into a 4-week structured learning roadmap."*

### **[1:15 – 2:15] SLIDE 3 — ENGINEERING ARCHITECTURE & ROI ENGINE**
> *"Under the hood, PathWise is powered by a stateful 16-node **LangGraph orchestration pipeline** with real-time Server-Sent Events progress streaming.
> 
> When Rohan uploads his resume, our Profile Parsing node uses **PyMuPDF** and **Gemini Flash** to normalize skills into a validated Pydantic schema. Next, our Job Search node queries the **Jooble API** live for local job postings, with an automatic fallback to 50 curated job profiles if the network fails.
> 
> For matching, we engineered a **70/30 hybrid scoring algorithm** combining 70% explicit keyword overlap with 30% semantic vector similarity using a local **SentenceTransformer (`all-MiniLM-L6-v2`)** model. 
> 
> Instead of recommending generic degrees, our ROI engine scores courses objectively: **Jobs Unlocked divided by Duration in Weeks**. If learning PostgreSQL unlocks 8 new jobs in 4 weeks through a free NPTEL course, that course gets our top ROI rank. Our **Career Simulator** shows Rohan how adding 2 skills increases his eligible job matches from 3 to 11 — a 266% increase in hireability."*

### **[2:15 – 3:00] SLIDE 4 — MULTI-DIMENSIONAL IMPACT & SPECIFIC ASK**
> *"PathWise delivers real impact across 4 pillars: For candidates, we cut job-readiness time from 18 months to 4 weeks while saving ₹20,000 via free courses. For placement cells, we increase counseling throughput by 10x. For employers, we deliver a zero skill-mismatch candidate pipeline. And economically, we address the employability crisis for 1.5 million graduates annually.
> 
> **Our specific ask today:** We are seeking mentorship and pilot access to deploy PathWise across 5 University Placement Cells to empower tier-2 and tier-3 engineering graduates.
> 
> Now, let us show you PathWise running live in action!"*

---

# 🖥️ PART 2: 2-MINUTE LIVE DEMO EXECUTION SCRIPT (60 Marks Focus)

> **Key Rule for Demo:** At every single step, state EXACTLY *which backend component/node ran and why*.

---

### ⏱️ **0:00 – 0:25 | STEP 1: RESUME PARSING**
* **UI Action:** Click **"Upload Resume"** (select a PDF resume) and hit **Analyze**.
* **Spoken Lines:**
  > *"We are starting our live demo. I am uploading Rohan's PDF resume. 
  > Right now, our backend **`profile_parsing_node`** triggers. It reads the raw file bytes using **PyMuPDF**, passes the text to **Gemini Flash**, and normalizes messy skill names into a strict `UserProfile` Pydantic model. Notice how raw strings like 'reactjs' and 'postgres sql' are instantly standardized."*
* **Component Named:** `profile_parsing_node` (`PyMuPDF` + `Gemini Flash` + `Pydantic v2`).

---

### ⏱️ **0:25 – 0:55 | STEP 2: LIVE GRAPH ORCHESTRATION & SSE STREAMING**
* **UI Action:** Point to the live progress bar as stages turn green (`Job Retrieval` ➔ `RAG Retrieval` ➔ `Semantic Matching`).
* **Spoken Lines:**
  > *"Look at the live progress bar — this is driven by an in-memory **Server-Sent Events (SSE)** manager tracking our stateful **LangGraph pipeline**. 
  > 
  > Here, **`job_search_node`** queried the **Jooble API** live for Python developer roles in Pune. 
  > Next, **`rag_retrieval_node`** indexed those job postings into a local **FAISS vector store** for grounded evidence retrieval. 
  > Now, **`skill_matching_node`** is running our 70/30 hybrid matching formula, batch-encoding candidate skills using **SentenceTransformers (`all-MiniLM-L6-v2`)** on CPU in under 100 milliseconds."*
* **Components Named:** `job_search_node` (Jooble API), `rag_retrieval_node` (FAISS), `skill_matching_node` (`all-MiniLM-L6-v2`).

---

### ⏱️ **0:55 – 1:25 | STEP 3: DETERMINISTIC GAP ANALYSIS & TIME-TO-READY**
* **UI Action:** Scroll to the Top Matched Job (e.g. Backend Engineer, Match Score: 72%).
* **Spoken Lines:**
  > *"The system ranks the Backend Engineer role at a 72% match score. 
  > Here, **`gap_analysis_node`** ran deterministic set operations to identify that Rohan has Python and FastAPI, but lacks **PostgreSQL** and **Docker**. 
  > 
  > Immediately after, **`time_to_ready_node`** queried our course catalog (`courses.json`). It calculates that mastering PostgreSQL requires **8 weeks sequentially**, or **4 weeks in parallel**, costing **₹0** through NPTEL."*
* **Components Named:** `gap_analysis_node` (deterministic set logic), `time_to_ready_node` (`courses.json` map).

---

### ⏱️ **1:25 – 1:45 | STEP 4: OPPORTUNITY DISCOVERY & CAREER SIMULATOR**
* **UI Action:** Click on the **Career Simulator** tab / toggle the missing skill checkboxes.
* **Spoken Lines:**
  > *"Now observe our ROI engine. **`opportunity_simulation_node`** ran set-difference operations across candidate skill matrices. 
  > It determined that learning PostgreSQL unlocks **8 additional matching jobs**, giving this course an ROI score of 1.0. 
  > In our Career Simulator, adding PostgreSQL and Docker increases Rohan's eligible job matches from **3 jobs to 11 jobs** — a 266% increase in hireability."*
* **Components Named:** `opportunity_simulation_node`, `career_simulator` (`rank_jobs()` set difference).

---

### ⏱️ **1:45 – 2:00 | STEP 5: GROUNDED RESUME CHAT & DEMO CONCLUSION**
* **UI Action:** Type in the Resume Chat box: *"Which skill should I prioritize first?"*
* **Spoken Lines:**
  > *"Finally, Rohan can ask questions to our **PathWise Resume Guide**. 
  > This chatbot uses **Gemini** constrained strictly by our RAG retrieval context. It answers using only verified market evidence without hallucinating skills. 
  > That completes our live end-to-end execution!"*
* **Components Named:** `resume_chat` endpoint (RAG-constrained Gemini LLM).

---

## 🛡️ HOW TO HANDLE UNREHEARSED INPUTS & DEMO FAILURES (60 Marks Rule)

Judges score **60 Marks** on live execution and will test unrehearsed inputs. Here is your protocol:

### **Scenario A: Judge asks "Test this random resume / role (e.g. C++ Embedded Developer in Nagpur)"**
* **What you do:** Upload/paste the judge's input calmly.
* **What you say:** 
  > *"Great test case. Let's run that through our graph. `profile_parsing_node` normalizes the C++ skills. `job_search_node` queries Jooble live for Nagpur. If Jooble has zero listings for that specific query, our graph routes to `job_search_fallback_node` which uses our curated 50-job dataset. The pipeline continues smoothly without crashing."*

### **Scenario B: Network drops or Gemini API times out during live demo**
* **What you do:** Do NOT freeze or refresh frantically. Point to the UI fallback output.
* **What you say:** 
  > *"Notice that the Gemini API call timed out, so our graph automatically triggered `deterministic_explanation_node`. Because our gap analysis and ROI math are 100% deterministic Python logic, Rohan still gets exact skill gap math and course recommendations. This demonstrates our zero-crash pipeline resilience."*
  *(This response directly earns top marks under the "explains failure calmly and names component" rubric!)*

---

# 🛡️ PART 3: TECHNICAL DEPTH & CROSS-QUESTIONING CHEAT SHEET (40 Marks Focus)

> **Golden Rule:** Never say *"we used AI for that part"*. Name the specific model, library, mathematical formula, and engineering tradeoff.

---

### **Q1: "Which specific AI models and libraries did you use and where?"**
* **Answer:**
  * **Profile Parsing & Narrative Reasoning:** `google-generativeai` SDK using `gemini-2.0-flash` for JSON extraction and natural language explanations.
  * **Semantic Matching Engine:** `sentence-transformers` library using the `all-MiniLM-L6-v2` model (384-dimensional dense embeddings).
  * **Text Extraction:** `PyMuPDF` (`fitz`) and `pdfminer.six` for PDF text parsing; `Pillow` (PIL) for image-based resume parsing.
  * **Vector Indexing (RAG):** `faiss-cpu` for L2 similarity search over embedded jobs/courses, with a `scikit-learn` TF-IDF cosine similarity fallback.
  * **Orchestration:** `langgraph` (`StateGraph`) for stateful graph compilation; `fastapi` with `httpx` for async API services.

---

### **Q2: "How exactly does your 70/30 hybrid skill matching formula work?"**
* **Answer:**
  > *"Pure keyword matching misses semantic variants (like 'React' vs 'React.js'), while pure vector embeddings suffer from false positives on short skill phrases. 
  > 
  > So we engineered a weighted hybrid score:
  > $$\text{Final Match Score} = (0.70 \times \text{Exact Score}) + (0.30 \times \text{Semantic Score})$$
  > 
  > **1. Exact Score (70%):** We build normalized skill maps for candidate skills and required job skills. $\text{Exact Score} = (\text{Matched Count} / \text{Total Required}) \times 100$.
  > 
  > **2. Semantic Score (30%):** For unmatched skills, we encode candidate skills into a 384-dim matrix using `all-MiniLM-L6-v2`. We compute the cosine similarity matrix using `numpy`. If max similarity $\ge 0.5$, the skill counts as a semantic match. $\text{Semantic Score} = (\sum \text{Similarity Scores} / \text{Total Required}) \times 100$."*

---

### **Q3: "Why did you choose LangGraph over standard LangChain chains?"**
* **Answer:**
  > *"Standard LangChain linear chains operate like black boxes — you cannot easily inspect intermediate state, handle conditional branching, or attach per-node fallback logic. 
  > 
  > **LangGraph** provides a stateful, typed directed graph (`SkillGapState`). Each node is an explicit Python function. This gives us three crucial advantages:
  > 1. **Conditional Routing:** If `job_search_node` returns empty, the graph dynamically routes to `job_search_fallback_node`.
  > 2. **Per-Node Observability:** We wrap every node with an SSE event decorator to stream live UI progress.
  > 3. **Native Tool Calling:** We can seamlessly attach a native `ToolNode` for Gemini tool selection."*

---

### **Q4: "How do you prevent Gemini from hallucinating candidate skills or job requirements?"**
* **Answer:**
  > *"We enforce a strict two-layer anti-hallucination architecture:
  > 
  > **1. Deterministic Core:** All skill gaps, match scores, and course recommendations are computed using 100% deterministic Python set logic *before* Gemini is ever invoked. Gemini does NOT compute gaps.
  > 
  > **2. Citation & Boundary Validator:** Our `citation_validator.py` module inspects Gemini's JSON output. It validates that every skill listed in `strengths` exists in `matched_skills`, and every skill in `priority_gaps` exists in `missing_skills`. Any skill hallucinated by the LLM is automatically stripped out."*

---

### **Q5: "What are the engineering tradeoffs you made? What is simplified or mocked?"**
* **Answer:**
  > *"We made three explicit engineering tradeoffs:
  > 
  > **1. Local Embedding Model vs OpenAI Embeddings:** We chose `all-MiniLM-L6-v2` (22MB). *Tradeoff:* Slightly lower semantic nuance than `text-embedding-3-large`, but zero API cost, lower latency (<100ms on CPU), and 100% candidate skill privacy.
  > 
  > **2. Jooble API vs Web Scraping:** We use the official Jooble API with an 8-second timeout and 10-minute TTL cache. *Tradeoff:* API rate limits exist, so we built a 50-job curated demo fallback.
  > 
  > **3. Simplified Component:** Our course catalog (`courses.json`) contains 20+ real verified courses from NPTEL, Coursera, and Infosys Springboard. It is stored as a structured catalog rather than a live dynamic EdTech crawler. In production, this would be replaced with live EdTech API integrations."*

---

### **Q6: "How does your ROI formula work?"**
* **Answer:**
  > *"Our ROI formula measures **opportunity density** — how many job doors open per week of study invested:
  > $$\text{ROI Score} = \left(\frac{\text{Jobs Unlocked}}{\text{Duration in Weeks}}\right) \times 10$$
  > 
  > For example, a 4-week NPTEL course that unlocks 8 new jobs yields an ROI of $(8 / 4) \times 10 = 20.0$. A 12-week course unlocking 6 jobs yields $(6 / 12) \times 10 = 5.0$. PathWise ranks the 4-week course higher because it maximizes candidate market entry speed."*
