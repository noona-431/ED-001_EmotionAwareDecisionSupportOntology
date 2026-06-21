# ED-001 Competency Questions & Reasoning Analysis

## Overview
This document demonstrates how the ED-001 ontology structure, SWRL rules, and class hierarchy directly answer three critical competency questions about emotion-driven decision-making in learning contexts.

---

## COMPETENCY QUESTION 1 (CQ1)
### **"Which learners are at high decision risk due to anxiety in high-difficulty, time-pressured examination contexts, and what is their predicted decision strategy?"**

### Why This Question Matters
This CQ directly tests the ontology's core capability: bridging from **Learning Context** (Layer 1) through **Emotion** (Layer 2) and **Cognitive State** (Layer 3) to **Decision Strategy** (Layer 4). This is the causal chain that no existing ontology models.

---

### How the ED-001 Ontology Answers CQ1

#### 1. **Class Hierarchy Support**
The ontology provides:
- **LearningContext** subclasses:
  - `ExaminationContext` (high-stakes, time-limited)
  - Data properties: `taskDifficulty` (High), `timePressure` (true)
  
- **EmotionalState** subclasses:
  - `AnxietyState` (valence=Negative, activation=Activating, objectFocus=Outcome)
  
- **CognitiveState** subclasses:
  - `HighCognitiveLoad` (cognitive load overload)
  - `NarrowedAttention` (anxiety-driven tunnel vision)
  
- **DecisionStrategy** subclasses:
  - `ImpulsiveDecision` (quick, unreflective)
  
- **RiskLevel** subclasses:
  - `HighRisk` (high probability of error or dropout)

#### 2. **Object Properties Implementing the Causal Chain**
```
LearningContext --[elicitedBy]--> EmotionalState
               |
               v
         AnxietyState
               |
               [modulates]
               v
         HighCognitiveLoad
               |
               [influencesDecision]
               v
         ImpulsiveDecision
               |
               [producesOutcome]
               v
         ErrorOutcome / DropoutEvent
```

Each arrow represents an explicitly defined OWL object property with domain/range constraints.

#### 3. **SWRL Rules Operating on the Chain**

**SWRL Rule 1 - Stress Detection:**
```
IF   hasEmotion(Learner, Anxiety) 
  ∧ isAttempting(Learner, Task)
  ∧ hasContext(Task, Context)
  ∧ taskDifficulty(Context, "High")
  ∧ timePressure(Context, true)
THEN hasCognitiveState(Learner, HighCognitiveLoad)
```

**SWRL Rule 2 - Decision Risk Inference:**
```
IF   hasCognitiveState(Learner, HighCognitiveLoad)
THEN hasDecisionStrategy(Learner, ImpulsiveDecision)
  ∧ hasRisk(Learner, HighRisk)
```

#### 4. **SPARQL Query to Answer CQ1**

```sparql
PREFIX ed: <http://www.semanticweb.org/ed001/ontology#>

# Query: Find all learners with anxiety in examination contexts
# that the ontology classifies as high-risk with impulsive strategy

SELECT DISTINCT ?learner ?emotion ?task ?difficulty ?timePressure ?cognitiveState ?strategy ?riskLevel
WHERE {
  # Layer 2: Emotion Detection (from ML classifier)
  ?learner ed:hasEmotion ?emotionState .
  ?emotionState rdf:type ed:AnxietyState .
  
  # Get emotional properties
  ?emotionState ed:valence ?valence ;
                ed:activation ?activation ;
                ed:intensity ?intensity .
  
  # Layer 1: Learning Context
  ?learner ed:isAttempting ?task .
  ?task ed:hasContext ?context .
  ?context rdf:type ed:ExaminationContext .
  ?context ed:taskDifficulty "High" ;
           ed:timePressure true .
  
  # Layer 3: Cognitive State (inferred by SWRL Rule 1)
  ?learner ed:hasCognitiveState ?cognitiveState .
  ?cognitiveState rdf:type ed:HighCognitiveLoad .
  
  # Layer 4: Decision Strategy (inferred by SWRL Rule 2)
  ?learner ed:hasDecisionStrategy ?strategy .
  ?strategy rdf:type ed:ImpulsiveDecision .
  
  # Layer 5: Risk Assessment (inferred by SWRL Rule 2)
  ?learner ed:hasRisk ?riskLevel .
  ?riskLevel rdf:type ed:HighRisk .
}
ORDER BY DESC(?intensity)
```

#### 5. **Example Instance Data (for query execution)**

```xml
<!-- Learner Instance -->
<rdf:Description rdf:about="http://www.semanticweb.org/ed001/data#Student_Alice">
  <rdf:type rdf:resource="http://www.semanticweb.org/ed001/ontology#Learner"/>
  <ed:learnerID>ALICE_2026_001</ed:learnerID>
  
  <!-- Layer 2: ML Classifier Output -->
  <ed:hasEmotion rdf:resource="http://www.semanticweb.org/ed001/data#Emotion_Anxiety_Alice_T1"/>
  
  <!-- Layer 1: Task Attempt -->
  <ed:isAttempting rdf:resource="http://www.semanticweb.org/ed001/data#Task_MathExam_001"/>
</rdf:Description>

<!-- Anxiety Emotion Instance -->
<rdf:Description rdf:about="http://www.semanticweb.org/ed001/data#Emotion_Anxiety_Alice_T1">
  <rdf:type rdf:resource="http://www.semanticweb.org/ed001/ontology#AnxietyState"/>
  <ed:valence>Negative</ed:valence>
  <ed:activation>Activating</ed:activation>
  <ed:intensity>0.87</ed:intensity>
  <ed:timestamp rdf:datatype="http://www.w3.org/2001/XMLSchema#dateTime">2026-01-15T10:30:00Z</ed:timestamp>
</rdf:Description>

<!-- Task Instance -->
<rdf:Description rdf:about="http://www.semanticweb.org/ed001/data#Task_MathExam_001">
  <rdf:type rdf:resource="http://www.semanticweb.org/ed001/ontology#LearningTask"/>
  <ed:hasContext rdf:resource="http://www.semanticweb.org/ed001/data#Context_Exam_Final"/>
</rdf:Description>

<!-- Context Instance -->
<rdf:Description rdf:about="http://www.semanticweb.org/ed001/data#Context_Exam_Final">
  <rdf:type rdf:resource="http://www.semanticweb.org/ed001/ontology#ExaminationContext"/>
  <ed:taskDifficulty>High</ed:taskDifficulty>
  <ed:timePressure rdf:datatype="http://www.w3.org/2001/XMLSchema#boolean">true</ed:timePressure>
  <ed:stakesLevel>High</ed:stakesLevel>
</rdf:Description>
```

#### 6. **Expected Reasoning Chain (Pellet/HermiT Inference)**

When the reasoner processes the above data with SWRL Rule 1 and Rule 2:

```
INPUT (from ML + manual assertion):
  hasEmotion(Student_Alice, Emotion_Anxiety_T1)
  isAttempting(Student_Alice, Task_MathExam_001)
  hasContext(Task_MathExam_001, Context_Exam_Final)
  taskDifficulty(Context_Exam_Final, "High")
  timePressure(Context_Exam_Final, true)

INFERENCE STEP 1 (SWRL Rule 1 fires):
  ✓ Rule 1 conditions matched
  → DERIVED: hasCognitiveState(Student_Alice, HighCognitiveLoad)
  
INFERENCE STEP 2 (SWRL Rule 2 fires):
  ✓ Rule 2 condition matched (HighCognitiveLoad is now asserted)
  → DERIVED: hasDecisionStrategy(Student_Alice, ImpulsiveDecision)
  → DERIVED: hasRisk(Student_Alice, HighRisk)

FINAL QUERY RESULT:
  ?learner = Student_Alice
  ?emotion = Emotion_Anxiety_T1 (intensity: 0.87)
  ?strategy = ImpulsiveDecision
  ?riskLevel = HighRisk
```

#### 7. **How This Answers CQ1**
✅ **Identifies learners** with anxiety (Layer 2)  
✅ **Links to context** (Layer 1): high difficulty, time pressure  
✅ **Infers cognitive state** (Layer 3): high load  
✅ **Predicts decision strategy** (Layer 4): impulsive  
✅ **Assesses risk** (cross-layer): high risk of error  

**NO existing ontology does all five of these steps.**

---

---

## COMPETENCY QUESTION 2 (CQ2)
### **"What emotions and cognitive states lead to optimal learning outcomes (mastery), and what is the inverse pattern predicting dropout risk?"**

### Why This Question Matters
This CQ tests whether the ontology can answer **bidirectional reasoning**: forward (Context→Outcome) and inverse (Outcome→Emotion). It validates the theoretical grounding in Control-Value Theory.

---

### How the ED-001 Ontology Answers CQ2

#### 1. **Class Hierarchy Support for Outcome Diversity**

The ontology distinguishes four outcome types:
- `LearningGain` (knowledge acquisition)
- `MasteryAchievement` (deep understanding, high performance)
- `ErrorOutcome` (incorrect response)
- `DropoutEvent` (abandonment)

Each is linked bidirectionally to decision strategies:
- `ExploratoryReasoning` → `LearningGain` / `MasteryAchievement`
- `ImpulsiveDecision` → `ErrorOutcome`
- `AvoidanceStrategy` → `DropoutEvent`

#### 2. **SWRL Rules for Positive Flow**

**SWRL Rule 3 - Positive Flow (Optimal Learning):**
```
IF   hasEmotion(Learner, Curiosity)
  ∧ hasCognitiveState(Learner, FocusedAttention)
THEN hasDecisionStrategy(Learner, ExploratoryReasoning)
  ∧ hasRisk(Learner, LowRisk)
```

**Extended (Domain Knowledge):**
```
IF   hasDecisionStrategy(Learner, ExploratoryReasoning)
  ∧ hasRisk(Learner, LowRisk)
THEN experiencedOutcome(Learner, LearningGain)  [with high probability]
```

#### 3. **SPARQL Query for Optimal Pattern**

```sparql
PREFIX ed: <http://www.semanticweb.org/ed001/ontology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

# Query: Identify optimal emotion-cognition-decision-outcome chain

SELECT ?learner ?emotion ?cogState ?strategy ?outcome ?outcomeScore
       (CONCAT("OPTIMAL: ", STR(?emotion), " → ", STR(?cogState), " → ", STR(?strategy)) AS ?chain)
WHERE {
  # Positive emotional state
  ?learner ed:hasEmotion ?emotion .
  ?emotion rdf:type ed:CuriosityState .  # or EnjoymentState
  ?emotion ed:intensity ?emotionalIntensity .
  FILTER (?emotionalIntensity > 0.6)
  
  # Optimal cognitive state
  ?learner ed:hasCognitiveState ?cogState .
  ?cogState rdf:type ed:FocusedAttention .
  
  # Inferred decision strategy
  ?learner ed:hasDecisionStrategy ?strategy .
  ?strategy rdf:type ed:ExploratoryReasoning .
  
  # Low risk classification
  ?learner ed:hasRisk ed:LowRisk .
  
  # Learning outcome
  ?learner ed:experiencedOutcome ?outcome .
  ?outcome rdf:type ed:MasteryAchievement ;
           ed:outcomeScore ?outcomeScore .
  FILTER (?outcomeScore > 0.75)
  
  # Order by strongest emotional intensity (most reliable signals)
} ORDER BY DESC(?emotionalIntensity) DESC(?outcomeScore)
```

#### 4. **SPARQL Query for Dropout Risk Pattern**

```sparql
PREFIX ed: <http://www.semanticweb.org/ed001/ontology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

# Query: Identify dropout risk patterns

SELECT ?learner ?emotion ?cogState ?strategy ?outcome 
       (CONCAT("DROPOUT RISK: ", STR(?emotion), " + ", STR(?cogState), " → ", STR(?strategy)) AS ?riskPattern)
WHERE {
  # Negative deactivating emotions (disengagement)
  ?learner ed:hasEmotion ?emotion .
  {
    ?emotion rdf:type ed:BoredomState .
  } UNION {
    ?emotion rdf:type ed:HopelessnessState .
  }
  
  # Degraded cognitive state
  ?learner ed:hasCognitiveState ?cogState .
  {
    ?cogState rdf:type ed:DistractedState .
  } UNION {
    ?cogState rdf:type ed:NarrowedAttention .
  }
  
  # Avoidance or persistence failure
  ?learner ed:hasDecisionStrategy ?strategy .
  ?strategy rdf:type ed:AvoidanceStrategy .
  
  # High risk classification
  ?learner ed:hasRisk ed:HighRisk .
  
  # Dropout outcome
  ?learner ed:experiencedOutcome ?outcome .
  ?outcome rdf:type ed:DropoutEvent .
  
} ORDER BY ?learner
```

#### 5. **Expected Query Results**

**Optimal Pattern (CQ2a - Forward Chain):**
```
?learner = Student_Bob
?emotion = Emotion_Curiosity_Bob
?cogState = CognitiveState_FocusedAttention_Bob
?strategy = DecisionStrategy_ExploratoryReasoning_Bob
?outcome = Outcome_MasteryAchievement_Bob
?outcomeScore = 0.89
?chain = "OPTIMAL: CuriosityState → FocusedAttention → ExploratoryReasoning"
```

**Dropout Pattern (CQ2b - Inverse Chain):**
```
?learner = Student_Carol
?emotion = Emotion_Boredom_Carol
?cogState = CognitiveState_DistractedState_Carol
?strategy = DecisionStrategy_AvoidanceStrategy_Carol
?outcome = Outcome_DropoutEvent_Carol
?riskPattern = "DROPOUT RISK: BoredomState + DistractedState → AvoidanceStrategy"
```

#### 6. **How This Answers CQ2**

✅ **Identifies all emotions** linked to MasteryAchievement  
✅ **Maps cognitive states** supporting optimal outcomes  
✅ **Reveals decision strategies** producing high performance  
✅ **Inverse mapping**: Identifies dropout risk patterns  
✅ **Validates CVT**: Curiosity (control+value) → Deep Learning  
✅ **Validates CVT**: Boredom (low value) + Hopelessness (low control) → Dropout  

---

---

## COMPETENCY QUESTION 3 (CQ3)
### **"Given a learner's detected emotion and current task context, what adaptive recommendation should the system generate, and what is the confidence in that recommendation?"**

### Why This Question Matters
This CQ demonstrates the **operational, real-time capability** of the ontology: translating ML emotion detection + context into machine-actionable, adaptive recommendations. This is the closing loop between symbolic reasoning and practical educational interventions.

---

### How the ED-001 Ontology Answers CQ3

#### 1. **Class Hierarchy for Recommendations**

The ontology defines three recommendation types, each paired to specific emotion-cognitive patterns:

```
AdaptiveRecommendation
├── StressReduction (for anxiety-driven high load)
│   └── Tactics: task decomposition, extended time, worked examples
├── EngagementBoost (for boredom or disengagement)
│   └── Tactics: gamification, challenge increase, novelty
└── ScaffoldingSupport (for help-seeking or low confidence)
    └── Tactics: hints, peer collaboration, explicit strategy instruction
```

#### 2. **SWRL Rules for Recommendation Generation**

**SWRL Rule 4 - Anxiety Intervention:**
```
IF   hasEmotion(Learner, Anxiety)
  ∧ hasRisk(Learner, HighRisk)
THEN receivesRecommendation(Learner, StressReduction)
```

**SWRL Rule 5 - Boredom Intervention:**
```
IF   hasEmotion(Learner, Boredom)
THEN receivesRecommendation(Learner, EngagementBoost)
```

**SWRL Rule 6 - Frustration Avoidance Prevention:**
```
IF   hasEmotion(Learner, Frustration)
  ∧ hasCognitiveState(Learner, DistractedState)
THEN hasDecisionStrategy(Learner, AvoidanceStrategy)
  ∧ hasRisk(Learner, HighRisk)
  ∧ receivesRecommendation(Learner, ScaffoldingSupport)
```

#### 3. **Extended Data Model: Recommendation Confidence**

Add to the ontology data properties:

```xml
<owl:DatatypeProperty rdf:about="http://www.semanticweb.org/ed001/ontology#confidence">
  <rdfs:domain rdf:resource="http://www.semanticweb.org/ed001/ontology#AdaptiveRecommendation"/>
  <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#float"/>
  <rdfs:comment>Confidence score 0.0-1.0: derived from emotion intensity and rule match strength</rdfs:comment>
</owl:DatatypeProperty>

<owl:DatatypeProperty rdf:about="http://www.semanticweb.org/ed001/ontology#actionDescription">
  <rdfs:domain rdf:resource="http://www.semanticweb.org/ed001/ontology#AdaptiveRecommendation"/>
  <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#string"/>
  <rdfs:comment>Human-readable action for learner or teacher (e.g., "Break this problem into 3 smaller steps")</rdfs:comment>
</owl:DatatypeProperty>

<owl:DatatypeProperty rdf:about="http://www.semanticweb.org/ed001/ontology#systemAction">
  <rdfs:domain rdf:resource="http://www.semanticweb.org/ed001/ontology#AdaptiveRecommendation"/>
  <rdfs:range rdf:resource="http://www.w3.org/2001/XMLSchema#string"/>
  <rdfs:comment>Machine-executable action (e.g., "ADJUST_TIME_LIMIT=+30%", "SHOW_WORKED_EXAMPLE=true")</rdfs:comment>
</owl:DatatypeProperty>
```

#### 4. **SPARQL Query to Answer CQ3**

```sparql
PREFIX ed: <http://www.semanticweb.org/ed001/ontology#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

# Query: Real-time recommendation generation for a specific learner
# Returns the most confident, actionable recommendation

SELECT ?recommendation ?recommendationType ?confidence ?actionDescription ?systemAction ?emotion ?cogLoad
WHERE {
  # Focus on specific learner (in practice, this comes from real-time session)
  ?learner ed:learnerID "STUDENT_ALICE_001" .
  
  # Current emotion (from ML classifier, real-time)
  ?learner ed:hasEmotion ?emotion .
  ?emotion ed:intensity ?emotionIntensity .
  
  # Current cognitive state (inferred from emotion + context)
  ?learner ed:hasCognitiveState ?cogState .
  ?cogState rdf:type ?cogLoad .
  
  # Current risk
  ?learner ed:hasRisk ?riskLevel .
  
  # Recommendation derived by SWRL rule
  ?learner ed:receivesRecommendation ?recommendation .
  ?recommendation rdf:type ?recommendationType ;
                 ed:confidence ?confidence ;
                 ed:actionDescription ?actionDescription ;
                 ed:systemAction ?systemAction .
  
  # Sort by confidence: most confident recommendations first
} ORDER BY DESC(?confidence) LIMIT 1
```

#### 5. **Real-Time Execution Example**

**Input Stream (from learning platform at time T):**
```xml
<!-- Real-time sensor/behavior signals captured -->
<ed:Learner rdf:about="http://example.com/learner/alice_session_2026_01_15">
  <ed:learnerID>ALICE_2026_001</ed:learnerID>
  <ed:responseSpeed rdf:datatype="...#float">0.45</ed:responseSpeed>  <!-- fast = stress -->
  <ed:errorRate rdf:datatype="...#float">0.62</ed:errorRate>           <!-- high errors -->
  <ed:questionSwitchingFrequency rdf:datatype="...#integer">7</ed:questionSwitchingFrequency>  <!-- frequent switches = anxiety -->
  <ed:isAttempting rdf:resource="#task_algebra_002"/>
  <ed:timestamp rdf:datatype="...#dateTime">2026-01-15T10:42:15Z</ed:timestamp>
</ed:Learner>

<!-- ML Emotion Classifier OUTPUT -->
<ed:AnxietyState rdf:about="#emotion_anxiety_alice_T2">
  <ed:intensity rdf:datatype="...#float">0.81</ed:intensity>
  <ed:confidence rdf:datatype="...#float">0.92</ed:confidence>
  <ed:timestamp rdf:datatype="...#dateTime">2026-01-15T10:42:15Z</ed:timestamp>
</ed:AnxietyState>

<rdf:Description rdf:about="#emotion_anxiety_alice_T2">
  <rdf:type rdf:resource="http://www.semanticweb.org/ed001/ontology#AnxietyState"/>
</rdf:Description>

<!-- Task Context -->
<ed:LearningTask rdf:about="#task_algebra_002">
  <ed:hasContext rdf:resource="#context_exam_final"/>
</ed:LearningTask>

<ed:ExaminationContext rdf:about="#context_exam_final">
  <ed:taskDifficulty>High</ed:taskDifficulty>
  <ed:timePressure rdf:datatype="...#boolean">true</ed:timePressure>
  <ed:timeRemaining rdf:datatype="...#integer">450</ed:timeRemaining>  <!-- 7.5 minutes left -->
</ed:ExaminationContext>
```

**Ontology Reasoning Process:**

```
Step 1: Assert emotion
  asserting: hasEmotion(alice, emotion_anxiety_T2)
             emotion_anxiety_T2 rdf:type AnxietyState
             intensity(emotion_anxiety_T2) = 0.81

Step 2: Apply SWRL Rule 1 (Stress Detection)
  ✓ Conditions match: AnxietyState + High difficulty + TimePressure
  → Deriving: hasCognitiveState(alice, HighCognitiveLoad)

Step 3: Apply SWRL Rule 2 (Decision Risk)
  ✓ Condition match: HighCognitiveLoad
  → Deriving: hasDecisionStrategy(alice, ImpulsiveDecision)
             hasRisk(alice, HighRisk)

Step 4: Apply SWRL Rule 4 (Anxiety Intervention)
  ✓ Conditions match: Anxiety + HighRisk
  → Deriving: receivesRecommendation(alice, StressReduction_001)
             confidence(StressReduction_001) = min(0.81, 1.0) = 0.81
```

**Query Result (sent to ITS immediately):**

```json
{
  "recommendation": {
    "type": "StressReduction",
    "confidence": 0.81,
    "actionDescription": "Your stress level is high. Let's take a moment to refocus. Next problem will have extended time and a worked example.",
    "systemAction": "ADJUST_TIME_LIMIT=+120, SHOW_WORKED_EXAMPLE=true, PLAY_RELAXATION_AUDIO=true",
    "targetLearner": "ALICE_2026_001",
    "timestamp": "2026-01-15T10:42:15Z",
    "emotion": "AnxietyState",
    "cognitiveState": "HighCognitiveLoad",
    "riskLevel": "HighRisk"
  }
}
```

#### 6. **Confidence Calculation**

The confidence score is derived from **emotion intensity** (from ML) and **rule match strength**:

```
confidence = emotion_intensity × rule_match_weight × context_weight

For Anxiety Intervention (Rule 4):
confidence = 0.81 (anxiety intensity) 
           × 1.0 (perfect rule match: emotion + risk both present)
           × 1.0 (high-stakes exam context validates intervention)
           = 0.81
```

#### 7. **How This Answers CQ3**

✅ **Accepts real-time emotion input** from ML classifier  
✅ **Links emotion to context** (high difficulty, time pressure)  
✅ **Applies SWRL reasoning** to infer cognitive & decision state  
✅ **Routes to appropriate recommendation** (Stress Reduction for anxiety)  
✅ **Calculates confidence** (0.81 = 81% confidence in this recommendation)  
✅ **Generates dual-audience output**:
   - Human: "Your stress level is high. Let's take a moment..."
   - Machine: "ADJUST_TIME_LIMIT=+120, SHOW_WORKED_EXAMPLE=true"  

✅ **Closes the loop**: ML detection → semantic reasoning → adaptive action  

---

---

## Summary: Why These Three CQs Validate ED-001

| Competency Question | What It Tests | Unique to ED-001 |
|---|---|---|
| **CQ1: At-Risk Learners** | Full 5-layer causal chain (Context → Emotion → Cognition → Decision → Risk) | No ontology models all 5 layers together |
| **CQ2: Outcome Patterns** | Bidirectional reasoning (forward prediction & inverse pattern discovery) + CVT validation | No ontology maps emotions to learning outcomes + dropout patterns |
| **CQ3: Real-Time Recommendations** | ML integration + SWRL reasoning + dual-audience output (human + machine) | No ontology bridges emotion detection to adaptive actions |

---

## Theoretical Grounding

Each CQ is grounded in established theory:

- **CQ1** operationalizes **Control-Value Theory** (Pekrun): High-control, high-value contexts elicit positive emotions; low-control, high-value contexts elicit anxiety
- **CQ2** validates **Cognitive Load Theory**: Negative activating emotions (anxiety, frustration) increase extraneous load; positive activating emotions (curiosity) optimize learning
- **CQ3** applies **Dual Process Theory**: High cognitive load shifts reasoning from deliberative (System 2) to heuristic (System 1), necessitating adaptive support

---

## Implementation Notes

To execute these queries in Protégé:
1. Load `ED-001_EmotionAwareDecisionSupport.owl`
2. Create a separate knowledge graph file with instances (learners, emotions, tasks, contexts)
3. Run **Pellet** or **HermiT** reasoner to infer SWRL-derived facts
4. Execute SPARQL queries via **SPARQLDLPlugin** or external SPARQL endpoint
5. Convert JSON/XML results to ITS API calls

---
