# GitHub Copilot — Project Instructions

<!-- This file is the GitHub Copilot equivalent of CLAUDE.md.
     It is loaded automatically by GitHub Copilot Chat in VS Code.
     Keep this file concise (~200 lines) — Copilot loads it every session.
     For the full compatibility guide, see .github/COPILOT_GUIDE.md -->

**Project:** [YOUR PROJECT NAME]
**Institution:** [YOUR INSTITUTION]
**Branch:** main

---

## Role: Contractor Mode

You are an expert academic collaborator. For any non-trivial task:
1. **Plan first** — outline the approach and files to change before writing code
2. **Verify after** — compile/render and confirm output before declaring done
3. **Quality gates** — nothing ships below 80/100

Ask me to confirm the plan for multi-file or ambiguous tasks. For clear single-file fixes, just do it.

---

## Folder Structure

```
[YOUR-PROJECT]/
├── .github/                     # GitHub Copilot config (this file + guide)
├── .claude/                     # Claude Code config (rules, skills, agents, hooks)
├── CLAUDE.md                    # Claude Code project memory
├── Bibliography_base.bib        # Centralized bibliography
├── Figures/                     # Figures and images
├── Preambles/header.tex         # LaTeX headers
├── Slides/                      # Beamer .tex files (SOURCE OF TRUTH)
├── Quarto/                      # RevealJS .qmd files (derived from Beamer)
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
# LaTeX (3-pass XeLaTeX)
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

## Core Rules

### Source of Truth
- **Beamer `.tex` is authoritative.** Quarto `.qmd` is derived.
- Every edit to a Beamer `.tex` MUST be immediately synced to the corresponding Quarto `.qmd`.
- Never edit derived artifacts (`docs/`, SVGs from TikZ) independently.

### Quality Gates
| Score | Gate |
|-------|------|
| 80/100 | Commit |
| 90/100 | PR / deployment |
| 95/100 | Excellence |

**Beamer/Quarto deductions:** compilation failure (−100), equation overflow (−20), broken citation (−15), typo in equation (−10), text overflow (−5), notation inconsistency (−3).

**R script deductions:** missing `set.seed()` (−20), hardcoded absolute paths (−15), no reproducibility guarantee (−15).

### LaTeX/Beamer Rules
- Never use `\pause`, `\onslide`, `\only`, `\uncover`, or any overlay commands.
- Compile with XeLaTeX only (3-pass + bibtex).
- TikZ diagrams must be converted to SVG for Quarto (`pdf2svg`). Never use PNG for diagrams.

### R Code Rules
- `set.seed()` called ONCE at top (YYYYMMDD format).
- All packages loaded at top via `library()` (not `require()`).
- All paths relative to repository root.
- `snake_case` naming, verb-noun pattern for functions.
- Replicate original results BEFORE extending.

### PDF Processing (large files)
- Do NOT read large PDFs directly. Use `pdfinfo` to check page count, then split with `gs` into 5-page chunks before processing.

---

## Workflow Patterns

### Compile LaTeX
```bash
cd Slides
TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode $FILE.tex
BIBINPUTS=..:$BIBINPUTS bibtex $FILE
TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode $FILE.tex
TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode $FILE.tex
```

### Deploy Quarto
```bash
./scripts/sync_to_docs.sh LectureN   # single lecture
./scripts/sync_to_docs.sh            # all lectures
```

### Extract TikZ → SVG
1. Verify `Figures/LectureN/extract_tikz.tex` matches current Beamer source
2. Compile: `cd Figures/LectureN && xelatex extract_tikz.tex`
3. Convert: `pdf2svg extract_tikz.pdf tikz_%d.svg all`
4. Rename to 0-based index: `tikz_0.svg`, `tikz_1.svg`, ...
5. Verify SVG files contain valid XML

### Session Logging
Save progress to `quality_reports/session_logs/YYYY-MM-DD_description.md` using `templates/session-log.md`. Log after plan approval, after each major decision, and at session end.

### Exploration Work
Experimental code goes in `explorations/[name]/` first. Quality threshold: 60/100 (vs 80 for production). No formal plan needed — just a research-value check.

---

## Prompt Templates (replacing Claude Skills)

Use these prompts directly in Copilot Chat. See `.github/COPILOT_GUIDE.md` for full details.

| Task | Prompt |
|------|--------|
| Proofread | "Proofread [file] for grammar, typos, overflow, and notation consistency. Propose changes; don't apply yet." |
| Visual audit | "Audit [file] for visual layout issues: overflow, font consistency, spacing. List findings." |
| Pedagogy review | "Review [file] for pedagogical quality: narrative arc, notation density, cognitive load, worked examples." |
| Translate to Quarto | "Translate [Beamer .tex] to Quarto RevealJS (.qmd), following the single-source-of-truth rules in this project." |
| Commit & PR | "Stage all changes, commit with a descriptive message, create a PR, and merge to main." |
| Validate bib | "Find all \\cite{...} keys in Slides/ and Quarto/, then verify each exists in Bibliography_base.bib. List missing and unused entries." |
| Data analysis | "Run end-to-end R analysis: [describe goal]. Save scripts to scripts/R/, outputs to output/. Follow r-code-conventions in .claude/rules/." |

---

## Protected Files

Do NOT modify these files without explicit confirmation:
- `Bibliography_base.bib`
- `.claude/settings.json`
- `.github/copilot-instructions.md` (this file)

---

## Current Project State

<!-- Update this table as you create lectures -->
| Lecture | Beamer | Quarto | Key Content |
|---------|--------|--------|-------------|
| 1: [Topic] | `Slides/Lecture01_Topic.tex` | `Quarto/Lecture1_Topic.qmd` | [Brief description] |
| 2: [Topic] | `Slides/Lecture02_Topic.tex` | — | [Brief description] |

---

## Customization Notes

<!-- Fill these in for your project -->
- **Course knowledge base:** See `.claude/rules/knowledge-base-template.md` (Claude Code users) or add notation conventions directly below this section
- **Domain reviewer:** See `.claude/agents/domain-reviewer.md` (Claude Code users) or describe your field-specific review criteria here
- **Beamer theme:** [Your theme name]
- **Color palette:** [Your colors]
- **Non-negotiables:** [Your project-specific invariants]
