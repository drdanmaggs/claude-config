---
description: Lightweight Explore-Plan-Code workflow for standard tasks (no subagents)
---

# EPC Lite - Fast Implementation

**Use this for:** Standard features, UI changes, simple CRUD, refactoring
**Don't use for:** Payments, auth, complex business logic (use /tdd instead)

## Stage 1: Quick Scope Check

Do we have clear acceptance criteria?

**If YES:**
- Great! Move to Stage 2.

**If NO:**
- Ask max 3 clarifying questions:
  1. What should work when this is done?
  2. What's the user-facing behaviour?
  3. Any edge cases I should know about?

**[WAIT FOR ANSWERS]**

## Stage 2: Quick Plan

Think through the approach (keep it simple):
- What files need changing?
- Any new components/functions needed?
- Roughly how long? (S: <1hr, M: 1-3hrs, L: consider /epc full)

If **L** → suggest using full `/epc` with issue-scoper.

Present plan in 3-5 bullets. Ask: **"Sound good?"**

**[WAIT FOR APPROVAL]**

## Stage 3: Implement

Create feature branch:
```bash
git checkout -b feat/[brief-description]
```

Build it:
- Write code following project patterns
- Add tests alongside (colocated)
- Keep commits small and logical

## Stage 4: Verify

Before finishing:
1. Run `npm run build` (or equivalent) - does it compile?
2. Run `npm run lint` - any errors?
3. Run `npm test` (if tests exist) - do they pass?
4. Quick manual check - does it work?

**If any fail:** Fix them now.

**If security-sensitive or complex:** Ask "Should I use code-reviewer for this?"

## Stage 5: Wrap Up

Show what was done:
```
✓ [What was built]
✓ Tests: [passing/added]
✓ Build: Clean
✓ Ready for: [commit/PR/further work]
```

Ask: **"Ready to commit?"**

If yes:
```bash
git add -A
git commit -m "feat: [clear description]"
git push -u origin feat/[branch-name]
```

---

## When to Use Full /epc Instead

Use `/epc` (with issue-scoper) if:
- Feature feels too big (>3 hours)
- Multiple acceptance criteria
- Unclear requirements
- Breaking into vertical slices would help

## When to Use /tdd Instead

Use `/tdd` (full TDD workflow) if:
- Payments or financial logic
- Authentication or authorisation
- Complex business rules with edge cases
- Refactoring critical path

---

**Remember:** This is 80% solution territory. Ship fast, iterate based on reality.
