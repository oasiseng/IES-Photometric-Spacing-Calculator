# Technical Audit — IES Photometric Spacing Calculator

Date: 2026-04-06 (UTC)

## Scope

This audit reviews:
- Photometric parser behavior (LM-63 handling)
- Point-by-point roadway illuminance math
- RP-8 alignment claims in UI and pass/fail logic
- Engineering and software reliability risks

## Executive Summary

The app is useful for preliminary spacing studies, but it is **not a full RP-8 design engine** today. The implementation currently evaluates horizontal illuminance and Avg/Min uniformity only, while roadway compliance commonly requires additional criteria (e.g., luminance-based classes, veiling luminance, surround, and context-specific criteria). For permit-grade design, this tool should be treated as screening only.

## Critical Findings

1. **Standards coverage is partial, but UI language can imply full RP-8 compliance.**
   - The checker only enforces `Eavg >= target` and `Avg/Min <= target`. It does not evaluate other roadway criteria used in RP-8 workflows.
   - Risk: false confidence that a layout is compliant when only two metrics were checked.

2. **Historical bug: no-pass cases were labeled as “Max Recommended Spacing.”**
   - Fixed in this update by clearly showing that no compliant spacing was found and avoiding recommendation highlighting when there are no passes.

3. **Historical bug: side-profile compliance badge used a hard-coded 4.0 ratio.**
   - Fixed in this update to use the selected uniformity target.

## High Findings

1. **Parser previously accepted malformed data silently.**
   - Fixed with explicit errors for missing `TILT=` line and missing numeric photometric data.

2. **Parser previously ignored scientific notation numbers.**
   - Fixed by allowing exponent notation (e.g., `1.23E+03`) in numeric parsing.

3. **External tilt file references (`TILT=<filename>`) were not explicitly handled.**
   - Fixed to reject unsupported external tilt references with a clear error message.

## Medium Findings (Not yet fixed in code)

1. **RP-8 class presets are simplified and should be documented as approximations, not code tables.**
   - Current dropdown values are hard-coded defaults and may not map to all road/user contexts.

2. **Calculation grid is fixed at 40×11 regardless of lane geometry.**
   - This is practical for speed, but not equivalent to all standard point grids used in formal studies.

3. **No glare/disability glare proxy checks.**
   - No TI/veiling luminance style checks are present.

4. **No pavement luminance model.**
   - RP-8 often depends on luminance method for higher classes; current app remains illuminance-based only.

## Recommended Next Steps

1. Add a **“Compliance scope” panel** that lists which criteria are and are not checked.
2. Add a **“strict roadway mode”** with additional required metrics and data inputs.
3. Include **audit trail export** (inputs, IES metadata, assumptions, pass/fail criteria) for engineering review.
4. Expand arrangement options (opposite/twin-center, median, overhang, tilt/aim).
5. Add automated regression tests with representative Type II/III/V photometry.

## What was fixed in this branch

- Improved parser validation and error handling for malformed IES files.
- Added scientific notation support in numeric parsing.
- Added unsupported external-tilt guardrail.
- Added spacing-range validation in UI calculation flow.
- Corrected “recommended spacing” behavior for no-pass scenarios.
- Corrected side-profile uniformity compliance to use user-selected target.
