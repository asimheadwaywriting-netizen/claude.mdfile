# Decisions Log

Append-only record of key decisions made during build sessions. Prevents Claude from re-inferring intent next session and silently breaking things that were intentional.

---

## When to Write an Entry
- Claude chose one approach over another
- User overrode something Claude built
- A requirement changed mid-build
- Something broke and was fixed a specific way

## Entry Format

```
### [Session Date] — [Workflow Name]
- **Decision:** [what was decided]
- **Why:** [the reasoning]
- **Alternative considered:** [what was rejected and why]
```

---

## Log

<!-- Paste session entries below this line -->
