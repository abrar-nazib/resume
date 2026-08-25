# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Two LaTeX documents for Nazib Abrar plus their compiled PDFs. There is no application code, no tests, and no dependency manifest. The editable sources are [latex/main.tex](latex/main.tex) (the 2-page résumé) and [latex/supplement.tex](latex/supplement.tex) (a 4-page technical supplement sent alongside it). Everything in [pdf/](pdf/) is build output that is intentionally committed, and [assets/](assets/) holds the images and link lists the supplement embeds.

## Build

Compiled with `pdflatex` (TeX Live 2023) from the repo root, writing artifacts into `pdf/`:

```bash
pdflatex -interaction=nonstopmode -output-directory=pdf latex/main.tex
```

This produces `pdf/main.pdf`. The committed deliverable is `pdf/nazib_abrar_mte_ruet_resume.pdf`, so after a successful build:

```bash
mv pdf/main.pdf pdf/nazib_abrar_mte_ruet_resume.pdf
```

The supplement builds the same way and is renamed the same way:

```bash
pdflatex -interaction=nonstopmode -output-directory=pdf latex/supplement.tex
mv pdf/supplement.pdf pdf/nazib_abrar_technical_attachment.pdf
```

It must be built from the repo root: its `\includegraphics` paths are repo-relative (`assets/...`), so compiling from inside `latex/` fails to find the images.

[latex/combined.tex](latex/combined.tex) is a third, optional deliverable for applications that accept only one file (Google Forms, single-upload portals). It contains no content of its own: it `\includepdf`s the two finished PDFs with a divider page between them, so it must be built **last**, after both of the above have been built and renamed.

```bash
pdflatex -interaction=nonstopmode -output-directory=pdf latex/combined.tex
mv pdf/combined.pdf pdf/nazib_abrar_resume_with_attachment.pdf
```

Because it consumes the renamed PDFs rather than the `.tex` sources, editing `main.tex` or `supplement.tex` does not change the fused output until those two are rebuilt first. It is 7 pages (2 + 1 divider + 4). The résumé and supplement remain the primary deliverables; the fused file is only for single-upload situations, and is never a replacement for them.

Single pass is enough — the document has no cross-references or bibliography needing a second run. Requires the `fontawesome5` and `sourcesanspro` packages (Debian: `texlive-fonts-extra`).

The `.aux`, `.log`, and `.out` files for both documents are tracked in git and change on every build; commit them along with the PDFs, matching existing history. Do not commit a stray `pdf/main.pdf`, `pdf/supplement.pdf`, or `pdf/combined.pdf` — rename them first.

## Typography

The body font is **Source Sans Pro**, set document-wide by `\usepackage[default]{sourcesanspro}` together with `\usepackage[T1]{fontenc}` — both near the top of the preamble. This replaces the LaTeX default (Computer Modern).

Source Sans Pro was chosen over other modern sans options (Lato, IBM Plex Sans) specifically because it ships real small caps, which the `\titleformat` block relies on via `\scshape` for section headers and the header uses for the name. Swapping in a font without small caps silently flattens those to normal case. Any font change also shifts metrics enough to move the page break — see below.

## Layout constraints

The résumé is deliberately **2 pages**. Page 2 is forced by a hardcoded `\newpage` in the middle of the Research & Projects section ([latex/main.tex:156](latex/main.tex#L156)), placed between the StereoLite and drone-navigation project blocks. Any content edit that changes vertical extent can push text past page 2 or leave page 1 short — always rebuild and confirm the log still reports `2 pages`, and move the `\newpage` if the break lands badly.

### Supplement layout

The supplement is **4 pages**, one topic per page, enforced by three `\newpage` calls: Embedded Linux platforms, Microcontroller platforms, Project Demonstrations + Kibo-RPC, then Certificates. The two platform pages each end with a `\suppFullFigure` — a full-`\textwidth` photo wrapped in `\vfill` so it centres in whatever vertical space the bullets leave.

`\suppFigure` and `\suppFullFigure` both wrap the image and its caption in a `minipage`. That is load-bearing: without it LaTeX will break between an image and its caption and strand the caption alone on the next page. Resizing any figure can change the page count — rebuild and confirm `4 pages`.

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

`docx/` and `job_description/` exist but are empty — earlier `.docx` sources and per-application job descriptions were removed.
