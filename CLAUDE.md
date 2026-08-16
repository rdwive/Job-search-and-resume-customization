# Job System

Three agents, handing work to each other through files. Agents never talk. They read files and write files.

1. **Sourcer** — reads CV and profile, searches the web, writes a ranked pool of real roles.
2. **Tailor** — takes one role, rewrites the CV for it.
3. **Reviewer** — judges a tailored CV as ATS and as hiring manager, writes a critique. Tailor rewrites from it. Loops up to 3 times.

## Running a full loop

When I ask to run a full loop of all 3 agents, ask me first, before doing anything:

1. Run for `cv` and `cv_LS` both, one after another, or just one of the two? If just one, ask which one.
2. Run just one cycle between Tailor and Reviewer and then stop, or run all 3 iterations?
3. After Sourcer runs, pick the top-fit job automatically, or stop and ask me to choose?

Don't proceed until all three are answered.

## Files

- `material/` — `cv.pdf`/`cv.md`/`cv.doc`/`cv.docx`, optionally also `cv_LS.pdf`/`cv_LS.md`/`cv_LS.doc`/`cv_LS.docx`, and `profile.md`. Written by me. If more than one CV file is present, agents ask which one to use before proceeding.
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
