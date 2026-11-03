# F1: Clickable Legend Card Filtering
<!-- koolade-artifact-id:v1 {"uid":"afb3553e-2eec-410f-9f68-8e7b6ff364a7","displayId":"F1","title":"Clickable Legend Card Filtering"} -->
<!-- koolade-change:v1 {"schemaVersion":1,"uid":"afb3553e-2eec-410f-9f68-8e7b6ff364a7","displayId":"F1","status":"draft"} -->

**Status:** Draft

## Intent
Board operators triaging the Kanban face mixed card types side by side with no way to visually isolate one type; focusing on a single type today means eyeballing and scrolling. The board legend already summarises those types, so making it an interactive filter lets anyone spotlight a type, release it, and stack more types on top. The requester asked for feedback-rich interaction: toggling each type on and off, stacking multiple selections, and a perceptible fade before non-matching cards disappear.

## Current Behavior
The legend above the board is decorative only; entries respond to nothing. All cards remain visible in their columns regardless of type, so type-focused review is done by eye, column by column.

## Desired Behavior
Each legend entry acts as a toggle: clicking it filters the board down to that type, and clicking it again releases it. Several types can remain selected at once, and a card stays visible while it matches any of the selected types (union), so each added type widens the view. Non-matching cards fade out and then leave the board, and the surviving cards glide smoothly into the freed slots; cards coming back return symmetrically. The selection is temporary: reloading or navigating away restores the full board.

## Scope
In:
- Turning the existing legend entries into accessible toggle controls with a visible selected state.
- Client-side filtering of board cards by card type, including stacked multi-selection.
- Hide/reveal transitions for affected cards plus the smooth re-settling of surviving cards, honouring reduced-motion preference (instant swap).
Out:
- Persisting or sharing the filter selection.
- Additional filter dimensions (status, category) or combined queries.
- Changing card content, ordering, or board counts.
- Server or data-model changes.

## Affected Product Areas
- Kanban board view: legend component and card-list rendering/state.

## Requirements
- REQ-001: Legend entries act as toggle controls that indicate on/off state, are operable by mouse and keyboard, and expose pressed state for assistive technology. Rationale: filtering must be discoverable, usable, and accessible, not a hover accident. ACs: AC-001.
- REQ-002: Selecting a single type confines the board to cards of that type; deselecting it restores the unfiltered board. Rationale: the core promised behaviour is per-type show/hide. ACs: AC-002, AC-003.
- REQ-003: Multiple legend types can be selected at the same time; a card shows when it matches any selected type (union), and changing one selection updates the board without disturbing the others. Rationale: the request promises stacking; the accepted union rule makes accumulated picks widen the view instead of collapsing it (settled on CLR-001). ACs: AC-004.
- REQ-004: A card that stops matching fades to zero opacity within a short window (about 150-400 ms) before leaving the board, after which the surviving cards glide smoothly into the freed slots; a card that matches again returns with a comparable reveal and settling; with reduced motion enabled, hide/show and repositioning are instant swaps. Rationale: the requester wants a visible fade and clean disappearance, and the settled exit style (animated reflow after the fade, chosen on CLR-002) commits the layout shift to coordinated motion rather than a hard snap or a stuck ghost. ACs: AC-005, AC-006.
- REQ-005: The filter selection is transient view state only: it is neither persisted nor transmitted, and reload or navigation restores the full board. Rationale: keeps scope client-side and predictable with no new data shape or sync surface. ACs: AC-007.

## Decisions and Assumptions
- The filter key is the card type the legend already distinguishes; entries are derived from live card data, not a hard-coded list. Reversible assumption.
- Stacking combination rule settled on CLR-001: match any selected type (union) — choosing at least one selected type keeps stacked picks additive; intersection would join later as an optional refinement, not the base behaviour.
- Vanished cards leave the layout so neighbours close the gap: after the fade completes, surviving cards settle into their new slots in one coordinated animated pass (“animated reflow after the fade”, settled on CLR-002); reduced motion degrades this to instant repositioning.
- The re-settling must hold up in dense columns and fast filter toggling without stutter, overlap, or stragglers (see AC-005, AC-006).
- Fade window is about 150-400 ms via CSS class transitions; reduced-motion preference switches to an immediate swap.
- No new empty-state UI: a filtered column with no matching cards simply shows as empty; no banners or hint copy.
- The interaction is client-side only; card payloads, ordering, counts, and endpoints are unchanged.

## Acceptance Criteria
- AC-001: Keyboard Tab focuses a legend entry, Enter or Space toggles it, the selected entry displays a distinct filled state, and assistive technology reports the pressed state.
- AC-002: From an unfiltered board, clicking exactly one type T leaves T cards visible and fades then hides every other card, with column layout staying intact.
- AC-003: After AC-002, clicking T again restores the hidden cards and the board matches its original unfiltered arrangement.
- AC-004: With types A and B selected, a card shows only if its type is A or B, and deselecting A hides only A cards while B cards remain.
- AC-005: Removal reaches zero opacity in roughly 150-400 ms before the card leaves layout; once the fade finishes, the surviving cards move into their new slots in one coordinated eased motion without hard snaps, and no instant pop-off occurs at normal motion settings.
- AC-006: Restored cards appear with a matching reveal and settle into place comparably smoothly, with no duplicated, overlapping, or permanently half-opaque cards remaining afterwards.
- AC-007: Reload or navigation shows no legend selection, presents the full board, and no filter flag survives in storage or network traffic.
