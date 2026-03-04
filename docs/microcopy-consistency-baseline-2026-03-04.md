# Microcopy Consistency Baseline (2026-03-04)

## Scope
Initial baseline for Mission Control UI copy consistency, with focus on task workflows and operational clarity.

Reviewed areas:
- `src/components/panels/task-board-panel.tsx`

## Baseline decisions

### 1) Case and label style
- Use **Title Case** for user-facing status and priority labels in UI surfaces.
- Keep machine values (`in_progress`, `quality_review`, `approved`) unchanged for API/storage.

### 2) Terminology normalization
- Status labels:
  - `inbox` → **Inbox**
  - `assigned` → **Assigned**
  - `in_progress` → **In Progress**
  - `review` → **Review**
  - `quality_review` → **Quality Review**
  - `done` → **Done**
- Priority labels:
  - `low` → **Low**
  - `medium` → **Medium**
  - `high` → **High**
  - `critical` → **Critical**
  - `urgent` → **Urgent**

### 3) Accessibility copy
- ARIA labels should use human-readable labels (not raw enum values).

## Implemented now

In `task-board-panel.tsx`:
1. Added centralized mapping dictionaries:
   - `statusLabel`
   - `priorityLabel`
2. Replaced raw enum display in task cards/detail view with mapped labels.
3. Updated task card ARIA label to use normalized copy.
4. Updated quality review dropdown labels from lowercase to Title Case:
   - `approved`/`rejected` → `Approved`/`Rejected`

## Known remaining opportunities (next pass)
- Expand baseline to other panels for consistent action verbs:
  - prefer explicit verbs like **Save**, **Retry**, **Refresh**, **Broadcast**, **Submit review**
- Normalize empty states globally:
  - pattern: `No <resource> yet.`
- Standardize success/error feedback tone:
  - concise, action-oriented, and blame-free
- Add a shared utility for enum-to-label transformation to avoid repeated mappings.

## Proposed follow-up deliverables
1. `docs/microcopy-style-guide.md` (single source of truth)
2. `src/lib/microcopy.ts` (central labels + common UI strings)
3. Sweep all panels to align CTA and empty-state wording
