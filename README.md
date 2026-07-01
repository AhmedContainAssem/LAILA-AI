# 🚀 Layla AI

> "LAILA is not powerful because of the model behind her. She's powerful because of the system around her. The model thinks. The system remembers, acts, observes, and adapts. Together, they become intelligent."

> "Don't make the model work harder. Build a system that lets the model think better."

## 🚀 What is Layla?
LAILA was built to bridge the massive gap between simple conversational chatbots and fully autonomous, local-first OS Agents. The objective was to create a highly intelligent entity capable of securely managing a local Windows workstation environment, completely offline if necessary, without the friction, slow response times, and context amnesia typical of standard LLM wrappers.

## ⚡ Features
- **Local-First & Autonomous:** Capable of running 100% offline via local LLMs (Ollama/Gemma-3 4B).
- **Absolute Grounding:** Zero tolerance for hallucinated evidence. If a tool did not run, LAILA cannot say it ran.
- **Resource Respect:** 0% CPU/GPU usage when idle, strict context token budgets when active.
- **Observability:** Complete transparency into the system's thought process, state, and resource consumption via the telemetry dashboard.

## 🏗 Architectural Overview
LAILA is a modular, event-driven, local web application utilizing a dual-stage cognitive pipeline. We employ **Reasoning-Action Decoupling**, separating the "thinking/executing" phase from the "talking/reporting" phase. It relies on a Python (Flask) backend, an SQLite memory engine, and specialized execution Pilots (SystemPilot, DocumentPilot).

## 🖥 Screenshots
*(Screenshots of the LAILA Dashboard and Chat Interface can be found in the `Screenshots/` directory.)*

## 🎥 Live Demonstrations
For visual overviews and short video demonstrations of the LAILA product in action, please visit the [Featured Section on LinkedIn](https://www.linkedin.com/in/ahmed-assem-874bb4400/details/featured/).

## 🚀 Quick Start
1. **Start Ollama** (with custom models path if needed):
   ```powershell
   $env:OLLAMA_MODELS="J:\Programs"
   ollama serve
   ```
2. **Start Flask Server**:
   ```powershell
   python app.py
   ```
3. **Access the Web UI**:
   Open your browser and navigate to `http://127.0.0.1:5000`.

## 📖 Documentation
- 👉 [Read the Engineering Handbook](Documents/Engineering-Handbook.md)
- 👉 [Read Laila's Philosophy](Documents/Lailas-Philosophy.md)
- 👉 [Read the Architectural Handbook](Documents/Architecture.md)
- 👉 [Read the Design Decisions](Documents/Design-Decisions.md)
- 👉 [Read the Lessons Learned](Documents/Lessons-Learned.md)
- 👉 [Read the Development History](Documents/Development-History.md)
- 👉 [Read the Roadmap](Documents/Roadmap.md)
