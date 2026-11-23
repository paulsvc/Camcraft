## #3701 - Preserve the Mach4 tap pitch

**What was decided:** Format the G84 feed with `fn()` at three metric
decimals or four inch decimals. Refuse a tap when the written F/S pitch
differs from the requested pitch by more than 0.01 percent. Keep the generic
`ff()` formatter unchanged. Put the regression tests in the dedicated
`mach4_feed_decimals` binary to keep the precision and refusal cases together.

**Why:** The coarse G84 feed changes the tap pitch without a warning.
More feed decimals preserve the requested metric feed. Inch conversion and
the integer S word can still round the written pitch beyond the limit, so
the pitch check must use both written words.

**Considered and dropped:** More decimals for every Mach4 feed would change
unrelated output and golden programs. This issue only requires the tap
feed change. More decimals without a pitch check would leave unsafe inch
feed and spindle speed rounding cases.

**Cost to change later:** Update the formatter, pitch guard and their
regression tests. No file format or public API migration is needed.
