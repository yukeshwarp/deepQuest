# deepQuest Application Documentation

## Overview

**deepQuest** is an AI-powered research automation platform featuring two distinct application flows: `dfsapp` (Depth-First Search Application) and `bfsapp` (Breadth-First Search Application). Both allow users to automate and orchestrate large-scale research tasks using LLMs, web search agents, and custom report generation.

## Application Architecture & Workflow

Below is a detailed architecture diagram describing the application workflow:
<img width="1302" height="778" alt="diagram-export-15-7-2025-12_33_32-pm" src="https://github.com/user-attachments/assets/04ed382a-00c9-4b75-a723-a1120f33a5d9" />

---

## Table of Contents

- [Installation](#installation)
- [Application Structure](#application-structure)
- [Core Pipeline](#core-pipeline)
  - [1. Initialization](#1-initialization)
  - [2. Research Planning](#2-research-planning)
  - [3. Step Execution](#3-step-execution)
  - [4. Replanning Logic](#4-replanning-logic)
  - [5. Report Generation & Evaluation](#5-report-generation--evaluation)
  - [6. Output & Download](#6-output--download)
- [Version Differences: dfsapp vs bfsapp](#version-differences-dfsapp-vs-bfsapp)
- [Key Modules](#key-modules)
- [Session Management](#session-management)
---

## Installation

### Prerequisites

- Python 3.8+
- Access to required APIs (Google, ArXiv, NewsAPI, SEC, Wikipedia)
- [Streamlit](https://streamlit.io/)

### Steps

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yukeshwarp/deepQuest.git
   cd deepQuest
   ```

2. **Install dependencies:**
   It's recommended to use a virtual environment. You may find a `requirements.txt` in the repository (if not, install dependencies as shown).
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install streamlit python-dotenv python-docx beautifulsoup4 markdown
   # Install any additional dependencies for web search APIs and OpenAI or compatible LLM client
   ```

3. **Set up environment variables:**
   - Copy `.env.example` (if present) to `.env` and fill in your API keys and credentials.
   - Example variables you may need:
     ```
     OPENAI_API_KEY=your_openai_key
     GOOGLE_API_KEY=your_google_key
     ARXIV_API_KEY=your_arxiv_key
     NEWSAPI_KEY=your_newsapi_key
     SEC_API_KEY=your_sec_key
     WIKIPEDIA_API_KEY=your_wikipedia_key
     ```
   - See the README or source code for the exact variable names.

4. **Run the application:**
   - For Depth-First Search mode:
     ```bash
     streamlit run dfsapp.py
     ```
   - For Breadth-First Search mode:
     ```bash
     streamlit run bfsapp.py
     ```

5. **Open the provided local web URL in your browser and start using deepQuest.**

---

## Application Structure

- `dfsapp.py` – Depth-First Search research orchestration UI.
- `bfsapp.py` – Breadth-First Search research orchestration UI.
- `planner.py` – Step planning and replanning logic (LLM-based).
- `writer.py` – Research report generation and evaluation logic.
- `dfs_stepexecutor.py` – Executes research steps (DFS strategy).
- `bfs_stepexecutor.py` – Executes research steps (BFS strategy).
- Other dependencies: Streamlit, python-docx, BeautifulSoup, markdown, dotenv, and various web search API clients.

---




**Components Explained:**
- **Streamlit UI:** Entry point for user queries, parameter input, and session management.
- **dfsapp.py / bfsapp.py:** Choose the research orchestration strategy (Depth-First or Breadth-First).
- **Step Executors:** Run each research step using LLMs and integrated web agents.
- **planner.py:** Handles planning and dynamic replanning of research steps.
- **writer.py:** Synthesizes final research report and runs automated evaluation/feedback cycles.
- **Session State:** All states (query, steps, report, context) are stored in Streamlit’s session for continuity.

---

## Core Pipeline

### 1. Initialization

- Loads environment variables.
- Configures logging.
- Sets up Streamlit UI with a main title and a sidebar for research steps.
- Initializes session state for query, steps, completed steps, context, and report.

### 2. Research Planning

- On user query input, `plan_research` (from `planner.py`) prompts the LLM to generate a step-by-step research plan, up to a configurable `max_steps` (default 20).
- Steps are presented in the sidebar, and the user can edit, remove, or add steps before proceeding.

### 3. Step Execution

- Execution is performed in parallel batches (batch size = 3).
- Each step is executed via `execute_step`:
  - Prompts the LLM to perform the step, always including detailed web search results (Google, ArXiv, NewsAPI, SEC, Wikipedia).
  - Results are appended to the context and marked as completed in the UI.
- The difference between the two apps:
  - `dfsapp` uses `dfs_stepexecutor.py`
  - `bfsapp` uses `bfs_stepexecutor.py`
  - Each executor uses a slightly different agent or web search orchestration strategy.

### 4. Replanning Logic

- After every 3 steps, the system may call `replanner` to allow the LLM to add more steps if the research is incomplete, respecting the `max_steps` and limiting replanning rounds.

### 5. Report Generation & Evaluation

- Once all steps are done, `report_writer` synthesizes a detailed markdown report, including:
  - Full source attribution.
  - Deep analysis.
  - A bibliography with a count of resources used.
- Optionally, `eval_agent` can be used to evaluate the report against the original research goal, with up to 3 feedback-improvement cycles.

### 6. Output & Download

- The report is displayed in the UI.
- Users can download the final report as a Word document (`.docx`), converted from markdown with preserved headings, bullet/number lists, and tables.

---

## Version Differences: dfsapp vs bfsapp

| Aspect            | dfsapp (Depth-First)                 | bfsapp (Breadth-First)                  |
|-------------------|--------------------------------------|-----------------------------------------|
| Executor module   | `dfs_stepexecutor.py`                | `bfs_stepexecutor.py`                   |
| Web Agent Import  | `deep_web_agent`                     | `spread_web_agent`                      |
| Research Strategy | Web crawler in DFS mode              | Web crawler in BFS mode                 |
| All other logic   | Largely identical, including UI, planning, report writing, etc.                 |

---

## Key Modules

### planner.py

- **plan_research(query, max_steps=20):**  
  Prompts the LLM to generate a numbered, actionable research plan for the query.
- **replanner(context, steps, ...):**  
  Allows the LLM to suggest new steps based on completed research, avoiding duplicates and exceeding max_steps.

### writer.py

- **report_writer(context):**  
  Generates a detailed, fully attributed research report in markdown, including a bibliography.
- **eval_agent(context, research_target, max_attempts=3):**  
  Evaluates if the report meets the research goal, with iterative improvement.

### dfs_stepexecutor.py / bfs_stepexecutor.py

- **execute_step(step, context):**  
  Executes individual research steps using LLM function calling and appropriate web search agents (Google, ArXiv, NewsAPI, SEC, Wikipedia).

---

## Session Management

- All user input and system states (steps, completed steps, context, report) are kept in Streamlit's `st.session_state` for reproducible, interactive research sessions.
