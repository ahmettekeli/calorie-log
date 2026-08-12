# Build Plan: Calorie Log

## 1. Core Domain Model & System Intent

**System Description:**  
Calorie Log is a numbers-first deficit ledger and running tally app. It is NOT a macro tracker or food diary — there is no food database or meal-logging workflow. 

The system operates on two independent, daily target-vs-actual tracking variables measured against a fixed maintenance calorie baseline:
- **Diet deficit**: diet_target vs. diet_actual
- **Exercise burn**: exercise_target vs. exercise_actual

Daily totals roll up into a weekly balance compared against maintenance_kcal (e.g., 2400 kcal/day default). 

---

## 2. System Scope (v0 MVP)

### In-Scope Features
- **Goal Setup:** User defines target weekly weight loss rate (kg or lb/week). System calculates required weekly deficit (7700 kcal/kg or 3500 kcal/lb) and evaluates daily target plans against this required deficit.
- **Maintenance Baseline Configuration:** Editable maintenance_kcal setting.
- **Daily Entry Ledger:** Tracking for diet_target, diet_actual, exercise_target, and exercise_actual.
- **Weekly Rollup Computation:** Dynamic calculation of weekly totals (target_total vs. actual_total).
- **Weekly Ledger View:** Tabular display mirroring structured paper log layouts.
- **Optional Weight Log:** Nullable weight tracking on any given day. When present, displays weekly moving averages instead of raw daily fluctuations.
- **Weekly Report & Burn Range:** Visual comparisons of diet vs. exercise and target vs. actual. Applies configurable uncertainty bounds on exercise actuals.
- **Milestone Date-Range Prediction:** Estimated target date range computed strictly when weight logs are available, factoring in combined burn uncertainty and physiological variance.
- **Authentication:** Single-user database auth.

### Out-of-Scope (v0 MVP)
- Food databases or barcode scanning
- Macro tracking (carbs, protein, fat)
- Multi-user accounts or social sharing
- Native mobile applications
- Auto-recalibrating adaptive target engines

---

## 3. Data Model & Technical Architecture

### Target Definition Logic
- **Loss Rate Formula:**  
  Required Weekly Deficit = Weekly Loss Rate (kg) * 7700 kcal
- **Cycling Modes:**
  - steady: Identical diet and exercise targets applied daily.
  - cycled: Separate target profiles for training days (training_day = true) vs. rest days (training_day = false).
- **Validation Rule:** The application must calculate total weekly planned deficit from active targets and display a visual warning if planned deficit does not match required deficit for the chosen goal.

### Database Schema (Supabase / Postgres)

- **settings**
  - id: uuid (PK)
  - maintenance_kcal: integer (default: 2400)
  - weekly_loss_rate: numeric
  - weight_unit: text ('kg' | 'lb')
  - cycling_mode: text ('steady' | 'cycled')
  - burn_uncertainty_low_pct: numeric (default: 0.20)
  - burn_uncertainty_high_pct: numeric (default: 0.30)

- **day_targets**
  - id: uuid (PK)
  - is_training_day: boolean (null if mode is 'steady')
  - diet_target: integer
  - exercise_target: integer

- **entries**
  - id: uuid (PK)
  - date: date (unique constraint)
  - diet_actual: integer
  - exercise_actual: integer
  - training_day: boolean

- **weight_entries**
  - id: uuid (PK)
  - date: date (unique constraint)
  - weight: numeric

### Uncertainty Handling Specifications
Wearable exercise burn metrics contain inherent estimation error. 
- Low/high uncertainty percentage bounds MUST be pulled dynamically from settings (burn_uncertainty_low_pct, burn_uncertainty_high_pct).
- Net weekly deficit MUST be rendered as a range calculation:
  Net Deficit (Low) = Diet Actual Deficit Total + (Exercise Actual Total * (1 - burn_uncertainty_high_pct))
  Net Deficit (High) = Diet Actual Deficit Total + (Exercise Actual Total * (1 - burn_uncertainty_low_pct))

### Domain Boundary Rules for Code Generation
1. **Pure Functional Domain Core:** Domain logic (computeWeeklyTotals, computeBurnRange, computeMilestoneRange, checkPlanAgainstGoal) must be implemented as side-effect-free pure functions isolated from React components and Database drivers.
2. **Immutable Entries & Dynamic Aggregations:** Daily raw log entries are strictly immutable. Target totals, deficit estimates, and predictions must always be computed on the fly.
3. **No Component Magic Numbers:** Constants (e.g., 7700 kcal/kg, default burn bounds) must reside in src/lib/constants.ts or settings.

---

## 4. Agent Team Roles & Operational Rules

| Role | Subagent Profile File | System Responsibilities |
|---|---|---|
| **Product Strategist** | .claude/agents/product-manager.md | Translates user requirements into issue specs, acceptance criteria, and UI wireframe descriptions. Operates fully offline without web search tools. |
| **Full-Stack Engineer** | .claude/agents/nextjs-developer.md | Executes Next.js (App Router), TypeScript, Tailwind CSS, shadcn/ui, and Supabase integration. Enforces Section 3 domain boundaries. |
| **QA/Test Engineer** | .claude/agents/qa-expert.md | Author and run unit tests (Vitest) and E2E tests (Playwright). Outputs PASS/FAIL evaluation comments to PRs. |

---

## 5. Technology Stack Specifications

- **Framework:** Next.js (App Router), TypeScript
- **Styling & UI:** Tailwind CSS, shadcn/ui
- **Database & Auth:** Supabase Postgres, Supabase Auth
- **Visualization:** Recharts / Tremor
- **Test Suite:** Vitest (unit/domain), Playwright (E2E)

---

## 6. Execution Order & Implementation Milestones

1. **Database & Schema Initialization:** Execute migrations for settings, day_targets, entries, and weight_entries.
2. **Authentication Setup:** Implement single-user Supabase auth guards.
3. **Goal Setup Engine:** Build settings interface for weekly loss rate, target deficit validation, and cycled/steady mode allocation.
4. **Daily Ledger Entry Component:** Form for logging daily diet_actual, exercise_actual, and optional daily weight.
5. **Weekly Ledger Table:** Implement tabular overview matching target-vs-actual rollup specs.
6. **Reports & Range Renderer:** Construct reports visualising exercise burn ranges and net weekly deficit bounds.
7. **Weight Trend Engine:** Compute moving-average weight trends when weight_entries exist.
8. **Milestone Date Range Estimator:** Generate predicted date ranges based on cumulative weight trend and deficit range computations.