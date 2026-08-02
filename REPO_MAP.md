# AI Job Search — Repo Map (for Paraguay adaptation)

## 1. What this repo actually is

A **Claude Code workflow**, not a standalone app. There's no server, no database, no UI.
Everything is:
- Markdown files that define "skills" and "commands" Claude reads and follows
- LaTeX templates Claude fills in and compiles to PDF
- Small TypeScript CLIs (run via `bun`) that scrape job portals
- A couple of Python utility scripts

You interact with it entirely by running `claude` in this folder and typing slash commands
(`/setup`, `/scrape`, `/apply`, etc.). The "intelligence" lives in the `.md` instruction files,
not in code.

Original author: Mads Lorentzen, a Danish geophysicist who built this for his own job search
in Denmark. That's why the shipped portal integrations and the README's default example are
Danish. The framework itself claims to be country/language-agnostic — the Danish parts are
meant to be a worked example, replaceable via `/add-portal`.

## 2. The three layers, and where each one lives

| Layer | Location | Danish-specific? |
|---|---|---|
| **Core workflow** (profiling, fit scoring, CV/cover-letter drafting, interview prep) | `.claude/commands/`, `.claude/skills/job-application-assistant/` | No — this is the generic engine |
| **Your personal data** | `CLAUDE.md` + the `01-...07-...md` files in `job-application-assistant/` | No — currently template placeholders, gets filled by `/setup` |
| **Job portal search** | `.agents/skills/*-search/` (CLI tools) | Was originally 4 Danish portals, now removed — see §5 |

## 3. Command → what it does → files it touches

All defined in `.claude/commands/*.md` (the prompt Claude follows) and often backed by a
skill folder in `.claude/skills/`.

- **`/setup`** — onboarding interview. Populates `CLAUDE.md` and all of
  `.claude/skills/job-application-assistant/01-*.md` through `08-*.md`, plus
  `cv/main_example.tex` and `.claude/skills/job-scraper/search-queries.md`.
  Three input modes: `documents/` folder, single CV paste, or interactive interview.
- **`/scrape`** — runs whichever `*-search` CLIs exist under `.agents/skills/` (auto-discovered,
  nothing to register), dedupes results. Logic lives in `.claude/skills/job-scraper/SKILL.md`.
- **`/rank`** — batch-scores scraped jobs against `04-job-evaluation.md`'s rubric.
- **`/apply <url>`** — the main event: fit evaluation → draft CV/cover letter (LaTeX) →
  reviewer agent critique → revise → compile with `lualatex`/`xelatex` → ATS text-layer check
  → present. Outputs to `cv/main_<company>_<role>.tex` and
  `cover_letters/cover_<company>_<role>.tex`.
- **`/interview`** — prep pack for a tracked application, pulls from `documents/applications/<company>_<role>/`.
- **`/outcome`** — records what happened, archives materials into `documents/applications/`,
  updates `job_search_tracker.csv`. Also has a `followup` mode for stale applications.
- **`/expand`** — enriches profile from GitHub/portfolio/Scholar/etc.
- **`/upskill`** — gap analysis between profile and tracked postings.
- **`/html-report`**, **`/notion-sync`**, **`/gmail-sync`** — reporting/sync integrations, read from the tracker/archives, don't need touching for a market swap.
- **`/add-portal`** — generates a new `*-search` skill for a local job board (this is your tool
  for adding Paraguay-specific portals).
- **`/add-template`** — swap the LaTeX CV/cover-letter templates for your own.
- **`/reset`** — wipes profile/documents data, requires typing `RESET`.

## 4. Profile files (the "brain" `/setup` fills in)

In `.claude/skills/job-application-assistant/`:

| File | Content |
|---|---|
| `SKILL.md` | Skill definition/trigger for the whole application-assistant skill |
| `01-candidate-profile.md` | Structured CV data (education, experience, skills) |
| `02-behavioral-profile.md` | Personality/behavioral assessment |
| `03-writing-style.md` | Tone/voice rules for generated text |
| `04-job-evaluation.md` | Fit-scoring rubric, deal-breakers, career goals |
| `05-cv-templates.md` | Profile-statement variants + LaTeX CV structure/tailoring rules |
| `06-cover-letter-templates.md` | Cover letter structure rules |
| `07-interview-prep.md` | Your STAR examples |
| `08-application-forms.md` | Presumably for filling out web application forms (not mentioned in README's file-structure table — worth opening if you use it) |

`CLAUDE.md` (repo root) is the master profile summary + workflow rules — this is the file
Claude reads automatically every session (it's the "project instructions" file).

Right now **all of these are still template placeholders** (`[YOUR_NAME]`, `[YOUR_CITY]`, etc.)
— confirmed by reading `CLAUDE.md`. Running `/setup` is what fills them in with your actual
Paraguay-based profile.

## 5. Job portal skills — current state

`.agents/skills/` contains only:
- `freehire-search/` — multi-market tech job aggregator, REST API, zero deps. Works globally via `--region`/`--country`/`--remote` flags.
- `linkedin-search/` — LinkedIn's public `jobs-guest` endpoints, country-agnostic, takes `--location` as an explicit flag (e.g. `-l "Asunción, Paraguay"`).

All four original Danish portal folders (`jobbank-search`, `jobdanmark-search`,
`jobindex-search`, `jobnet-search`) are deleted, and every reference to them has been cleaned
up (as of 2026-08-01):
- `README.md` — file-structure diagram, "Job search tools" section, and the `/apply` example
  URL no longer mention the Danish portals or `jobindex.dk`.
- `SETUP.md` — install loops (§3) and the `/apply` example (§6) updated to just `linkedin-search`/`freehire-search`.
- `.github/workflows/ci.yml` — CI matrix (`cli-checks` job) trimmed to the two remaining tools.
- `.claude/commands/add-portal.md`, `.claude/commands/setup.md`, `.claude/skills/job-scraper/SKILL.md`,
  `.claude/skills/job-scraper/search-queries.md` — internal references/examples that pointed at
  `jobindex-search` or "Danish demos" now point at `linkedin-search`/`freehire-search` instead.
- `.claude/commands/apply.md`, `.claude/skills/job-application-assistant/06-cover-letter-templates.md`,
  `.claude/skills/job-application-assistant/04-job-evaluation.md` — illustrative Danish-language
  examples (cover-letter closing, language-matching examples, review-site example) replaced with
  Spanish examples or generic phrasing, since these are otherwise-generic instruction files.
- `tools/lint_skills.py` and `tests/` had no hardcoded references — nothing needed there. Confirmed
  via `python3 -m unittest tests.test_lint_skills tests.test_security_guards` — all 23 tests pass.

**Deliberately left alone** — not stale, just Denmark-flavored and harmless:
- `CHANGELOG.md`, `CONTRIBUTING.md` — historical/policy text describing the *original* upstream
  repo accurately; rewriting history here would be wrong.
- `salary_lookup.py`, `tools/convert_salary_excel.py`, `tools/README_SALARY_TOOL.md`, and their
  tests — a genuine fuzzy-matching feature (handles Nordic characters, `A/S`/`ApS` legal suffixes)
  that happens to have been built against Danish demo data. It's idle, not broken, for a
  Paraguay profile; ripping it out would mean rewriting several tests for no functional gain.
  Revisit only if you want Spanish/Guaraní-specific normalization instead.

**For Paraguay**, `linkedin-search` and `freehire-search` already work out of the box (just
point `-l`/`--country` at Paraguay/Asunción). If you want a local Paraguayan job board
(e.g. computrabajo.com.py, empleospy.com), `/add-portal` is the tool to generate a new skill for it.

## 6. CV / Cover letter generation

- `cv/main_example.tex` — moderncv (banking style) LaTeX template, placeholder content.
- `cover_letters/cover.cls` + `cover_example.tex` — custom LaTeX class with bundled Lato/Raleway
  fonts (`cover_letters/OpenFonts/`).
- Compiled with `lualatex` (CV) and `xelatex` (cover letter) — **not** pdflatex.
- `/add-template` lets you swap in a different template/toolchain (LaTeX, Typst, etc.) if
  moderncv doesn't suit you.
- CLAUDE.md's Verification Checklist (in your project instructions) is mandatory after every
  CV/cover-letter generation — 2-page CV, 1-page cover letter, PDF text-layer ATS check, etc.

## 7. Supporting files/tools

- `salary_lookup.py` + `tools/convert_salary_excel.py` — optional salary benchmarking, BYO data
  (`salary_data.json`, gitignored). No Danish/Paraguay data shipped — you'd need your own source
  if you want this (e.g. glassdoor/local salary surveys for Paraguay/LatAm).
- `tools/lint_skills.py` — CI lint that checks skill/command/settings.json structure.
- `tools/security_guards.py` — CI guard: permission allowlist, gitignore rules.
- `tools/check_upstream_updates.py` — compares your fork's personalized files against upstream
  by `framework_version` markers, so you can pull framework improvements without clobbering
  your data. Relevant since you're diverging from upstream (removing Danish stuff).
- `job_search_tracker.csv` — the application-tracking spreadsheet (gitignored, doesn't exist
  yet until you generate data).
- `documents/` — drop folder for your real CV/LinkedIn export/diplomas/references/past
  applications; `/setup` Path A reads this. Has its own README (read above) explaining exact
  naming conventions.
- `.claude/settings.json` — shared permission allowlist (what Bash commands Claude can run
  without asking). Currently scoped to `bun run`, salary_lookup.py, pdftotext.
- `AGENTS.md` — points other agent tools (Codex, Antigravity, Gemini CLI) at the same canonical
  files, in case you ever use a non-Claude-Code tool against this repo.

## 8. What "language/country-agnostic" actually means here

The README's central claim is that only the **portal search layer** is Denmark-specific;
fit-evaluation, CV drafting, cover-letter writing, and interview prep are generic and driven
entirely by whatever you put in your profile files. That claim holds up on inspection — I
didn't find hardcoded Danish assumptions in `.claude/skills/job-application-assistant/` (the
scoring rubric, writing style, and templates are all fill-in-the-blank, not Denmark-coded).
CV language is even an explicit `/setup` question (`[YOUR_CV_LANGUAGE]` in CLAUDE.md), so
Spanish output for Paraguay is supported natively, not a hack.

## 9. Suggested order of operations for your Paraguay adaptation

1. ~~Doc cleanup of stale Danish references~~ — done, see §5.
2. Run `/setup` to populate `CLAUDE.md` + the profile files with your real data (name, Paraguay
   location, Spanish/English, experience, etc.) — right now everything is still placeholder text.
3. Decide on portals: keep `linkedin-search` + `freehire-search` as-is (already global), and
   optionally run `/add-portal` for a Paraguayan/LatAm board if you want local coverage beyond
   LinkedIn.
4. Confirm LaTeX toolchain is installed locally (`lualatex`/`xelatex`) per SETUP.md §1, or swap
   in a template you prefer via `/add-template`.
5. Test end-to-end with a real posting: `/apply <url>` and watch the fit-evaluation +
   CV/cover-letter output before trusting it on a real application.

## 10. Maintaining this map

This file is a plain, manually-referenced doc — not wired into a skill or auto-loaded like
`CLAUDE.md`, on purpose (so it doesn't inject repo-maintenance context into normal `/apply`/`/scrape`
runs, and so it can't compete with the 12 existing slash commands' own trigger matching). When
you're doing repo-structure work, `@`-mention this file, and after a structural change (added/
removed a portal, command, or skill; changed what a command touches) ask for it to be updated,
or expect it to be updated in the same turn as the change.
