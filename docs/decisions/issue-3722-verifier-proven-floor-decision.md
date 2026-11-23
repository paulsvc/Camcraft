## Decision: Verifier proven floor excludes over-deep dives

**Created:** 2026-11-23 by main session  
**Issue:** #3722 - Verifier: decide whether a named over-deep dive stands as a proven floor for the later laws  
**Priority:** P1 (safety-critical)

### Problem

`Rest::cut` takes the chords of every non-Rapid move, including feed dives that go deeper than allowed by plunge depth law (`PlungeTooDeep`, #3679). When a dive is over-deep, it unsafely removes material below what was verified as safe. Later moves then read this column as cleared to the dive bottom.

The question: should a verifier treat an unsafe removal as proven floor, or exclude over-deep dives from the clearing set?

### Decision

**Exclude over-deep dives from proven floor.** An over-deep dive removes material unsafely according to plunge depth law. If a later move relies on material removed by an over-deep dive, it may be gouging since only part of that zone was safely cut.

The rest calculation should only include material removed SAFELY by previous operations. This follows the verifier's purpose: detect unsafe material removals before they're posted.

### Implementation

1. Modify post processing's Rest::cut to skip over-deep dives when computing proven floor
2. Pinning test: verify that an operation after an over-deep dive correctly detects potential gouge (since only part of the zone was safely cut)
3. Document the decision in `docs/decisions/`

### Done means

A `docs/decisions/` entry names the choice, and pinning tests verify the chosen reading.
