# Advanced Techniques

Supplementary techniques referenced from the main skill. These are situational — use them when the main workflow surfaces a need, not as mandatory steps.

---

## Handling Trace Convergence (Fan-in)

When a single output depends on multiple inputs converging:

1. Identify all upstream branches
2. Pick *one* to follow completely
3. Mark the others as "unexplored — converges at [location]"
4. Complete your slice on the chosen branch
5. Return to unexplored branches as separate traces

Trying to follow all branches simultaneously leads to context overflow. Complete one path, then widen.

---

## Recognizing Dead Code

Real codebases contain ghost code — branches that never execute, features behind frozen flags, deprecated paths. Don't waste cycles on code that doesn't matter.

1. Start from entry points, trace callers of suspicious functions
2. Check feature flags — find their actual values
3. Check for TODO/DEPRECATED/UNUSED comments
4. Cross-reference git history: unchanged 18+ months with no callers → likely dead

Assign confidence: **live** / **probably live** / **suspicious** / **likely dead**

---

## Verification Procedures

**Golden rule**: If you didn't see it fail, you don't know if it works.

### Pure Functions
Identify claimed behavior → list edge cases → execute with representative inputs → execute with edge cases → observe failure modes.

**Key question**: Do actual production inputs match your test inputs?

### Impure Functions
Identify external dependency → determine failure modes (slow, down, bad data) → simulate if possible → observe graceful vs. catastrophic degradation.

**Key question**: What is the blast radius when this dependency fails?

### When You Can't Execute
Trace error handling paths · Check for defensive code (null checks, type guards) · Look for tests (they reveal what the author thought mattered) · Assume unhandled for impure calls without evidence of error handling.

### Decision Tree

```
Is the function pure?
├─ Yes → Test edge cases with representative inputs
└─ No → What external dependency?
         ├─ Identify failure modes
         ├─ Can you simulate failure?
         │   ├─ Yes → Do it, observe behavior
         │   └─ No → Check error handling paths
         └─ Assume unhandled if no evidence
```
