# Code Review: Reputation/Title Dialogue Script

This review covers the supplied script blocks (`@main`, `@init`, `@varReset`, `@diagDataHandler`, `@btn1Handler`, `@btn2Handler`, etc.) and focuses on correctness, style, and maintainability.

> Engine assumption: `goto @label` uses call/return behavior (jump, execute, return to next instruction).

## High-risk logic findings

No unresolved high-risk logic issues remain after applying current author-note constraints.

## Confirmed assumptions / author notes

1. `L$title` is intentionally 0-based.
2. CSV table rows are intentionally 1-based.
3. Table-row to list-index conversion is explicit: `listIndex = tableRowIndex - 1`.
4. `@getRPQualifiedIndex` is intentionally implemented with `n$rawRPIndex` (internal baseline) and `n$rpIndex` (final qualified index).
5. In TeaJie engine, each `#if` controls following `#act` statements. Unconditional `#if` blocks in `@btn1Handler` are intentional scope resets.
6. In Special Ring engine, `n$` variables are default-initialized to `0`.
7. `n$btn1` is intentionally a tab selector enum (not a boolean active flag):
   - `n$btn1 = 0` renders Button 1 content (default view)
   - `n$btn1 = 1` renders Button 2 content
   This is intentional and lifecycle-consistent with engine conventions.
8. Dynamic variable names are intentionally used to simulate key-value storage in current engine constraints.

## Medium-risk runtime findings

1. **Hard-coded bounds (`1`, `18`, `19`, `20`)**  
   Logic depends on fixed title count.
   - **Action:** derive bounds from `n$titleCount`.

2. **Neighbor-index edge safety for `L$titleName`**  
   UI renders `[current-1, current, current+1]` and requires explicit clamping.
   - **Action:** clamp neighbor indices before table/list access.

3. **`@btn2Handler` depends on external reset ordering for `n$i`**  
   Handler behavior assumes `n$i` is already reset before entry.
   - **Action:** initialize `n$i` at `@btn2Handler` entry.

## Maintainability findings

1. Layout uses many literal coordinate values (`447`, `258`, `233`, `109`, etc.).
   - **Action:** move UI coordinates to named constants/config.

2. Tab/button setup logic is duplicated.
   - **Action:** consolidate through helper label or table-driven descriptor.

3. Variable naming is mixed and heavily abbreviated (`n$i`, `n$j`, `s$show`).
   - **Action:** use role-based naming for state and UI buffers where practical.

4. `@btn1Handler` includes multiple responsibilities.
   - **Action:** split into data-preparation and rendering labels.

## Refactor direction

1. Centralize rendering flow (`renderDialog(tabIndex, pageIndex)` pattern):
   - clear buffers,
   - render shared header/tab,
   - render tab body,
   - open dialog.
2. Use `n$titleCount`-derived bounds.
3. If engine capability allows in future, replace dynamic-name mapping with explicit map/list.
4. Add a boundary helper for previous/current/next title selection.

## Priority order

1. Initialize handler-local runtime variables at handler entry points.
2. Replace fixed bounds with data-driven bounds from `n$titleCount`.
3. Add inline comments for dynamic-name key-value simulation intent and required naming constraints.
4. Add inline comments where intentional TeaJie `#if` scope-reset pattern is used.
