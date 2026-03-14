# Debug Investigation Template

**File location**: `/docs/debug/[YYYY-MM-DD]_[bug-name]_investigation.md`

---

## Hypothesis Tracking Matrix

**Purpose**: Track all hypotheses and systematically eliminate possibilities. Update after each test to narrow down root cause.

| ID | Hypothesis | Probability | Status | Evidence | Time |
|----|-----------|-------------|--------|----------|------|
| H1 | [Description] | 🔴 High | ⏳ Pending | - | - |
| H2 | [Description] | 🔴 High | ⏳ Pending | - | - |
| H3 | [Description] | 🟡 Medium | ⏳ Pending | - | - |
| H4 | [Description] | 🟡 Medium | ⏳ Pending | - | - |
| H5 | [Description] | 🟠 Low | ⏳ Pending | - | - |

**Status Legend**:
- ⏳ Pending - Not yet tested
- 🔍 Testing - Currently investigating
- ✅ Ruled Out - Confirmed NOT the cause
- ❌ ROOT CAUSE - Confirmed as the problem

**How to use**:
1. List ALL possible causes before testing (aim for 10+)
2. Test high probability (🔴) first
3. After each test, update Status and Evidence
4. Narrow scope by ruling out possibilities
5. Continue until root cause found

---

## Debug Investigator Report: [Bug Name]

### 0. Context
- **Observed Bug**: [Specific description - what exactly happens]
- **Environment**: [OS / framework / version / branch]
- **Reproduction Steps**: 
  1. [Step 1]
  2. [Step 2]
  3. [Observed behavior]

---

### 1. Hypothesis & Analysis

#### 1.1 Possible Causes (WHAT)
List ALL possible causes (aim for 10+):

- 🔴 **H1**: [Most likely cause based on symptoms]
  - Why: [reasoning]
  - How to test: [specific test method]
  
- 🔴 **H2**: [Another high-probability cause]
  - Why: [reasoning]
  - How to test: [specific test method]

- 🟡 **H3**: [Medium probability cause]
  - Why: [reasoning]
  - How to test: [specific test method]

- 🟡 **H4**: [Another medium probability]
  - Why: [reasoning]
  - How to test: [specific test method]

- 🟠 **H5**: [Low probability but possible]
  - Why: [reasoning]
  - How to test: [specific test method]

[Continue listing all possibilities...]

#### 1.2 Reasoning (WHY)
- H1 → [underlying mechanism or failure point]
- H2 → [underlying mechanism or failure point]
- H3 → [underlying mechanism or failure point]

#### 1.3 Information Gathering

**Self-gathered**:
- ✅ [What I checked - e.g., git log, error logs]
- ✅ [What I verified - e.g., config files]
- ❌ [What I couldn't access - e.g., production logs]

**Information gaps (need from user)**:
- [ ] "I need to see [X]. Can you [specific action] and share the output?"
- [ ] "I need to verify [Y]. Can you check [specific location]?"

#### 1.4 Research Notes
- [Source code inspection findings]
- [Third-party library documentation]
- [Similar issues found online]

---

### 2. Decision
- **Testing Order**: H1 (🔴) → H2 (🔴) → H3 (🟡) → ...
- **Testing First**: H1 - [name]
- **Rationale**: [why this is most likely based on evidence]

---

### 3. Fix Implementation (Iteration 1)

**Testing Hypothesis**: H1 - [name]

**Change Applied**: 
- File: `path/to/file.ext`
- Change: [what was modified]

**Rationale**: [why this should address H1]

**Verification Method**: [how to confirm]

---

### 4. Verification (Iteration 1)

#### 4.1 Test Result
- Status: ✅ CONFIRMED / ✅ RULED OUT / ⏳ INCONCLUSIVE
- Evidence: [what happened when tested]
- Time spent: [X minutes]

#### 4.2 Findings
[What was learned from this test]

#### 4.3 Next Step
- [If confirmed: proceed to comprehensive testing]
- [If ruled out: test H2 next]
- [If inconclusive: gather more information]

---

### 5. Loop

**Iteration Log**:

| Iteration | Hypothesis | Status | Time | Finding |
|-----------|-----------|--------|------|---------|
| 1 | H1 | ✅ Ruled Out | 5 min | [finding] |
| 2 | H2 | ✅ Ruled Out | 10 min | [finding] |
| 3 | H3 | ❌ CONFIRMED | 15 min | [root cause found] |

**Total iterations**: 3
**Total time**: 30 minutes

---

### 6. Resolution Summary

**Final Fix**: 
- File: `path/to/file.ext`
- Change: [what was changed]

**Root Cause**: 
[Detailed explanation of what actually caused the bug]

**Why It Wasn't Caught Earlier**: 
[Gap in testing, edge case, recent change, etc.]

**Prevention**:
- Immediate: [what to do now]
- Long-term: [tests to add, monitoring to improve]

**Lessons Learned**:
1. [Key insight 1]
2. [Key insight 2]
3. [What would speed up debugging next time]

---

## Status: ✅ RESOLVED

**Date resolved**: [YYYY-MM-DD]
**Archive location**: `/docs/debug/resolved/[YYYY-MM-DD]_[bug-name]_investigation.md`
