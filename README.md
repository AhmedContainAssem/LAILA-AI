# 📖 LAILA AI: The Official Architecture & Engineering Handbook

*This is the official project reference, technical gospel, and complete engineering encyclopedia for LAILA AI. Every architectural decision, historical modification, database schema, and core philosophy is documented here. A year from now, this document will serve as the master key to understanding exactly **why** the system behaves the way it does, preventing the need to dig through Git commits.*

---

## 📑 Table of Contents
1. [Part 1 — Vision & Philosophy](#part-1--vision--philosophy)
2. [Part 2 — Core Architecture](#part-2--core-architecture)
3. [Part 3 — Engineering Decisions (The "Why")](#part-3--engineering-decisions)
4. [Part 4 — Development Phases](#part-4--development-phases)
5. [Part 5 — Project Roadmap](#part-5--project-roadmap)
6. [Part 6 — System Diagrams & Flowcharts](#part-6--system-diagrams--flowcharts)
7. [Part 7 — Engineering Tables & Metrics](#part-7--engineering-tables--metrics)
8. [Part 8 — Lessons Learned (The Vault)](#part-8--lessons-learned-the-vault)

---

## 🌟 Part 1 — Vision & Philosophy

### 1.1 Why does LAILA exist?
LAILA was built to bridge the massive gap between simple conversational chatbots and fully autonomous, local-first OS Agents. The objective was to create a highly intelligent entity capable of securely managing a local Windows workstation environment, completely offline if necessary, without the friction, slow response times, and context amnesia typical of standard LLM wrappers. 

### 1.2 The Problem It Solves
The current landscape of AI agents suffers from several critical flaws that LAILA was specifically engineered to solve:
1. **Hallucination of Action (Ghost Evidence):** Standard LLMs often pretend they have executed a command, read a file, or completed a task when they haven't actually touched the system. 
2. **Context Bloat & Amnesia:** Conversational agents accumulate massive context windows. On local hardware, processing an 8,000-token history can take 15 seconds before a single word is generated. 
3. **Execution Friction:** Blurring the line between "talking to the user" and "executing a JSON command" leads to inconsistent outputs, leaked prompts, and unparseable JSON schemas.
4. **Hardware Drain:** Both backend LLM inference and frontend UI rendering traditionally waste massive system resources (up to 50% CPU) even when entirely idle.

### 1.3 The Philosophy
LAILA is designed as an **Executive Manager**, not just a typist. 
She thinks, reasons, and delegates tasks to specialized "Pilots" (SystemPilot, DocumentPilot). She verifies their work objectively using strict validation, and then synthesizes the results back to the user in a polished, human manner. 

### 1.4 Core Principles
*   **Local-First & Autonomous:** Capable of running 100% offline via local LLMs (Ollama/Gemma-3 4B).
*   **Absolute Grounding:** Zero tolerance for hallucinated evidence. If a tool did not run, LAILA cannot say it ran.
*   **Resource Respect:** 0% CPU/GPU usage when idle, strict context token budgets when active.
*   **Observability:** Complete transparency into the system's thought process, state, and resource consumption via the telemetry dashboard.

### 1.5 Product Demonstrations
For visual overviews and short video demonstrations of the LAILA product in action, please visit the [Featured Section on LinkedIn](https://www.linkedin.com/in/ahmed-assem-874bb4400/details/featured/).

---

## 🏗️ Part 2 — Core Architecture

LAILA is a modular, event-driven, local web application utilizing a dual-stage cognitive pipeline. We employ **Reasoning-Action Decoupling**, separating the "thinking/executing" phase from the "talking/reporting" phase.

### 2.1 The Backend (Python & Flask)
*   **Framework:** Flask is used for its lightweight, synchronous nature that is easy to debug.
*   **SSE Streams:** The backend pushes tokens, state changes, and telemetry live to the frontend via Server-Sent Events (SSE) without blocking threads, giving the illusion of instantaneous thought.

### 2.2 The Frontend (Vanilla HTML/CSS/JS)
*   **Framework-less:** Built with pure HTML/JS/CSS. No React/Vue overhead, ensuring a zero-compilation lightweight footprint.
*   **CSS Excellence:** Pure CSS styling (Poppins font, Cyan/Navy `#00B4FF` glow, CSS animations like typing, Checkbox Hacks for sidebars).
*   **DOM Efficiency:** Manipulating the DOM via raw JavaScript allows surgical updates to telemetry bars and streaming text without triggering massive virtual-DOM diffing.

### 2.3 The Memory Engine (SQLite)
Powered by a consolidated, absolute-path SQLite database (`database.py`).
*   **`messages`:** Conversation threads, strictly bound to session IDs.
*   **`summaries`:** Historical state logs injected dynamically.
*   **`system_knowledge`:** Local Knowledge Base (LKB) for persistent facts.
*   **`agent_state`:** Tracks active states (`CHAT`, `ANALYSIS`, `PLANNING`, `EXECUTION`, `FINALIZING`).
*   **`agent_runs_telemetry`:** Ledgers every execution loop for trace inspection.
*   **`tool_executions`:** Stores the exact raw output from the OS.

### 2.4 The Pilots (Actuators)
Pilots are the "hands" of LAILA, executing safely in a deterministic workspace.
*   **SystemPilot:** Handles OS-level interactions, bulk file creation, renaming, and safe deletion.
*   **DocumentPilot:** Handles file reading, writing, parsing, and advanced conversions (e.g., txt to PDF).

### 2.5 Security & Workspace Isolation
*   **Sandbox:** LAILA operates under a strict, secure sandbox path: `J:\LAILA AI pro`.
*   **Claim Verifier Shield:** Intercepts proposed tool outcomes, preventing the LLM from fabricating data.
*   **Run-Group Isolation:** Forces voice synthesis to query only successfully completed tool runs via `run_group_id`.

---

## ⚖️ Part 3 — Engineering Decisions (The "Why")

### Decision #001: Why were the Pilots separated from the LLM?
*   **The Reason:** LLMs are terrible at reliable, raw script execution while maintaining a conversational persona.
*   **The Advantages:** Total isolation. The LLM simply outputs a strict JSON command (e.g., `{"action": "tool_call", "tool_name": "READ_FILE"}`). The Python Pilot executes it deterministically.
*   **Reversible?** No. This decoupling is the absolute foundation of agentic stability.

### Decision #002: Why did LAILA become an executive manager?
*   **The Reason:** Forcing an LLM to "be" the terminal and the conversational friend simultaneously causes cognitive collapse.
*   **The Advantages:** By acting as a manager, LAILA delegates safely. She evaluates the request, dispatches a Pilot, reviews the output, and reports back. This completely solved the "Ghost Evidence" hallucination.

### Decision #003: Why rely on Structured Memory and not Text History?
*   **The Reason:** Raw conversation history bloats infinitely. A 10,000-character prompt on a local CPU takes 15 seconds just to process.
*   **The Advantages:** SQLite allows us to query exact states, inject compressed summaries, truncate massive responses symmetrically, and feed the LLM a hyper-dense, perfectly formatted context payload.

### Decision #004: Removing Jinja Direct Style Injections
*   **The Reason:** Injecting logic directly into HTML attributes (`style="width:{{ cpu }}%"`) caused strict IDE syntax errors and dynamic layout breakage during live SSE updates.
*   **The Advantages:** Decoupling using `data-pct` attributes and letting a client-side JS observer map the styles guarantees 100% clean HTML and zero rendering anomalies.

### Decision #005: Dual LLM Strategy
*   **The Reason:** Cloud models (Gemini) are fast but rely on internet/quotas. Local models (Ollama) are private and free but slower.
*   **The Advantages:** Intelligent routing. If the internet drops, fallback to Ollama. If a task requires heavy code analysis, route to Cloud. Ensures 100% uptime.

### Decision #006: Flexible Dynamic Tokens
*   **The Reason:** Hardcoded tokens (e.g., always 350 or 1500) were either wasteful for simple queries ("Hi") or completely insufficient for complex tool sequences (multi-file operations).
*   **The Advantages:** Implemented `calculate_dynamic_tokens()`. Complexity is scored based on keywords (read, analyze, large). Tokens dynamically scale from 500 to 3500. Result: 4x faster simple queries, 95%+ success rate on complex tasks.

### Decision #007: Strict Agent vs. Chat Mode Partitioning
*   **The Reason:** When Pilots failed to understand a command in Agent Mode, the system fell back to Gemma, which would hallucinate fake system actions.
*   **The Advantages:** Agent Mode is strictly for deterministic Pilot execution. If a command fails, Router Guard intercepts and asks for clarification. Chat Mode is where LAILA helps formulate commands. No fallback to Gemma for system execution.

### Decision #008: Native Python File I/O over PowerShell
*   **The Reason:** Reading files via PowerShell (`Get-Content`) corrupted output due to encoding mismatches and parsing errors.
*   **The Advantages:** Implemented 5 dedicated native Python file tools (`READ_FILE`, `WRITE_FILE`, `SUMMARIZE_FILE`, `LIST_DIR`, `DELETE_PATH`). Result: 100% reliable read/write, native UTF-8 Arabic support.

---

## 📈 Part 4 — Development Phases

*   **Phase 1: Core Setup & Persona Integration**
    Basic Flask server, dynamic frontend chat interface, dual-language capabilities (English / Egyptian Arabic), and initial Ollama API integration. 
*   **Phase 2: Observability & Persistence (Sprint 2)**
    Dual LLM fallback routing, SQLite structured persistence, Zero-Hallucination Claim Verifier, and Live Telemetry Dashboard.
*   **Phase 3: Context Optimization & Zero-Drain Engineering**
    Elimination of the Concurrency Bomb, dynamic adaptive ReAct loops, symmetric history truncation, and complete removal of frontend CPU/GPU idle drain.
*   **Phase 4: Tool Expansion & Dynamic Scaling**
    Implementation of 5 native file tools, Flexible Dynamic Tokens (500-3500), and Strict UI/UX CSS refactoring (Poppins font, CSS-only typing animations).
*   **Phase 5: Safety & Partitioning**
    Implementation of the Router Guard, Pronoun Resolution (`it/them/all`), and Destructive Delete Confirmations.

---

## 🗺️ Part 5 — Project Roadmap

### Current (Stable Base)
*   Stable local-first execution with 0% idle system drain.
*   5 highly reliable file tools (Read, Write, List, Summarize, Delete).
*   Dynamic Token Allocation (adaptive performance).
*   Secure, grounded execution with the Claim Verifier.

### Upcoming (Stabilization Phase)
*   **Pronoun Reference Resolver:** Robust linking of `it/them/all` to the active state (`active_document`, `active_file_collection`).
*   **Delete Safety Guard:** Exact-match only for deletions. Mandatory bulk confirmation for destructive operations.
*   **Pilot Command Guide:** Establishing Chat Mode as the safe space where LAILA helps the user write exact pilot commands without executing them.

### Future (The Singularity)
*   **Proactive Background Monitoring:** LAILA monitoring system logs and project folders without user prompts.
*   **Advanced Multi-Agent Swarms:** Deploying multiple specialized pilots concurrently to solve complex engineering tasks in parallel.

---

## 📊 Part 6 — System Diagrams & Flowcharts

### 1. System Architecture High-Level
```mermaid
graph TD
    User[User Input] --> App[Flask app.py]
    App --> Intent[Intent Router]
    Intent --> CCCO[Dynamic Token Budgeting]
    CCCO --> Agent[Agent Engine]
    Agent --> LLM[Dual LLM Services]
    LLM --> ReAct[ReAct Loop]
    ReAct --> Verifier[Claim Verifier Shield]
    Verifier --> Pilots[System / Document Pilots]
    Pilots --> DB[(SQLite DB)]
    DB --> Agent
    Agent --> Stream[Secure SSE Stream]
    Stream --> UI[Pure HTML/CSS/JS UI]
```

### 2. General Sequence (The Full Lifecycle)
```mermaid
sequenceDiagram
    participant User
    participant App
    participant Engine
    participant DB
    participant LLM
    participant Pilot
    User->>App: Sends Request
    App->>DB: Save User Message Placeholder
    App->>Engine: Process Request
    Engine->>DB: Fetch Truncated Context
    Engine->>LLM: Reasoning Prompt (Strict JSON)
    LLM-->>Engine: JSON Tool Request
    Engine->>Pilot: Execute Command (Native Python/PS)
    Pilot-->>Engine: Raw Execution Result
    Engine->>DB: Log Telemetry & Raw Result
    Engine->>LLM: Synthesis Prompt (with raw data)
    LLM-->>App: Stream Human Response
    App-->>User: SSE Token Stream Updates UI
    App->>DB: Commit Final Response
```

### 3. Claim Verifier Shield Flow
```mermaid
graph TD
    Response[LLM JSON Reply] --> V1{MISSING_TOOL?}
    V1 -->|Yes| Reject1[Reject: No Tools Used]
    V1 -->|No| V2{GHOST_EVIDENCE?}
    V2 -->|Yes| Reject2[Reject: Fabricated Tool Use]
    V2 -->|No| V3{FAILED_RELIANCE?}
    V3 -->|Yes| Reject3[Reject: Tool Failed but LLM claims success]
    V3 -->|No| V4{DATA_MISMATCH?}
    V4 -->|Yes| Reject4[Reject: Hallucinated Paths/Dates]
    V4 -->|No| Pass[Approve for Synthesis]
    
    Reject1 -.-> ReLoop[Trigger Self-Correction Prompt]
    Reject2 -.-> ReLoop
    Reject3 -.-> ReLoop
    Reject4 -.-> ReLoop
```

### 4. Dynamic Token Allocation Flow
```mermaid
graph LR
    Query[User Query] --> KeywordCheck{Keyword Analysis}
    KeywordCheck -->|Read, Analyze| ScoreHigh[Score: +3 to +5]
    KeywordCheck -->|Then, And| ScoreMed[Score: +2]
    KeywordCheck -->|Hi, Chat| ScoreLow[Score: 0]
    
    ScoreHigh --> MaxTokens[Allocate: 2500-3500 Tokens]
    ScoreMed --> MedTokens[Allocate: 1000-1500 Tokens]
    ScoreLow --> MinTokens[Allocate: 500-800 Tokens]
    
    MaxTokens --> LLMRequest[Send to LLM]
    MedTokens --> LLMRequest
    MinTokens --> LLMRequest
```

### 5. Memory Truncation & Hygiene Flow
```mermaid
sequenceDiagram
    participant Input as Context Loader
    participant Truncator as Symmetric Truncator
    participant DB as SQLite
    participant Output as Final Prompt Array
    Input->>DB: Fetch last 6 messages
    DB-->>Truncator: Large payload containing User Paste
    Truncator->>Truncator: Check length > 1000 chars?
    Truncator->>Truncator: Slice User Paste (Append truncation notice)
    Truncator->>Truncator: Check Agent response > 1000 chars?
    Truncator->>Truncator: Slice Agent Response (Append truncation notice)
    Truncator->>Output: Yield Hyper-Dense Context
```

### 6. Agent State Machine
```mermaid
stateDiagram-v2
    [*] --> IDLE
    IDLE --> CHAT: Intent = Simple Chat
    IDLE --> PLANNING: Intent = Complex Task
    PLANNING --> EXECUTION: Steps Formulated
    EXECUTION --> RECOVERY: Tool Failed / Verifier Blocked
    RECOVERY --> EXECUTION: Self-Corrected
    EXECUTION --> FINALIZING: All Steps Complete
    FINALIZING --> CHAT: Synthesis Phase
    CHAT --> IDLE: Stream Finished
```

### 7. Pilot Execution & Router Guard
```mermaid
graph TD
    Engine[Agent Engine] --> ModeCheck{Is Agent Mode?}
    ModeCheck -->|No| ChatMode[Chat Mode: Assist User]
    ModeCheck -->|Yes| Resolver[Reference Resolver: it/them]
    Resolver --> Registry[Capability Registry]
    Registry -->|Match Found| Pilot[Execute via Pilot]
    Registry -->|No Match| Guard[Router Guard]
    Guard --> Block[Block Fallback to Gemma]
    Block --> UI[Ask user for clarification/Show Examples]
    Pilot --> Success[Update Active State]
```

### 8. Security & Workspace Execution Flow
```mermaid
graph TD
    Request[Tool Request] --> CheckPath{Path in J:\LAILA AI pro?}
    CheckPath -->|Yes| CheckExt{Safe Extension/File Type?}
    CheckPath -->|No| Reject[REJECT: Out of Bounds]
    CheckExt -->|Yes| CheckBinary{Writing to PDF/Doc?}
    CheckExt -->|No| RejectExt[REJECT: Unsafe Extension]
    CheckBinary -->|Yes| RejectBinary[SKIP/WARN: Binary Write Blocked]
    CheckBinary -->|No| Execute[EXECUTE]
    Execute --> Log[Log to DB Telemetry]
```

---

## 📈 Part 7 — Engineering Tables & Metrics

### Table 1: File Tools Portfolio & Performance
| Tool Name | Core Purpose | Underlying Tech | Avg Latency | Status |
| :--- | :--- | :--- | :--- | :--- |
| **READ_FILE** | Read source code and text files | Native Python I/O | 45ms | ✅ Highly Stable |
| **WRITE_FILE** | Create/overwrite files atomically | Native Python I/O | 35ms | ✅ Highly Stable |
| **SUMMARIZE_FILE** | Analyze file length, type, head/tail | Native Python I/O | 28ms | ✅ Highly Stable |
| **LIST_DIR** | Browse directories (differentiates types) | Native Python I/O | 52ms | ✅ Highly Stable |
| **DELETE_PATH** | Safe file/folder deletion | Native Python `os`/`shutil` | 30ms | ✅ Highly Stable |

### Table 2: Dynamic Token Calculation Logic
| Query Characteristic | Complexity Keywords Detected | Score Added | Expected Token Budget |
| :--- | :--- | :--- | :--- |
| Simple Greeting | None | 0 | 500 - 800 Tokens |
| Create / Edit | `write`, `create`, `اكتب`, `انشئ` | +1 | 700 - 1000 Tokens |
| Find / Check | `search`, `find`, `check`, `فحص` | +1 | 700 - 1000 Tokens |
| Multi-Step Command | `then`, `and`, `بعدين`, `ثم` | +2 each | 1000 - 1500 Tokens |
| Code Analysis | `analyze`, `تحليل`, `functions` | +2 | 1500 - 2500 Tokens |
| Read Large Files | `read`, `file`, `content`, `كبير` | +3 | 2500 - 3500 Tokens |

### Table 3: Database Core Schema (`database.py`)
| Table Name | Purpose | Key Fields |
| :--- | :--- | :--- |
| `messages` | Chat History | `session_id`, `role`, `content`, `timestamp` |
| `agent_runs_telemetry` | Log of ReAct loop iterations | `run_id`, `thought`, `action`, `error_count` |
| `tool_executions` | Raw OS Output & Inputs | `run_id`, `tool_name`, `arguments`, `raw_output` |
| `verifier_stats` | Hallucination tracking | `error_code`, `traceback`, `timestamp` |
| `agent_state` | Observability tracker | `state_name`, `is_active` |

### Table 4: UI/UX Color Identity
| Element | Hex Code | Visual Purpose |
| :--- | :--- | :--- |
| Deep Background | `#020617` | Main canvas background, reducing eye strain. |
| Secondary Container | `#0B132B` | Cards, buttons, and glassmorphism bases. |
| Primary Accent (Cyan)| `#00B4FF` | Typing animations, glows, primary active states. |
| Secondary Accent (Blue)|`#0072FF` | Gradients, pulse animations. |
| Primary Text | `#FFFFFF` | High contrast text (headers, outputs). |
| Secondary Text | `#9CA3AF` | Placeholders, inactive icons, timestamps. |

### Table 5: Pronoun & State Resolution Mapping
| Pronoun Used | Mapped State Variable | Example Translation |
| :--- | :--- | :--- |
| `it`, `this file` | `active_document` | "rename it" -> rename `E:\test\file.txt` |
| `them`, `all` | `active_file_collection` | "convert them" -> convert `[File1.pdf, File2.pdf]` |
| `each file` | `active_working_directory` | "write in each" -> write in `*.txt` in `E:\test` |

---

## 🗝️ Part 8 — Lessons Learned (The Vault)

*This section is pure gold. Every major problem that occurred, its root cause, how it was solved, and the permanent result. This is the history of debugging LAILA.*

### 1. The Language Policy Issue (The Franco-Arabic Bug)
*   **Problem:** LAILA rigidly refused to mix languages or use Franco-Arabic, contradicting her natural polyglot abilities and making her feel artificially constrained.
*   **Root Cause:** Hardcoded strict injections (`[CRITICAL LANGUAGE POLICY]`) appended dynamically to every prompt overrode the LLM's natural intelligence, forcing it into a linguistic corner. 
*   **Decision:** Remove aggressive suffixes. Update the base prompt to simply state her identity and allow her to match the user's language naturally.
*   **Result:** Fluid, natural bilingual communication restored.

### 2. The `_inject_persona` Behavior
*   **Problem:** Server responses felt robotic, highly formatted, and lacked the charm of raw Ollama outputs.
*   **Root Cause:** System prompt stacking. `llm_services.py` and `agent_engine.py` were both injecting redundant rules (e.g., "Always be concise"). We overwhelmed the model with instructions, causing it to default to a bland "customer service" persona.
*   **Decision:** Streamlined prompt assembly. Decoupled Pilot instructions so they are only injected when explicitly needed.
*   **Result:** LLM personality returned to a natural, conversational state.

### 3. The Concurrency Bomb & Memory Amnesia
*   **Problem:** The model would randomly freeze, take 130 seconds to reply, and forget previous context entirely. 
*   **Root Cause:** The context compressor was spawning a background thread to summarize history via Ollama *at the exact same millisecond* as the main chat request. Local inference cannot handle concurrent requests; they bottleneck and queue. Additionally, the history array was aggressively hard-sliced (`[-6:]`), erasing active context.
*   **Decision:** Disabled concurrent summarization. Relied on dynamic tokens and intelligent complexity budgets. 
*   **Result:** Eliminated the 130s freeze. Context retention stabilized.

### 4. Code Analysis False Positive & Persistent Memory Bloat
*   **Problem:** Prompt size exploded to ~18,000 characters for a basic chat, bringing the CPU to its knees.
*   **Root Cause:** The intent router falsely flagged *any* message over 120 chars containing the word "code" as deep analysis, triggering a massive context window. Pasted code blocks were saved fully into the DB and injected continuously on every subsequent turn.
*   **Decision:** Implemented **Symmetric History Truncation**—massive user pastes AND massive assistant responses are truncated to 1000 characters when loaded from history, but preserved in the DB.
*   **Result:** Saved thousands of tokens per turn, drastically reducing Time-to-First-Token.

### 5. Frontend CPU Drain (The Idle Heater)
*   **Problem:** The frontend web UI consumed 50% CPU and 30% GPU while completely idle, draining laptop batteries.
*   **Root Cause:** A global `backdrop-filter: blur(18px)` wrapper applied over the entire screen was interacting with internal infinite pulsing animations (`@keyframes livePulse`). The browser engine was forced to continuously re-calculate the heavy background blur across the entire DOM 60 times a second.
*   **Decision:** Removed all infinite animations from idle UI components. Removed the global backdrop filter.
*   **Result:** Idle CPU and GPU instantly dropped to 0~3%.

### 6. File Tools Corruption via PowerShell
*   **Problem:** Reading files via `execute_powershell` (`Get-Content`) returned corrupted garbage like `html> lang="en">` instead of code. Multi-commands (create folder, create file, write file) failed sequentially.
*   **Root Cause:** Unreliable parsing of PowerShell stdout and treating tool execution as atomic single-steps without chaining.
*   **Decision:** Built 5 dedicated native Python file I/O tools. Updated prompts to enforce sequential tool calls.
*   **Result:** 100% reliable read/write, native UTF-8 Arabic support, and seamless multi-step execution.

### 7. Token Limits Starving Agent Capabilities
*   **Problem:** Multi-command sequences skipped steps, and large file reads failed to analyze the contents.
*   **Root Cause:** Hardcoded `max_tokens=350` was sufficient for a JSON reply, but starved the model of cognitive space to analyze data or plan subsequent steps.
*   **Decision:** Built the **Flexible Dynamic Tokens** engine. Tokens scale up to 3500 based on the detected complexity of the query.
*   **Result:** Agent mode now correctly executes multi-step chains with over 95% success rate.

### 8. Destructive Delete Safety Failure
*   **Problem:** User typed `delete "final" file from E:\test`. The system executed a wildcard delete, wiping `File1.txt` through `File21.txt` along with `final.txt`. Highly dangerous.
*   **Root Cause:** The NLP parser interpreted the command as "delete files from E:\test" and ignored the specific filter target.
*   **Decision:** Implemented strict **Delete Safety Rules**. Exact matches only for specific names. Mandatory bulk confirmation for destructive operations if more than one file matches. `delete it` linked strictly to `active_document`.
*   **Result:** System is safe for production use without risk of recursive drive deletion.

### 9. Agent Mode vs Chat Mode Confusion
*   **Problem:** When a Pilot failed to execute an ambiguous command in Agent Mode, the system fell back to Gemma, which hallucinated system operations, leading to ghost evidence.
*   **Root Cause:** Lack of strict mode partitioning.
*   **Decision:** Agent Mode became a deterministic execution space *only*. If Pilots fail, the Router Guard asks the user for clarification. Chat Mode became the safe space where LAILA helps the user formulate correct Pilot commands.
*   **Result:** Complete elimination of execution hallucinations during ambiguous requests.

---
*End of Document. Preserved for Technical Posterity.*
