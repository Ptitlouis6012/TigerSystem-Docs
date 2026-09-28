# The material swatch — the official convention

A material's colour is stored as data, not as a picture. The **swatch** is that
data drawn: the coloured shape that stands for the material in a list, on a
card, in a rack slot, next to its name. This page is the **official TigerSystem
convention** for producing it, so the same material shows the same swatch
everywhere — in Tiger Studio, in the mobile app, on the web, in your own
integration.

**"Material", not "spool", is deliberate.** A TigerTag identifies a filament
spool, but also an accessory, a spare part, a resin — and the product types the
protocol has yet to gain. The convention is written against `id_type` in
general: nothing below reads the product type, so a swatch is produced the same
way whatever the material is.

It is normative. A bicolor filament that shows a smooth ramp in one app and a
hard diagonal split in another is a bug in whichever one departed from this page
— not a matter of taste. If you cannot reproduce a rule exactly (a platform
with no conic gradient, say), implement the closest equivalent described in
[Non-CSS platforms](#non-css-platforms) and say so; do not invent a different
picture.

- **Convention version:** 1.2 — bicolor and tricolor are a **135° ramp** (1.1 drew bicolor as a
 hard diagonal split and tricolor as a camembert; 1.0 split bicolor vertically)
- **Reference renderer:** [`material-swatch-playground.html`](./material-swatch-playground.html) —
 open it in any browser, no server, no dependency. Every case, every box shape,
 live colour pickers, and the exact CSS it produces.

---

## Three shapes, and only three

| Shape | When | Geometry |
|---|---|---|
| **Ramp** | **Two or three** hard-edged colours — bicolor, tricolor — plus rainbow and the catalogue's declared `gradient` type | A smooth linear ramp at **135°**, no hard edge — pointing to the bottom-right, so the first colour sits top-left |
| **Camembert** (pie) | **Four or more** hard-edged colours — any list of N ≥ 4 | N equal conic sectors, first colour starting at **12 o'clock**, sweeping **clockwise** |

Said plainly: **two or three hard colours blend into a ramp, four or more make a
camembert** — and there is exactly one angle in the whole system, 135°, shared
by every ramp in it.

**Bicolor and tricolor are a ramp — the same shape as a declared gradient.**
Same axis, same order (first colour top-left), no hard edge anywhere: 2 or 3
hard colours blend exactly like a catalogue `gradient` of the same length.
Implement it once, in the function that splits a list of colours (N ≤ 3 →
ramp, N ≥ 4 → pie), so every caller gets it; never draw it mirrored.

Why a ramp for two or three colours: a hard edge still read as two or three
separate objects — on a thumbnail, and even more on the colour frame round a
product photo, where the photo hid the centre and left disjoint coloured bars.
A ramp reads as one multi-tone material on any box shape. Why a pie from four
colours up: a ramp through four or more colours turns to mud, while sector
boundaries stay angular and recognisable — measured from the box centre, so N
sectors stay legible whatever the box's aspect ratio. Why 135°: it is the one
angle of the system, shared by every ramp, whether it has 2, 3 or more smooth
stops.

---

## Where the data comes from

Two layers feed the renderer, and they are not the same thing.

### The chip — at most three colours

A TigerTag carries **three colour slots and an aspect**. Nothing else: there is
no gradient type on the chip, and there never was.

| Field | Type | Meaning |
|---|---|---|
| `color_r` / `color_g` / `color_b` | `int 0-255` | Slot 1 |
| `color_r2` / `color_g2` / `color_b2` | `int 0-255` | Slot 2 |
| `color_r3` / `color_g3` / `color_b3` | `int 0-255` | Slot 3 |
| `color_a` | `int 0-255` | Alpha — **ignored for rendering**; never blend a material colour |
| `id_aspect1` / `id_aspect2` | `int` | Either slot may carry the colouring aspect |

Three aspect ids change the shape (their reference table also carries an
authoritative `color_count`):

| id | label | `color_count` | Shape |
|---|---|---|---|
| `252` | Bicolor | 2 | **Ramp**, 135° |
| `24` | Tricolor | 3 | **Ramp**, 135° |
| `145` | Rainbow | 3 | Ramp, 135° |

Every other aspect (`Silk`, `Matt`, `Glitter`, …) has `color_count` ≤ 1 and does
not affect the shape. **Match on the id, not on the label** — labels are display
strings and may be translated.

> **Never count the slots to guess how many colours there are.** A chip
> document always carries all three slots: absent components are stored as `0`,
> so slots 2 and 3 read as pure black on a mono material. **The number of colours
> comes from the aspect, never from the slots.**

### The catalogue — a richer, cloud-only description

A product from the official catalogue can describe its colour more precisely
than a chip can. These two fields exist **only** in cloud/product data — they are
never written to a chip:

| Field | Type | Meaning |
|---|---|---|
| `online_color_list` | `string[]` | Ordered colours, `RRGGBB` or `RRGGBBAA`, `#` optional. Order is meaningful: index 0 is the first sector / first stop. |
| `online_color_type` | `string` | Rendering instruction: `mono`, `multi`, `gradient`, `conic_gradient`. Any other value, or absent, is treated as `multi`. |

When both layers are present, **the catalogue wins** — it is the more precise
description of the same product.

---

## The decision ladder

Evaluate **in this order, first match wins**. The order encodes a priority: the
catalogue outranks the chip, an explicit colour type outranks a guess, and the
aspect only speaks when there is no online colour list.

Let `LIST` = `online_color_list` after [normalisation](#normalisation), `TYPE` =
`online_color_type`, `SLOTS` = the non-null chip slots.

| # | Condition | Result |
|---|---|---|
| 1 | `LIST ≥ 2` and `TYPE == "conic_gradient"` | Smooth conic sweep, closing on the first colour |
| 2 | `LIST ≥ 2` and `TYPE == "gradient"` | Ramp — same shape `LIST` would get at 2-3 colours anyway; the declared type only matters once `LIST.length ≥ 4` |
| 3 | `LIST ≥ 2` | **Ramp** of `LIST` when 2 or 3 colours, **camembert** of `LIST.length` sectors when 4 or more |
| 4 | `LIST == 1` | Solid colour — **outranks the chip colour** |
| 5 | aspect **Rainbow** *and* **Tricolor** | Ramp, 3 stops |
| 6 | aspect **Rainbow** *and* **Bicolor** | Ramp, 2 stops |
| 7 | aspect **Rainbow** | Ramp over `SLOTS`; 1 slot → solid; 0 slots → the 6-colour default |
| 8 | aspect **Tricolor** | **Ramp**, 135°, over `SLOTS` (slot 3 missing → a 2-colour ramp over slots 1 and 2, it does not repeat slot 1) |
| 9 | aspect **Bicolor** | **Ramp**, 135°, over slots 1 and 2 |
| 10 | otherwise | Solid slot 1; nothing at all → `#1c2030` |

Defaults when an aspect carries no usable colour:

| Case | Defaults |
|---|---|
| Rainbow, no colours | `#ff0000 #ff8800 #ffff00 #00cc00 #0000ff #8b00ff` |
| Rainbow + Tricolor | `#ff4d4d #ffd93d #4da3ff` |
| Rainbow + Bicolor | `#ff7a00 #8a2be2` |
| Tricolor | `#cccccc #888888` (slot 3 missing → 2-colour ramp, slot 1 is not repeated) |
| Bicolor | `#cccccc #ffffff` |
| Nothing | `#1c2030` |

---

## Normalisation

Applied to every entry of `online_color_list` before the ladder is evaluated:

1. Trim, strip a leading `#`.
2. If 8 characters (`RRGGBBAA`), **keep the first 6** — alpha is dropped, never
 blended.
3. Accept only `^[0-9a-fA-F]{6}$`. **Anything else is dropped from the list**,
 not defaulted — a malformed entry must never silently become black.
4. Re-add `#` for output.

Dropping happens *before* the ladder runs, so `["ff0000", "oops"]` is a
**one-colour** list (rule 4), not a two-sector pie.

Chip slots convert to `#` + two hex digits per component, only when all three
components are numbers.

---

## The exact expressions

With `c1…cN` the normalised colours and `step = 360 / N`:

```css
/* Ramp — 2 or 3 hard colours: rule 3 with 2-3 colours, rules 5-9. Same shape
   as a declared gradient, one angle for every ramp in the system. */
linear-gradient(135deg, c1, c2, …)

/* Camembert — four or more hard colours: rule 3 with ≥ 4 colours */
conic-gradient(c1 0deg <step>deg, c2 <step>deg <2·step>deg, …)

/* Catalogue-declared conic gradient — rule 1 (first colour repeated to close) */
conic-gradient(from 0deg, c1, c2, …, c1)

/* Mono — rules 4, 10 */
# RRGGBB
```

### Test vectors

Any implementation must reproduce these exactly.

| Input | Expected |
|---|---|
| `{online_color_list:["FF5722"]}` | `#FF5722` |
| `{online_color_list:["000000FF"]}` | `#000000` |
| `{online_color_list:["e02424","2463e0"]}` | `linear-gradient(135deg, #e02424, #2463e0)` |
| `{online_color_list:["e02424","2463e0","22a06b"]}` | `linear-gradient(135deg, #e02424, #2463e0, #22a06b)` |
| `{online_color_list:["e02424","2463e0","22a06b","f0a020"]}` | `conic-gradient(#e02424 0deg 90deg, #2463e0 90deg 180deg, #22a06b 180deg 270deg, #f0a020 270deg 360deg)` |
| `{online_color_list:["e02424","2463e0"],online_color_type:"gradient"}` | `linear-gradient(135deg, #e02424, #2463e0)` |
| `{color_r:224,color_g:36,color_b:36,color_r2:36,color_g2:99,color_b2:224,id_aspect2:252}` | `linear-gradient(135deg, #e02424, #2463e0)` |
| `{id_aspect1:145}` | `linear-gradient(135deg, #ff0000, #ff8800, #ffff00, #00cc00, #0000ff, #8b00ff)` |
| `{}` | `#1c2030` |

---

## The TigerTag watermark

Any surface that paints a material colour **without** a product photo carries the
TigerTag logo on top, as a watermark.

| Rule | Value |
|---|---|
| Position | Top-right corner of the tile |
| Opacity | **1** — always, on every surface |
| Size | A **percentage** of the tile, never a fixed pixel size, so it scales with the surface |
| Variant | **Dark background → the plain WHITE logo. Light background → the contoured BLACK logo.** |

The two logo files are **not tintable variants of one another** — each ships
with its own baked-in fill, and the rule is which *file* to use. Never apply a
CSS filter, a mask colour or an opacity to make one stand in for the other: the
contoured artwork is a different drawing, not the white one inverted.

**Choosing the variant** — take the relative luminance of the **first colour** of
the produced expression:

```
luminance = (0.299·R + 0.587·G + 0.114·B) / 255   // dark when < 0.5
```

If you extract that colour by matching the first `#` hex in a CSS string, match
**8 digits first**, then 6, then 4, then 3. Otherwise an `#RRGGBBAA` matches its
first six digits, fails the word boundary that follows, and the whole match is
lost — which reads as "light" and puts a black logo on a black spool. Drop the
alpha and expand shorthand before computing.

---

## Non-CSS platforms

Flutter, SwiftUI, Android or any canvas renderer implements the same three shapes.
Angles are always measured the CSS way: **0° points up, angles increase
clockwise.**

- **Camembert** — a sweep gradient centred on the tile, starting at 12 o'clock,
 clockwise, with hard stops at each `k · 360/N` degrees. Flutter:
 `SweepGradient` with `transform: GradientRotation(-pi/2)`. Do not approximate
 it with pie slices drawn as paths unless the tile is square — the sector
 boundaries must follow the box. Only for **four or more** hard colours.
- **Ramp** — a linear gradient from the **top-left** corner to the
 **bottom-right**, evenly spaced stops, no hard edge. Used for **two or
 three** hard colours (bicolor, tricolor), for rainbow, and for the
 catalogue's declared `gradient` type. Flutter: `Alignment.topLeft →
 Alignment.bottomRight`.

---

## Changing the convention

This page is the source. A change here is a change to every TigerSystem surface:
amend it first, bump the convention version, then re-align the implementations
and check them against the reference renderer.

---

**▲ [Documentation index](../../README.md)** · **Related:** [The TigerTag chip](../concepts/tigertag-chip.md), [The `.ttag` format](./ttag-format.md), [Developer overview](./README.md)
