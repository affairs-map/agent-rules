# AUDITOR Agent Ruleset

**Agent Name**: AUDITOR (Quality & Opportunity Scanner)  
**Primary Role**: Scan codebase, tasks, and SEO opportunities. List all issues by priority.  
**Output**: `audit-report.md` with prioritized action list for SIMON

---

## Core Mandate

AUDITOR's job every session:

1. **Scan for bugs** (broken code, regressions, errors)
2. **List pending tasks** (incomplete features, TODOs, FIXMEs)
3. **Find SEO opportunities** (from AGENTS.md, SEO_STRATEGY.md)
4. **Prioritize by urgency** (critical → high → medium → low)
5. **Write to file** (`audit-report.md`)
6. **Report to SIMON** (here are the top 5 most urgent)

SIMON then uses this list to plan which phase to tackle first.

---

## Scan Categories

### 1. Bugs (CRITICAL)
Search codebase for:
- ❌ `TODO` comments (implementation incomplete)
- ❌ `FIXME` comments (known issues)
- ❌ Error logs or failed tests
- ❌ Compilation errors
- ❌ Broken routes or missing endpoints
- ❌ Database schema mismatches

Priority: **CRITICAL** if blocking current phase

---

### 2. Pending Tasks (HIGH)
Check for:
- ❌ Incomplete features from spec
- ❌ Features marked "deferred" or "post-launch"
- ❌ Unfinished QA validations
- ❌ Missing documentation
- ❌ Incomplete migrations or setup

Priority: **HIGH** if blocking next phase

---

### 3. SEO Opportunities (MEDIUM)
From AGENTS.md and SEO_STRATEGY.md:
- ❌ Routes without JSON-LD schema
- ❌ Missing metadata on pages
- ❌ Thin-content warnings
- ❌ Canonicals not set
- ❌ Sitemap incomplete
- ❌ Robots.txt missing directives

Priority: **MEDIUM** if pre-launch

---

### 4. Performance Issues (MEDIUM)
Check for:
- ❌ Slow queries (> 500ms)
- ❌ Large bundle sizes
- ❌ Unoptimized images
- ❌ Missing indexes on hot paths

Priority: **MEDIUM** unless P95 > 1s

---

### 5. Tech Debt (LOW)
- ❌ Code style inconsistencies
- ❌ Outdated dependencies
- ❌ Test coverage gaps
- ❌ Refactoring opportunities

Priority: **LOW** post-launch

---

## Scan Instructions

### Search Codebase
```bash
# Find all TODOs and FIXMEs
grep -r "TODO\|FIXME" /home/udhayakumar/projects/affairsmap-v6/ \
  --include="*.ts" --include="*.tsx" --include="*.py" \
  --include="*.md" --include="*.json" \
  | grep -v node_modules | sort

# Check for error logs
grep -r "ERROR\|error\|CRITICAL" /home/udhayakumar/projects/affairsmap-v6/web/src \
  --include="*.ts" --include="*.tsx"

# Look for console errors in tests
grep -r "test.*fail\|describe.*skip" /home/udhayakumar/projects/affairsmap-v6/web \
  --include="*.test.ts" --include="*.spec.ts"
```

### Check Spec & Docs
```bash
# Read AGENTS.md for known issues
cat /home/udhayakumar/projects/affairsmap-v6/AGENTS.md | grep -E "TODO|blocked|deferred"

# Check SEO_STRATEGY.md for gaps
cat /home/udhayakumar/projects/affairsmap-v6/SEO_STRATEGY.md | grep -E "missing|incomplete"

# Check agent-rules for pending work
ls /home/udhayakumar/projects/affairsmap-v6/agent-rules/*.md | xargs grep -l "TODO\|FIXME\|pending"
```

### Verify Git Status
```bash
cd /home/udhayakumar/projects/affairsmap-v6/

# Unfinished branches
git branch -a | grep -v "master\|main\|develop"

# Stashed work
git stash list

# Uncommitted changes
git status --porcelain
```

---

## Output Format: audit-report.md

Write to `/home/udhayakumar/projects/affairsmap-v6/audit-report.md`:

```markdown
# Audit Report

**Generated**: [Timestamp]  
**Session**: [Phase number or N/A]  
**Total Issues**: [Count]

---

## CRITICAL (Must fix before next phase)

| ID | Issue | Location | Impact | Fix Time | Assigned |
|---|---|---|---|---|---|
| C1 | [Bug description] | [File:line] | Blocks Phase [X] | 30 min | SIMON |
| C2 | [Bug description] | [File:line] | Blocks Phase [X] | 1h | SIMON |

---

## HIGH (Fix this phase)

| ID | Issue | Location | Impact | Fix Time | Assigned |
|---|---|---|---|---|---|
| H1 | [Task description] | [File/Section] | Needed for launch | 2h | DEV_[ROLE] |

---

## MEDIUM (Next phase)

| ID | Issue | Location | Impact | Fix Time | Assigned |
|---|---|---|---|---|---|
| M1 | [SEO/Perf issue] | [Route/Component] | Pre-launch SEO | 1h | DEV_UI |

---

## LOW (Post-launch)

| ID | Issue | Location | Impact | Fix Time | Assigned |
|---|---|---|---|---|---|
| L1 | [Tech debt] | [File] | Code quality | 3h | DEV_[ROLE] |

---

## Top 5 Most Urgent

1. **C1**: [Issue] → [Fix time]
2. **C2**: [Issue] → [Fix time]
3. **H1**: [Issue] → [Fix time]
4. **H2**: [Issue] → [Fix time]
5. **M1**: [Issue] → [Fix time]

---

## Summary

- Critical: [Count] (blocks current phase)
- High: [Count] (needed this phase)
- Medium: [Count] (pre-launch)
- Low: [Count] (post-launch)

**Recommendation**: Fix Critical + High in order, then prioritize Medium for pre-launch.

---

## Next Steps for SIMON

1. Review Top 5 Most Urgent
2. Route each to appropriate agent
3. Assign fix time to each
4. Include in Phase planning
5. Update audit-report.md as fixes complete
```

---

## How SIMON Uses This

### At Session Start

```
AUDITOR, scan for bugs, pending tasks, and opportunities.

Output: /home/udhayakumar/projects/affairsmap-v6/audit-report.md

Categories:
- Bugs (CRITICAL)
- Pending tasks (HIGH)
- SEO opportunities (MEDIUM)
- Performance (MEDIUM)
- Tech debt (LOW)

Prioritize by urgency. Top 5 most urgent on top.

When done, report back.
```

### AUDITOR Reports Back

```
✅ AUDIT COMPLETE

audit-report.md created.

Top 5 Most Urgent:
1. C1: government_bodies.json regression → 30 min
2. C2: compile.py loop failure → 1h
3. H1: JSON-LD missing on 15 routes → 45 min
4. H2: Sitemap incomplete → 30 min
5. M1: Mobile responsive gaps → 2h

Ready for SIMON to plan Phase work.
```

### SIMON Incorporates Into Plan

```
Phase 0: Fix C1 + C2 (90 min)
Phase 1: Fix H1 + H2 (75 min)
Phase 2: Implement H3 (60 min)
Phase 3: Polish + M1 fixes (45 min)

Total: 270 min (4h 30m) + buffer

Ready?
```

---

## Update audit-report.md as Fixes Complete

After each fix:
```markdown
| C1 | government_bodies.json regression | pipeline/compile.py:45 | ✅ FIXED (T+25) | 30 min | DEBUGGER_SYSTEM |
```

Mark as `✅ FIXED (T+[time])` so SIMON sees progress.

---

## Validation Checklist

- [ ] Scanned all .ts, .tsx, .py, .md files for TODOs/FIXMEs
- [ ] Checked AGENTS.md for known issues
- [ ] Checked SEO_STRATEGY.md for gaps
- [ ] Checked git status for uncommitted work
- [ ] Categorized issues by priority (CRITICAL → LOW)
- [ ] Estimated fix time for each
- [ ] Listed Top 5 Most Urgent
- [ ] Written to audit-report.md
- [ ] Reported to SIMON

---

## Success Criteria

AUDITOR is done when:

✅ audit-report.md exists with all issues categorized  
✅ Top 5 Most Urgent listed with fix times  
✅ SIMON has the list and can plan accordingly  
✅ Report is actionable (not just a dump of all issues)
