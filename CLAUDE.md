# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Lecture material, not software: slides and notes for "Skalierbare Systeme" at DHBW Karlsruhe, written in pandoc Markdown and compiled to PDF with LaTeX. The default branch is the year (`2026`). `TODO.md` is the working plan: session table with dates and lecturers, the AI-engineering thread, cross references between sessions, conventions and open points. Read it before changing content. The tooling is shared with the Web Engineering lecture (github.com/DHBW-KA-Webengineering/Lecture_Webengineering), which is the pattern for anything not yet present here, such as exercises or media.

## Build

Requires PowerShell 7 (`pwsh`, the script uses `ForEach-Object -Parallel`), pandoc, TeX Live with lualatex and the metropolis beamer theme, and the `pandoc-plantuml` filter. CI uses the container `ghcr.io/lukaspanni/pandoc-builder` and installs the theme with tlmgr.

```powershell
./build.ps1 Material/Slides/01_Grundlagen_Systemdesign.md   # one file -> build/Slides/01_Grundlagen_Systemdesign.pdf
./build.ps1                                                 # everything, plus build/script.pdf and build.zip
```

- Files under `Material/Slides/` are built as beamer slides with `--slide-level 2`; everything else under `Material/` as a plain PDF. The output folder mirrors the parent folder name.
- The full run concatenates all slide files into `build/script.pdf` through a temporary `Material/Slides/99_Script.md`, which it deletes afterwards. Do not create a file with that name.
- The script fails hard if any PDF does not compile and prints the pandoc/LaTeX log for each failure.
- There is no lint or test. The CI workflow (`.github/workflows/create-release.yml`) builds on every push and is the compile check when the toolchain is not installed locally. On the default branch it also publishes `build.zip` as a GitHub release, which is where students download the PDFs.

## How a slide file works

- `#` is a section and produces a title slide (`section-titles: true`), `##` is one slide, `###` is a block heading inside a slide. Keep a slide at seven bullets or fewer; metropolis at 12pt overflows beyond that.
- Every session file starts with the same frontmatter (copy `template_Slides.md`) and a time plan in an HTML comment: minutes per section against 180 content minutes for a 4-VE session (90 for session 7). Update the plan when content moves.
- HTML comments are the author-note channel (`<!-- TODO(Lu): ... -->`); pandoc drops them from the PDF. Put a blank line before a comment so it does not attach to the preceding list.
- Arrows are written as `\rightarrow{}`, as in the sibling lecture. Image paths are relative to the Markdown file (`rebase_relative_paths`), media lives in `Material/Slides/media/`. ```` ```plantuml ```` blocks are rendered by the filter.
- The `author` field names the one lecturer who holds the session; only session 7 names both.

## Content conventions

- Slide and note content is German with the English technical terms the field uses. README, `TODO.md`, comments and commit messages are English.
- One file per session, numbered `NN_Topic.md` in session order. Each content session ends with a summary and a pointer to the next session. An "AI Engineering" section is optional and goes in where it fits the topic.
- No exercises or assignments; each content session has one "Beispiel" slide that the lecturer works through in plenum.
- Each topic has one owning session; other sessions reference it as "(Vertiefung in Vorlesung N)" or "(Bezug Vorlesung N)". When moving content, fix these references and the cross-reference list in `TODO.md`.
- The three module-handbook references on the "Literatur: Modulhandbuch" slide in session 1 are fixed; anything else is "weiterführende Literatur".
