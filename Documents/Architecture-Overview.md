<div align="center">

  <img src="../docs/diagrams/hero-banner.svg" alt="LAILA Architecture Overview" width="100%" />

  <br/><br/>

  [![Architecture](https://img.shields.io/badge/Architecture-Dual--Stage%20Decoupled-0A192F?style=for-the-badge&logo=flask&logoColor=00B4FF)](Architecture-Overview.md)
  [![Pilots](https://img.shields.io/badge/Pilots-Modular%20Sub--Engines-7928CA?style=for-the-badge&logo=windows&logoColor=white)](Architecture-Overview.md)
  [![Security](https://img.shields.io/badge/Security-DPAPI%20%7C%20Local%20Sandbox-00F5A0?style=for-the-badge&logo=shield&logoColor=black)](Engineering-Overview.md)

  <br/><br/>

  <img src="../docs/diagrams/four-pillars.svg" alt="The Four Core Pillars" width="100%" />

</div>

<br/>

# 🏗️ LAILA Architecture Overview

LAILA is architected around a dual-stage cognitive pipeline that enforces **Reasoning-Action Decoupling**—strictly segregating the non-deterministic reasoning layer from deterministic actuators and the operating system interface.

---

## 1. Dual-Workspace Decoupled Pipeline

LAILA provides two dedicated runtime workspaces designed to eliminate context pollution and execution bleed:

<div align="center">
  <img src="../docs/diagrams/dual-workspace-pipeline.svg" alt="Dual-Workspace Decoupled Runtime Pipeline" width="100%" />
</div>


---

<div align="center">
  <img src="../docs/diagrams/pilots-deepdive-card.svg" alt="SystemPilot & DocumentPilot Subsystems" width="100%" />
</div>

---

## 2. Deterministic Actuators (The Pilots)

Pilots serve as the high-speed, zero-latency actuators of LAILA. They operate deterministically with **zero LLM tokens and zero prompt inference overhead**.

### 🛠️ SystemPilot
* **Operating System Automation:** Application launching, path resolution, file discovery, and directory inspections.
* **Power Management:** Workstation locking, clean restart, and shutdown with safety guardrails.
* **Action Center Schedulers:** Native Windows 11 notifications, background alarms, and scheduled reminders via `APScheduler`.

### 📄 DocumentPilot (4-Engine Modular Suite)
* **`PdfEngine`:** Surgical PDF manipulations—merging multi-document streams via `pypdf`, page range splitting, rotation (90°/180°/270°), page deletion, keyword searching, and semi-transparent ReportLab vector watermarking.
* **`OfficeEngine`:** Word (`.docx`) document processing, tabular transformations (`.xlsx` ↔ `.csv`), and multi-table extraction from complex PDFs into structured Excel sheets.
* **`ImageEngine`:** Cross-conversion across 13 formats (PNG, JPG, WebP, ICO, TIFF, BMP, PSD, TGA, GIF) and multi-image compilation into unified PDF portfolios.
* **`TextEngine`:** Multi-encoding text parsing, Markdown generation, and Arabic bidirectional layout synthesis (`arabic_reshaper` + `python-bidi`).

---

<div align="center">
  <img src="../docs/diagrams/doc-engine-suite.svg" alt="DocumentPilot 4-Engine Modular Suite" width="100%" />
</div>

---

## 3. Hybrid Memory Hierarchy

LAILA rejects unstructured, infinitely growing raw text histories. The memory architecture is anchored on a local, consolidated **SQLite relational database**:

* **Conversation Threads:** Strictly bound by session IDs with symmetrical truncation to prevent token bloating.
* **Operational Memory (LKB):** Dynamic key-value operational state tracking working directories, active files, and pending conversions.
* **Trace Telemetry:** Every execution loop, tool invocation, and decision path is ledgered into local telemetry storage for full observability.
* **System Knowledge:** User preferences, hardware configurations, and long-term facts stored permanently without token cost.

---

<div align="center">
  <img src="../docs/diagrams/arch-memory-resilience.svg" alt="Hybrid Memory Hierarchy & Circuit Breakers" width="100%" />
</div>

---

## 4. Security, Isolation & Offline Privacy

* **Zero Cloud Leak:** Local inference routes directly to the embedded Ollama engine. No user files, document text, or system metadata ever touch external servers in local mode.
* **Hardware-Backed Encryption:** Provider API credentials (OpenRouter, Gemini, Groq) are protected using the **Windows Data Protection API (DPAPI)**—encrypted with keys tied directly to the user's Windows login.
* **File Safety Gateway:** Dynamic path sanitization, traversal protection, and pre-execution safety confirmations for destructive commands.

---

## 5. Executive Floating Modals (Pilot Studio & Keep-Notebook)

LAILA completely eliminates in-page DOM churn and view-switching latency by hosting mission-critical productivity environments in **Floating Glassmorphic Windows**:

* **Executive Keep-Notebook (`Alt + N`):** Persistent personal knowledge vault backed by an isolated `notebook_notes` SQLite table. Features autonomous AI drafting, interactive Markdown checklists, 6-color glass palettes, and multi-format document exporting (PDF/DOCX/MD/CSV) without touching conversational memory.
* **Executive Pilot Studio (`Alt + P`):** High-speed OS actuator station with a Split-Pane interface. The left pane provides drag-and-drop file staging, native Windows WinForms STA file/folder pickers, and native Windows Explorer file reveals. The right pane provides non-destructive intent preflight previews, multi-model sync, and real-time multithreaded SSE execution streaming.

---

## 6. Multi-Session Sidebar & Google ADK Vector Memory Bank (LTM)

LAILA isolates chat sessions while preserving unified user intelligence through a decoupled memory pipeline:
* **Strict Session Boundaries:** Each conversation thread is indexed by a dedicated `session_id` in SQLite, ensuring zero prompt history bleed between threads.
* **Google ADK Vector Memory Bank (`session_memory_vectors`):** Conceptual session summaries are distilled asynchronously and embedded as dense floating-point vectors stored directly in SQLite for sub-millisecond retrieval.
* **Shared Cognitive Vault (`user_longterm_memory`):** Automatically extracts and clusters recurring user interests, projects, and key facts across all sessions using dense vector cosine similarity.
* **Zero-Regex Semantic Recall ("Do you remember when...?"):** Autonomous intent anchoring (English & Arabic) queries historical conversation vector embeddings without brittle regexes or context bloat.
* **Fast Prior Continuity:** New chat sessions automatically pull the summary of the latest preceding session, ensuring natural flow without context amnesia.
* **Modular Clean Architecture:** Built on strict separation of concerns, ensuring high-throughput local execution, memory safety, and zero context pollution.

---

<div align="center">
  <img src="../docs/diagrams/core-advantages-card.svg" alt="Dual-Stage Memory & Decoupled Workspaces" width="100%" />
</div>

---

<div align="center">

  <img src="../docs/diagrams/runtime-snapshot.svg" alt="Runtime Architecture Snapshot" width="100%" />

  <br/><br/>

  <sub>Designed &amp; Engineered by <b>Ahmed Assem</b> • Founder &amp; Lead AI Architect</sub>

</div>