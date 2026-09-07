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

<div align="center">

  <img src="../docs/diagrams/runtime-snapshot.svg" alt="Runtime Architecture Snapshot" width="100%" />

  <br/><br/>

  <sub>Designed &amp; Engineered by <b>Ahmed Assem</b> • Lead AI Architect</sub>

</div>