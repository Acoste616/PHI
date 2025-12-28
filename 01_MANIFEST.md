# PHI MANIFEST
## Personal Health Intelligence - Digital Biological Twin
**Version:** 1.0.0  
**Last Updated:** 2024-12-27  
**Status:** PRODUCTION READY  

---

## 1. VISION STATEMENT

> "Know thyself through data. Act through intelligence. Thrive through optimization."

PHI (Personal Health Intelligence) is not another health tracking app. It is a **Digital Biological Twin** - an AI-powered system that understands the Founder's body and mind better than the Founder understands themselves.

PHI exists to answer one question: **"What should I do RIGHT NOW to maximize my energy, focus, and performance?"**

---

## 2. THE "HIGH ENERGY" PHILOSOPHY

### 2.1 Core Belief
The Founder operates on a **High Energy** profile:
- Peak performance comes in bursts
- Energy is the primary currency (not time)
- Optimization beats motivation
- Data-driven decisions outperform gut feelings (but gut feelings validate data)

### 2.2 The PHI Promise
| Traditional Apps | PHI Approach |
|-----------------|--------------|
| "You slept 7.2 hours" | "Your sleep was restorative. HRV indicates peak cognitive window at 9-11 AM. Schedule your hardest work now." |
| "You walked 8,000 steps" | "Your movement pattern suggests pre-meeting anxiety. Consider 5 min breathing before the call." |
| "Log your mood" | "Based on your biometrics and yesterday's pattern, I predict an energy dip at 3 PM. Pre-empting with caffeine cutoff at 1 PM." |

### 2.3 The Three Pillars of High Energy
```
┌─────────────────────────────────────────────────────────────┐
│                    HIGH ENERGY FRAMEWORK                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐       │
│   │   RECOVER   │   │   PERFORM   │   │   ADAPT     │       │
│   │             │   │             │   │             │       │
│   │ • Sleep     │   │ • Focus     │   │ • Learn     │       │
│   │ • HRV       │   │ • Energy    │   │ • Predict   │       │
│   │ • Readiness │   │ • Output    │   │ • Optimize  │       │
│   └─────────────┘   └─────────────┘   └─────────────┘       │
│         ▲                 │                 │                │
│         └─────────────────┴─────────────────┘                │
│                    FEEDBACK LOOP                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. THE "DIGITAL TWIN" CONCEPT

### 3.1 What is a Digital Biological Twin?
A Digital Twin in engineering is a virtual replica of a physical system that updates in real-time and can predict future states. 

PHI applies this concept to **human biology and psychology**:

```
┌─────────────────────────────────────────────────────────────┐
│                    DIGITAL TWIN MODEL                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   PHYSICAL WORLD              DIGITAL WORLD                  │
│   ┌──────────────┐           ┌──────────────┐               │
│   │              │  sensors  │              │               │
│   │   FOUNDER    │ ────────► │   PHI TWIN   │               │
│   │   (Body)     │           │   (Model)    │               │
│   │              │ ◄──────── │              │               │
│   └──────────────┘  actions  └──────────────┘               │
│                                     │                        │
│                                     ▼                        │
│                              ┌──────────────┐               │
│                              │  PREDICTIONS │               │
│                              │  & INSIGHTS  │               │
│                              └──────────────┘               │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Twin Components
| Component | Physical Source | Digital Representation |
|-----------|----------------|----------------------|
| **Sleep State** | Oura Ring | Sleep Score, HRV, Deep Sleep % |
| **Readiness** | Oura Ring | Readiness Score (0-100) |
| **Activity** | Health Connect | Steps, Active Calories, Workouts |
| **Context** | Chat Input | Episodic Memory Database |
| **Psychology** | Self-Report | Energy Level, Mood Tags, Commander Mode |

### 3.3 Twin Intelligence Levels
1. **Level 1 - Reactive:** "Your HRV is low today" (current state)
2. **Level 2 - Correlative:** "Low HRV + late dinner = poor recovery" (pattern)
3. **Level 3 - Predictive:** "Based on your pattern, tomorrow's readiness will be 65" (forecast)
4. **Level 4 - Prescriptive:** "To hit 85 readiness tomorrow: bed by 10 PM, no alcohol, 30 min walk" (action)

**PHI targets Level 4 from Day 1.**

---

## 4. NON-NEGOTIABLE PRINCIPLES

### 4.1 PRIVACY: Sovereign Data Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    PRIVACY HIERARCHY                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   TIER 1: NEVER LEAVES DEVICE                               │
│   ├── Raw biometric data (HR, HRV readings)                 │
│   ├── Health Connect raw exports                            │
│   └── Episodic memory database                              │
│                                                              │
│   TIER 2: PROCESSED LOCALLY, AGGREGATES MAY SYNC            │
│   ├── Daily scores (Sleep, Readiness)                       │
│   ├── Trend summaries (7-day averages)                      │
│   └── Anonymized patterns                                   │
│                                                              │
│   TIER 3: SENT TO CLOUD AI (Gemini)                         │
│   ├── Conversation context (no raw data)                    │
│   ├── Summarized insights for long-term analysis            │
│   └── User queries and AI responses                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Rule:** If Gemini API fails, PHI MUST remain functional with Ollama local fallback.

### 4.2 PROACTIVITY: Ask Before Being Asked
PHI does not wait for the user to open the app. PHI **initiates**:
- Morning: "Your readiness is 78. Here's your battle plan."
- Midday: "Check-in: What are you working on? How's energy 1-10?"
- Pre-crash: "Predicted energy dip in 90 min. Preventive action: [X]"
- Evening: "Reflection time. How did today match predictions?"

### 4.3 ACTIONABILITY: No Vanity Metrics
Every data point must answer: **"So what? What do I do with this?"**

| ❌ Bad Output | ✅ PHI Output |
|--------------|--------------|
| "HRV: 45ms" | "HRV below your baseline. Avoid high-stress decisions before noon." |
| "Sleep: 6h 32m" | "Sleep debt accumulating. Prioritize 8h tonight or expect 30% productivity drop." |
| "Steps: 4,200" | "Movement deficit. 20 min walk now will boost afternoon focus by ~25%." |

### 4.4 ADAPTATION: Commander Modes
PHI adapts its communication style to the Founder's current state:

| Mode | Trigger | Communication Style | UI Theme |
|------|---------|--------------------|---------| 
| **HIGH ENERGY** | Readiness > 80, User confirms | Direct, aggressive, challenge-oriented. "Let's crush it." | Red/Orange, Bold |
| **ZEN MASTER** | Readiness 50-80, Recovery day | Calm, supportive, patience-focused. "Easy does it." | Blue/Teal, Soft |
| **SOCIAL BATTERY** | Social event detected | Social energy management, introvert recharge tips | Purple, Balanced |
| **RECOVERY** | Readiness < 50, Illness detected | Gentle, protective, rest-focused. "Your body needs this." | Green, Minimal |

---

## 5. SUCCESS METRICS

### 5.1 North Star Metric
**Average Weekly Readiness Score > 75**

### 5.2 Supporting Metrics
| Metric | Target | Measurement |
|--------|--------|-------------|
| Daily Check-in Completion | > 80% | User responds to midday prompt |
| Prediction Accuracy | > 70% | Predicted vs actual energy levels |
| Action Compliance | > 50% | User follows PHI recommendations |
| Sleep Consistency | < 45 min variance | Bedtime standard deviation |

### 5.3 Anti-Metrics (What We DON'T Optimize For)
- ❌ Time spent in app (addiction is failure)
- ❌ Number of notifications (noise is failure)
- ❌ Data volume collected (hoarding is failure)

---

## 6. FOUNDER-SYSTEM CONTRACT

### What PHI Promises:
1. Never sell or share identifiable health data
2. Function offline with core features
3. Explain every recommendation ("Why am I telling you this?")
4. Learn and adapt to individual patterns within 14 days
5. Respect "Do Not Disturb" periods absolutely

### What Founder Promises:
1. Wear Oura Ring consistently (>90% nights)
2. Respond to check-in prompts honestly
3. Give PHI 30 days to calibrate before judging accuracy
4. Report when predictions are wrong (feedback loop)

---

## 7. TECHNICAL PHILOSOPHY

### 7.1 Build for Resilience
```
IF cloud_api_fails:
    fallback_to_local_ai()
    
IF local_ai_fails:
    show_raw_data_with_simple_rules()
    
IF all_else_fails:
    show_last_known_good_state()
    never_show_error_screen_without_actionable_recovery()
```

### 7.2 Build for Speed
- First Meaningful Paint: < 2 seconds
- Data sync: Background, never blocking UI
- AI response: < 5 seconds (or show typing indicator)

### 7.3 Build for the Founder
- No jargon in UI (translate HRV to "Recovery Power")
- No configuration hell (smart defaults, minimal settings)
- No guilt trips ("You missed your goal" → "Let's adjust the goal")

---

## 8. VERSION HISTORY

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2024-12-27 | Initial Manifest - Production Ready |

---

**END OF MANIFEST**

*This document is the philosophical foundation of PHI. All technical decisions must align with these principles. When in doubt, refer back to Section 4: Non-Negotiable Principles.*
