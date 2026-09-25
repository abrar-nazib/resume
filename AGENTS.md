# Repository Guide

## Purpose and layout

This repository publishes Nazib Abrar's résumé materials. It contains no
application code, package manifest, or automated test suite.

- `latex/main.tex` is the primary two-page résumé.
- `latex/supplement.tex` is the four-page technical attachment distributed
  alongside the résumé.
- `latex/combined.tex` creates the optional single-upload PDF: the résumé, a
  divider page, and the attachment.
- `assets/` contains images, certificate PDFs, and source lists embedded by
  the attachment.
- `pdf/` contains intentionally committed compiled deliverables and LaTeX
  auxiliary files.

Read `CLAUDE.md` before making substantive edits. It documents the typography,
custom macro behavior, content conventions, and layout decisions in more
detail.

## Editing rules

- Edit the `.tex` sources; do not edit generated PDFs or `.aux`, `.log`, or
  `.out` files by hand.
- Preserve Source Sans Pro and its small-caps behavior unless a deliberate
  typographic redesign is requested.
- Keep résumé entries achievement-oriented and concrete about technologies.
- Keep external links in the existing `\href{...}{\underline{...}}` style.
- Do not simplify apparently extra arguments after `\resumeSubheading` or
  `\resumeProjectHeading` without changing the macro and every affected call
  site. The macros accept two arguments; trailing groups/text intentionally
  form subtitle lines in several entries.
- Treat image/caption wrappers in the attachment as layout-critical. Captions
  must stay with their images.

## Build and verify

Run all commands from the repository root. A TeX Live installation with
`fontawesome5` and `sourcesanspro` is required (on Debian,
`texlive-fonts-extra` provides them).

```bash
# Primary résumé
pdflatex -interaction=nonstopmode -output-directory=pdf latex/main.tex
mv pdf/main.pdf pdf/nazib_abrar_mte_ruet_resume.pdf

# Technical attachment
pdflatex -interaction=nonstopmode -output-directory=pdf latex/supplement.tex
mv pdf/supplement.pdf pdf/nazib_abrar_technical_attachment.pdf

# Optional single-upload document; run only after the two commands above
pdfannotextractor pdf/nazib_abrar_mte_ruet_resume.pdf
pdfannotextractor pdf/nazib_abrar_technical_attachment.pdf
pdflatex -interaction=nonstopmode -output-directory=pdf latex/combined.tex
mv pdf/combined.pdf pdf/nazib_abrar_resume_with_attachment.pdf
```

After a source or asset change, rebuild the relevant document and inspect the
compiler output and PDF. Confirm that the résumé remains two pages, the
attachment remains four pages, and the combined file remains seven pages.
`combined.tex` consumes the renamed PDFs rather than the `.tex` sources, so
always rebuild the résumé and attachment before rebuilding it.
It uses PAX annotation maps to retain clickable links from both inputs;
regenerate the `.pax` files with `pdfannotextractor` after either component PDF
changes and before building the combined PDF.

Commit the updated named PDFs together with their corresponding tracked LaTeX
auxiliary and `.pax` files. Do not leave `pdf/main.pdf`,
`pdf/supplement.pdf`, or `pdf/combined.pdf` in the working tree; rename them to
the committed deliverable names above.

## Layout-sensitive areas

- `main.tex` has a deliberate `\newpage` in Research & Projects to force the
  two-page layout. Reassess its placement after changes that affect height.
- `supplement.tex` has three deliberate `\newpage` commands, one topic per
  page. Figure sizing and vertical spacing affect its four-page layout.
- Build from the repository root because graphics paths are repository-relative.
