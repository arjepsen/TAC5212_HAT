AUDIO HAT — JLCPCB QUOTATION PREPARATION (DRAFT, NOT MANUFACTURING RELEASE)
Prepared from uploaded for_quotation.zip on 2026-10-18.

TARGET REQUEST
  5 fabricated four-layer boards, 2 SMT-assembled on TOP side, remaining 3 unpopulated.
  JLCPCB Terms 2026 state that with Economic PCBA the other 3 may pass through reflow
  and be coated with solder/tin; they are not necessarily untouched bare PCBs.
  All through-hole parts including header J1, radial C18-C21, and wire sockets J2-J6 to be hand-soldered.
  2 PCB-only mounting holes need no assembly.
  JLCPCB may add assembly edge rails, fiducials, and tooling features for economic PCBA;
  inspect their online assembly preview/engineer notes and ensure mechanical clearances.

INCLUDED
  JLCPCB_BOM_DRAFT.csv  — 42 SMT positions; no DNP, THT, or mounting holes.
  JLCPCB_CPL_DRAFT.csv  — 42 corresponding Top-side locations and raw KiCad footprint rotations.
  component_manifest_ALL.csv — component details, DNP/manual/assembly flags, audit origins.
  BOM_GROUPED_for_2_boards.csv — quantity list for 2 assembled boards and remaining manually fitted parts.
  fp-lib-table — optional project footprint library registration if missing in your local project.

IMPORTANT — NOT YET AN UPLOAD-COMPLETE PACKAGE
  This environment DOES NOT have KiCad/pcbnew/kicad-cli, so Gerber/Excellon exports
  cannot be generated and independently checked here. Export from KiCad 10 (board
  F.Cu, In1.Cu, In2.Cu, B.Cu, F/B.Mask, F/B.SilkS, F.Paste, Edge.Cuts;
  plated + non-plated holes), inspect with KiCad GerbView and ZIP Gerbers/drills.
  Re-run ERC and DRC on the saved final project.
  The CPL rotation values are read directly from KiCad board footprints, not run through a
  JLC correction library; MUST confirm all part orientations using JLC's placement preview,
  especially IC1, U1, U2, D1/D2 and polarised parts.

UNRESOLVED COMPONENTS BEFORE MEANINGFUL QUOTE
  U1: genuine Analog Devices LT3042 10-MSOP-EP; no selected JLCPCB part number in project.
      Do not let an automatically matched substitute such as Tokmas 'LT3042EMSE' be accepted.
      Analog Devices LT3042EMSE#TRPBF (LCSC C461518) or LT3042IMSE#PBF
      (LCSC C580011) are authentic alternatives in the same nominal package, but not
      a user-approved choice and JLCPCB PCBA availability is UNVERIFIED.
  FB1: exact Murata BLM21PG221SN1D -> C85840, listed Extended at JLCPCB.
  FB2: exact Murata BLM18SP601SN1D -> LCSC C3716583; JLCPCB PCBA matching
      and Basic/Extended status must still be confirmed on the quote site.
  All OTHER JLCPCB codes copied from the user's schematic. They are NOT asserted as
      freshly verified stock or as compatible with all given footprints; check at matching stage.

PROJECT-SPECIFIC ENGINEERING CONDITIONS
  R9/R10 are DNP in the latest schematic and PCB. JP1 is SMT-populated.
  R2 changed position and is 33 ohm 0402: center-to-center IC1 distance 4.47 mm.
  IC1 center pad 25 is single F.Cu+F.Mask, no main-pad F.Paste, and four
  0.94mm F.Paste apertures (~80.15% coverage). Verified from uploaded board.
  Native design contains no separate fp-lib-table in the uploaded ZIP, although
  the two custom footprint .pretty folders were supplied.
  Mechanical, audio driver initialization, final safety and fabrication validation
  are NOT signed off. DO NOT ORDER YET.

HOW TO QUOTE
  1. In KiCad 10, run ERC/DRC; refill zones. Plot Gerber + Excellon drill files.
  2. Upload Gerber ZIP at JLCPCB; confirm 4 layers, thickness and pricing.
  3. Enable assembly and select 2 units, TOP side; request 5 fabricated in total.
  4. Upload BOM_DRAFT and CPL_DRAFT; inspect component matching and rotations.
  5. Confirm the three unpopulated boards and their reflow condition; record line items,
     setup, extended parts, shipping, taxes/VAT. Stop before payment.

Manufacturer/source references (for exact part identities, not guaranteed stock):
  FB1: https://jlcpcb.com/partdetail/MurataElectronics-BLM21PG221SN1D/C85840
  FB2: https://www.lcsc.com/product-detail/C3716583.html
  ADI LT3042 family: https://www.analog.com/en/products/lt3042.html
  ADI LT3042 genuine LCSC: https://lcsc.com/product-detail/Linear-Voltage-Regulators-LDO_Analog-Devices_C461518.html
  JLC KiCad 10 guide: https://jlcpcb.com/help/article/how-to-generate-the-bom-and-centroid-file-from-kicad

  JLCPCB Terms for 5 PCBs/2 PCBA: https://jlcpcb.com/help/article/terms-and-conditions-of-jlcpcb-assembly-service
