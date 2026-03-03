# Code Review: Reputation/Title Dialogue Script

This review covers the supplied script blocks (`@main`, `@init`, `@varReset`, `@diagDataHandler`, `@btn1Handler`, `@btn2Handler`, etc.) and focuses on correctness, style, and maintainability.

> Note: this review assumes the clarified engine semantics where `goto @label` behaves like a subroutine call (jump to label, execute it, then return and continue after the `goto`).

## High-risk logic bugs

1. **`n$btn1` semantics are inverted/ambiguous**  
   In `@diagDataHandler`, condition `equal n$btn1 0` routes to `@btn1Handler`. But `@btn1` sets `n$btn1 0`, `@btn2` sets `n$btn1 1`. This makes button-state naming confusing and easy to misuse.
   - **Fix:** rename state vars to explicit meaning (e.g., `n$activeTab`) and compare against tab constants.

2. **Possible unsafe dynamic variable name construction**  
   `mov n$<$str(s$titleName)> <$str(n$j)>` creates variables from CSV field values. If source data contains unexpected characters/collisions, this can overwrite unrelated vars.
   - **Fix:** use a dedicated map/list structure rather than dynamic global variable names.


## Confirmed assumptions from follow-up

1. `L$title` is intentionally 0-based.
2. CSV table rows are intentionally 1-based.
3. When crossing from table-row index to list index, explicit conversion is applied (`listIndex = tableRowIndex - 1`), so this mapping is treated as correct and not flagged as an off-by-one defect.
4. Author note: `@getRPQualifiedIndex` is intended to keep outer loop behavior stable via `n$rawRPIndex` (internal baseline) and `n$rpIndex` (final qualified index), so this is not treated as a loop-mutation defect in this review.
5. Engine note: in TeJie, each `#if` controls all following `#act` statements, and unconditional `#if` blocks in `@btn1Handler` are intentionally used to reset control scope so subsequent actions execute in the intended sequence.

## Medium-risk bugs / runtime pitfalls

1. **Hard-coded clamp boundaries (`1`, `18`, `19`, `20`)**  
   Logic assumes a fixed title count; if CSV changes, arrows/page behavior can break.
   - **Fix:** compute boundaries from `n$titleCount`.

2. **Potential out-of-range `L$titleName` access around edges**  
   The UI renders `[current-1, current, current+1]`. Near first/last indices, this can point outside valid IDs unless clamped before list construction.
   - **Fix:** clamp computed neighbor indices before reading table/title resources.

3. **`@btn2Handler` depends on `n$i` reset externally**  
   The loop in `@btn2Handler` relies on `n$i` being 0 at entry; currently this comes indirectly via `@diagDataHandler`/`@varReset`.
   - **Fix:** initialize `n$i` at the start of `@btn2Handler` for local correctness.

## Style and maintainability issues

1. **Magic numbers for coordinates and layout** (`447`, `258`, `233`, `109`, etc.)
   - **Improve:** extract constants at top-level config section.

2. **Duplicate tab/button initialization code**
   - **Improve:** factor into helper label(s) or table-driven tab descriptor.

3. **Mixed Chinese/English naming with overloaded short vars (`n$i`, `n$j`, `s$show`)**
   - **Improve:** adopt naming convention by role (`idxTitle`, `uiShowBuffer`, etc.).

4. **Large monolithic handlers** (`@btn1Handler`)
   - **Improve:** split into pure data-prepare labels and pure UI-render labels.

## Suggested refactor direction

1. Introduce a single `n$activeTab` with values `{1,2,3}`.
2. Centralize `renderDialog(activeTab, pageIndex)` path:
   - clear display buffers
   - render common header/tab
   - render tab-specific body
   - open dialog
3. Replace hard-coded limits with `n$titleCount` derived values.
4. Build safe lookup map for title name → id using list records, not dynamic var names.
5. Add boundary-safe helper for previous/current/next title calculation.

## Priority fixes first

1. Replace ambiguous button flags with `n$activeTab`.
2. Normalize handler-local initialization so each handler does not rely on external reset ordering.
3. Replace hard-coded bounds with data-driven limits derived from `n$titleCount`.
4. Add short inline comments documenting intentional `#if` scope-reset blocks in `@btn1Handler` to reduce maintenance confusion.
