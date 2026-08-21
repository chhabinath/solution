Yes. For a **30-hour hackathon**, don't over-engineer this. You need a stack that lets two people build a convincing MVP quickly while still showing real AI/system depth.

# Recommended tech stack for ComputePilot

```text
                    ┌──────────────────────┐
                    │      iQOO PHONE      │
                    │                      │
                    │   Android App        │
                    │   Kotlin             │
                    │   Jetpack Compose    │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │   ComputePilot AI    │
                    │                       │
                    │ Task Classification   │
                    │ Workload Estimation   │
                    │ Decision Engine       │
                    └──────────┬────────────┘
                               │
               ┌───────────────┼────────────────┐
               │               │                │
               ▼               ▼                ▼
           Phone/NPU        Phone CPU        Laptop
          Local Model       Lightweight      Heavy Work
               │               │                │
               └───────────────┼────────────────┘
                               │
                        Office Kit / API
                               │
                               ▼
                    ┌──────────────────────┐
                    │       LAPTOP         │
                    │                      │
                    │ Python FastAPI       │
                    │ Heavy AI / Processing│
                    │ Ollama               │
                    └──────────────────────┘
```

## 1. Phone: Native Android + Kotlin

**Use:**

* Kotlin
* Android SDK
* Jetpack Compose
* CameraX
* Android microphone/voice APIs
* Android device telemetry APIs

Why Kotlin?

Because this project is fundamentally about **the phone itself**, not just a web UI.

You need access to things like:

* battery
* thermal state
* network
* CPU/device state
* camera
* microphone
* local inference

Native Android gives you the cleanest path.

### UI

Use:

**Jetpack Compose**

Don't waste hackathon time building XML layouts.

---

# 2. Local AI: Ollama initially, Snapdragon acceleration if available

For development before the event:

**Ollama**

Use a small open-source model for:

```text
User request
     ↓
Task classification
     ↓
Complexity estimation
     ↓
Privacy requirement
     ↓
Latency requirement
```

For example:

```text
"Summarize this image"

→ task = vision
→ complexity = low
→ latency = high
→ privacy = high
→ recommended = phone
```

You could prototype with a small model such as:

* Qwen
* Gemma
* Phi

But **don't commit to a specific model until you know what the provided iQOO/Qualcomm environment supports**.

At the hackathon, the priority should be:

> **Use the provided/local model tooling and, if available, Snapdragon NPU acceleration.**

That's much stronger than simply saying "we used an LLM."

---

# 3. AI architecture: DON'T let the LLM make everything

This is important.

Bad architecture:

```text
User
 ↓
LLM
 ↓
"Use laptop"
```

That's difficult to trust and demonstrate.

Instead:

```text
             User Request
                  ↓
             Local Model
                  ↓
          Task Characteristics
                  ↓
        ┌─────────────────────┐
        │ ComputePilot Router │
        └──────────┬──────────┘
                   ↓
       ┌───────────┼───────────┐
       ↓           ↓           ↓
    Phone         NPU        Laptop
```

The model answers:

```json
{
  "task": "document_analysis",
  "complexity": 0.91,
  "privacy": 0.82,
  "latency": 0.35,
  "compute": 0.89
}
```

Then your deterministic **Decision Engine** decides where to execute.

That gives you much better technical credibility.

---

# 4. Decision Engine: Kotlin

I'd actually keep the routing logic on the phone.

Something conceptually like:

```text
                    Task
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     Battery       Thermal      Network
        │            │            │
        └────────────┼────────────┘
                     ↓
              Device availability
                     ↓
              ComputePilot Score
                     ↓
          ┌──────────┴──────────┐
          ↓                     ↓
        PHONE                 LAPTOP
```

The router calculates something like:

```text
Phone Score
Laptop Score
```

and selects the lower-cost option.

You can include weights for:

* latency
* battery
* thermal risk
* privacy
* compute capacity
* network cost

---

# 5. Laptop backend: Python + FastAPI

Your laptop should run a small server.

### Stack

**Python**

**FastAPI**

**Uvicorn**

**Ollama**

Potentially:

**PyTorch / Transformers**

depending on the workload.

Architecture:

```text
Android
   │
   │ HTTP/WebSocket
   ▼
FastAPI
   │
   ├── Task Manager
   ├── AI Processor
   ├── File Processor
   └── Result Manager
          │
          ▼
        Ollama
```

Since you're already comfortable with Python/FastAPI, this will be much faster for you than building the laptop side in Java.

---

# 6. Communication

For the prototype:

### REST

Use REST for:

```text
POST /task
POST /execute
GET /status
GET /result/{id}
```

### WebSocket

Use WebSocket if you want live updates:

```text
PHONE
  │
  │ WebSocket
  ▼
LAPTOP
  │
  ├── Processing 20%
  ├── Processing 50%
  ├── Processing 80%
  └── Complete
```

For a 30-hour hackathon, **REST first**.

Add WebSocket only if the demo needs live progress.

---

# 7. Office Kit

This is where we need to be careful.

Your hackathon material says Office Kit provides:

* screen mirroring
* clipboard
* file transfer
* remote control

and the phone comes pre-paired. 

Don't build your own replacement for Office Kit.

Use it as part of the workflow.

For example:

```text
Phone
  │
  │ "Heavy task"
  ▼
ComputePilot
  │
  │ Office Kit / paired PC
  ▼
Laptop
  │
  │ Processing
  ▼
Result
  │
  ▼
Phone
```

**But we need to verify exactly what programmatic interfaces Office Kit exposes during the event.**

If Office Kit doesn't provide an API for automated task transfer, don't pretend it does.

We can make the laptop-side service communicate over the local network while using Office Kit visibly for the phone↔PC experience where appropriate.

---

# 8. Database

You don't need PostgreSQL.

Seriously.

For a 30-hour hackathon:

### Phone

**Room / SQLite**

for:

* task history
* routing decisions
* device measurements

### Laptop

**SQLite**

for:

* task records
* execution history
* performance measurements

That's enough.

---

# 9. What data should we collect?

This is actually more important than the database.

Every execution should produce something like:

```json
{
  "task": "document_analysis",
  "complexity": 0.91,
  "battery": 18,
  "temperature": 41,
  "network": "good",
  "phone_score": 0.82,
  "laptop_score": 0.31,
  "decision": "LAPTOP",
  "execution_time": 12.4
}
```

Then you can show the judge:

> **Why did ComputePilot choose the laptop?**

That's powerful.

---

# 10. Three workloads only

Don't build ten.

Build these three:

### Workload 1 — Camera understanding

```text
Camera
 ↓
Local AI
 ↓
Phone
```

Example:

> "Read this document."

---

### Workload 2 — Heavy document analysis

```text
PDF
 ↓
ComputePilot
 ↓
Laptop
 ↓
Ollama
 ↓
Result
 ↓
Phone
```

---

### Workload 3 — AI generation

```text
Prompt
 ↓
Task classifier
 ↓
Decision Engine
 ↓
Phone OR Laptop
```

These three are enough to demonstrate the concept.

---

# 11. Optional: a lightweight ML model

This is where you can make the project genuinely **AI/ML**, rather than just an LLM wrapper.

Collect execution data:

```text
task_size
input_tokens
battery
temperature
cpu_usage
network_latency
phone_execution_time
laptop_execution_time
```

Then train something simple:

**XGBoost / Random Forest / Gradient Boosting**

to predict:

```text
expected_execution_time
```

for:

```text
PHONE
LAPTOP
```

Then your router chooses the predicted best option.

Example:

```text
                Task
                  │
                  ▼
          Feature extraction
                  │
                  ▼
          ML prediction model
             /           \
            /             \
           ▼               ▼
    Phone: 8.4 sec    Laptop: 2.1 sec
           \               /
            \             /
             └─────┬─────┘
                   ▼
                LAPTOP
```

That is much more interesting in front of an AI/ML judge.

---

# 12. Tech stack I would actually lock

### 📱 Android

| Component          | Technology                                       |
| ------------------ | ------------------------------------------------ |
| Language           | **Kotlin**                                       |
| UI                 | **Jetpack Compose**                              |
| Camera             | **CameraX**                                      |
| Voice              | Android Speech APIs                              |
| Local storage      | Room / SQLite                                    |
| Device information | Android APIs                                     |
| Local AI           | Snapdragon-compatible runtime / provided tooling |
| Networking         | Retrofit / OkHttp                                |

### 💻 Laptop

| Component        | Technology             |
| ---------------- | ---------------------- |
| Language         | **Python**             |
| API              | **FastAPI**            |
| Server           | Uvicorn                |
| Local LLM        | **Ollama** initially   |
| ML               | scikit-learn / XGBoost |
| Data             | SQLite                 |
| Heavy processing | Python                 |
| Communication    | REST initially         |

### 🔗 Integration

| Component     | Technology                                       |
| ------------- | ------------------------------------------------ |
| Phone ↔ PC    | **Office Kit + local network where appropriate** |
| API           | REST                                             |
| Live updates  | WebSocket, optional                              |
| Serialization | JSON                                             |

---

# 13. What NOT to use

For this hackathon, I'd deliberately avoid:

❌ React Native
❌ Flutter
❌ Spring Boot backend
❌ PostgreSQL
❌ Kubernetes
❌ Docker-heavy architecture
❌ Microservices
❌ Kafka
❌ Redis
❌ LangChain everywhere
❌ RAG unless a workload actually requires it

Those aren't bad technologies.

They're just **bad choices for a 30-hour project where your differentiator is the device/system architecture**.

Your existing Java/Spring Boot experience doesn't mean Spring Boot belongs here.

---

# 14. Team of two

I'd divide it like this:

### You — AI + backend + routing

```text
Task classifier
      ↓
Workload estimator
      ↓
Decision engine
      ↓
ML prediction
      ↓
FastAPI laptop service
      ↓
Ollama
```

### Teammate — Android/device

```text
Jetpack Compose
      ↓
Voice
Camera
      ↓
Battery/thermal/network data
      ↓
Local inference
      ↓
Office Kit integration
      ↓
Demo UI
```

Then integrate.

---

# 15. The final architecture

```text
                         USER
                           │
                   Voice / Camera / Text
                           │
                           ▼
                 ┌────────────────────┐
                 │    iQOO Android    │
                 │                    │
                 │   ComputePilot    │
                 └─────────┬──────────┘
                           │
                     Local AI Model
                           │
                           ▼
                  Task Characteristics
                           │
              ┌────────────┴────────────┐
              │                         │
       Device telemetry           Laptop status
       • Battery                  • Available
       • Thermal                  • Load
       • Network                  • Capability
       • Current load
              │                         │
              └────────────┬────────────┘
                           ▼
                  ┌─────────────────┐
                  │ Decision Engine │
                  └────────┬────────┘
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
              📱 PHONE           💻 LAPTOP
              Local AI           FastAPI
              NPU/CPU            Ollama
                  │                 │
                  └────────┬────────┘
                           ▼
                         RESULT
                           │
                           ▼
                        📱 PHONE
```

## One important correction to our earlier plan

**Don't make the ML prediction model mandatory for V1.**

Build in this order:

**V1:** Rule/score-based router → working demo
**V2:** Local AI task classifier → intelligent task understanding
**V3:** ML performance predictor → predicts phone vs laptop execution cost
**V4:** Snapdragon/NPU optimization → if the event tooling allows it

That way, if something breaks during the 30-hour event, you still have a working project.

And the final pitch becomes:

> **“ComputePilot is an AI workload router for the phone era. Instead of asking users to decide where their work should run, it predicts the cost of running a task and automatically chooses the best available compute surface—phone or PC—based on performance, battery, thermal state, latency, and privacy.”**

That's the architecture I'd build around.
