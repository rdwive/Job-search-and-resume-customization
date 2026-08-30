# Job System

Three agents, handing work to each other through files. Agents never talk. They read files and write files.

1. **Sourcer** — reads CV and profile, searches the web, writes a ranked pool of real roles.
2. **Tailor** — takes one role, rewrites the CV for it.
3. **Reviewer** — judges a tailored CV as ATS and as hiring manager, writes a critique. Tailor rewrites from it. Loops up to 3 times.

## Running a full loop

When I ask to run a full loop of all 3 agents, ask me first, before doing anything:

1. Which of these do you want to run — pick any one, or more than one:
   - `cv.pdf`, using the target-company priority list
   - `cv.pdf`, without the target-company priority list
   - `cv_LS.pdf`, using the target-company priority list
   - `cv_LS.pdf`, without the target-company priority list

   Run each one picked, one after another.
2. Run just one cycle between Tailor and Reviewer and then stop, or run all 3 iterations?
3. After Sourcer runs, pick the top-fit job automatically, or stop and ask me to choose?

Don't proceed until all three are answered.

## Target company priority

`material/target-companies.md` (optional) lists companies I specifically want prioritized. Sourcer only uses it when told to for that run (see "Running a full loop" above — using the list is a per-run choice, not automatic just because the file exists). When told to use it, Sourcer searches for these companies explicitly, folding each tier's rationale into the scoring for any role that matches. The target list doesn't override the `cv`/`cv_LS` industry split — a target company only gets searched if it's already in scope for that run (Life Science-side targets surface on a `cv_LS` run, others on a `cv` run).

## Job id prefix

Every role's id starts with a prefix set by that run's configuration — which CV file was used, and whether the target-company list was on. Sourcer applies it to every role from that run, not just target-company matches:

- `cv.pdf` + target list → `X_P-`
- `cv.pdf`, no target list → `X_N-`
- `cv_LS.pdf` + target list → `LS_P-`
- `cv_LS.pdf`, no target list → `LS_N-`

E.g. a Databricks role from a `cv.pdf` + target-list run is `X_P-databricks-staff-pm-ai-platform`; a Natera role from a `cv_LS.pdf` run with the target list off is `LS_N-natera-director-pm-ai-precision-medicine`. The prefix carries through automatically to the role's posting file and its `output/[id]/` folder, and every file inside it — Tailor and Reviewer don't do anything special for it, they just use the id they're given.

In `output/job-pool.json`, `X_P-`/`LS_P-` roles (target list was on) sort as their own block above `X_N-`/`LS_N-` roles, each block ordered by match score.

This id prefix is independent of the `_LS` tag used elsewhere for files built from the `cv_LS` CV variant — the file tag depends on which CV Tailor/Reviewer are told to build from for a given tailoring pass, not on how the role was originally sourced.

## Files

- `material/` — `cv.pdf`/`cv.md`/`cv.doc`/`cv.docx`, optionally also `cv_LS.pdf`/`cv_LS.md`/`cv_LS.doc`/`cv_LS.docx`, and `profile.md`. Written by me. If more than one CV file is present, agents ask which one to use before proceeding.
- `material/target-companies.md` — optional. Companies I specifically want prioritized, grouped with the kind of role to look for at each. Sourcer searches for these explicitly when told to for a run — see "Target company priority" and "Job id prefix" above.
- `output/postings/[id].txt` — full posting text, one file per role. Written by Sourcer.
- `output/job-pool.json` — id, title, company, location, remote, salary, source, link, posted date, match score /100, likelihood score /100, reasoning for each, gaps list. Two-line summary only, never full posting text.
- `output/[id]/` — one folder per role id, holding Tailor's and Reviewer's output for that role while the loop is running:
  - `cv-v1.md`, `v2`, `v3` — tailor output. New number each pass, never overwrite. Named `cv_LS-vN.md` instead if the rewrite was built from the `cv_LS` CV variant.
  - `changes-v1.md`, `v2`, `v3` — tailor's change log for that CV version, showing its changes against the original CV in `material/`. Written every pass, including v1. Number matches the CV version it documents. `changes_LS-vN.md` for the `cv_LS` variant.
  - `critique-v1.md`, `v2`, `v3` — reviewer output. Number and variant both match the CV version it judges (`critique_LS-v2.md` judges `cv_LS-v2.md`).
  - No RTF is written per pass — see below.

  **Only one RTF is ever produced per role, on the final pass** — when the tailor/reviewer loop concludes (3-loop cap reached, or a version accepted earlier), whichever version number that turns out to be:
  - `Changes_Final.rtf` (or `Changes_LS_Final.rtf`) — a single, consolidated Word-openable redline (strikethrough for cuts, underline for reworded/added text) comparing the final accepted CV version directly against the original CV in `material/` — the whole journey in one document. E.g. if the candidate accepts the v2 rework and a v3 pass is never needed, `Changes_Final.rtf` is built directly from `cv-v2.md`, not `cv-v3.md`. The role's job posting link (from `output/job-pool.json`) is included as plain text at the very top of the file, before the candidate's name.
  - **Folder cleanup:** once `Changes_Final.rtf` is written, every earlier `cv-vN.md`, every `changes-vN.md` (including the one matching the final version), and every `critique-vN.md` are deleted. The folder ends up holding just two files for that CV variant: the final `cv-vN.md`/`cv_LS-vN.md` and `Changes_Final.rtf`/`Changes_LS_Final.rtf`. Tailor's tools don't include delete access, so Tailor reports which files are superseded and whoever is running the loop deletes them.

## Two rules, no exceptions

- **Never invent a job.** Every role comes from a search result with a link that opens. Nothing found means say so, not fill the gap.
- **Never invent experience.** Tailoring is reframing what is on my CV. Adding a skill I do not have is the one unforgivable failure here.

## How to work with me

**1. Think before coding.**
State your assumptions before acting on them. If my request has two readings that would build different things, show me both rather than picking one. If a simpler approach exists, say so and push back. If something is unclear, stop and name what is confusing.

**2. Simplicity first.**
The minimum that solves the problem. Nothing speculative. No features I did not ask for, no abstraction for something used once, no configurability I did not request, no error handling for things that cannot happen. If you wrote 200 lines and it could be 50, rewrite it.

**3. Surgical changes.**
Touch only what I asked about. Do not improve nearby code, comments or formatting. Do not refactor what is not broken. Match the style already there. If you spot unrelated dead code, mention it, do not delete it. Clean up only the mess your own change made. Every changed line should trace back to something I asked for.

**4. Goal-driven execution.**
Turn my request into something checkable before you start. For anything with more than one step, give me the plan as steps with a verification against each:
```
1. [step] → verify: [check]
2. [step] → verify: [check]
```
Never say done without telling me the command or check that proves it.

## Me

Not a software engineer. Plain English, gloss jargon on first use.
