# AGENTS.md -- Academic Project Development with OpenAI Codex

<!-- HOW TO USE: Replace [BRACKETED PLACEHOLDERS] with your project info.
     Customize Beamer environments and CSS classes for your theme.
     Keep this file under ~200 lines — Codex loads it every session.
     See the guide at docs/workflow-guide.html for full documentation. -->

**Project:** [YOUR PROJECT NAME]
**Institution:** [YOUR INSTITUTION]
**Branch:** main

---

## Core Principles

- **Plan first** -- create a plan before non-trivial tasks; save plans to `quality_reports/plans/`
- **Verify after** -- compile/render and confirm output at the end of every task
- **Single source of truth** -- Beamer `.tex` is authoritative; Quarto `.qmd` derives from it
- **Quality gates** -- nothing ships below 80/100
- **[LEARN] tags** -- when corrected, save `[LEARN:category] wrong → right` to MEMORY.md

---

## Folder Structure

```
[YOUR-PROJECT]/
├── AGENTS.md                    # This file (Codex instructions)
├── CLAUDE.md                    # Claude Code instructions (if also using Claude Code)
├── Bibliography_base.bib        # Centralized bibliography
├── Figures/                     # Figures and images
├── Preambles/header.tex         # LaTeX headers
├── Slides/                      # Beamer .tex files
├── Quarto/                      # RevealJS .qmd files + theme
├── docs/                        # GitHub Pages (auto-generated)
├── scripts/                     # Utility scripts + R code
├── quality_reports/             # Plans, session logs, merge reports
├── explorations/                # Research sandbox
├── templates/                   # Session log, quality report templates
└── master_supporting_docs/      # Papers and existing slides
```

---

## Commands

```bash
# LaTeX (3-pass, XeLaTeX only)
cd Slides && TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode file.tex
BIBINPUTS=..:$BIBINPUTS bibtex file
TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode file.tex
TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode file.tex

# Deploy Quarto to GitHub Pages
./scripts/sync_to_docs.sh LectureN

# Quality score
python scripts/quality_score.py Quarto/file.qmd
```

---

## Quality Thresholds

| Score | Gate | Meaning |
|-------|------|---------|
| 80 | Commit | Good enough to save |
| 90 | PR | Ready for deployment |
| 95 | Excellence | Aspirational |

---

## Workflow: Plan-First

**For any non-trivial task, draft a plan before writing code.**

1. **Check MEMORY.md** — read any `[LEARN]` entries relevant to this task
2. **Requirements Specification (for complex/ambiguous tasks)**:
   - Ask 3–5 clarifying questions when the task is high-level or vague
   - Create `quality_reports/specs/YYYY-MM-DD_description.md` with MUST/SHOULD/MAY requirements
   - Declare clarity status per requirement: CLEAR / ASSUMED / BLOCKED
3. **Draft the plan** — what changes, which files, in what order
4. **Save to disk** — write to `quality_reports/plans/YYYY-MM-DD_short-description.md`
5. **Present to user** — wait for approval
6. **Implement via the orchestrator loop** (see below)

---

## Orchestrator Protocol: Contractor Mode

**After a plan is approved, work autonomously through this loop:**

```
Plan approved
  │
  Step 1: IMPLEMENT — Execute plan steps
  │
  Step 2: VERIFY — Compile, render, check outputs
  │         If verification fails → fix → re-verify (max 2 retries)
  │
  Step 3: REVIEW — Check for issues by file type (see rubrics below)
  │
  Step 4: FIX — Apply fixes (critical → major → minor)
  │
  Step 5: RE-VERIFY — Confirm fixes are clean
  │
  Step 6: SCORE — Apply quality-gates rubric
  │
  └── Score >= threshold?
        YES → Present summary to user
        NO  → Loop back to Step 3 (max 5 rounds)
```

**"Just do it" mode:** When user says "just do it" / "handle it", skip final approval, auto-commit if score >= 80, still run the full loop.

---

## Quality Gates & Scoring Rubrics

### Quarto Slides (.qmd)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Compilation failure | -100 |
| Critical | Equation overflow | -20 |
| Critical | Broken citation | -15 |
| Major | Text overflow | -5 |
| Major | Notation inconsistency | -3 |
| Minor | Long lines (>100 chars) | -1 (EXCEPT documented math formulas) |

### R Scripts (.R)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Syntax errors | -100 |
| Critical | Hardcoded absolute paths | -20 |
| Major | Missing set.seed() | -10 |
| Major | Missing figure generation | -5 |

### Beamer Slides (.tex)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | XeLaTeX compilation failure | -100 |
| Critical | Undefined citation | -15 |
| Critical | Overfull hbox > 10pt | -10 |

**Enforcement:** Score < 80 → block commit; score < 90 → allow commit with warning.

---

## Session Logging

Log to `quality_reports/session_logs/YYYY-MM-DD_description.md` at three points:
1. **Post-plan** — immediately after plan approval: goal, approach, rationale
2. **Incremental** — whenever a design decision is made or a problem is solved (1–3 lines)
3. **End-of-session** — summary, quality scores, open questions, blockers

Quality reports generated **only at merge time** → `quality_reports/merges/YYYY-MM-DD_[branch-name].md`.

---

## Verification Protocol

At the end of EVERY task, verify the output works:

- **Quarto/HTML:** Run `./scripts/sync_to_docs.sh LectureN`, open HTML, check images
- **LaTeX/Beamer:** Compile with xelatex, check for overfull hbox warnings
- **R Scripts:** Run `Rscript scripts/R/filename.R`, verify output files created

---

## Specialized Review Agents

Reference these patterns when reviewing files (full agent definitions in `.claude/agents/`):

| Agent | What It Does |
|-------|-------------|
| `proofreader` | Grammar, typos, overflow, consistency review |
| `slide-auditor` | Visual layout audit (overflow, font, spacing) |
| `pedagogy-reviewer` | 13-pattern pedagogical review |
| `r-reviewer` | R code quality and reproducibility |
| `quarto-critic` | Adversarial QA comparing Quarto against Beamer |
| `quarto-fixer` | Implements fixes from the critic |
| `verifier` | End-to-end task completion verification |

---

## Available Workflow Skills

Reference these workflows (full skill definitions in `.claude/skills/`):

| Skill | What It Does |
|-------|-------------|
| `compile-latex` | 3-pass XeLaTeX compilation with bibtex |
| `deploy` | Render Quarto + sync to GitHub Pages |
| `proofread` | Grammar/typo/overflow review |
| `visual-audit` | Slide layout audit |
| `pedagogy-review` | Teaching quality review |
| `review-r` | R code quality review |
| `qa-quarto` | Adversarial critic-fixer loop (max 5 rounds) |
| `slide-excellence` | Combined multi-agent review |
| `translate-to-quarto` | Full 11-phase Beamer-to-Quarto translation |
| `validate-bib` | Cross-reference citations against bibliography |
| `commit` | Stage, commit, create PR, and merge to main |
| `lit-review` | Literature search, synthesis, gap identification |
| `research-ideation` | Generate research questions and empirical strategies |
| `data-analysis` | End-to-end R analysis with publication-ready output |

---

<!-- CUSTOMIZE: Replace the example entries below with your own
     Beamer environments and Quarto CSS classes. -->

## Beamer Custom Environments

| Environment       | Effect        | Use Case       |
|-------------------|---------------|----------------|
| `[your-env]`      | [Description] | [When to use]  |

## Quarto CSS Classes

| Class              | Effect        | Use Case       |
|--------------------|---------------|----------------|
| `[.your-class]`    | [Description] | [When to use]  |

---

## Current Project State

| Lecture | Beamer | Quarto | Key Content |
|---------|--------|--------|-------------|
| 1: [Topic] | `Lecture01_Topic.tex` | `Lecture1_Topic.qmd` | [Brief description] |
| 2: [Topic] | `Lecture02_Topic.tex` | -- | [Brief description] |

---

## Context Survival

When approaching context limits, save to disk before resuming:
1. Update MEMORY.md with any `[LEARN]` entries from this session
2. Ensure session log is current
3. Save active plan to `quality_reports/plans/`

After a new session starts:
1. Read `AGENTS.md` + most recent plan in `quality_reports/plans/`
2. Check `git log --oneline -10` and `git diff`
3. State what you understand the current task to be
