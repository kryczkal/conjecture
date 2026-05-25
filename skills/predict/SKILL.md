---
name: predict
description: >
  State a prediction before acting. Use PROACTIVELY at the start of any significant
  task — implementation, refactor, investigation, brainstorm. The prediction is a bet
  about what will happen, specific enough to be wrong. After the task completes, add
  a scorecard. The wiki accumulates predictions and scorecards to track learning rate.
  This skill should be used automatically before major actions, not only when invoked.
---

# Predict

Before acting, state what you expect. After acting, record what happened. That's it.

## When to use (PROACTIVE — do not wait for user to invoke)

Use at the start of any task that:
- Changes code (implementation, refactor, bug fix)
- Makes a recommendation (architect, brainstorm, code review)
- Investigates something (bug-hunter, debugging)
- Takes more than 5 minutes

Do NOT use for trivial tasks (rename a variable, fix a typo, answer a factual question).

## The prediction (before acting)

State in 1-3 sentences:

```
**Prediction:** [what I expect to happen / what I expect to find / how long this will take]
**Fail condition:** [what observation would prove this prediction wrong]
```

Write it in the chat before starting work. Not in a file — in the conversation. Keep it light. One prediction per task, not per sub-step.

Examples:
- "**Prediction:** This bug is in the auth middleware — the token expiry check uses < not <=. Fix is a 1-line change. **Fail:** If the issue is elsewhere or requires >3 files changed."
- "**Prediction:** The architect review will find coupling between engine and frontend via shared telemetry types. **Fail:** If the top finding is about something else entirely."
- "**Prediction:** Spectator mode needs zero backend changes — the SSE endpoint is already open. **Fail:** If any backend route needs modification."

## The scorecard (after acting)

When the task is complete, append to the conversation:

```
**Scorecard:**
- Accuracy: exact | partial | wrong
- Surprise: [what I didn't expect, in one sentence]
```

That's all. Two fields. If the prediction was exact, say so and move on. If it was wrong, the surprise is the learning.

## Filing to the wiki (optional, for significant predictions)

If the prediction is about something worth tracking long-term (an architectural claim, an LLM behavioral observation, a process hypothesis), file it to the wiki:

```
wiki/predictions/open/YYYY-MM-DD-<slug>.md
```

Use the wiki's prediction format (from CLAUDE.md). Update index.md and log.md. This is for predictions that future sessions should know about — not for every 1-line bet in a conversation.

## What this produces over time

After 20+ tasks with predictions and scorecards, patterns emerge:
- "I consistently underestimate implementation time by 30%"
- "My architecture predictions are exact but my cost estimates are always wrong"
- "When I predict something is 'trivial,' it's wrong 40% of the time"

These patterns are the system's learning rate. They tell the user where to trust the LLM and where to double-check. They tell the LLM where its model of the codebase is accurate and where it's drifting.

## Anti-patterns

- Don't make predictions so vague they can't be wrong ("this will probably work")
- Don't skip the scorecard — a prediction without follow-up is useless
- Don't make the prediction longer than the task — keep it 1-3 sentences
- Don't file every prediction to the wiki — only ones with long-term relevance
