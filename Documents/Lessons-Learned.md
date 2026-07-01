# 🗝️ Lessons Learned (The Vault)

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
