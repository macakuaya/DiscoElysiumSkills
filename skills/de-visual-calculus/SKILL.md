---
name: de-visual-calculus
description: "Disco Elysium VISUAL CALCULUS skill: spatial planning for complex UI layouts. Activates in plan mode when building or modifying layouts with multiple positioned elements, component composition, or stacking-context decisions. Produces ASCII diagrams of the layout container and every component used, plus a source-of-truth ranking and stacking note, before any code is written."
---

# Visual Calculus

*INTELLECT (#6CC6CE) - See the invisible architecture. Map the load-bearing structures before you build.*

## Voice

Architectural, spatial, methodical. Surveys the site before breaking ground. Commits to dimensions and stacking on paper before touching CSS.

## When to Activate

Plan mode, layout/UX work, AND any of:
- 2+ positioned or z-indexed elements in the same stacking context
- 2+ design system or repo components composed into one view
- Touching a container that already has positioned children
- Building a new screen or section from primitives

Stay out of the way for: single-element tweaks, color changes, icon swaps, copy-only edits.

## Source-of-truth ranking

Fix the ranking before reading any source:

1. User-annotated screenshot (dashed boxes, arrows, circles) — annotation is canonical
2. Plain user screenshot — visual layout is canonical
3. Design tool frame (Figma, Sketch, etc.) — visual reference
4. Design tool layer tree — structural reference only; does NOT dictate DOM ancestry or stacking
5. Inferred patterns from past corrections — do NOT extrapolate

If sources conflict, the higher source wins.

## Format conventions

Apply to every artifact below:

- Use box-drawing characters (`┌─┐│└┘`) for container outlines.
- Annotate each line on the right with invariants only: `z-index`, `position`, key padding, overflow, dimensions.
- Do not annotate every CSS property — only what affects layout or stacking.
- Use three buckets in component enumerations: design system / existing repo / new.

## Workflow

Produce these artifacts in order, before writing any code.

### 1. Component enumeration

Skeleton:

```
Design system primitives:
- <Component>            <package>

Existing repo components:
- <Component>            <path>

New wrappers / primitives:
- <element or wrapper>   (new)
```

If a component doesn't exist yet, mark it `(new)` so the user can override the choice before code is written.

### 2. ASCII layout container diagram

Skeleton:

```
.<container-name> (<dimensions>, <flex/grid>, <overflow>)
┌──────────────────────────────────────────────┐
│ <element>                                    │  z:<n>  position:<…>  <key props>
│ <element>                                    │  z:<n>  position:<…>  <key props>
│ <element>                                    │
└──────────────────────────────────────────────┘
```

Mark every positioned child. Leave un-positioned children with no annotation column.

### 3. ASCII per-component diagrams

For each component in the enumeration, sketch internal structure: padding, dimensions, child order, alignment. Skeleton:

```
<ComponentName props=…>
.<class-or-name> (<key props>)
┌──────────────────────────────────────────────┐
│ <inner-element>                              │
│ <inner-element>                              │
└──────────────────────────────────────────────┘
```

Make *invariants* legible — not every CSS property.

### 4. Stacking note

Three lines max, referencing the container diagram and any prior decisions earlier in the session:

```
Stacking note:
- <new element> sits between <element A (z:N)> and <element B (z:M)>.
- To render over <conflicting element (z:P)>, needs <position> with z-index ≥ <Q>.
- Choosing z:<X> because <reason>.
```

### 5. Verification handoff

End the plan with an explicit verification checklist of 2–4 specific things the user will see in the rendered output. Never declare a visual change "done", "fixed", or "matches the screenshot" — this model cannot see rendered pages.

Skeleton:

```
Ready to verify. Please confirm in the rendered output:
- [ ] <specific visible thing 1>
- [ ] <specific visible thing 2>
- [ ] <specific visible thing 3>
```

### 6. Optional: browser-use delegation for pixel fidelity

For layouts where pixel-perfect accuracy matters — the user has expressed a "100−1=0" standard, has asked for 1:1 matching against a design frame, or the work is shipping to production — delegate visual verification to a `browser-use` subagent after the implementation lands.

The subagent can:
- Navigate to the dev server
- Take screenshots of the rendered layout
- Compare against the source design (Figma frame, screenshot)
- Report measured discrepancies (spacing, color, alignment) that the parent agent cannot see

Trigger this only when the user opts in or when pixel-fidelity is explicit. The verification handoff in Step 5 is sufficient for most work; browser-use delegation is the escalation path.

## Example trigger

```
[🔵 VISUAL CALCULUS — Medium] Complex layout: <N> positioned wrappers,
<N> design system components, new content crossing the stacking context
of an existing absolute background. Mapping it out before editing.
```
