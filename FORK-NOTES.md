# Flash Bridge fork of slicer-cli

This is a fork of [Helio-Additive/slicer-cli](https://github.com/Helio-Additive/slicer-cli)
for Flash Bridge, which runs the OrcaSlicer engine (`bin/slicer_cli-orcaslicer`)
on a 4 GB single-board computer. It stays on Helio's v0.1.0-dev-19 and the
OrcaSlicer 2.4.2 pin (`8500fcd`); it changes nothing about the G-code, the
profiles or the resources tree.

## Why the fork exists

On a 1,005,210-triangle painted helmet (two colours, 254 mm tall, 1,270
layers) the unpatched engine peaks at 2.47 GB resident on Linux x86_64 with
the profile defaults (arc fitting on, resolution 0.012). Slicing itself
finishes at 1.29 GB; the G-code export then takes it to 2.53 GB in about
19 seconds. The G-code is 112 MB: 3.17 million G1 moves plus 516,000 arcs.
The extra 1.24 GB is `GCodeProcessorResult::moves`, one `MoveVertex` per
move (about 300 bytes each including vector growth; arc moves also carry
`interpolation_points`), which the desktop app keeps for its coloured
preview. A command-line slicer never reads it. On the board (4 GB RAM,
1.9 GB zram swap) the kernel killed that slice at 2.8 GB resident.

Target: helmet peak at or below 1.1 GB with `--resolution=0.05
--enable-arc-fitting=0`, and at or below 1.6 GB with the profile defaults,
with byte-identical G-code and unchanged print-time and filament statistics.

## What is changed

1. `.github/workflows/slicer-cli-ci.yml`: the macOS and Windows jobs
   (`build-nonlinux`, `package`) are removed; they need paid runners and the
   bridge ships only the Linux arm64 package. `release` depends on
   `package-linux` only, and accepts a release that was created on the GitHub
   web page before the tag's CI run (upstream refused a published release).
   Both engines still build on Linux because `package-linux` copies and tests
   both programs; the bridge's pack script deletes `bin/slicer_cli`.

2. Override files (compiled in place of the engine file at the same relative
   path under `references/OrcaSlicer/src/`; see `docs/building.md`, "The
   override layer"):

   | File | What it changes |
   |-|-|
   | (none yet; the lean-mode `libslic3r/orcaslicer/libslic3r/GCode/GCodeProcessor.cpp` is added in a later commit) | |

## Releases

Tags are `slicer-cli/v0.1.0-dev-19-ffbN` (Helio's version plus `-ffbN`,
N counting this fork's releases), so the asset is named
`slicer-cli-v0.1.0-dev-19-ffbN-linux-arm64.tar.gz` and keeps Helio's layout.
The release is created on the GitHub web page with "Create new tag on
publish"; the tag push starts CI, and the `release` job attaches the assets.

## Rebasing on a new Helio release

1. Pull upstream `main` (or the release tag) into this fork; take Helio's
   version of every file except the ones listed above.
2. Re-apply the workflow trim (section 1) to the new workflow file.
3. For each override file, start again from the newly pinned engine file
   (`references/OrcaSlicer/src/libslic3r/...`) and re-apply the override's
   hunks; an override is a whole-file copy frozen at the commit it was taken
   from, so the old copy must not be kept as is.
4. Build, then re-measure with the same models and the same command before
   tagging.
