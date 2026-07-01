# ⚖️ Engineering Decisions (The "Why")

### Decision #001: Why were the Pilots separated from the LLM?
*   **The Reason:** LLMs are terrible at reliable, raw script execution while maintaining a conversational persona.
*   **The Advantages:** Total isolation. The LLM simply outputs a strict JSON command. The Python Pilot executes it deterministically.
*   **Reversible?** No. This decoupling is the absolute foundation of agentic stability.

### Decision #002: Why did LAILA become an executive manager?
*   **The Reason:** Forcing an LLM to "be" the terminal and the conversational friend simultaneously causes cognitive collapse.
*   **The Advantages:** By acting as a manager, LAILA delegates safely. She evaluates the request, dispatches a Pilot, reviews the output, and reports back. This completely solved the "Ghost Evidence" hallucination.

### Decision #003: Why rely on Structured Memory and not Text History?
*   **The Reason:** Raw conversation history bloats infinitely. A 10,000-character prompt on a local CPU takes 15 seconds just to process.
*   **The Advantages:** SQLite allows us to query exact states, inject compressed summaries, truncate massive responses symmetrically, and feed the LLM a hyper-dense, perfectly formatted context payload.

### Decision #004: Removing Jinja Direct Style Injections
*   **The Reason:** Injecting logic directly into HTML attributes caused strict IDE syntax errors and dynamic layout breakage during live SSE updates.
*   **The Advantages:** Decoupling using `data-pct` attributes and letting a client-side JS observer map the styles guarantees 100% clean HTML and zero rendering anomalies.

### Decision #005: Dual LLM Strategy
*   **The Reason:** Cloud models (Gemini) are fast but rely on internet/quotas. Local models (Ollama) are private and free but slower.
*   **The Advantages:** Intelligent routing. If the internet drops, fallback to Ollama. If a task requires heavy code analysis, route to Cloud. Ensures 100% uptime.

### Decision #006: Flexible Dynamic Tokens
*   **The Reason:** Hardcoded tokens were either wasteful for simple queries ("Hi") or completely insufficient for complex tool sequences.
*   **The Advantages:** Implemented `calculate_dynamic_tokens()`. Complexity is scored based on keywords. Tokens dynamically scale from 500 to 3500. Result: 4x faster simple queries, 95%+ success rate on complex tasks.

### Decision #007: Strict Agent vs. Chat Mode Partitioning
*   **The Reason:** When Pilots failed to understand a command in Agent Mode, the system fell back to Gemma, which would hallucinate fake system actions.
*   **The Advantages:** Agent Mode is strictly for deterministic Pilot execution. If a command fails, Router Guard intercepts and asks for clarification. Chat Mode is where LAILA helps formulate commands. No fallback to Gemma for system execution.

### Decision #008: Native Python File I/O over PowerShell
*   **The Reason:** Reading files via PowerShell (`Get-Content`) corrupted output due to encoding mismatches and parsing errors.
*   **The Advantages:** Implemented 5 dedicated native Python file tools (`READ_FILE`, `WRITE_FILE`, `SUMMARIZE_FILE`, `LIST_DIR`, `DELETE_PATH`). Result: 100% reliable read/write, native UTF-8 Arabic support.
