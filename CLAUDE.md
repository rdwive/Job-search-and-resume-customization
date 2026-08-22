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

`material/target-companies.md` (optional) lists companies I specifically want prioritized. Sourcer only uses it when told to for that run (see "Running a full loop" above — using the list is a per-run choice, not automatic just because the file exists). When told to use it, Sourcer searches for these companies explicitly and tags a matching role's id with a `P-` prefix (e.g. `P-roche-principal-pm-diagnostics-platform`). That prefix carries through automatically to its posting file, its `output/[id]/` folder, and every file inside it — Tailor and Reviewer don't do anything special for it, they just use the id they're given.

The target list doesn't override the `cv`/`cv_LS` industry split — a target company only gets searched if it's already in scope for that run (Life Science-side targets surface on a `cv_LS` run, others on a `cv` run). In `output/job-pool.json`, `P-` roles sort as their own block above all other roles, each block ordered by match score.

Separately: every role found on a `cv_LS` run gets a trailing `-LS` suffix on its id (e.g. `natera-director-pm-ai-precision-medicine-LS`), since a `cv_LS` run only ever searches Life Science, Medical Device, Healthcare, Diagnostics, Biotechnology, and Pharmaceutical industries to begin with — the suffix just marks that. A `cv` run never produces it. This combines independently with `P-` across the four run types:

- `cv.pdf` + target list → `P-` prefix on matches only
- `cv.pdf`, no target list → neither
- `cv_LS.pdf` + target list → `-LS` on every role, plus `P-` on matches (e.g. `P-roche-principal-pm-diagnostics-platform-LS`)
- `cv_LS.pdf`, no target list → `-LS` on every role, never `P-`

This id-level `-LS` suffix is independent of the `_LS` tag used elsewhere for files built from the `cv_LS` CV variant — a role can carry the id suffix, the file tag, both, or neither (the file tag depends on which CV Tailor/Reviewer are told to build from for a given tailoring pass, not on how the role was originally sourced).

## Files

- `material/` — `cv.pdf`/`cv.md`/`cv.doc`/`cv.docx`, optionally also `cv_LS.pdf`/`cv_LS.md`/`cv_LS.doc`/`cv_LS.docx`, and `profile.md`. Written by me. If more than one CV file is present, agents ask which one to use before proceeding.
- `material/target-companies.md` — optional. Companies I specifically want prioritized, grouped with the kind of role to look for at each. Sourcer searches for these explicitly and marks matches — see "Target company priority" below.
- `output/postings/[id].txt` — full posting text, one file per role. Written by Sourcer.
- `output/job-pool.json` — id, title, company, location, remote, salary, source, link, posted date, match score /100, likelihood score /100, reasoning for each, gaps list. Two-line summary only, never full posting text.
- `output/[id]/` — one folder per role id, holding all of Tailor's and Reviewer's output for that role:
  - `cv-v1.md`, `v2`, `v3` — tailor output. New number each pass, never overwrite. Named `cv_LS-vN.md` instead if the rewrite was built from the `cv_LS` CV variant.
  - `changes-v1.md`, `v2`, `v3` — tailor's change log for that CV version, showing its changes against the original CV in `material/`. Written every pass, including v1. Number matches the CV version it documents. `changes_LS-vN.md` for the `cv_LS` variant.
  - `changes-v1.rtf`, `v2`, `v3` — same pass, same number, but a Word-openable visual redline of the CV text itself (strikethrough for cuts, underline for reworded/added text) rather than a prose log. `changes_LS-vN.rtf` for the `cv_LS` variant.
  - `Changes_Final.rtf` — written once, when the tailor/reviewer loop concludes for this role (3-loop cap reached, or accepted earlier). A single consolidated Word-openable redline comparing the final CV version directly against the original CV in `material/`, not just the last pass's increment. `Changes_LS_Final.rtf` for the `cv_LS` variant.
  - `critique-v1.md`, `v2`, `v3` — reviewer output. Number and variant both match the CV version it judges (`critique_LS-v2.md` judges `cv_LS-v2.md`).

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
