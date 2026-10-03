# PROMPT FOR THE INTEGRATION AI
# (paste everything below the line into the AI working on the repo)

---

## Mission: ship the 1,191-question MCQ bank into Know Your Pill

**Repo:** `zammucanva/know-your-pill-2026` (Next.js static export → GitHub Pages).
**Branch to integrate:** `feat/mcq-bank-1191` — delivered as a patch file `kyp-mcq-bank-1191.patch` (apply instructions below). You HAVE push access (you already pushed `feature/drug-audience-mode`), so you can push this branch too.

### GETTING THE BRANCH (run first)
```bash
git -C <your-clone> fetch origin main && git -C <your-clone> checkout -b feat/mcq-bank-1191 origin/main
git -C <your-clone> am /path/to/kyp-mcq-bank-1191.patch
git -C <your-clone> push -u origin feat/mcq-bank-1191
```
If `git am` hits a conflict on `worklog.md` (your Phase 0 entry): `git checkout --theirs worklog.md` is NOT wanted — instead keep BOTH logs: resolve by appending, then `git am --continue`.

### FIRST STEP AFTER APPLY
Read `data/mcq/HANDOFF.md` on the branch. Single source of truth: architecture map, numbers, verification checklist, hard rules.

### What is already done in the commit (do NOT redo)
- `data/mcq/` — 18 QA-passed JSON batches (1,191 questions, all 145 drug monographs) + master index + plan + merge report + `mixed-unassigned.json` (18 class-summary questions parked).
- **1,173 questions already appended** to `microQuizzes` in all 145 `src/lib/kyp/data/drugs/<slug>.ts` files — exactly the 6-field MicroQuiz schema, idempotent by id; the 471 pre-existing `quiz-*` items untouched; zero id collisions.
- Scripts in `scripts/mcq-bank/`: `merge-mcq-bank.py` (idempotent), `quiz-import-check.ts`, `mcq_4step_check.py` (QA gate for future batches).
- Already verified in the source environment: `bunx tsc --noEmit` = clean; runtime import check = clean (sertraline 18 / alprazolam 11 / lithium 11 / clozapine 11 / fluvoxamine 14 quizzes).

### What automatically works with zero UI changes
`src/lib/kyp/custom-test/engine.ts` (`authoredQuestions()`) harvests every `microQuizzes` entry → Custom Test authored pool grows 471 → **1,644** from the data alone.

### Your job, in order
1. **Verify** — HANDOFF.md §6 checklist: `bunx tsc --noEmit` → `bun run scripts/mcq-bank/quiz-import-check.ts` → `bun run build` → re-run `python3 scripts/mcq-bank/merge-mcq-bank.py` (must report `inserted=0, skipped=1173`) → browser spot-checks: `/drugs/sertraline` (old inline quizzes still render), `/quiz` Custom Test with Sertraline (authored ≈ 18), `/drugs/class/ssri` (aggregate counts updated).
2. **Ship** — open PR `feat/mcq-bank-1191` → `main`; merge when CI/build is green (GitHub Pages workflow redeploys).
3. **Decide the 18 `mixed` questions** (HANDOFF §5): recommended = Custom Test "class summary" mixed deck; alternative = rotated summary question on each `/drugs/class/<classId>` page.
4. **Phase 2 — inline rendering expansion (only if explicitly requested).** `src/app/drugs/[slug]/page.tsx` renders inline quizzes ONLY for `QUIZ_RENDER_SECTIONS = {mechanism, timeline, side-effects, monitoring, contraindications, evidence-practice}` and only ONE per section (`.find()`). To surface more: `.find()` → `.filter()`, extend whitelist with the bank's real anchors (`quick-facts`, `high-yield-summary`, `knowledge-graph`, `neural-pathways`, `pathways`, `neurotransmitters`, `brain-regions`, `brain`, `top`) AFTER verifying each anchor id exists in rendered HTML, update `renderedQuizCount`, cap ~4 quizzes per section with a "show more" expander.

### Hard rules (breaking any = rejected PR)
- Never renumber/reword bank question ids (idempotency depends on them).
- Never add `drugSlug`/`domain`/`difficulty`/`style` to MicroQuiz objects — the TS type has exactly 6 fields; the engine derives its own difficulty. Metadata stays in `data/mcq/*.json`.
- New question batches must pass `scripts/mcq-bank/mcq_4step_check.py` before entering `data/mcq/`.
- If a question looks wrong: cross-check the in-repo monograph + Katzung 14e / K.D. Tripathi 7e, fix with a note — don't silently delete.
