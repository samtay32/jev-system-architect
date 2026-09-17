---
name: jev-system-architect
description: >
  Identify, design, review, and improve opportunities to use TypeSafe AI's
  Jev/System One as a semantic decision layer inside normal software.
  Use when a system contains unstructured data, brittle parsing, prompt-to-JSON
  workflows, semantic routing, classification, ranking, verification,
  guardrails, extraction from known candidates, or other places where
  deterministic code needs bounded human-like judgment.
---

# Jev System Architect

## Purpose

Use this skill to determine:

1. Whether Jev belongs in a system at all
2. Exactly where the Jev boundary should be
3. Which judgments should become Choice, Score, or Noul questions
4. What state each judgment should see
5. Which questions should run together
6. How ordinary code should compose the results
7. How uncertainty should change system behavior
8. How the integration should be evaluated before production use

This is primarily an architecture and decomposition skill.

For exact SDK syntax, request/response fields, models, limits, or version-specific API behavior, read the current TypeSafe documentation and, when available, use TypeSafe's official agent skill.

Do not invent API fields.

---

## Core Mental Model

Jev is not the application.

Jev is not the workflow engine.

Jev is not an autonomous agent.

The preferred architecture is:

```
APPLICATION STATE
      ↓
bounded semantic questions
      ↓
     JEV
      ↓
typed judgments + probabilities
      ↓
DETERMINISTIC APPLICATION CODE
      ↓
route / rank / verify / ask / escalate / execute
```

The model supplies programmable judgment.

Code owns:

- control flow
- permissions
- calculations
- deterministic rules
- persistence
- transactions
- side effects
- tool execution
- authorization
- policy enforcement

Use Jev only where ordinary software needs semantic understanding of unstructured or ambiguous information.

---

## Trigger Conditions

Consider Jev when the system contains a boundary shaped like:

```
unstructured or contextual information
                ↓
     a bounded judgment is required
                ↓
 software needs a typed value to continue
```

Strong candidates include:

- intent classification
- semantic routing
- moderation decisions
- document classification
- evidence verification
- citation checking
- semantic guardrails
- risk signals
- message interpretation
- candidate selection
- ranking dimensions
- relevance judgments
- sentiment or urgency
- user-request interpretation
- tool/function selection
- extraction from a known candidate set
- RAG passage filtering
- workflow triage
- quality judgments
- policy-to-case comparison
- fuzzy matching
- entity alignment
- escalation decisions

Also inspect code for symptoms such as:

```
LLM → prose → parser → JSON → validation → retry
```

or:

```
large regex tree
huge if/else semantic classifier
prompt-based classifier returning arbitrary JSON
agent loop being used only to make a bounded decision
expensive reasoning model used for a trivial classification
```

These are potential Jev insertion points.

---

## Do Not Use Jev By Default

Prefer ordinary code for:

- arithmetic
- exact dates already represented structurally
- database lookups
- exact string matches
- permissions
- authorization
- schema validation
- deterministic business rules
- known transformations
- sorting known numbers
- cryptographic operations
- exact parsers that already solve the problem reliably

Prefer a generative model when the required output is fundamentally open-ended:

- writing
- summarization
- explanation
- brainstorming
- code generation
- long-form synthesis
- open-ended research
- multi-step reasoning

Jev may support those systems by making bounded decisions around them, but it should not be forced into a generative role.

---

## Step 1 — Find the Judgment Boundaries

Inspect the workflow from input to outcome.

Mark every point where the software effectively asks:

> "I have information, but I need to understand what it means before I know which deterministic path to take."

For each candidate boundary, write:

**INPUT STATE:** What information exists?

**JUDGMENT:** What semantic determination is needed?

**OUTPUT:** What bounded value would allow code to continue?

**ACTION:** What will code do with the answer?

**STAKES:** What happens if the judgment is wrong?

Do not immediately implement Jev.

First determine whether deterministic code can solve the problem more reliably.

---

## Step 2 — Apply the Jev Fit Test

A judgment is a strong Jev candidate when most of these are true:

- [ ] Input contains natural language or contextual data.
- [ ] The required output can be bounded.
- [ ] Software needs the answer programmatically.
- [ ] A knowledgeable human could make the judgment quickly.
- [ ] The judgment can be described explicitly.
- [ ] Uncertainty is useful information.
- [ ] The result can be tested against examples or labels.
- [ ] Code can own the action after the judgment.

Be skeptical when the task requires:

- deep deliberation
- long chains of reasoning
- many dependent intermediate conclusions
- open-ended generation
- autonomous planning

Decompose first.

---

## Step 3 — Make Questions Atomic

A Jev question should represent one coherent judgment.

**BAD:**

Analyze this customer and determine whether we should refund them, how angry they are, whether fraud is likely, and which department should handle the case.

**BETTER:**

- `topic` → Choice
- `refund_requested` → Noul
- `fraud_signal` → Noul
- `frustration` → Score

Then compose those answers in code.

The goal is:

```
complex behavior = many small judgments + deterministic composition
```

Do not hide application policy inside a giant semantic question when code could express that policy explicitly.

---

## Step 4 — Select the Correct Primitive

### Choice

Use when exactly one answer should be selected from a defined set.

Shape: **Which one?**

Examples:

- billing | technical | sales | account
- refund | rebook | information
- python | javascript | rust | other

Add an "other", "unknown", or "none" outcome when the supplied options may not cover the input.

### Noul

Use for a proposition whose probability of being true is useful.

Shape: **Is this true?**

Examples:

- Does the customer request a refund?
- Does this passage answer the user's question?
- Does this message request a credential?
- Does the resume state professional Python experience?

Interpret the output as the probability of yes.

Do not interpret ~0.5 as "medium intensity."

It means yes/no are approximately equally plausible.

### Score

Use when the judgment exists on an ordered dimension.

Shape: **Where on this defined spectrum does the state fall?**

Examples:

- calm / concerned / very frustrated
- irrelevant / partially relevant / strongly relevant / direct answer

Each level should describe a concrete semantic condition.

Avoid undefined labels like bad / okay / good / great unless their meaning is explicitly defined.

---

## Step 5 — Design State Deliberately

Send the smallest state that contains the evidence required for the questions.

Prefer structured state:

```json
{
  "ticket": {
    "message": "...",
    "sender": {}
  },
  "customer": {
    "plan": "...",
    "orders": []
  },
  "policy": {
    "refund_policy": "..."
  }
}
```

rather than one giant prose blob.

Principles:

- relevant > comprehensive
- structured > flattened
- observed facts > assumed model knowledge
- current information > stale embedded knowledge

Keep observed facts distinguishable from inferred judgments.

When questions concern specific state fields, point explicitly at those fields.

---

## Step 6 — Fan Out Independent Questions

If multiple questions can be answered from the same existing state, prefer asking them together.

```
                    ┌── topic
                    ├── urgency
STATE ── JEV ───────┼── refund_requested
                    ├── frustration
                    └── fraud_signal
```

Do not create unnecessary serial calls such as:

```
Jev → answer → Jev → answer → Jev → answer
```

when all three questions could have been evaluated from the original state.

Ask speculative questions when they are cheap and may be useful later.

Example: `category`, `bug_severity`, `reproduction_quality`, `refund_requested`, `frustration`

If `category = billing`, code simply ignores the bug-specific answers.

---

## Step 7 — Use Multiple Jev Stages Only for Real Dependencies

A second Jev request is justified when the first answer is required to:

- fetch new evidence
- construct new state
- choose the next candidate set
- determine which questions can meaningfully be asked
- traverse a hierarchy
- retrieve documents required for the next judgment

Example:

```
STEP 1  Which product family?
   ↓
code retrieves products in that family
   ↓
STEP 2  Which exact product best matches?
```

Do not serialize questions merely because the workflow conceptually contains several steps.

---

## Step 8 — Keep Composition in Code

Jev produces semantic signals.

Code decides what those signals mean operationally.

Example:

```
topic = billing
refund_requested = 0.94
frustration = high
```

Application code may decide:

```
if topic == billing:
    route_to_billing()

if refund_requested > REFUND_FLAG_THRESHOLD:
    add_refund_flag()

if frustration > PRIORITY_THRESHOLD:
    increase_priority()
```

Business logic should remain inspectable.

Do not ask Jev "What should our entire application do?" when the answer can instead be constructed from smaller judgments.

---

## Step 9 — Treat Uncertainty as a First-Class Signal

Never treat typed output as equivalent to truth.

Typed output means the interface is constrained.

It does not mean the semantic judgment cannot be wrong.

For Choice and Score, inspect probabilities and confidence.

For Noul, inspect the yes probability directly.

Design behavior such as:

- **HIGH CERTAINTY** → automatic path
- **MEDIUM CERTAINTY** → ask for confirmation / gather more evidence / stronger model / flag for review
- **LOW CERTAINTY** → do not guess / escalate

Thresholds are application policy.

They must reflect cost of false positives/negatives, reversibility, user/financial/security/regulatory impact.

A threshold copied from an example is not evidence that it is appropriate for this system.

---

## Step 10 — Match Confidence Gates to Risk

Do not use one universal confidence threshold.

Example:

- show_help_article — low consequence
- route_support_ticket — moderate consequence
- cancel_subscription — higher consequence
- move_money — very high consequence

The semantic judgment may be identical while the required certainty and confirmation path differ.

For consequential actions, Jev should usually inform a controlled decision boundary rather than independently authorize the side effect.

---

## Step 11 — Design Evaluation Before Automation

Before productionizing a Jev boundary, identify representative cases.

Include: easy positives, easy negatives, ambiguous cases, missing-evidence cases, edge cases, adversarial cases, rare but costly failures.

Record: state, question, expected behavior, Jev answer, probabilities/confidence, application action, ground-truth outcome when available.

Evaluate the system behavior, not merely whether the top label looked reasonable.

For thresholded systems, measure behavior at several thresholds.

Especially measure: coverage / automation rate, false-positive rate, false-negative rate, human-review rate, cost of errors, latency, cost.

Tune thresholds using actual target-domain data.

---

## Step 12 — Centralize Jev Policy

Keep Jev questions, criteria, weights, and threshold constants easy to find and review.

Prefer a structure such as:

```
ai/
  jev/
    questions.*
    thresholds.*
    evaluation/
    client.*
```

or the equivalent structure appropriate for the project.

Do not scatter semantic policy across dozens of route handlers.

Human reviewers should be able to inspect:

- What is Jev judging?
- What definitions did we give it?
- What thresholds cause actions?
- Which actions are reversible?
- What happens when confidence is low?

---

## Step 13 — Preserve Raw Judgments

When practical, retain the useful raw outputs separately from the action derived from them.

For example:

```
refund_probability = 0.91
frustration_score = 1.72
topic = billing
topic_confidence = 0.88
```

instead of storing only `priority = high`.

This allows re-evaluating thresholds, changing weights, auditing behavior, comparing model versions, investigating failures, and downstream analytics.

Do not retain sensitive data beyond the application's legitimate requirements.

---

## Step 14 — Look for Jev Around Generative AI

Jev is often valuable around an LLM rather than replacing it.

Examples:

```
INPUT → Jev guardrail → LLM → Jev citation/quality verification → code decides whether to return
```

or:

```
query → retrieval → Jev relevance / injection judgments → selected context → reasoning model
```

or:

```
user request → Jev intent/function selection → deterministic typed function → result
```

This can reduce the amount of application control entrusted to an open-ended generative model.

---

## Architecture Review Procedure

When asked to review an existing system for Jev opportunities:

### 1. Map the workflow

Identify: inputs, deterministic steps, semantic decisions, model calls, parsers, routers, tool calls, side effects, human-review points.

### 2. Produce a Jev opportunity map

For each candidate:

- **LOCATION:** Current code/module/workflow
- **CURRENT METHOD:** How the judgment is made today
- **JEV CANDIDATE:** Yes / No / Maybe
- **WHY:** Why a bounded semantic decision is or is not appropriate
- **STATE:** Minimum required evidence
- **PRIMITIVE:** Choice / Score / Noul
- **QUESTIONS:** Proposed atomic judgments
- **COMPOSITION:** How code should use them
- **UNCERTAINTY:** What happens when the answer is unclear
- **RISK:** Consequence of a wrong judgment
- **EVALUATION:** How to validate it

### 3. Prioritize by leverage

Prefer opportunities where Jev could replace fragile semantic code, repeated LLM parsing, expensive trivial reasoning calls, unbounded agent loops, duplicated classifiers, or manual review of obvious cases — while keeping system behavior bounded and testable.

### 4. Recommend the smallest viable Jev boundary

Do not redesign the whole application around Jev unless the architecture genuinely requires it.

Insert the smallest useful semantic primitive first.

---

## Implementation Rule

Before writing actual TypeSafe API code:

1. Read the current TypeSafe documentation.
2. Read the relevant primitive documentation.
3. Read the closest applicable cookbook.
4. Use the official TypeSafe agent skill if available.
5. Confirm the installed SDK version.
6. Do not invent request or response fields.
7. Preserve the project's existing language, framework, and conventions unless there is a concrete reason not to.

This skill provides the architecture.

The live TypeSafe documentation is the source of truth for implementation details.

---

## Example — Support System

Current architecture:

```
customer message → large LLM prompt → generated JSON → parser / retry → route
```

Potential Jev architecture:

```
customer message + account context
                ↓
               JEV
       ┌────────┼──────────┐
       ↓        ↓          ↓
    topic   frustration  refund?
   Choice      Score       Noul
       └────────┼──────────┘
                ↓
             CODE
       ┌────────┼─────────┐
       ↓        ↓         ↓
    billing  priority   review
```

The key architectural improvement is not merely replacing one model with another.

It is moving from **MODEL OWNS INTERPRETATION + RESPONSE SHAPE** to **MODEL SUPPLIES SMALL JUDGMENTS / CODE OWNS THE SYSTEM**.

---

## Example — RAG Pipeline

Instead of sending every retrieved passage directly to a reasoning model:

```
query → retriever → candidate passages → Jev judgments per candidate
  - Does this passage address the query?
  - Does it contain suspicious instructions?
  - Does it contradict the user's premise?
  - How directly relevant is it?
→ code filters / ranks → reasoning model
```

Jev acts as a semantic control layer.

---

## Example — Tool Calling

Instead of allowing an agent to freely choose an arbitrary next action:

```
user request → JEV → Choice: function + Choice/Noul: bounded argument judgments
→ confidence/risk gate → ordinary typed function call
```

Code still validates arguments, authorization, permissions, and side effects.

---

## Red Flags

Stop and reconsider the architecture when you see:

- one giant question making many unrelated judgments
- Jev deciding its own next tool repeatedly
- business logic hidden inside prompt prose
- deterministic calculations delegated to Jev
- automatic high-stakes side effects with no risk gate
- thresholds copied blindly from examples
- serial Jev calls whose questions could run together
- huge irrelevant state
- confidence interpreted as guaranteed correctness
- Noul 0.5 interpreted as "medium"
- typed output described as hallucination-proof
- SDK fields written from memory instead of current docs

---

## Completion Standard

A good Jev integration should make it easy to answer:

- Why is AI needed here?
- What exact judgment is the model making?
- Why is that judgment atomic?
- Why is this the correct primitive?
- What evidence can the model see?
- Which judgments run together?
- What remains deterministic?
- What happens when Jev is uncertain?
- What happens when Jev is wrong?
- How do we measure whether this is better?
- Can a human easily review the questions and thresholds?

If those answers are unclear, the Jev boundary is not finished.

---

## Default Instruction to the Agent

When this skill activates:

> Inspect the system for places where deterministic software needs bounded semantic judgment over unstructured or contextual state. Do not assume Jev should be used. Prefer normal code whenever the problem is deterministic. For strong candidates, design the smallest possible Jev boundary using atomic Choice, Score, or Noul questions; batch independent judgments over the same state; keep control flow and side effects in code; design uncertainty handling according to action risk; and define how the behavior will be evaluated. Before implementing TypeSafe-specific API code, consult the current official documentation and official TypeSafe skill.