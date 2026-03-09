# GitHub Copilot Compatibility Guide

This guide answers two questions:
1. **Does every feature in this repo work with GitHub Copilot instead of Claude Code?**
2. **What is the best practice for keeping this fork updated from the origin repo?**

---

## Part 1: Feature Compatibility

### TL;DR

| Category | Claude Code | GitHub Copilot | Workaround |
|----------|------------|----------------|------------|
| Project instructions | `CLAUDE.md` | `.github/copilot-instructions.md` ✅ | Already created |
| Always-on rules | `.claude/rules/*.md` (no `paths:`) | Inline in copilot-instructions.md ✅ | Already created |
| Path-scoped rules | `.claude/rules/*.md` (with `paths:`) | Not supported — paste into chat when relevant | Manual |
| Skills (slash cmds) | `.claude/skills/*/SKILL.md` | Not supported — use prompt templates | Prompt templates in copilot-instructions.md |
| Sub-agents | `.claude/agents/*.md` | Not supported — use personas in prompts | Prompt templates in this guide |
| Hooks | `.claude/hooks/` + `settings.json` | **Not supported** | Described below |
| Permission model | `settings.json` `allow/deny` | Not supported | N/A |
| `$CLAUDE_PROJECT_DIR` env var | Built-in | Not available | Use absolute paths |

### What Works Without Changes

- **Project instructions** — `.github/copilot-instructions.md` is read automatically by GitHub Copilot Chat in VS Code (and similar environments). This file condenses `CLAUDE.md` and the always-on rules into Copilot's format.
- **Folder structure, commands, quality gates** — Copilot will follow instructions in `copilot-instructions.md` just as Claude follows `CLAUDE.md`.
- **All content files** — Beamer `.tex`, Quarto `.qmd`, R scripts, bibliography — work identically.
- **Scripts** — `scripts/sync_to_docs.sh`, `scripts/quality_score.py`, and all shell/R scripts run independently of the AI tool.

### What Does NOT Work

#### 1. Hooks (`.claude/hooks/`)

Claude Code's hook system fires scripts at specific lifecycle events:

| Hook | What It Does | Copilot Alternative |
|------|-------------|---------------------|
| `notify.sh` | Desktop notification when Claude needs attention | No equivalent |
| `protect-files.sh` | Blocks edits to protected files (e.g., `Bibliography_base.bib`) | State in copilot-instructions.md: "Do NOT modify these files" |
| `pre-compact.py` | Saves context state before auto-compression | No equivalent — save important context manually |
| `post-compact-restore.py` | Restores context after compaction | No equivalent |
| `context-monitor.py` | Warns when context window is filling up | No equivalent |
| `verify-reminder.py` | Reminds to compile/render after edits | No equivalent |
| `log-reminder.py` | Blocks Claude from stopping without updating session log | No equivalent |

**Impact:** The automatic file protection, context survival, and session log enforcement are lost. These behaviors can be partially compensated by including the rules explicitly in your prompts and being disciplined about:
- Never asking Copilot to edit `Bibliography_base.bib` or `settings.json`
- Manually saving session summaries to `quality_reports/session_logs/`
- Manually updating `MEMORY.md` with `[LEARN]` tags

#### 2. Skills / Slash Commands (`.claude/skills/`)

Claude Code supports slash commands like `/compile-latex`, `/deploy`, `/proofread`. These are not available in GitHub Copilot. Use the prompt templates in `copilot-instructions.md` or the expanded versions below.

#### 3. Sub-agents (`.claude/agents/`)

Claude Code can spawn specialized sub-agents (proofreader, slide-auditor, etc.) running in parallel. GitHub Copilot Chat does not support spawning multiple agents. Instead, ask Copilot to **adopt a persona**:

> "Act as a strict proofreader. Read `[file]` and check for: grammar, typos, overflow, notation consistency. List every issue with line number. Do NOT apply changes yet."

#### 4. `settings.json` Permission Model

Claude Code's `permissions.allow` list restricts which bash commands can run without explicit approval. GitHub Copilot has no equivalent — it relies on the user to approve terminal commands in the VS Code integrated terminal.

---

## Part 2: Using Skills as Prompt Templates

The `.claude/skills/` files contain detailed instructions for specific workflows. You can paste or paraphrase these as prompts in Copilot Chat.

### Compile LaTeX (`/compile-latex`)

```
Compile [filename].tex with 3-pass XeLaTeX:
  cd Slides
  TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode [filename].tex
  BIBINPUTS=..:$BIBINPUTS bibtex [filename]
  TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode [filename].tex
  TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode [filename].tex
Report any errors or overfull hbox warnings.
```

### Proofread (`/proofread`)

```
You are a strict academic proofreader. Read [file] and check for:
1. Grammar (subject-verb agreement, articles, prepositions)
2. Typos (misspellings, duplicated words, search-replace corruption)
3. Overflow (overfull hbox in LaTeX, content exceeding slide boundaries in Quarto)
4. Consistency (notation, citation style, terminology)
5. Academic quality (informal abbreviations, missing words, awkward phrasing)

Produce a numbered list of findings with location (line number or slide title) and proposed fix.
Do NOT apply any changes yet.
```

### Visual Audit (`/visual-audit`)

```
You are a visual slide auditor. Read [file] and check for:
1. Text overflow / overfull hbox
2. Font size inconsistencies (too many reductions)
3. Spacing issues (crowded slides, inconsistent margins)
4. TikZ label overlaps
5. Image sizing problems

List findings with severity (Critical/Major/Minor) and location.
```

### Pedagogy Review (`/pedagogy-review`)

```
You are a pedagogy expert reviewing a lecture slide deck. Read [file] and evaluate:
1. Narrative arc — does the lecture build logically?
2. Prerequisite clarity — are student prerequisites clearly stated?
3. Notation density — is notation introduced gradually?
4. Worked examples — are concepts illustrated concretely?
5. Cognitive load — are slides too dense?
6. Pacing — is there variety in slide types?
7. Notation consistency — are symbols used consistently throughout?

Score each dimension 1-5 and provide an overall score and top 3 recommendations.
```

### Translate to Quarto (`/translate-to-quarto`)

```
Translate [Beamer .tex file] to Quarto RevealJS (.qmd). Rules:
- Beamer .tex is the source of truth; .qmd is derived
- Every frame becomes a ## slide
- TikZ diagrams: reference pre-compiled SVGs from Figures/LectureN/tikz_N.svg (0-indexed)
- Use CSS classes for Beamer environment equivalents (see .claude/agents/beamer-translator.md for mapping)
- Preserve all mathematical notation exactly
- Every \cite{key} becomes [@key]
Save output to Quarto/[LectureN_Topic].qmd
```

### Validate Bibliography (`/validate-bib`)

```
1. Find every \cite{...}, \citet{...}, \citep{...} key used in Slides/ and Quarto/
2. Find every @key entry in Bibliography_base.bib
3. Report: (a) cite keys used in slides but missing from .bib, (b) .bib entries never cited
```

### Adversarial QA (`/qa-quarto`)

```
Round 1 — Critic: Compare [Quarto .qmd] against [Beamer .tex] and find every discrepancy:
missing content, wrong math, layout differences, broken citations, missing TikZ diagrams.
Score harshly. If score >= 90, output "APPROVED". Otherwise list all findings.

Round 2 — Fixer: Implement every finding from the critic. Apply fixes to [Quarto .qmd].

Repeat until critic outputs "APPROVED" (max 5 rounds).
```

### Commit & PR (`/commit`)

```
1. git status && git diff --stat
2. git add -A
3. git commit -m "[descriptive message]"
4. gh pr create --title "[title]" --body "[summary of changes]" --base main
5. gh pr merge --merge --auto
```

### Data Analysis (`/data-analysis`)

```
Run end-to-end R analysis for: [describe goal].
Follow the R code conventions in .claude/rules/r-code-conventions.md:
- set.seed() once at top (YYYYMMDD format)
- All packages loaded via library() at top
- All paths relative to repository root
- snake_case naming, verb-noun pattern
Save scripts to scripts/R/, outputs to output/.
Run the script and verify it executes without errors.
```

---

## Part 3: Best Practice for Keeping Upstream Updated

### The Problem

You want to:
- Pull new features, skills, and rules from the origin repo (`psantanna/claude-code-my-workflow`)
- Keep your own GitHub Copilot settings (`.github/copilot-instructions.md`) without conflicts
- Possibly keep your own project customizations (filled-in `CLAUDE.md`, custom rules)

### The Solution: Two Zones

```
Upstream touches:            You customize:
.claude/                     .github/          ← No conflicts possible
CLAUDE.md (template)         Your filled-in CLAUDE.md (or keep it as template)
README.md                    .github/COPILOT_GUIDE.md
templates/                   quality_reports/ (your logs, plans)
scripts/                     Slides/, Quarto/, Figures/ (your content)
```

**Key insight:** The upstream repo never writes to `.github/`. Your `copilot-instructions.md` lives there and is never touched by upstream merges.

### Setup

```bash
# Add the upstream remote once
git remote add upstream https://github.com/psantanna/claude-code-my-workflow.git
git remote -v  # Verify: origin = your fork, upstream = origin repo
```

### Pulling Upstream Updates

```bash
# Fetch latest from upstream
git fetch upstream

# See what's new
git log upstream/main --oneline -20

# Merge upstream into your branch
git merge upstream/main

# Resolve any conflicts (unlikely if you follow the two-zone rule above)
# Your .github/ files are never touched by upstream
git push origin main
```

### What to Customize (and Where)

| What | Where | Upstream risk |
|------|-------|---------------|
| Copilot project instructions | `.github/copilot-instructions.md` | ✅ Never touched |
| This guide | `.github/COPILOT_GUIDE.md` | ✅ Never touched |
| Lecture mapping | `.claude/rules/beamer-quarto-sync.md` (your fork only) | ⚠️ Merge conflict possible |
| Knowledge base | `.claude/rules/knowledge-base-template.md` (fill in your content) | ⚠️ Merge conflict possible |
| Domain reviewer | `.claude/agents/domain-reviewer.md` (fill in your field) | ⚠️ Merge conflict possible |
| Workflow quick ref | `.claude/WORKFLOW_QUICK_REF.md` | ⚠️ Merge conflict possible |
| `CLAUDE.md` (project name, etc.) | `CLAUDE.md` | ⚠️ Merge conflict possible |

**Recommendation for files in the ⚠️ column:** After filling them in, copy your changes to a backup file (e.g., `.github/my-rules-backup/`) so you can reapply them after an upstream merge.

### Merge Conflict Resolution

If upstream updates a file you've customized:

```bash
# After git merge upstream/main produces a conflict:
git diff HEAD..upstream/main -- .claude/rules/knowledge-base-template.md

# Option A: Keep yours (discard upstream change)
git checkout HEAD -- .claude/rules/knowledge-base-template.md

# Option B: Keep upstream (discard your customization)
git checkout upstream/main -- .claude/rules/knowledge-base-template.md

# Option C: Manually merge in your editor, then:
git add .claude/rules/knowledge-base-template.md
git merge --continue
```

### Recommended Workflow

1. **Fork the repo** and clone locally
2. **Fill in your customizations** in `.github/copilot-instructions.md` (project name, institution, lecture mapping, etc.)
3. **Add the upstream remote**: `git remote add upstream https://github.com/psantanna/claude-code-my-workflow.git`
4. **Monthly:** `git fetch upstream && git merge upstream/main` to get new skills, rules, and features
5. **If conflicts:** Use the guidance above to resolve them

---

## Part 4: Summary

### Quick Answers

**Q: Does every feature work with GitHub Copilot?**

No. The following are Claude Code-specific and have no equivalent in GitHub Copilot:
- Hooks (automatic file protection, context monitoring, session log enforcement, desktop notifications)
- Slash commands / Skills (`/compile-latex`, `/deploy`, `/proofread`, etc.)
- Sub-agents (parallel specialist review)
- The `settings.json` permission model

**Q: What needs to change?**

- ✅ **Already done:** Created `.github/copilot-instructions.md` with project context, core rules, and prompt templates
- ✅ **Already done:** Created this guide with full prompt templates for every skill
- 📋 **You do:** Fill in `[YOUR PROJECT NAME]`, `[YOUR INSTITUTION]`, and the lecture table in `.github/copilot-instructions.md`
- 📋 **You do:** Add upstream remote (`git remote add upstream ...`)

**Q: How do I keep upstream features while keeping my Copilot settings?**

- Keep all your customizations in `.github/` — upstream never writes there
- Regularly run `git fetch upstream && git merge upstream/main` to get new features
- For files both you and upstream touch (e.g., `CLAUDE.md`, `.claude/rules/knowledge-base-template.md`), resolve conflicts manually keeping your content

---

*See also: [GitHub Copilot custom instructions documentation](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)*
