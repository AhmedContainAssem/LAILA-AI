# LAILA AI v1.0.0 Beta — Autonomous Desktop AI Workstation

We are excited to announce the inaugural public release of **LAILA AI (v1.0.0-beta)** — an autonomous local AI workstation and desktop agent designed for local sovereignty, multi-provider intelligence, and deep Windows OS copilot automation.

---

## 🌟 Key Highlights & Architectural Features

### 1. Decoupled Dual-Stage Runtime Architecture
- **Desktop Shell (`LAILA.exe`)**: Ultra-lightweight native Windows desktop shell powered by PyWebView2, featuring system tray minimization, single-instance mutex guards, and smooth glassmorphism UI.
- **Backend Service (`laila-backend.exe`)**: High-performance headless API service managing process lifecycles, memory optimization, and cognitive task routing.
- **Embedded Ollama Engine**: Includes local inference binaries with automated daemon management on port `11434`.

### 2. Multi-Provider Hybrid Intelligence & Circuit Breakers
- Seamlessly transition between on-device local models and leading cloud frontier models:
  - **Local Sovereignty**: Ollama (Gemma 4 E2B, Llama 3, DeepSeek, Qwen)
  - **Frontier Cloud**: Google Gemini 2.0 / 1.5, OpenAI (GPT-4o / GPT-5), Groq Cloud, and OpenRouter
- **Floating Engine Error Modal**: Complete protection against context and database pollution. Engine limits (e.g. OpenAI HTTP 429 quota exhaustion, Groq rate limits, provider key expiration) are cleanly intercepted into an interactive floating window with 1-click fallback to local offline models.

### 3. Native SystemPilot & DocumentPilot 4-Engine Suite
- **SystemPilot**: Autonomous Windows automation, process inspection, file system navigation, app launching, and system command orchestration.
- **DocumentPilot Suite**:
  - `ImageEngine`: OCR, layout analysis, format conversions, and computer vision.
  - `PDFEngine`: Document parsing, bidirectional text extraction (full Arabic support), table parsing, and PDF-to-Word conversions.
  - `OfficeEngine`: Word (`.docx`), Excel (`.xlsx`), and presentation data extraction.
  - `TextEngine`: Syntax highlighting, code generation, and markdown transformations.

### 4. Multi-Session Isolated Chat & Full Session Continuity
- **Multi-Session Sidebar**: Isolated session storage per chat thread with live title search and strict context boundaries.
- **Instant Session Continuity**: Cookie and client synchronization ensuring active session persistence and refresh recovery.
- **Dynamic Context-Aware Greetings**: Device-time responsive greetings with exact English & Arabic localization and Easter eggs.
- **Custom Precision UI Cursor**: Custom GPU-composited pinpoint dot with smooth lerp trailing ring and zero idle CPU impact.

### 5. Real-Time Hardware Telemetry Dashboard
- Dedicated `/dashboard` displaying live system vitals:
  - **GPU Acceleration**: Dedicated NVIDIA GPU detection (e.g., GeForce GTX series), real-time GPU Load %, and live VRAM gauge.
  - **CPU & Memory**: Core utilization, active threads, and RAM gauges.

---

## 🚀 Quickstart & Installation

1. **Download Archive**: Download `LAILA-Beta-1.0.zip` from the Assets section below.
2. **Extract**: Extract the contents to any directory (e.g. `C:\LAILA` or `D:\LAILA`).
3. **Launch**: Double-click **`LAILA.exe`** to start the application.
   > **Note**: No Python installation or command-line setup is required. All runtimes, engines, and WebView2 bootstrapper components are self-contained.

---

## 💻 System Requirements

- **Operating System**: Windows 10 (64-bit) or Windows 11 (64-bit)
- **Processor**: Intel Core i5 / AMD Ryzen 5 or better
- **System Memory**: 8 GB RAM minimum (16 GB recommended for running 7B+ local models)
- **Storage**: ~3.5 GB free disk space
- **Graphics (Optional)**: NVIDIA GPU with CUDA support (for accelerated local model inference)

---

## 🔒 Security & Privacy Notice

LAILA AI runs completely private and offline-first by default. Your chats, database records, and files never leave your machine unless you explicitly configure and select a cloud API provider.

---

**Founder & Lead AI Architect**: [Eng. Ahmed Assem](https://www.linkedin.com/in/ahmed-assem-874bb4400/)
**Repository & Documentation**: [AhmedContainAssem/LAILA-AI](https://github.com/AhmedContainAssem/LAILA-AI)
