# Skalierbare Systeme

Content (slides & notes) for the lecture "Skalierbare Systeme" @ DHBW Karlsruhe, winter semester 2026, by Lukas Panni and Silas Schnurr.

To download pre-built PDFs, head over to [Releases](https://github.com/dhbw-ka-scalable-systems/lecture/releases).
The latest version can be downloaded [here](https://github.com/dhbw-ka-scalable-systems/lecture/releases/latest/download/build.zip).

Please report any issue with notes or slides as an issue.
For general questions use [Discussions](https://github.com/dhbw-ka-scalable-systems/lecture/discussions) instead.

## Structure

- `Material/Slides/NN_Topic.md`: one file per session, built to beamer slides
- `Material/Notes/`: notes, built to plain PDFs
- `TODO.md`: session plan, conventions and open points
- `template_Slides.md`, `template_Notes.md`: starting points for new files

## Building locally

Requires pandoc, a TeX Live with lualatex and the metropolis beamer theme, and the `pandoc-plantuml` filter. On CI the container `ghcr.io/lukaspanni/pandoc-builder` provides all of this.

```powershell
./build.ps1                                                # everything, plus build/script.pdf and build.zip
./build.ps1 Material/Slides/01_Grundlagen_Systemdesign.md  # a single file
```

Licensed under CC BY-SA 4.0, see `LICENSE`.
