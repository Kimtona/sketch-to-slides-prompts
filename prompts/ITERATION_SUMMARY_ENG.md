# Iteration Summary — Sketch-to-Slides (personal reference)

English translation of [`ITERATION_SUMMARY.md`](ITERATION_SUMMARY.md) (the Korean original is kept unchanged).

These are notes for writing the **Iteration Process** section of the report. For the verbatim prompts and their numbers (P01…), see [`PROMPT_LOG.md`](PROMPT_LOG.md).
Times are KST (2026-11-06); commits refer to the `math-slides` repository. Results and numbers are only the real values recorded in the development log (`docs/SKETCH_DEVLOG.md`, `examples/composite/measurements/`).
This document is a working note, not a submission document (the list of verbatim prompts is `PROMPT_LOG.md`).

## Overview

| Stage | Prompts | Focus |
|---|---|---|
| 1. Initial implementation | P01–P02 | Connect the local model (Ollama) → place the result inside the image area |
| 2. Bug fixing and stabilization | P03–P17 | Broken equations from paper captures, curve/color recovery, test port collisions, porting partial equation coloring, merging two accounts' histories |
| 3. Iterative improvement | P18–P22 | Table/list structure conversion, real-use test feedback (composite screenshot), speed (Document Mode), editing converted equations |
| 4. Final verification and release | P23–P31 | Integrated main, v2.0.0 docs/tag/release, submitting the prompt log |

Timeline: first implementation commit 00:45 (before the first prompt in this log) → Ollama 01:42 → frame placement 02:00 → LaTeX 02:39 → pixel-based color/curves 03:20 → (interrupted by the usage limit, resumed in the afternoon) equation color port 15:45 → integration 16:38 → structure conversion 17:34 → composite image 18:24 → Document Mode 18:52 → equation click fix 19:40 → v2.0.0 21:59–22:28.

## Per-prompt summary

### P01 · Connect a real vision model (Ollama instead of a paid API)
- **Background:** The prompts for the design and initial implementation before this (IR + layout converter `e638e41`, AI bridge `d5bd7e2`, preview and AI review loop `c271518`, CLI `e335104`, docs `a1ad4f0`, 00:45–00:54) are not included in this log.
- **Purpose:** Test and record real conversions with a local model (`qwen2.5vl:7b`) at no cost.
- **Problems at the time:** The model returned coordinates 1.3–1.5× too large, sometimes looped in its output, and LaTeX control characters were corrupted (experiments 2–3).
- **Result:** Ollama added as the default provider `7aaf63a`; local-model experiments and README recorded `4c9470c` (01:42). Uniform rescaling, a token cap, and a review policy for Korean handwriting and equations came in at this point.

### P02 · Place converted elements inside the original image area
- **Purpose:** UX change so the result appears where the original image was, not across the whole slide.
- **Result:** Rebuild inside the image frame `bd801e3`; real GUI runs recorded `28ac5fa` (02:00).

### P03 · Broken equations from paper captures
- **Purpose:** Fix inline equations `\(..\)` showing as plain text and display equations coming out too wide when converting a PPO paper capture (reproduce → fix → test; no new provider and no prompt optimization).
- **Cause:** The adapter did not distinguish math spans and wrapped `\commands`, corrupting `\(\pi_{\theta}\)` into `\($\pi$_{$\theta$}\)`. Long equations had no width fitting.
- **Result:** Math-span awareness plus shrinking by MathJax's real typeset width `651c252` (02:39).

### P04 · Curved arrows / curved lines (separate session)
- **Purpose:** Add curved arrows and curved connectors to the editor, and have Sketch-to-Slides recover curves in hand drawings.
- **Result:** Quadratic Bézier curves `d0bf69b` (02:42). (Because this ran concurrently, P05 asks to avoid conflicts.)

### P05–P06 · Avoiding conflicts between concurrent sessions; test ports
- **Purpose (P05):** Another session is developing the curve feature, so commit only the LaTeX-related files without conflicts.
- **Problem (P06):** The tests used a fixed port (5187), so they collided when several sessions ran them at once.
- **Result:** Fixed at the root with OS-assigned ports, process-group shutdown, and so on `56d7500` (02:50). All GUI tests have used `tests/test-ports.mjs` since.

### P07–P08 · FullDuplex hand drawing: curved arrows and color recovery
- **Purpose:** Analyze and improve the inaccurate curved arrows and lime highlight marks in a real Ollama conversion (the original `test3.jpeg` was provided, P08).
- **Cause (measured, from the devlog):** The curvature was already lost at the model output stage (two straight `arrow`s, one arrow missing). The highlighter's color name was right, but MathSlides could not express a light, thick marker and dropped the `explains` relation, and the model's color name varied between runs.
- **Result:** Recovery by measuring ink color, thickness, whether it is a highlighter, and the bend of curves mistaken for straight lines directly from the image pixels `362062a` (03:20).

### P09 · Investigation (no code changes): moving inside an equation and partial selection
- **Purpose:** Investigation only, as the token limit was near. Locate the old feature of "moving inside an equation with the arrow keys, selecting part of it, and changing its color".
- **Result:** Confirmed that the feature exists nowhere in this account's code or git history (an equation is a single `atom` node; clicking it opens a LaTeX popup), and wrote it up as an investigation note (`docs/MATH_CURSOR_INVESTIGATION.md`, a personal note not committed to the repository). Then found an implementation in the collaborator's account and ported it in P10.

### P10–P12 · Porting the partial equation coloring developed in another account
- **Purpose:** Bring two commits from the collaborator's account (`51a5129` symbol selection and partial color, `af41278` selection stabilization) into the current Sketch-to-Slides tree (P10), continue the work interrupted by the usage limit (P11), and reflect it in main once tests passed (P12).
- **Result:** Ports `4ef4d47`, `b02f038` + a test for equations converted by Sketch `2c5ef98` (15:45–15:54), merged into main.

### P13–P14 · Shortcut investigation (no code changes)
- **Purpose:** Find the shortcuts for rectangle / text / line / arrow, and for the feature that created a 2px rectangle when dragging on an empty canvas while holding a key (Quick Draw = ⌥⇧+drag).
- **Result:** Listed only the shortcuts actually implemented (no code changes).

### P15–P17 · Integrating the histories of the two macOS accounts and settling main
- **Purpose:** Merge the two separately developed MathSlides histories into one (backup branches, integration worktree, conflict resolution, full tests), check the status (P16), and settle on GitHub's default main (P17).
- **Result:** Merge `b3cc797` + integrated-tree verification `b97c51f` (16:38–16:39); the remote main became `b97c51f` (no force push).

### P18 · Tables, formatting, and lists as native elements (structure conversion)
- **Purpose:** Recover tables, bold/italic/strikethrough, lists, headings, quotes, and equations from an image as native elements (plan first, feature branch and worktree).
- **Result:** Native Table/list/quote conversion `821c5d5`, records `8eae918` (17:34). The 7B model does not recognize bold/italic/strikethrough (recorded as a limitation).
- **Also found and fixed:** Repeated or duplicated output made a table take 5 minutes and vanish, or inflate to 6×4 → salvage of finished elements, repetition detection, duplicate removal.

### P19–P20 · Problems revealed by real-use testing → the real composite image
- **Purpose (P19):** Analyze and fix problems the automated tests missed (table/list failure on a composite screenshot, no partial color on converted equations, 202+ seconds). **P20:** Re-requested using the real input image (`~/Downloads/test_composite.png`) (pin it as a regression test, save the model output, 60-second target).
- **Cause (measured):** The model read the image correctly but emitted table cells, list items, and equations all as plain `text`. The structure was lost in the adapter: table detection failed on a composite page because of an "at least 80% of the page's text" condition, `1.` lines became one-item lists, and bullet glyphs were absent. Speed: 1,481 output tokens ÷ about 6.6 tokens/s (244.9 s).
- **Result:** Region-based table detection, list merging by counting numbers, bullets inferred from indentation, and equations the model emitted as `text` turned into equations `87f7a70` (18:24). Fixed in `Canvas.tsx` a bug where, right after Apply, the group selection made dragging an equation move the group. The real image's 5×2 table, 3 bullets, 3 numbered items, and 2 equations were recovered as native elements.

### P21 · Approval of Document Mode (speed)
- **Purpose:** Add a mode that is much faster via Markdown transcription (no coordinates), keeping Layout Mode, distinguishing them in the UI, and making Document Mode the default.
- **Evidence at the time:** One measurement each: JSON (with boxes) 244.9 s vs Markdown 55.3 s, with accurate table/list/equation/quote.
- **Result:** Document Mode added `c95ecd2` (18:52): heights are measured and content is laid out top to bottom, overflowing onto new slides (one Undo cancels them). Repeated real-GUI measurements: Document 35.1–35.3 s (model warm), Layout 215.0 s. The cache effect (23.6 s) is excluded from the representative values.

### P22 · Converted equations do not enter edit mode
- **Purpose:** Investigate the symptom that an equation created directly with `/math` is edited on click while a converted one only shows a blue border (including why the existing `math-leaf-sketch` test passed but missed it; reproduce with real mouse input, then fix).
- **Cause:** Right after Apply the whole set of converted elements is group-selected → clicking a box that holds an equation, away from a symbol, only "narrowed the group to one element" and did not enter editing. The existing test called `startEditing` directly and skipped the click path. (Everything else — types, equation structure, click flow — was confirmed identical by comparing with real input.)
- **Result:** A click now narrows the selection and starts editing immediately `c62a36f` (19:40) + `tests/math-leaf-document.mjs`, which uses only real mouse/keyboard events.

### P23 · Final integration (verification, cleanup, merge to main, assignment material)
- **Purpose:** Final verification, file cleanup, merge and push to main, and organizing the assignment material (workflow, models, problems and fixes, mode comparison, timing measurements).
- **Result:** Fast-forwarded main to `6db351a`, pushed after confirming typecheck, build, and the full `npm test` passed; preserved `docs/SKETCH_ASSIGNMENT_NOTES.md` and `examples/composite/measurements/`.

### P24–P25 · v2.0.0 release preparation (minimal validation + full instructions)
- **Purpose:** With the deadline close, prepare the README, version, and release notes without further testing. P24 is a fragment of the instructions; P25 is the full text (12 README sections, reproducibility, no commit/push/tag/Release without approval).
- **Result:** README rewritten in English, version 1.3.1 → 2.0.0 (`package.json`/lockfile), `docs/RELEASE_NOTES_v2.0.0.md`, detailed docs split out (`docs/USER_GUIDE.md`, `docs/SKETCH_TO_SLIDES.md`). Left as local changes until approved.

### P26–P30 · Release procedure (approval step by step)
- **P26:** Step 1 approved → commit and push `0af9adf` (21:59). Also includes the instruction to answer in Korean from then on.
- **P27:** Created and pushed tag `v2.0.0` → GitHub Actions (tests, package, release) succeeded and produced a Draft Release (DMG 131,739,703 B + SHA256SUMS).
- **P28:** Check that the README makes clear Ollama and the model are installed separately, and update the Draft body → added the install distinction and the `ollama list` / `curl …/api/tags` check steps.
- **P29:** README change committed and pushed `945f1de` (22:27); Draft links checked.
- **P30:** Publication approved → v2.0.0 Release published (draft: false, Latest); the tag stays at `0af9adf`.

### P31 · Prompt-log repository
- **Purpose:** Organize the verbatim prompt list and a personal summary in a separate public repository for submission (the request to write this document).

## Flow usable for the report's Iteration Process

1. **Implementation → gap with reality:** After the initial implementation, connecting a real model (P01) exposed coordinate drift, repeated output, and LaTeX corruption, each fixed with a unit-testable correction (rescaling, salvage, escape repair).
2. **Measurement-driven improvement:** Broke "it is slow" down into token count and speed (1,481 tokens ≈ 215–245 s) and cut it to 35 s by changing the output format (JSON + coordinates → Markdown, 176 tokens) (P20→P21). The prompt-cache effect on repeated runs of the same image is reported separately.
3. **Finding in real use what automated tests missed:** the table/list failure on a composite screenshot (P19–P20) and the first-click problem on converted equations (P22). Both times it was handled in the order "cause first → failing test that reproduces it → minimal fix", and real model outputs were saved as fixtures so the problems can be reproduced (`tests/fixtures/sketch/`).
4. **Demand for honest reporting:** Failures and unmet targets are reported as they are (the 60-second goal was missed until Document Mode was approved, the 7B model cannot see bold/italic, Layout Mode does not recognize block quotes).
5. **Collaboration and safety measures:** avoiding conflicts between concurrent sessions (P05–P06); merge/push/tag/publish only after approval (P17, P26–P30); no force pushes.

## Reference links
- Project: https://github.com/Kimtona/math-slides (release: https://github.com/Kimtona/math-slides/releases/tag/v2.0.0)
- Development log: `docs/SKETCH_DEVLOG.md` (Stages 9–13); assignment-oriented summary: `docs/SKETCH_ASSIGNMENT_NOTES.md`; measurements: `examples/composite/measurements/`
