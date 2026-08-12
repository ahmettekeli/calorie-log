## Project
Calorie Deficit Tracker — Numbers-first deficit ledger & weekly weight-loss tracker.
Stack: Next.js (App Router), TypeScript, Tailwind CSS, shadcn/ui, Supabase (Postgres/Auth), Vitest, Playwright.

## Commands
- Install: `npm install`
- Dev/Start: `npm run dev`
- Build: `npm run build`
- Test (all / single file): `npm run test` / `npx vitest run path/to/file.test.ts`
- E2E Test: `npx playwright test`
- Lint / format: `npm run lint` / `npm run format`

## Conventions
- Domain logic MUST live in pure, side-effect-free functions separate from UI/DB (`computeWeeklyTotals()`, `computeBurnRange()`, `computeMilestoneRange()`, `checkPlanAgainstGoal()`).
- Raw entry logs are immutable. Derived metrics (targets, estimates, predictions) must always be computed dynamically, never hardcoded or stored as final state in DB tables.
- Zero magic numbers in components. Put constants like `7700` (kcal/kg constant) or burn-uncertainty ranges (`0.20`, `0.30`) in `src/lib/constants.ts` or read from the `settings` table.
- Wearable exercise burns are range estimates; always render exercise actuals and net weekly deficits as low/high ranges in reports and milestone predictions.

## Behavioral rules
- Small/scoped tasks: just implement. Don't add process overhead.
- Ambiguous or multi-interpretation tasks: state the ambiguity in one line, don't guess silently.
- Multi-step or ambiguous-scope tasks only: state a brief plan with a verify step per item before starting.
- Minimum code that solves the problem — no speculative abstractions, config, or error handling for cases not in scope.
- Touch only what the task requires. Don't refactor, reformat, or "improve" adjacent code. Remove only imports/vars your own change made unused.

## Do NOT
- Scan node_modules/, dist/, .git/, lockfiles
- Read the whole repo for small/scoped tasks, ask which files are relevant instead
- Paste full logs, extract only the relevant lines instead
- Make repo-wide refactors without proposing a plan first

## Source of truth
- Data Model & Scope: `deficit-tracker-build-plan.md`
- DB Schema / Client: `src/lib/supabase/`
- Constants: `src/lib/constants.ts`
- Env vars: `.env.local` , `.env.production`

## Session hygiene
- Prefer small, scoped sessions over one long running session
- Summarize before continuing a long session
- Keep explanations and diffs concise; don't restate unchanged code