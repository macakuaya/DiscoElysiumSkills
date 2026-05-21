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

### Figma protocol

When the user provides a figma.com URL or node id, calling `get_design_context` for that node is mandatory before writing any CSS. `get_metadata` alone is insufficient — it returns layer geometry but not tokens, fonts, or surface treatment. Improvising visual values (colors, fonts, shadows, radii, blur, borders) when a design source exists is a process failure. If the Figma MCP response itself instructs you to call `get_design_context`, that instruction is non-negotiable.

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

### 2. DS primitive audit

For every design-system or repo component in the enumeration that appears inside a Figma frame, you MUST do this BEFORE writing any CSS. Skipping it is the most common way Visual Calculus produces "50% right" output: the outer box is correct, the inner component drifts.

For each such primitive:

a. Read the DS's rendered CSS for the primitive's classes (e.g. `.cc-radio-button-component`, `.cc-radio-button-label`, `.cc-button-primary`). The bundled stylesheet is the source of truth, not the docs.
b. Compare against the Figma component's `data-node-id` rendering returned by `get_design_context`.
c. Produce a divergence table — every property that differs must be either explicitly accepted as deviation or overridden via a scoped `:deep()` rule.

Watch especially for:

- **font-family** — DS components rarely declare it; the label inherits from `body`. Common cause of Chess Sans / Inter mismatches.
- **min-height / min-width** — DS often forces 20px rows where Figma allots 16px, inflating overall component height.
- **internal margin on the input/icon** — DS uses margins as vertical-alignment hacks that shift visual centers.
- **internal padding on labels** — DS often uses `padding-left` for icon-to-text gap; Figma uses `gap` on the wrapper. Picking one without the other yields double-spacing or no spacing.
- **color tokens** — DS may use a generic default token; Figma may specify a brighter / dimmer variant for the surface in question.

Skeleton:

```
DS primitive: <ComponentName>
| Property      | DS default              | Figma spec        | Action               |
|---------------|-------------------------|-------------------|----------------------|
| font-family   | (inherits Chess Sans)   | Inter Regular     | :deep override       |
| min-height    | 2rem                    | 16px              | :deep min-height: 0  |
| input margin  | 0.3rem 0 0.2rem 0       | 0                 | :deep margin: 0      |
```

If the table has zero rows, state that explicitly: "Audited <Component>, no divergence." Silent skipping is not allowed.

### 2.5. Container decoration checklist (mandatory before drawing children)

For every container frame that appears in the layout — including "new"
wrappers that aren't DS primitives — list its OWN decorations BEFORE
enumerating children. These belong to the frame itself, not to any
child, and the layer tree hides them as properties on the parent — the
single most common Figma → code drift after typography. The Step 2
audit only covers DS primitives; new wrappers fall straight through
without this checklist.

For each container, write a one-line row:

```
.<container>  stroke-top:<weight, color, opacity | none>
              stroke-right:<…>   stroke-bottom:<…>   stroke-left:<…>
              fill:<color, opacity | none>
              effect:<drop-shadow / inner-shadow / blur | none>
              radius:<n | 0>   padding:<t r b l>
```

If a side has no stroke, write `none` explicitly. Silent omission is
the failure mode — "I didn't see it" means "I didn't look." Same
anti-silent-skip discipline as the DS primitive audit in Step 2.

Watch especially for:

- **1px hairline dividers at 5–10% opacity** — almost invisible in the
  Figma canvas, trivially missed when scanning by eye.
- **Asymmetric strokes** — e.g. `border-top` + `border-bottom` but no
  sides. Easy to assume "no border" if you only check one side.
- **Effect-mode shadows / inner-shadows** — applied via Figma's
  Effects panel, not visible in the layer tree until expanded.
- **Container fill vs. ancestor fill** — a wrapper that "looks
  transparent" may actually carry its own translucent tint that
  composites differently from the parent.

### 3. ASCII layout container diagram

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

### 4. ASCII per-component diagrams

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

At the end of each diagram, write a dimensional sum and verify it against the Figma frame's reported size. Example:

```
total height = padding-top + header + 4·(gap + row) + padding-bottom
            = 16 + 16 + 4·(8 + 16) + 16
            = 144  ✓ matches Figma frame size (200×144)
```

If the math doesn't match the frame, the spec has drifted. Back up before writing CSS — either a row height is wrong, a gap is wrong, or padding is wrong. Don't wallpaper over the discrepancy by tweaking values until it visually fits.

### 5. Stacking note

Three lines max, referencing the container diagram and any prior decisions earlier in the session:

```
Stacking note:
- <new element> sits between <element A (z:N)> and <element B (z:M)>.
- To render over <conflicting element (z:P)>, needs <position> with z-index ≥ <Q>.
- Choosing z:<X> because <reason>.
```

### 6. Verification handoff

End the plan with an explicit verification checklist of 2–4 specific things the user will see in the rendered output. Never declare a visual change "done", "fixed", or "matches the screenshot" — this model cannot see rendered pages.

Skeleton:

```
Ready to verify. Please confirm in the rendered output:
- [ ] <specific visible thing 1>
- [ ] <specific visible thing 2>
- [ ] <specific visible thing 3>
```

### 7. Optional: browser-use delegation for pixel fidelity

For layouts where pixel-perfect accuracy matters — the user has expressed a "100−1=0" standard, has asked for 1:1 matching against a design frame, or the work is shipping to production — delegate visual verification to a `browser-use` subagent after the implementation lands.

The subagent can:
- Navigate to the dev server
- Take screenshots of the rendered layout
- Compare against the source design (Figma frame, screenshot)
- Report measured discrepancies (spacing, color, alignment) that the parent agent cannot see

Trigger this only when the user opts in or when pixel-fidelity is explicit. The verification handoff in Step 6 is sufficient for most work; browser-use delegation is the escalation path.

## Example trigger

```
[🔵 VISUAL CALCULUS — Medium] Complex layout: <N> positioned wrappers,
<N> design system components, new content crossing the stacking context
of an existing absolute background. Mapping it out before editing.
```
