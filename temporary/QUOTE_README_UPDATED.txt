AUDIO HAT — UPDATED JLCPCB QUOTATION DRAFT (not production release)
Built 2026-10-18 from user-uploaded for_quotation.zip and existing 42-SMT draft.

QUOTE INTENT
5 four-layer bare PCBs; assemble 2 of them, leaving 3 unassembled.
42 SMT placements on Top plus 2 radial through-hole capacitor positions on Top
(C18,C19), each on BOTH assembled PCBs (4 individual caps total).
J1 40-pin 2x20 female socket on B.Cu: HAND SOLDER yourself, OMIT from
assembly list to avoid potential double-sided assembly tier/processing.
C20,C21 100uF line-output coupling: DNP for QUOTE. IMPORTANT: line outputs
DO NOT WORK until these series capacitors are installed. Keep footprints for
future retrofit. R9,R10 remain DNP per user schematic.

INCLUDED FILES
JLCPCB_BOM_DRAFT.csv: 44 positions = original 42 SMT plus C18/C19.
JLCPCB_CPL_DRAFT.csv: same 44 positions; THT positions from exact board
footprint anchor (C18 116.575,121.377651 angle 90; C19 114.947349,111 angle 0).
component_manifest_ALL.csv: all refs with quote-only population decisions.
BOM_GROUPED_for_2_boards.csv: grouped quantities and population status.
fp-lib-table: project footprints setup helper retained from earlier draft.

CAUTION — CONDITIONAL THT ASSEMBLY
Panasonic EEU-FR1A681 = LCSC C407875, 680uF 10V radial 8mm diameter,
3.5mm pitch; appears in stock at LCSC; JLCPCB PCBA selection eligibility and
stock NOT independently confirmed. Require explicit part matching in quote.
Through-hole C18/C19 may be manually/wave soldered; confirm whether JLCPCB
Economic with top-side SMT and THT wave service accepts chosen parts. Their
recommended documentation asks for BOM and CPL including through-hole parts.
Polarity: inspect placement preview and capacitor polarity relative to board,
ESPECIALLY C18 and C19. The CPL gives KiCad footprint rotation only, not
manufacturer packing rotation, and requires JLC placement verification.

VERSION LIMITATION
The KiCad schematic/PCB in original for_quotation.zip still mark C20 and C21
as populated; these are quotation-specific DNP omissions only. Do not use this
BOM for production without reconciling schematic, PCB flags and order settings.
Underlying SMT rotation and part-matching notes from the previous draft still
apply, along with unverified U1 ADI LT3042 source and FB2 assembly.
GERBERS/EXCELLON DRILLS NOT INCLUDED — KiCad export from saved final PCB.
Not validated for mechanical fit or approved for manufacturing.

J1 SOURCE
Current KiCad footprint: Connector_PinSocket_2.54mm:PinSocket_2x20_P2.54mm_Vertical on B.Cu.
Project schematic: ZHOURI 2.54-2*20 / JLCPCB C2977589, 8.5mm insulator height.
Possible hand-solder sources: RaspberryPi.dk 2x20 female header SKU9808
(19 DKK, no height specification; verify); Kiwi Electronics (Netherlands)
KW-1650 / MPN ZL262-40DG, 8.5mm socket with 3mm tails, €0.99 excl VAT
(verify pin fit in final PCB, and mechanical clearance to Pi active cooler).

See previous QUOTE_README for other sourcing and manufacturing uncertainties.
