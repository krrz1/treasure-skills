---
name: clarity-check
description: "Use this skill when you receive a technical task request with a vague action verb — test, optimize, fix, debug, refactor, deploy, add, improve — and there isn't enough context to know which specific approach is wanted. The ambiguity isn't in the words themselves, but in the fact that different reasonable interpretations lead to meaningfully different work: 'test this' could mean unit tests, integration tests, or manual CLI verification; 'fix the CI failure' could mean patching the test, fixing the underlying code, or adjusting the pipeline config. Use this skill to check if your interpretation is reliable before diving in. Skip for clearly specified micro-tasks: renaming a variable, adding a comment, fixing a typo, changing a loop style."
---

# Intent Reliability Check

## Core Idea

The same prompt can lead different models toward completely different interpretations. The problem isn't whether the prompt is clear — it's whether **your understanding is reliable**. "How do I test this?" is grammatically clear, but it could mean unit tests, integration tests, manual verification, or CI setup. These are fundamentally different directions.

The key question: **If I pick the wrong interpretation, will the user be clearly dissatisfied with the result?**

---

## Workflow

### Step 1: Enumerate Interpretations (internal, quick)

Before doing anything, run through:

- How many meaningfully different ways can this request be interpreted? (list 2–3)
- What would the output/action look like for each interpretation?

You don't need to enumerate every possibility — only find **substantially different paths**. If all interpretations converge on the same general direction, you're fine.

### Step 2: Assess Reliability

Use two dimensions:

**1. Outcome Divergence (most important)**
How different are the actions corresponding to each interpretation?
- Different types of work ("write tests" vs "run tests" vs "set up CI") → high divergence, low reliability
- Same direction, different details ("use pytest" vs "use unittest") → low divergence, high reliability

**2. Context Anchoring**
Does the conversation history, codebase, or user's description provide clues to disambiguate?
- User previously said "I just need to verify the feature works" → strong anchor, high reliability
- No context at all → weak anchor, low reliability

### Step 3: Decide Based on Reliability

| Reliability Score | Action |
|-------------------|--------|
| High (75+/100) | Proceed directly, no declaration needed |
| Medium (55–74/100) | State your interpretation in one sentence before acting |
| Low (<55/100) | **Pause and ask one clarifying question with concrete options** |

### Step 4: How to Ask (Low Reliability Only)

Don't use open-ended questions ("Can you be more specific?") — that dumps all the cognitive burden on the user. Instead, **enumerate your interpretations and let the user choose**.

Format:
```
I have a few interpretations, confidence ~XX/100 — please help me confirm:

A. [Interpretation A] — action: [one sentence describing what you'd do]
B. [Interpretation B] — action: [one sentence describing what you'd do]

Which is closer to what you need?
```

Why this works:
- The user sees your interpretations and can immediately tell if you're on track
- Even if none fit, the options give them a concrete reference point to describe what they actually want
- Avoids the "I wasn't clear → you asked → I'm still not clear" loop

---

## Example

**Prompt:** "I added todo.py to kiyo_cli — how do I test it?"

**Internal enumeration:**
- Interpretation A: Write unit tests for the functions in todo.py (pytest + mocks)
- Interpretation B: Verify the todo feature works end-to-end after integration into the CLI (run CLI, check actual behavior)
- Interpretation C: Add todo.py tests to the existing CI pipeline

Outcome divergence: High (three completely different types of work)
Context anchoring: Low (no context clues)
→ Reliability ~35/100, needs clarification

**Correct output:**
```
I have a few interpretations, confidence ~35/100 — please help me confirm:

A. Write unit tests for todo.py's function logic (pytest, mock external dependencies)
B. Verify the todo feature works end-to-end inside kiyo_cli (actually run CLI commands to check)
C. Add todo.py tests to the existing CI pipeline

Which is closer to what you need?
```

---

## When to Skip This Check

- Pure information requests ("What does this code do?") — explanations can't really be "wrong"
- Clearly reversible micro-tasks ("Rename this variable to snake_case") — easy to fix if wrong
- The user has already provided enough context in the conversation to resolve any ambiguity

---

## Notes

**Stay completely silent at high reliability.** All reasoning happens internally — the user should never see your score, your analysis steps, or your "do I need to clarify?" judgment. At high scores (75+/100), just act. The medium-score one-liner is about the task itself ("I'm treating this as X, proceeding..."), not about the check process. The user should only feel the skill's presence when you're actually asking them something.

**Interpretations need real differences, not nitpicking.** When enumerating, look for paths that lead to genuinely different deliverables — not invented ambiguity. "for loop vs map" is not a real difference; "write tests vs run tests" is.

**Options must be concrete, not categorical.** "Functional testing vs unit testing" is weak — users may not know these terms. Describe the actual action: "run CLI to verify end-to-end behavior" beats "integration testing".
