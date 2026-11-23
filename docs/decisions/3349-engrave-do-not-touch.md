## #3349 - "Do not touch" support for Engrave operations

**What was decided:** Added `machining2d::trace_avoid` (split out from the 397-line `slots.rs` into its own module) and enabled `OpType::Engrave` in `do_not_touch_applies`. For non-empty protected items, `trace_avoid` trims each loop per Z level outside `AvoidSpec::keepout_region(z, z, AVOID_PAD)`, merges wrap-around breaks at the original seam, rotates unbroken closed loops when their start/retract sits under an overhead clearance obstacle (`keepout_region(z, clearance_z, AVOID_PAD)`), trims open runs whose plunge or retract endpoint is overhead-blocked, elevates rapid links over protected items (`elevate_rapid_links`), and refuses by item name if no safe cutting room remains.

**Why:** An `Engrave` path cuts a centerline curve directly rather than clearing an area, so offsetting or detouring around a protected item would carve unintended lines into the part. Trimming blocked spans and lifting connecting rapids over protected items preserves exact engraving geometry everywhere safe while keeping flutes, shank, and holder clear across cuts, plunges, retracts, and links. Empty `AvoidSpec` delegates directly to `trace` so unprotected `Engrave` operations produce identical toolpaths.

**Alternatives considered and dropped:**
- Detouring the centerline around keep-out polygons (rejected: creates uncommanded cuts in the finished surface).
- Trimming `trace` only in `crates/app` post-generation (rejected: belongs in `camcraft-motion`'s 2D generator alongside `pocket_avoid`, keeping geometry/motion planning in `motion` and verification in `sim`/`app`).

**Cost to change later:** Low. `trace_avoid` sits behind `do_not_touch_applies` and `OpParams::Engrave` dispatch in `generate.rs`; extending it to `OpType::Trace` later is a one-line change in `do_not_touch_applies` and `generate.rs`.
