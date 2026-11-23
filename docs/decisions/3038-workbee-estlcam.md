## #3038 - Ooznest WorkBee 1000x1500 machine file (Estlcam control)

**What was decided.** A new library machine,
`assets/machines/ooznest-workbee-1000x1500.mch` ("Ooznest WorkBee 1000x1500
(Estlcam)"), a 3-axis `mill`:

- X travel 1200 mm, Y travel 770 mm, Z travel 88 mm (written `-L..L`, the
  library's travel rule).
- X and Y feed and rapid 2500 mm/min. Z feed and rapid are not stated.
- Spindle power 2.2 kW. No rpm range is stated.
- No coolant list, no tool count, no `controller.post`.

**The work area takes the smaller value on each axis** (maintainer
decision, 2026-11-23, on PR #3736). The WorkBee's owner (MisterMaker,
relayed by mkinney on the issue) thinks that his frame is a "1500 x 1000"
and gave a work area of 1200 x 800 mm (X x Y). The public Ooznest page for
the 1000x1500 frame gives 1270 x 770 (long axis taken as X, as the owner
states it):

- X 1200 mm: the owner's figure, inside the published 1270.
- Y 770 mm: the published figure; the owner's 800 is more than it.
- Z 88 mm: the published figure; the owner has not stated Z.

A smaller travel can refuse a program that the machine could run. It never
passes a move past the real end of travel. **The WorkBee's owner must
confirm his frame size and his real X/Y/Z travel** before the values are
final.

The first version of this PR kept the public figures with Ooznest's axis
names (X 770, Y 1270). The maintainer replaced it with the rule above.

**Why.**

- Size: the issue names a WorkBee 1500 x 1000. Ooznest sells that frame as
  "1000x1500". Its product page is the public source for the work area.
- Feeds: the owner asked for conservative maxima for his lead screws. The
  page publishes "Maximum Feedrate (mm/min) 2500" and no rapid rate. The file
  states 2500 for the X/Y rapid too: no faster than the published maximum.
  The page gives no Z rate, so Z stays unstated (absent is not zero).
- Spindle: the owner states 2.2 kW. He has not confirmed the rpm range, so no
  `min_speed` or `max_speed` is stated.
- Coolant: Estlcam's M08/M09 and M10/M11 switch "Ausgang 1" and "Ausgang 2"
  (general outputs), not a named coolant. No coolant is stated.
- No post is named. Estlcam's own page for programs from other CAM programs
  (below) sets rules that no built-in post is confirmed to meet:
  - only G00, G01, G02 and G03 run; every other G word is ignored with no
    message;
  - a repeated G word must not be left out ("Sich wiederholende G-Befehle
    duerfen nicht ausgelassen werden"): each motion line needs its own G
    word, so modal output is not safe;
  - one G word and one M word for each line; later words on the line are
    ignored;
  - no cycles, no subprograms, no variables, no expressions;
  - G02/G03 in I/J form only (no R), I/J relative, XY plane only, no full
    circles, helix permitted;
  - absolute coordinates only; no work offset or coordinate system change;
  - units are the length and feed units set in Estlcam (G20/G21 are
    ignored);
  - M words: M00, M01, M03, M05, M06, M08, M09, M10, M11; others ignored;
  - comments in parentheses; UTF-8 or ASCII; any decimal count; no file
    extension rule is stated.

  A post for these rules changes posted G-code. That is under the safety
  floor (`effort:high`), so it is a follow-up issue (#3735) and not part of this
  machine file. Until it ships, the user picks a post for each setup.

**Considered and dropped.**

- The owner's 1200 x 800 mm as stated: Y 800 is more than the published Y.
- The public 770 x 1270 with Ooznest's axis names: the owner's long axis is
  X.
- `controller.post: "Grbl 1.1"` (the issue body's first plan): Estlcam is not
  grbl, and a grbl post that leaves out repeated G words breaks the rule
  above.
- A 6000-24000 rpm range (typical for 2.2 kW VFD spindles): not confirmed.

**Cost to change later.** Edit the JSON values in the one file and the
pinning test `ooznest_workbee_states_the_smaller_envelope_and_feed`. A
rename also changes the picker row in `crates/app/src/posting_files.rs` and
the list in `every_shipped_machine_mounts_and_speaks_its_declaration`.

**Sources (read 2026-11-23).**

- Ooznest, WorkBee Z1+ 1000x1500 product page:
  https://ooznest.co.uk/product/black-1000x1500mm-workbee-z1-cnc-router-machine-ethernet-uk-power/
  ("Working Area (Xmm x Ymm) 770x1270", "Maximum Z-Axis Travel (mm) 88",
  "Maximum Feedrate (mm/min) 2500", "Transmission Type Lead Screw Drive").
- Ooznest, WorkBee machine list:
  https://ooznest.co.uk/product-category/self-build-solutions/workbee-cnc-machine/workbee-machines/
- Estlcam, "Anforderungen an CNC-Programme":
  https://www.estlcam.de/steuerung_cnc_programme.php
- Estlcam, hardware overview: https://www.estlcam.de/steuerung_hardware.php
  and https://www.estlcam.de/hardware.php
- The owner's frame, drive, work area and spindle: issue #3038 comments
  (relayed by mkinney, 2026-11-20). Not a public source.
