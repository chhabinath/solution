## **ComputePilot**

> **AI decides where your task should run.**

The system has three execution targets:

```text
                  USER TASK
                      │
                      ▼
              ┌───────────────┐
              │  AI Router    │
              └───────┬───────┘
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       PHONE       NPU/AI      LAPTOP
      CPU/GPU      LOCAL AI    HEAVY AI
          │           │           │
          └───────────┼───────────┘
                      ▼
                   RESULT
```

But the router doesn't simply ask **"which device is faster?"**

It considers:

```text
Latency
Battery
Temperature
Privacy
Task complexity
Network
Model availability
Current workload
```

and makes a decision.

---

# 1. What would the user actually do?

Suppose your phone has the app open.

You say:

> **"Summarize this document."**

The system receives:

```json
{
  "task": "summarize_document",
  "document_size": "2 MB",
  "privacy": "high",
  "battery": 82,
  "temperature": "normal",
  "network": "good"
}
```

The router might decide:

> **LOCAL**

because the document is small and privacy matters.

---

Then:

> **"Analyze these 500 pages and compare them with my project repository."**

Now:

```text
Task complexity = HIGH
Document size = LARGE
Repository = LARGE
Phone compute = LIMITED
Laptop = AVAILABLE
```

Decision:

> **LAPTOP**

Office Kit transfers the workload.

---

Then:

> **"Identify what's written on this receipt."**

Decision:

> **PHONE / LOCAL**

because camera + lightweight vision inference is faster and doesn't need a laptop.

---

# 2. The important part: build a scoring engine

Don't start with reinforcement learning or some fancy deep-learning scheduler.

That would be unnecessary hackathon engineering.

Start with a **cost function**.

For example:

```text
Score(device) =
    w1 × latency
  + w2 × battery_cost
  + w3 × thermal_risk
  + w4 × privacy_risk
  + w5 × compute_cost
```

Lower score = better execution target.

For example:

```text
              Phone    NPU     Laptop

Latency        2        1         4
Battery        4        1         2
Thermal        3        1         2
Privacy        1        1         4
Compute        4        2         1
```

The weights depend on the task.

### Privacy-sensitive task

Increase:

```text
privacy_weight
```

### Battery-critical situation

Increase:

```text
battery_weight
```

### Real-time task

Increase:

```text
latency_weight
```

That's already an interesting systems problem.

---

# 3. Where does the AI come in?

This is important.

Don't call the scoring function itself "AI."

Use AI for:

### **Task classification**

```text
User request
     ↓
Local model
     ↓
Task classification
```

Example:

```text
"Extract text from this image"
        ↓
VISION
        ↓
LIGHT
        ↓
LOCAL
```

versus:

```text
"Compare these 500 documents"
        ↓
DOCUMENT_ANALYSIS
        ↓
HEAVY
        ↓
LAPTOP
```

The AI predicts:

```json
{
  "task_type": "document_analysis",
  "complexity": 0.92,
  "latency_requirement": 0.30,
  "privacy_requirement": 0.85,
  "estimated_compute": 0.88
}
```

Then the **scheduler** makes the final decision.

That separation makes your architecture much more credible.

---

# 4. Your architecture

For a two-person team, I'd make it:

```text
                    iQOO PHONE
                        │
             ┌──────────┴──────────┐
             │                     │
          Voice                  Camera
             │                     │
             └──────────┬──────────┘
                        ▼
                 Intent Parser
                        │
                        ▼
              Local Task Classifier
                        │
                        ▼
              ┌──────────────────┐
              │ ComputePilot     │
              │ Decision Engine  │
              └────────┬─────────┘
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
    PHONE CPU        PHONE NPU        LAPTOP
       │               │                │
       └───────────────┼────────────────┘
                       │
                       ▼
                  Result Manager
                       │
                       ▼
                    PHONE UI
```

---

# 5. Phone ↔ laptop

This is where **Office Kit** becomes meaningful.

Don't use it just because the judging criteria says you should.

Use it because your architecture actually needs it.

Example:

```text
Phone:
"Analyze repository"

        ↓

ComputePilot:

"Estimated workload:
HIGH"

        ↓

"Recommended:
LAPTOP"

        ↓

Office Kit

        ↓

Laptop executes

        ↓

Result → Phone
```

The user doesn't manually decide where the computation goes.

**The AI router decides.**

That's your core product.

---

# 6. How do you demonstrate NPU?

This needs some care.

Don't claim:

> "We're directly controlling the Snapdragon NPU."

unless the hackathon provides an SDK/API that actually lets you do that.

Instead, structure the architecture so that **local AI inference is your NPU-capable execution path**.

For example:

```text
Local model
    ↓
Android inference runtime
    ↓
Available hardware acceleration
    ↓
NPU / GPU / CPU
```

If the event provides the appropriate Qualcomm/Snapdragon tooling, you can optimize the model for that hardware.

If not, your demo can still show:

> **LOCAL AI EXECUTION**

and make the hardware-accelerated path an architectural target.

Don't fake hardware capabilities. A technical judge will catch it immediately.

---

# 7. Thermal prediction

This is where the project becomes much more interesting.

Instead of just reading:

> temperature = 40°C

you collect a short history:

```text
Time   CPU   Battery   Temp   Load
-----------------------------------
0      20%    80%     34°C    LOW
1      35%    79%     35°C    MED
2      52%    78%     37°C    MED
3      68%    77%     39°C    HIGH
4      82%    75%     41°C    HIGH
```

Your model predicts:

```text
Thermal risk in next 5 min:
HIGH
```

Now the router changes its decision.

For example:

> Normally → LOCAL

but:

> High thermal risk → LAPTOP

That's the **predictive** part.

---

# 8. Battery becomes another dimension

Suppose:

```text
Battery = 12%
```

and the user asks:

> "Run this AI analysis."

Your system calculates:

```text
Phone execution:
Battery cost = HIGH

Laptop execution:
Phone battery cost = LOW
```

Therefore:

> **Move computation to laptop.**

This creates a very nice demo.

---

# 9. Privacy makes the architecture more interesting

Imagine the user says:

> "Analyze this private document."

Your router should recognize:

```text
Privacy requirement = HIGH
```

Then:

```text
Cloud
❌

Laptop
maybe

Phone local
✅
```

So the system isn't blindly optimizing speed.

It's optimizing:

> **performance subject to privacy constraints.**

That's much more sophisticated.

---

# 10. Your actual MVP should have only 3 workloads

Do **not** attempt 30 workloads.

Build three extremely polished ones.

### Workload A — Image understanding

```text
Camera
 ↓
Local AI
 ↓
Phone
```

Example:

> "What's on this document?"

---

### Workload B — Text/document analysis

```text
Phone
 ↓
Decision Engine
 ↓
Laptop
 ↓
Heavy processing
 ↓
Phone
```

---

### Workload C — AI generation

```text
User prompt
 ↓
Task classifier
 ↓
Battery/thermal/privacy evaluation
 ↓
LOCAL or LAPTOP
```

That's enough.

---

# 11. The UI could be ridiculously simple

Main screen:

```text
┌──────────────────────────────┐
│        ComputePilot          │
│                              │
│     "What do you want?"      │
│                              │
│        🎤 Speak              │
│                              │
│        📷 Camera             │
│                              │
├──────────────────────────────┤
│ Device Status                │
│                              │
│ Battery       78%            │
│ Temperature   Normal         │
│ Network       Good           │
│ Laptop        Connected      │
│                              │
├──────────────────────────────┤
│ Current AI Strategy          │
│                              │
│ LOCAL NPU                    │
└──────────────────────────────┘
```

When you submit something:

```text
┌──────────────────────────────┐
│       COMPUTE DECISION       │
├──────────────────────────────┤
│ Task: Document Analysis      │
│                              │
│ Complexity      HIGH         │
│ Privacy         HIGH         │
│ Battery cost    MEDIUM       │
│ Thermal risk    LOW          │
│                              │
│ Recommended:                │
│                              │
│        💻 LAPTOP             │
│                              │
│ Reason:                     │
│ Large workload + high       │
│ compute requirement         │
└──────────────────────────────┘
```

That screen itself communicates the idea to judges immediately.

---

# 12. Your killer demo

Don't start by explaining architecture.

Start with:

### Scene 1

Phone battery:

**15%**

Laptop connected.

You say:

> **"Analyze this 200-page document."**

ComputePilot:

> **Laptop recommended.**

Because:

```text
Battery impact: HIGH
Compute requirement: HIGH
Privacy: HIGH
Laptop available: YES
```

The laptop performs the work.

---

### Scene 2

Now:

> **"Read this document with my camera."**

Phone:

> **Local execution recommended.**

Because:

```text
Small workload
Low latency required
No laptop needed
```

---

### Scene 3

Now artificially increase workload/thermal state during the demo.

The system says:

> **"Local execution no longer optimal. Moving workload to laptop."**

That's the moment the judge should understand:

**This isn't an AI chatbot.**

It's a **dynamic compute orchestration system**.

---

# 13. What makes this recruitable

For an iQOO/vivo engineer, your project lets you discuss:

* Edge AI
* On-device inference
* NPU acceleration
* AI model routing
* workload prediction
* thermal management
* battery optimization
* latency optimization
* privacy-aware computing
* distributed systems
* Android
* device-to-PC computing
* AI agents

That's a **much stronger engineering story** than:

> "We built a chatbot."

---

# 14. Your two-person division

### You — AI/Systems

Build:

* task classifier
* workload estimator
* decision engine
* thermal/battery prediction
* routing algorithm
* laptop execution API

### Teammate — Android/Integration

Build:

* Android application
* voice
* camera
* device telemetry
* local inference integration
* Office Kit integration
* UI/demo

Then both integrate.

---

# 15. One thing I'd change from our earlier idea

I would **not call it "AI that predicts the phone's future computational needs"** in the final pitch.

That's technically accurate but sounds like a research project.

Use:

# **ComputePilot**

### *Your AI decides where your work should run.*

Then the technical explanation:

> ComputePilot predicts workload requirements and dynamically routes AI tasks between local phone execution and a paired PC based on latency, battery, thermal state, privacy, and compute availability.

That's crisp.

---

## The real MVP target

Don't try to build:

> **"A replacement for Android's scheduler."**

You won't finish it in 30 hours.

Build:

> **"An intelligent AI workload router that demonstrates what a future phone-to-PC compute layer could look like."**

That's achievable.

And importantly, the hackathon already gives you the exact ingredients that make the concept interesting: **iQOO phone, local/open-source AI, phone-in-the-loop, and Office Kit**. 

If we pursue this, the next thing I'd do is **design the actual 30-hour MVP: exact tech stack, 3 workloads, model choice, Android architecture, backend architecture, data flow, APIs, and what each of the two team members builds hour-by-hour.**
