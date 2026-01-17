
# Orchestration Runbook: Product Owner (PO) / Software Engineer (SE) Agile Loop

## Objective
Build a repo using:
- a **General Assistant (GA)** for repo comprehension + red-flag auditing
- a **Software Engineer (SE)** agent for implementation
- a **Product Owner (PO)** reviewer for strict PASS/FAIL gating

All development proceeds via **work orders**, not by free-form chat.

---

## Repo control files (must exist)
These files are the “operating system” of the repo:

- `AGENT_RULES.md` (canonical constraints, portable)
- `PO_SEED.md` (PO role prompt)
- `SE_SEED.md` (SE role prompt)
- `SPEC.md` (milestones 0–14, architecture)
- `STATE.md` (status + progress tracker, aligned with SPEC)
- `WORK_ORDERS/current.md` (active scope only)

Protected subsystem:
- `./broll/` must not be modified unless explicitly permitted in `WORK_ORDERS/current.md`

---

## Stage 1 — Context building (human ↔ GA)
**Goal:** Clarify requirements before touching code.

1. Open a GA chat in your tool of choice.
2. Discuss product intent and constraints until you are satisfied the plan is stable.
3. Freeze the outcome into repo files (Stage 2).

**Output:** Clear intent + stable constraints (no coding yet).

---

## Stage 2 — Create / update the control files (human)
**Goal:** Put the plan into versioned repo artifacts.

Ensure these exist and are correct:

- `AGENT_RULES.md`
- `PO_SEED.md`
- `SE_SEED.md`
- `SPEC.md`
- `STATE.md`
- `WORK_ORDERS/current.md`

Rule:
- `WORK_ORDERS/current.md` must describe **only the next 1–2 hours of work**.

**Output:** Repo is “agent-ready”.

---

## Stage 3 — GA repo audit (read-only)
**Goal:** Catch contradictions, missing constraints, and footguns early.

1. Set GA to **Ask/Chat mode** (read-only).
2. Give GA an audit prompt instructing it to:
   - read all control files
   - check internal consistency
   - flag BLOCKERS/MAJOR/MINOR
   - avoid opinionated suggestions

3. Apply only necessary fixes manually:
   - contradictions between `STATE.md` and `SPEC.md`
   - missing protection of `./broll/`
   - unclear acceptance tests
   - missing `WORK_ORDERS/current.md`

**Output:** Repo passes sanity audit.

---

## Stage 4 — Commit “scaffold state” (human)
**Goal:** Create a clean baseline checkpoint.

1. Create a new branch (recommended):
   - `attempt/001-bootstrap`
2. Commit all control files + any audit fixes.

Example commit message:
- `bootstrap: add repo control docs and work order loop`

**Output:** A stable, recoverable baseline.

---

## Stage 5 — SE initialization (Ask mode)
**Goal:** Ensure the SE understands the rules before touching code.

1. Open a dedicated SE chat thread:
   - Name: `SE — Implementer`
2. Set SE to **Ask/Chat mode** initially (no edits).
3. Prompt SE:

“Read `SE_SEED.md`, `AGENT_RULES.md`, `STATE.md`, `SPEC.md` (Milestone context only), and `WORK_ORDERS/current.md`.  
Confirm you understand:  
- you must implement ONLY `WORK_ORDERS/current.md`  
- no new deps without approval  
- do not touch `./broll/`  
Then wait.”

**Output:** SE confirms constraints in writing.

---

## Stage 6 — PO initialization (Ask mode)
**Goal:** Ensure PO uses strict PASS/FAIL and doesn’t drift into redesign.

1. Open a dedicated PO chat thread:
   - Name: `PO — Reviewer`
2. Set PO to **Ask/Chat mode** (read-only forever).
3. Prompt PO:

“Read `PO_SEED.md`, `AGENT_RULES.md`, `STATE.md`, `SPEC.md`, and `WORK_ORDERS/current.md`.  
Confirm your review output format: PASS/FAIL + BLOCKERS/MAJOR/MINOR + pasteable corrective prompt.”

**Output:** PO confirms review contract.

---

## Stage 7 — Enable SE implementation mode
**Goal:** Allow the SE to edit the repo only after constraints are acknowledged.

1. Switch SE from **Ask mode → Agent mode**.

**Output:** SE is now allowed to implement.

---

## Stage 8 — Execution loop (work order implementation + gating)
**Goal:** Complete one work order at a time with strict acceptance testing.

### 8.1 Run the current work order (SE)
Prompt SE:

“Implement `WORK_ORDERS/current.md` exactly.  
No future work. No new dependencies unless allowed. Do not touch `./broll/`.  
Return the full DoD packet and include the exact commands you ran.”

SE output must include:
- files changed
- commands to run
- expected outputs
- tests run
- assumptions / limitations

### 8.2 Review against the work order (PO)
Paste the SE DoD packet into the PO chat and prompt:

“Review against `WORK_ORDERS/current.md`.  
Output PASS or FAIL.  
If FAIL: list BLOCKERS/MAJOR/MINOR and end with a pasteable corrective prompt for SE.”

### 8.3 Correction loop (only if FAIL)
If PO verdict = **FAIL**:

1. Paste PO’s corrective prompt back into SE, prefixed with:

“PO verdict = FAIL. Implement ONLY the required fixes below.  
Do not do anything else. Re-run acceptance tests. Return updated DoD packet.”

2. SE fixes.
3. PO re-reviews.
4. Repeat until PO verdict = **PASS**.

**Stop rule:** No new work order is started until PASS.

### 8.4 Commit + archive (only if PASS)
Once PO verdict = **PASS**:

1. Human sanity-checks quickly:
   - no scope creep
   - no `./broll/` edits
   - acceptance tests actually run

2. Commit changes with a milestone-aligned message.

Example:
- `M0-A: tm validate EpisodeSpec + examples + tests`

3. Archive the completed work order:
- move `WORK_ORDERS/current.md` → `WORK_ORDERS/history/<timestamp>_M0-A.md`

4. Create the next `WORK_ORDERS/current.md`.
5. Repeat Stage 8.

**Output:** One completed milestone slice, committed and checkpointed.

---

## Stage 9 — Progress tracking (human)
**Goal:** Keep state consistent and visible.

After each PASS:
- update `STATE.md` milestone checklist
- optionally add a tag:
  - `git tag m0a-pass`

**Output:** Clear milestone progress and rollback points.

---

## Stage 10 — Rollback protocol (when things go sideways)
**Goal:** Escape bad agent runs quickly.

If SE drifted or broke things:
- reset branch to last known good commit OR delete attempt branch and restart

Recommended:
- keep clean attempt branches:
  - `attempt/00X-*`
- merge only PASSed work into `main`

---

## Operating principles (do not violate)
- Work order defines scope. Not chat.
- PO is strict reviewer. Not a brainstormer.
- SE implements. Not a product designer.
- Human is the final gatekeeper.
- Small vertical slices beat “big intelligent systems”.

---

## Minimal command summary
Typical cycle commands after SE implementation:
- `tm --help`
- `tm validate examples/episode1.yaml`
- `tm validate examples/episode2.yaml`
- `pytest -q`



