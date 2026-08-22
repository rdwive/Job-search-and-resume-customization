# Job System

A three-agent job search pipeline built for [Claude Code](https://claude.com/claude-code). Each agent has one job, reads and writes files, and never talks to the others directly — they hand off work purely through the filesystem.

```
Sourcer  →  finds real, open roles and ranks them
Tailor   →  rewrites your CV for one role at a time
Reviewer →  critiques the tailored CV as an ATS and as a hiring manager, Tailor rewrites from it (up to 3 rounds)
```

## What it does

1. **Sourcer** reads your CV and a profile of what you want, searches the web, and writes a ranked pool of real job postings — each with a match score (fit) and a likelihood score (realistic shot at an interview), both with plain-English reasoning and an honest gaps list.
2. **Tailor** takes one role from that pool and rewrites your CV for it: reordering, rewording with the posting's own language where it's honestly true, cutting irrelevant experience, and rewriting the summary — without ever adding a skill, tool, or achievement that isn't already on your CV.
3. **Reviewer** reads the tailored CV twice — once as an ATS parser checking keyword coverage and formatting, once as a skeptical hiring manager checking narrative, seniority, and red flags — and writes a critique. Tailor rewrites from that critique. This loops up to three times, and every round produces a plain-language change log plus a Word-openable visual redline, so you can see exactly what changed and why.

Two rules the whole system is built around:

- **Never invent a job.** Every role comes from a real search result with a link that opens.
- **Never invent experience.** Tailoring means reframing what's already on your CV — adding something you didn't do is the one unforgivable failure.

## Requirements

- [Claude Code](https://claude.com/claude-code) (Anthropic's CLI, VS Code/JetBrains extension, or desktop app)
- Your CV as a PDF, Word doc, or Markdown file
- (Optional) a second, differently-focused version of your CV, e.g. one leaning into a specific industry — see [Two-CV support](#two-cv-support) below

## Setup

1. Clone or download this repo:
   ```bash
   git clone <this-repo-url>
   cd job-system
   ```
2. Drop your CV and a short profile of what you're looking for into `material/`:
   - `material/cv.pdf` (or `.md` / `.doc` / `.docx`)
   - `material/profile.md` — plain English: target titles, industries, location/remote preferences, salary floor, notice period, anything you want ruled in or out
3. Open the folder in Claude Code. The three agents are already defined in `.claude/agents/` — Claude Code picks them up automatically.

## Usage

Talk to Claude Code in plain English from inside the project. A few ways to drive it:

**Find roles:**
> "Run the sourcer agent"

**Tailor your CV for one role** (role ids come from `output/job-pool.json`):
> "Run tailor for role id acme-principal-pm-data-platform"

**Get a critique, and loop:**
> "Run reviewer on that CV version, then have tailor address the critique"

**Run the whole pipeline in one go:**
> "Let's run all 3 agents for a full loop"

This last one triggers a short check-in first — Claude will ask:

1. Which CV variant(s) to run, each independently combined with whether to use your target-company priority list (see [Target company priority](#target-company-priority) below) — you can pick any one or several, and each runs in sequence.
2. How many tailor/review rounds to run — one pass, or the full 3-round loop.
3. Whether to auto-pick the top-ranked role after Sourcer runs, or stop and let you choose.

## Output layout

```
material/
  cv.pdf                      # your CV (pdf/md/doc/docx)
  cv_LS.pdf                   # optional second CV variant (see below)
  profile.md                  # what you're looking for
  target-companies.md         # optional: companies to prioritize (see below)

output/
  job-pool.json                # every role Sourcer has found, ranked and scored
  postings/[id].txt            # full posting text per role
  [role-id]/                   # everything Tailor + Reviewer produced for one role
    cv-v1.md, v2, v3             # successive tailored CV versions
    changes-v1.md, v2, v3        # plain-language change log per version, vs. the original CV
    changes-v1.rtf, v2, v3       # same thing as a Word-openable visual redline (strikethrough/underline)
    Changes_Final.rtf            # one consolidated redline: final version vs. original CV
    critique-v1.md, v2, v3       # Reviewer's ATS + hiring-manager critique per version
```

Every role id starts with a prefix set by how it was sourced — see [Job id prefix](#job-id-prefix).

## Two-CV support

If you're targeting more than one type of role — say, a generalist track and an industry-specific track — you can keep a second CV as `material/cv_LS.pdf` (or `.md`/`.doc`/`.docx`). Sourcer will ask which one to use when both are present, steer its search toward the industries that variant is written for, and every output file gets tagged accordingly (`cv_LS-v1.md`, `critique_LS-v1.md`, `Changes_LS_Final.rtf`, etc.) so the two tracks never collide.

## Target company priority

If there are specific companies you want prioritized, list them in `material/target-companies.md` — grouped into tiers, each with a fit level, target companies, target role/title patterns, and why they fit. Sourcer only uses this list on runs where you tell it to (see [Usage](#usage) above) — having the file present doesn't turn it on by itself. When it's used, Sourcer runs dedicated searches for those companies alongside its normal search and folds each tier's rationale into scoring for any role that matches.

## Job id prefix

Every role's id starts with a prefix set by that run's configuration — which CV file was used, and whether the target-company list was on:

- `cv.pdf` + target list → `X_P-`
- `cv.pdf`, no target list → `X_N-`
- `cv_LS.pdf` + target list → `LS_P-`
- `cv_LS.pdf`, no target list → `LS_N-`

E.g. a Databricks role from a `cv.pdf` + target-list run is `X_P-databricks-staff-pm-ai-platform`; a Natera role from a `cv_LS.pdf` run with the target list off is `LS_N-natera-director-pm-ai-precision-medicine`. The prefix carries through automatically to the posting file and the role's output folder — Tailor and Reviewer don't do anything special for it. In `job-pool.json`, `X_P-`/`LS_P-` roles (target list was on) sort as their own block above `X_N-`/`LS_N-` roles, each block ordered by match score.

This id prefix is unrelated to the `_LS` tag used in filenames for CVs built from the `cv_LS` variant (e.g. `cv_LS-v1.md`) — that's a separate, file-level convention tied to which CV Tailor/Reviewer build from, not to how the role was originally sourced.

## Customizing

The whole system is three markdown files:

- [`.claude/agents/sourcer.md`](.claude/agents/sourcer.md)
- [`.claude/agents/tailor.md`](.claude/agents/tailor.md)
- [`.claude/agents/reviewer.md`](.claude/agents/reviewer.md)

plus [`CLAUDE.md`](CLAUDE.md), which documents the file conventions and the ground rules all three agents follow. Edit these directly to change scoring criteria, search sources, tailoring style, or critique format — no code involved.
