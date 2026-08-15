# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-source LaTeX résumé for Nazib Abrar, plus its compiled PDF. There is no application code, no tests, and no dependency manifest. The only editable source file is [latex/main.tex](latex/main.tex); everything in [pdf/](pdf/) is build output that is intentionally committed.

## Build

Compiled with `pdflatex` (TeX Live 2023) from the repo root, writing artifacts into `pdf/`:

```bash
pdflatex -interaction=nonstopmode -output-directory=pdf latex/main.tex
```

This produces `pdf/main.pdf`. The committed deliverable is `pdf/nazib_abrar_mte_ruet_resume.pdf`, so after a successful build:

```bash
mv pdf/main.pdf pdf/nazib_abrar_mte_ruet_resume.pdf
```

Single pass is enough — the document has no cross-references or bibliography needing a second run. Requires the `fontawesome5` package (Debian: `texlive-fonts-extra`).

`pdf/main.aux`, `pdf/main.log`, and `pdf/main.out` are tracked in git and change on every build; commit them along with the PDF, matching existing history. Do not commit a stray `pdf/main.pdf` — rename it first.

## Layout constraints

The résumé is deliberately **2 pages**. Page 2 is forced by a hardcoded `\newpage` in the middle of the Research & Projects section ([latex/main.tex:154](latex/main.tex#L154)), placed between the StereoLite and drone-navigation project blocks. Any content edit that changes vertical extent can push text past page 2 or leave page 1 short — always rebuild and confirm the log still reports `2 pages`, and move the `\newpage` if the break lands badly.

## Custom macro layer

The document defines its own commands near the top rather than using a résumé class. Two arity details matter because call sites look wrong but are load-bearing:

- `\resumeSubheading{#1}{#2}` takes **two** arguments (bold title, right-aligned date) and renders them in a `tabularx` row. Several call sites pass extra brace groups or trailing bare text — e.g. Education passes the degree/CGPA line unbraced after the two args, and the Awards entries append `{}{}` / `{}`. TeX consumes only the first two and typesets the rest as ordinary body text on the following line. That is how the subtitle lines appear. Don't "fix" the extra groups; either keep the pattern or change the macro and all call sites together.
- `\resumeProjectHeading{#1}{#2}` is the same shape but italicizes `#2`; project entries pass an empty second group.

Other pieces: `\resumeSubHeadingList` / `\resumeItemList` are `enumitem` list environments (the outer one is label-less with tight `nosep` spacing), `\resumeItem` is a `\small` bullet, and `\resumeSubheadingSpacer` inserts the 2pt gap between consecutive entries within one section. Spacing is tuned via negative `itemsep` and the `\titleformat` block — small changes there ripple across the whole page.

## Content conventions

- Sections in order: Education, Work Experience, Research & Projects, Leaderships and Awards, Problem-Solving Experience, Certificates. Each is delimited by a `%-----------NAME-----------` comment banner.
- Experience and project bullets are achievement-oriented and name concrete technologies; keep that register when adding entries.
- External links use `\href{...}{\underline{...}}`; the header uses `fontawesome5` glyphs (`\faPhone`, `\faEnvelope`, `\faGithub`).

## Other directories

`docx/` and `job_description/` exist but are empty — earlier `.docx` sources and per-application job descriptions were removed. `pdf/updated_resume_abrar.pdf` is a legacy artifact from before the LaTeX rewrite and is not produced by this build.
