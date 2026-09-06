# RFC-001: LAILA Runtime Foundational Architecture

> **Document Status:** Official Baseline / RFC-001 (CLOSED & FINAL)  
> **Location:** `Documents/LAILA_Runtime_Foundational_Architecture.md`  
> **Target Audience:** Core Architects, System Engineers, and Extension Developers  
> **Primary Rule:** Every Pull Request or feature addition must answer:  
> *"Where does this component fit within LAILA Runtime, and what state does it operate in?"*

---

## 1. Vision & Philosophical Principles

LAILA Runtime is a zero-dependency, self-healing **Runtime Platform** (analogous to the JVM, .NET Runtime, or Electron Main Process) designed to orchestrate desktop AI applications.

### Core Principles

1. **Zero-Dependency User Experience:** The user simply downloads, installs, and launches LAILA. They are never required to install or manage Python, Ollama, CUDA, or models.
2. **Decoupled Architecture:** LAILA Runtime is 100% agnostic of the underlying AI model (Gemma, DeepSeek, Qwen, or Cloud APIs). The backend or model can be completely swapped without modifying the Runtime Platform.
3. **Strict Division of Responsibility:**
   - **LAILA Runtime Platform:** Manages processes, TCP ports, configuration, self-healing, plugins, MCP host, and application lifecycle.
   - **LAILA Backend Engine:** Manages AI domain logic, warm-up execution, memory hygiene, prompt engineering, database persistence, and pilots.
   - **Shared Security Layer (`core/security`):** Manages OS-native credential encryption (Windows DPAPI) shared across Runtime and Backend.
4. **5-Stage Self-Healing Supervisor:** System or process failures (crashes, port conflicts, service drops) trigger a multi-stage recovery pipeline with escalation thresholds.
5. **Event-Driven Subsystem Communication:** All Runtime components communicate exclusively via the internal Event Bus to ensure 100% loose coupling.
6. **Zero Developer Credentials:** Release builds must never ship with developer API keys. User-configured credentials (Gemini, Groq, OpenRouter) are encrypted via Windows DPAPI.

---

## 2. System Layering & Component Architecture

```
                     ┌───────────────────────────┐
                     │           USER            │
                     └─────────────┬─────────────┘
                                   │
                     ┌─────────────▼─────────────┐
                     │        Frontend UI        │
                     │  (Tauri / Webview Window) │
                     │  + Provider Setup Wizard  │
                     └─────────────┬─────────────┘
                                   │  IPC / WebSockets
 ┌─────────────────────────────────▼──────────────────────────────────┐
 │                       LAILA RUNTIME PLATFORM                       │
 │                    (System Supervisor & Host)                      │
 │                                                                    │
 │ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────────┐ │
 │ │ Process Manager  │ │ Environment &    │ │ 5-Stage Recovery     │ │
 │ │ (Job Objects)    │ │ Port Manager     │ │ Pipeline Subsystem   │ │
 │ └──────────────────┘ └──────────────────┘ └──────────────────────┘ │
 │ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────────┐ │
 │ │ State Machine    │ │ Shared Security  │ │ Config, Update &     │ │
 │ │ Supervisor       │ │ (core/security)  │ │ Telemetry Host       │ │
 │ └──────────────────┘ └──────────────────┘ └──────────────────────┘ │
 │ ┌────────────────────────────────────────────────────────────────┐ │
 │ │            Internal Event Bus (Pub/Sub Broker)                 │ │
 │ └────────────────────────────────────────────────────────────────┘ │
 └─────────────────────────────────┬──────────────────────────────────┘
                                   │  REST / HTTP / Local Sockets
                     ┌─────────────▼─────────────┐
                     │   Python Backend Engine   │
                     │   (app.py / Flask SSE)    │
                     ├───────────────────────────┤
                     │ • Memory Hygiene          │
                     │ • Warm-up Engine          │
                     │ • Pilots & Tool Execution │
                     │ • SQLite Database & RAG   │
                     └─────────────┬─────────────┘
                                   │
                     ┌─────────────▼─────────────┐
                     │     LLM Service Engine    │
                     │  (Ollama / Local Model)   │
                     └───────────────────────────┘
```

---

## 3. The Runtime State Machine

LAILA Runtime operations are strictly governed by a deterministic Finite State Machine (FSM). Features and subsystems may only execute within their permitted states.

```
       ┌───────┐
       │  OFF  │
       └───┬───┘
           │ (Launch)
       ┌───▼───┐
       │BOOTING│
       └───┬───┘
           │
┌──────────▼──────────────┐
│  CHECKING_DEPENDENCIES  │
└──────────┬──────────────┘
           │ (Missing binaries/models/keys)
┌──────────▼──────────────┐
│       INSTALLING        │
└──────────┬──────────────┘
           │ (Setup complete)
┌──────────▼──────────────┐
│     STARTING_OLLAMA     │
└──────────┬──────────────┘
           │
┌──────────▼──────────────┐
│    STARTING_BACKEND     │
└──────────┬──────────────┘
           │
┌──────────▼──────────────┐
│     WAITING_BACKEND     │ ◄─── (Retry / Self-Heal)
└──────────┬──────────────┘        │
           │ (HTTP 200 Ready)      │
┌──────────▼──────────────┐        │
│      BACKEND_READY      │        │
└──────────┬──────────────┘        │
           │                       │
┌──────────▼──────────────┐        │
│        UI_READY         │        │
└──────────┬──────────────┘        │
           │                       │
┌──────────▼──────────────┐        │
│         ACTIVE          │ ───────┤ (Process drop / Error detected)
└──────────┬──────────────┘        │
           │ (Anomalous exit)      │
       ┌───▼──────┐                │
       │RECOVERING├────────────────┘
       └───┬──────┘
           │ (User exit / Recovery Escalation Failure)
    ┌──────▼──────┐
    │SHUTTING_DOWN│
    └──────┬──────┘
           │
       ┌───▼───┐
       │  OFF  │
       └───────┘
```

---

## 4. The 5-Stage Recovery Pipeline

When an anomaly occurs in `ACTIVE` or `WAITING_BACKEND` states, the Runtime transitions to `RECOVERING` and executes this 5-stage pipeline:

```
┌─────────────┐   ┌─────────────┐   ┌───────────────┐   ┌──────────────┐   ┌────────────┐
│ 1.DETECTION │──►│ 2.DIAGNOSIS │──►│ 3.RECOVERY ACT│──►│ 4.VERIFICATION│──►│ 5.ESCALATE │
└─────────────┘   └─────────────┘   └───────────────┘   └──────────────┘   └────────────┘
```

1. **Detection:** Watchdog catches non-zero process exit, lost connection to Ollama, or TCP port drop.
2. **Diagnosis:** Identifies root cause (Backend crash, Ollama service drop, Port collision, VRAM OOM).
3. **Recovery Action:** Re-spawn process, re-bind free TCP port, or trigger emergency VRAM unload.
4. **Verification:** Polls `/health` endpoint up to 3 times to confirm restoration.
5. **Escalation:** If 3 recovery attempts fail, escalate to `INSTALLING` / `Repair Wizard` mode.

---

## 5. Event Bus & Subsystem Communication Rules

Direct cross-module method invocation is strictly prohibited. All Runtime modules communicate exclusively by publishing and subscribing to events via the **Internal Event Bus**.

### Event Taxonomy

1. **Infrastructure Events:**
   - `PortAllocatedEvent` (Assigned TCP port)
   - `OllamaStartedEvent` (Service PID & endpoint)
   - `BackendStartedEvent` (Backend PID)
   - `BackendReadyEvent` (Health status & routes)
   - `BackendCrashedEvent` (Exit code & log)
   - `RecoverySucceededEvent` (Restored FSM state)

2. **Domain Events:**
   - `ConfigurationChangedEvent` (Runtime or User preference updated)
   - `ModelMissingEvent` (Target model identifier)
   - `CredentialsMissingEvent` (Provider name)
   - `InstallationStartedEvent` (Target task)
   - `InstallationFinishedEvent` (Installed artifacts)
   - `RepairRequestedEvent` (Component target for recovery escalation)

---

## 6. State Ownership Matrix

| State Domain | Owner | Contents |
| :--- | :--- | :--- |
| **Runtime Configuration State** | **LAILA Runtime Platform** | Dynamic TCP ports, process PIDs, application directory paths, log destinations. (`laila_runtime.json`) |
| **User Preference State** | **LAILA Runtime Platform** | Theme, UI language/locale, active provider selection. (`user_preferences.json`) |
| **Secure Credential State** | **Shared Security (`core/security`)** | Windows DPAPI encrypted API keys. (`credentials.enc`) |
| **Infrastructure State** | **LAILA Runtime Platform** | FSM state, watchdog timers, plugin & MCP registry statuses. |
| **AI & Session State** | **Backend Engine** | Conversation context, model parameters, DB tables, memory hygiene logs, pilot states. |
| **Presentation State** | **Frontend UI** | Input box content, Provider Settings UI, modal dialog states, local animations. |