# Code Review: Reputation/Title Dialogue Script

This review covers the supplied script blocks (`@main`, `@init`, `@varReset`, `@diagDataHandler`, `@btn1Handler`, `@btn2Handler`, etc.) and focuses on correctness, style, and maintainability.

> Engine assumption: `goto @label` uses call/return behavior (jump, execute, return to next instruction).

## High-risk logic findings

1. **`n$btn1` state naming does not match branch usage**  
   In `@diagDataHandler`, `equal n$btn1 0` routes to `@btn1Handler`, while `@btn1` writes `n$btn1 0` and `@btn2` writes `n$btn1 1`.
   - **Action:** replace split flags with one explicit tab state (for example `n$activeTab`), and branch by tab constant.

2. **Dynamic variable-name construction can cause name collision risk**  
   `mov n$<$str(s$titleName)> <$str(n$j)>` creates runtime variable names from CSV values.
   - **Action:** replace dynamic variable names with explicit list/map storage.

## Confirmed assumptions / author notes

1. `L$title` is intentionally 0-based.
2. CSV table rows are intentionally 1-based.
3. Table-row to list-index conversion is explicit: `listIndex = tableRowIndex - 1`.
4. `@getRPQualifiedIndex` is intentionally implemented with `n$rawRPIndex` (internal baseline) and `n$rpIndex` (final qualified index).
5. In TeaJie engine, each `#if` controls following `#act` statements. Unconditional `#if` blocks in `@btn1Handler` are intentional scope resets.

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
   - **Action:** use role-based naming for state and UI buffers.

4. `@btn1Handler` includes multiple responsibilities.
   - **Action:** split into data-preparation and rendering labels.

## Refactor direction

1. Use a single `n$activeTab` state.
2. Centralize rendering flow (`renderDialog(activeTab, pageIndex)` pattern):
   - clear buffers,
   - render shared header/tab,
   - render tab body,
   - open dialog.
3. Use `n$titleCount`-derived bounds.
4. Replace dynamic-name variable mapping with explicit map/list.
5. Add a boundary helper for previous/current/next title selection.

## Priority order

1. Replace button-state flags with `n$activeTab`.
2. Initialize handler-local runtime variables at handler entry points.
3. Replace fixed bounds with data-driven bounds from `n$titleCount`.
4. Add inline comments where intentional TeaJie `#if` scope-reset pattern is used.
