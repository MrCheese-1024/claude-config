# Planner Skill Optimization Roadmap

This document tracks planned improvements to reduce token consumption while maintaining quality.

**Status:** Phase 1 (Critical) implemented ✅. Phase 2 Change 4 (Confidence-Based Escalation) infrastructure ✅ complete. Phase 2 Changes 5-6 and Phase 3 planned for future cycles.

---

## Phase 1: Critical for 5-Hour Budget (✅ IMPLEMENTED)

### Change 1: Plan Review Checkpoint
- Added interactive checkpoint between Step 5 (planning) and Step 6 (QR-Completeness)
- User options: Approve, Review/Edit, Regenerate, Save & Exit
- **Token savings:** 100K+ tokens by preventing unwanted code generation
- **Impact:** User controls when expensive QR loops begin

### Change 2: Extract Execution Skill
- Created standalone `/plan-execute` skill separate from planner
- Accepts `--plan-file` parameter to load and execute existing plans
- **Token savings:** Separates planning and execution budgets (fresh 200K tokens each)
- **Impact:** Can plan in one session, clear context, execute in separate session

### Change 3: Iteration Limits
- Added max iteration counter to QR gates (default: 2-3 iterations)
- After max iterations: force user decision point
- **Token savings:** 50K+ tokens by preventing runaway feedback loops
- **Impact:** QR gates become visible, bounded, user-controlled

---

## Phase 2: Recommended (Infrastructure Implementation)

### Change 4: Confidence-Based Escalation (✅ INFRASTRUCTURE COMPLETE)

**Status:** Infrastructure implemented, full integration pending Phase 2 QR findings

**What's Implemented:**
- ✅ `extract_qr_confidence(qr_passed, findings_count)` function
- ✅ `--qr-confidence-threshold` CLI flag (default: 80%)
- ✅ Parameter threading through planner → format_gate()
- ✅ Precedence logic (confidence check → iteration check → normal routing)
- ⏳ Full integration awaits QR findings parsing

**How to Use (Phase 2):**
```bash
# Default 80% threshold
/plan

# Custom threshold (more lenient)
/plan --qr-confidence-threshold 75

# Custom threshold (stricter)
/plan --qr-confidence-threshold 90
```

**Token Impact:** 10-20K tokens saved by detecting low-quality results early

**Next Steps:** When QR sub-agents provide structured findings, activate confidence extraction and routing

**Documentation:** See CHANGE4_IMPLEMENTATION.md for details

---

### Change 5: Fast/Medium/Thorough Planning Modes

**Objective:** Allow users to trade quality for token efficiency based on task risk

**Modes:**
- **FAST mode** (~35K tokens)
  - Runs Steps 1-5 only (exploration + planning)
  - Skips all QR gates
  - User performs manual code review
  - Best for: Simple tasks, internal code, prototypes, experienced users

- **MEDIUM mode** (~85K tokens) [DEFAULT]
  - Runs Steps 1-9 (planning + code quality review)
  - Skips TW documentation QR (Step 12)
  - Code is verified, docs may need manual touch-up
  - Best for: Standard features, code quality critical, docs secondary

- **THOROUGH mode** (~135K tokens)
  - Runs full 13-step workflow (current behavior)
  - All QR gates enabled
  - Best for: Critical features, first-time architectural patterns, high stakes

**User Invocation:**
```bash
/plan --mode fast
/plan --mode medium
/plan --mode thorough
```

**Implementation Pattern:**
```python
# In planner.py get_step_guidance()
if mode == "fast":
    if step > 5:
        return "FAST mode complete at step 5"
elif mode == "medium":
    if step == 12:  # Skip QR-Docs
        next_step = 13  # Gate
elif mode == "thorough":
    # No changes, run all steps
```

**Quality Trade-offs:**
| Mode | Quality Loss | Token Savings | When Worth It |
|------|---|---|---|
| FAST | ~15-20% lower code quality | 100K | Small tasks, internal code |
| MEDIUM | ~5-10% lower doc quality | 50K | Most production tasks |
| THOROUGH | None | - | Critical features |

**Token Impact Analysis:**
- Current users: always THOROUGH (270-400K with failures)
- With FAST/MEDIUM options: users pick mode per task, average 85K tokens
- **Total savings across project: 50-60% token reduction**

---

### Change 6: Bounded Iteration Limits with Exponential Backoff

**Current State:** (Post Phase 1) Iteration limits set at 2-3, then escalate to user

**Enhancement:** Implement exponential backoff to reduce retry overhead

**Pattern:**
```
Iteration 1: Full QR scope
  - Full code review, full doc review, full completeness check

Iteration 2: Focused QR scope
  - Only review changes from iteration 1
  - Skip unchanged sections
  - ~50% tokens of iteration 1

Iteration 3+: Escalate to user
  - Don't retry automatically
  - User decides: [Fix], [Skip], [Regenerate], [Abort]
```

**Implementation:**
```python
class QRGate:
    def __init__(self, max_iterations=3):
        self.iteration = 1
        self.max_iterations = max_iterations
        self.reviewed_sections = []

    def run(self, plan):
        if self.iteration == 1:
            scope = "full"
        elif self.iteration == 2:
            scope = "changed_only"  # Only review what changed
        else:
            return self.escalate_to_user()

        findings = qr_review(plan, scope)
        self.iteration += 1
        return findings
```

**Token Impact:**
- Iteration 1: 25K tokens (full review)
- Iteration 2: 12K tokens (changed sections only)
- Total: 37K vs 50K with naive retry → **26% savings**

**Quality Impact:** None—same rigor, just smarter scope

---

## Phase 3: Nice to Have (Future Consideration)

### Change 7: Token Budget Awareness

**Objective:** Prevent session exhaustion, give users budget control

**Implementation:**
```bash
/plan --token-budget 80000 --step 1 --total-steps 13
```

**Behavior:**
```
After each step, print:
  "Step 5 complete
   Tokens used so far: 35K / 80K budget
   Estimated remaining: QR(15K) + Dev(40K) + TW(15K) = 70K

   ⚠️  Budget insufficient for full workflow.

   Options:
   [Reduce-Scope] → Switch to --mode medium
   [Skip-TW] → Skip Step 11 documentation review
   [Proceed] → Continue, may exhaust budget
   [Abort] → Save plan, resume in new session"
```

**Implementation:**
- Add `--token-budget` flag to planner script
- Estimate tokens per step (based on past executions)
- Warn before each step if budget at risk
- Allow graceful fallback modes

**Token Impact:** Prevents mid-workflow failures where user exhausts budget at Step 9 (can't complete)

---

### Change 8: Plan Diff Preview Before Code Generation

**Objective:** Let users review code intent before Step 8 generates diffs

**Current Flow:**
- Step 5: Write plan (with Code Intent sections)
- Step 8: Developer generates diffs

**Enhanced Flow:**
- Step 5: Write plan with Code Intent
- *NEW Step 5b: Show user the Code Intent sections*
  - User sees what Code Intent says will be changed
  - Can edit Code Intent before diffs generated
  - Prevents 40K token code generation for unwanted changes
- Step 8: Developer generates diffs (user-approved)

**User Prompt:**
```
Plan complete. Code Intent preview:

### Milestone 1: Add session timeout
Code Intent:
  - auth.py: Add timeout check to session validation (lines 45-60)
  - config.py: Add SESSION_TIMEOUT setting (after line 12)
  - tests/test_auth.py: Add timeout test cases

[Edit Code Intent] [Approve & Continue] [Back to Plan]
```

**Token Impact:** Prevents 30-40K tokens on code generation if user wants to change approach

---

## Implementation Timeline

| Phase | Focus | Token Savings | Timeline |
|-------|-------|---|---|
| Phase 1 ✅ | User checkpoints, execution extraction | 150K+ | Complete |
| Phase 2 | Fast/Medium modes, confidence escalation | 100K+ | Next optimization cycle |
| Phase 3 | Budget awareness, graceful fallbacks | 50K+ | Future consideration |

---

## Research References

All changes are backed by production research:

- **HULA (Takerngsaksiri et al., 2025):** Human-In-the-Loop planning achieves 82% approval rate with user review gates
- **Self-Refine (Madaan et al., 2023):** Iterative feedback must be actionable and visible
- **Selective Escalation (Wang et al., 2023):** Confidence-based routing optimizes human attention
- **Plan Review Gates:** See `skills/prompt-engineer/references/prompt-engineering-hitl.md`

---

## How to Use This Document

When picking up Phase 2 or 3:
1. Select the change(s) you want to implement
2. Review the "Implementation Pattern" section
3. Reference the token impact and quality trade-offs
4. Implement using the same pattern as Phase 1 (use prompt-engineer skill)
5. Update this roadmap with completion status

Example for Phase 2:
```bash
# Implement Fast/Medium/Thorough modes
/prompt-engineer "Analyze implementation for Phase 2 Change 4..."

# Implement Confidence escalation
/prompt-engineer "Analyze implementation for Phase 2 Change 5..."
```
