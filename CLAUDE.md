# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **learning journal (TIL)**, not a software project. It documents *reverse-engineering* of popular Claude Code plugins ("why do they work"), building toward eventually authoring an original plugin. There is **no build system, no tests, no dependencies, no runtime code** — all content is Markdown prose plus third-party source excerpts. Do not invent build/lint/test commands.

Author's framing: prove *understanding of plugin internals + hands-on authorship*, not just usage. Prioritize mechanism-level analysis over feature summaries.

## Language

All prose is written in **Korean**. Match this when adding or editing entries. Code/JSON/identifiers stay in their original language.

## Structure

- `entries/NN-topic.md` — the actual TIL writeups (numbered, e.g. `01-frontend-design.md`). Each is dated and analyzes real plugin source. This is the primary deliverable.
- `sources/<plugin>/` — **third-party source excerpts** copied verbatim for analysis. See below.
- `README.md` — index + roadmap. The "다음 할 일 (Next)" section is the resume point when continuing on another machine; the progress table and roadmap must be updated when an entry is added.
- `.gitignore` excludes `.claude/` and `.omc/` (local harness state) — don't commit those.

## `sources/` is not our code — critical convention

Everything under `sources/` was written by other authors (e.g. Anthropic's Frontend Design, Jesse Vincent's Superpowers) and is included under their licenses (Apache-2.0, MIT). Rules:

- **Do not edit, refactor, or "improve" files in `sources/`** — they are evidence, kept verbatim to match the original.
- Each plugin folder keeps its original `LICENSE` file. Preserve license/attribution.
- Only the *specific files cited by an entry* are excerpted here, not whole repos. `sources/README.md` maps each file to the entry that cites it and records the version snapshot.
- When a new entry analyzes a new plugin, add the excerpted files under `sources/<plugin>/`, its `LICENSE`, and a row in `sources/README.md` (folder, author, license, upstream link, version).

## Cross-linking convention

Entries and source docs link to each other with **relative paths** (`../sources/...`, `../entries/...`). Keep these working when moving/renaming files. Entries reference source files by path so a reader can jump from analysis to the exact original.

## Adding a new TIL entry

1. Create `entries/NN-topic.md` with a title, `> 대상:` and `> 날짜:` header lines (see `01-frontend-design.md` for the format).
2. Excerpt any newly-analyzed source into `sources/<plugin>/` + add its `LICENSE` and a `sources/README.md` row.
3. Update `README.md`: the 진행 상황 progress table and the 다음 할 일 roadmap.
