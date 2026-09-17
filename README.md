# Jev System Architect Skill

**A system-architecture skill for TypeSafe AI's Jev / System One.** Continuously asks: *Where does this system contain fuzzy semantic judgment that should become a small Jev decision primitive?*

TypeSafe already publishes an official skill that gives coding agents current API/SDK context and implementation guidance. The missing layer is architecture: keep deterministic logic, control flow, and side effects in code; use Jev for narrow structured judgments; batch independent questions; and route uncertainty explicitly.

**Install both:** TypeSafe's official skill **and** this one.

---

## Install

Copy into your agent skills folder (Cursor / Claude Code / compatible harness):

```bash
# from this repo
cp -R skills/jev-system-architect ~/.cursor/skills/jev-system-architect
# or wherever your agent loads skills from
```

Or clone and point your agent at `skills/jev-system-architect/SKILL.md`.

---

## What this skill is for

Use when a system contains unstructured data, brittle parsing, prompt-to-JSON workflows, semantic routing, classification, ranking, verification, guardrails, extraction from known candidates, or other places where deterministic code needs bounded human-like judgment.

It helps you determine:

1. Whether Jev belongs in a system at all  
2. Exactly where the Jev boundary should be  
3. Which judgments should become Choice, Score, or Noul questions  
4. What state each judgment should see  
5. Which questions should run together  
6. How ordinary code should compose the results  
7. How uncertainty should change system behavior  
8. How the integration should be evaluated before production use  

This is primarily an **architecture and decomposition** skill.

For exact SDK syntax, request/response fields, models, limits, or version-specific API behavior, read the current [TypeSafe documentation](https://typesafe.ai) and use TypeSafe's official agent skill when available. **Do not invent API fields.**

---

## Core mental model

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

The model supplies programmable judgment. Code owns control flow, permissions, calculations, persistence, transactions, side effects, tool execution, authorization, and policy enforcement.

---

## Related

- TypeSafe AI / Jev docs — source of truth for API and SDK details  
- TypeSafe official agent skill — implementation guidance  

### Use case / related

**[jevlike](https://github.com/vinnylarouge/jevlike)** — An independent research starter (MIT) for training small local option-scorers that share the same I/O shape as a Jev-like Choice: context text + a changing list of N text options → one probability per option in one pass (not token-by-token generation).

Not TypeSafe's commercial Jev and not a reverse-engineered copy of TypeSafe's private design. Useful when you want to experiment with or train a small local option-scorer while using `jev-system-architect` for where Choice/Score/Noul boundaries belong in your application.

Includes Doom/chess demos and a Wikispeedia example.

---

## License

MIT — see `LICENSE`.

Built for public reuse by [Ripe Avocado](https://x.com/crypto1618).
