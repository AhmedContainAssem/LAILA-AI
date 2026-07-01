# 🏗️ Core Architecture & Diagrams

LAILA is a modular, event-driven, local web application utilizing a dual-stage cognitive pipeline. We employ **Reasoning-Action Decoupling**, separating the "thinking/executing" phase from the "talking/reporting" phase.

## 2.1 The Backend (Python & Flask)
*   **Framework:** Flask is used for its lightweight, synchronous nature that is easy to debug.
*   **SSE Streams:** The backend pushes tokens, state changes, and telemetry live to the frontend via Server-Sent Events (SSE) without blocking threads, giving the illusion of instantaneous thought.

## 2.2 The Frontend (Vanilla HTML/CSS/JS)
*   **Framework-less:** Built with pure HTML/JS/CSS. No React/Vue overhead, ensuring a zero-compilation lightweight footprint.
*   **CSS Excellence:** Pure CSS styling (Poppins font, Cyan/Navy `#00B4FF` glow, CSS animations like typing, Checkbox Hacks for sidebars).
*   **DOM Efficiency:** Manipulating the DOM via raw JavaScript allows surgical updates to telemetry bars and streaming text without triggering massive virtual-DOM diffing.

## 2.3 The Memory Engine (SQLite)
Powered by a consolidated, absolute-path SQLite database (`database.py`).
*   **`messages`:** Conversation threads, strictly bound to session IDs.
*   **`summaries`:** Historical state logs injected dynamically.
*   **`system_knowledge`:** Local Knowledge Base (LKB) for persistent facts.
*   **`agent_state`:** Tracks active states (`CHAT`, `ANALYSIS`, `PLANNING`, `EXECUTION`, `FINALIZING`).
*   **`agent_runs_telemetry`:** Ledgers every execution loop for trace inspection.
*   **`tool_executions`:** Stores the exact raw output from the OS.

## 2.4 The Pilots (Actuators)
Pilots are the "hands" of LAILA, executing safely in a deterministic workspace.
*   **SystemPilot:** Handles OS-level interactions, bulk file creation, renaming, and safe deletion.
*   **DocumentPilot:** Handles file reading, writing, parsing, and advanced conversions (e.g., txt to PDF).

## 2.5 Security & Workspace Isolation
*   **Sandbox:** LAILA operates under a strict, secure sandbox path: `J:\LAILA AI pro`.
*   **Claim Verifier Shield:** Intercepts proposed tool outcomes, preventing the LLM from fabricating data.
*   **Run-Group Isolation:** Forces voice synthesis to query only successfully completed tool runs via `run_group_id`.

## System Diagrams & Flowcharts

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
