# IES Photometric Spacing Calculator

**A free, browser-based roadway lighting spacing tool.** Upload a manufacturer IES file, set your pole height and road parameters, and get optimal spacing recommendations with interactive heatmaps — all per IES RP-8 standards.

**Zero dependencies. Single HTML file. No backend. No install.**

Built by [Oasis Engineering LLC](https://oasisengineering.com) • Live at [windcalculations.com](https://windcalculations.com)

![License](https://img.shields.io/badge/license-MIT-green)
![Version](https://img.shields.io/badge/version-2.1-blue)

---

## What It Does

Point-by-point photometric calculation using the inverse square law with cosine correction:

```
E = (I × cos θ) / d²
```

Where `I` is the interpolated candela intensity from the IES file at the vertical and horizontal angles to the calculation point, `θ` is the angle of incidence, and `d` is the distance from luminaire to point.

For each spacing interval, the tool evaluates a 40 × 11 zone-center grid across the roadway between two consecutive poles — including contributions from an adaptive window of neighboring poles (minimum ±3 per side, extending until poles sit at least 12 mounting heights beyond the module) — and reports:

- **Eavg** — Average horizontal illuminance (footcandles)
- **Emin** — Minimum illuminance on the grid
- **Emax** — Maximum illuminance on the grid
- **Avg/Min** — Uniformity ratio per IES RP-8 convention
- **Max/Min** — Uniformity ratio for glare assessment
- **Pass/Fail** — Against user-selected IES RP-8 road classification targets

## Features

- **IES LM-63 Parser** — Type C photometry with:
  - `TILT=NONE` and `TILT=INCLUDE` handling (skips tilt data block correctly); external tilt files (`TILT=<filename>`) rejected with a clear error instead of silently misparsing
  - Scientific-notation and `.5`-style numbers
  - Candela multiplier application; number-of-lamps applied to display lumens
  - Absolute photometry detection (lumens ≤ 0) with exact cos-difference candela web integration
  - Bilateral symmetry (0°–180°), quadrant symmetry (0°–90°), and axially symmetric (single horizontal plane) files
  - Truncated-file detection; non-Type-C, TILT=INCLUDE, and ballast-factor ≠ 1.0 conditions surfaced as warnings instead of being silently ignored
- **IES RP-8 Road Classification** — Dropdown auto-populates target illuminance and uniformity based on road type, pedestrian activity level, and speed (verify values against the RP-8 edition governing your project):
  - Local / Residential (15–25 mph)
  - Collector (30–35 mph)
  - Minor Arterial (35–45 mph)
  - Major Arterial / Highway (45–55 mph)
  - Manual override mode
- **Pole Arrangements** — Single-side and staggered (opposite-side with 180° fixture rotation)
- **Light Loss Factor** — Adjustable 0.60–1.00; defaults to 0.85 since RP-8 targets are maintained values
- **Demo IES File** — One-click synthetic LED cobra-head file for trying the tool without manufacturer data
- **Live Recalculation** — After the first run, results update automatically (debounced) as inputs change
- **CSV Export** — Download the full spacing table with a header documenting the inputs
- **Illuminance Heatmap** — Color-mapped canvas showing fc distribution between two poles, with pole markers
- **Side Profile Visualization** — SVG light cone diagram with dimension callouts and animated luminaire indicators
- **Poles per Mile** — Cost-relevant metric in every results row

## Screenshots

Upload an IES file → Set parameters → Get results:

```
┌─────────────────────────────────────────────────────┐
│ ★ Max Recommended Spacing: 120 ft                   │
│   Eavg: 1.20 fc  |  Avg/Min: 3.2:1  |  44 poles/mi │
│                                                      │
│ [████████ Illuminance Heatmap ████████]              │
│ [📐 Side Profile with Light Cones    ]              │
│                                                      │
│ Spacing  Eavg   Emin   Avg/Min  Status              │
│ 80 ft    1.79   1.01   1.8:1    ✓ PASS              │
│ 100 ft   1.44   0.62   2.3:1    ✓ PASS              │
│ 120 ft   1.20   0.38   3.2:1    ✓ PASS  ★           │
│ 140 ft   1.04   0.22   4.7:1    ⚠ MARGINAL          │
│ 160 ft   0.92   0.11   8.4:1    ✗ FAIL              │
└─────────────────────────────────────────────────────┘
```

## Quick Start

1. Download `ies-spacing-calculator.html`
2. Open it in any modern browser
3. Upload a `.ies` file (IESNA LM-63 format) — or click **Try with demo IES file**
4. Adjust parameters and click **Calculate** (after the first run, results update live)

That's it. No server, no build step, no dependencies.

### Self-Hosting

```bash
# It's one file. Just serve it.
python3 -m http.server 8000

# Or drop it into any web server
cp ies-spacing-calculator.html /var/www/html/tools/lighting-calculator.html
```

## Engineering Notes

### IES Type C Coordinate System

The calculator correctly implements the IES Type C photometric coordinate convention for roadway luminaires:

- **0° horizontal** = Perpendicular to curb line (across the road, +Y direction)
- **90° horizontal** = Parallel to curb line (along the road, +X direction)
- **0° vertical** = Nadir (straight down)

This is implemented via `atan2(dx, dy)` (not `atan2(dy, dx)`) to align the mathematical angle with the IES convention where the primary beam axis of a cobra/roadway fixture points across the street.

### Staggered Arrangement

Far-side poles in staggered configuration are rotated 180° so the fixture's "street side" distribution illuminates the road surface rather than shining backwards. This is applied as a `rotDeg` offset to the horizontal angle before querying the candela array.

### Grid Placement

Calculation points are placed at **zone centers** (half-step offset), not on boundaries. This prevents unrealistic Emax values at theoretical coordinates directly under the pole and aligns with IES RP-8 standard calculation methodology.

### Pole Influence Window

Each grid point sums contributions from an adaptive window of poles: at least 3 modules beyond each end of the analyzed module, extending further (12 × mounting height) when spacing is tight relative to pole height. Convergence-tested against very wide windows — residual error is below 0.1% for typical roadway geometry. (v2.0 used a fixed ±1-module window, which underestimated Eavg by up to ~1.8% and Emin by ~2.3% at tight spacings.)

### Interpolation Bounds

Bilinear interpolation fractions on the candela web are clamped to [0, 1], and vertical angles are clamped to the file's actual angular range — the calculator never extrapolates past the photometric data.

### Uniformity Convention

Uniformity is expressed as **Avg/Min** (e.g., 4.0:1), the North American IES RP-8 convention, rather than the European CIE Uo decimal format (Emin/Eavg = 0.25). Lower ratios indicate better uniformity.

### Known Limitations

| Limitation | Impact | Workaround |
|---|---|---|
| Point-source approximation | Negligible at ≥15 ft mounting heights | Not suitable for bollards (<5 ft) |
| No luminaire tilt angle input | May underestimate throw for tilted fixtures | Use fixture IES that includes tilt in the photometric test |
| TILT=INCLUDE multipliers not applied | Candela web read as-is | Use a TILT=NONE file for the installed orientation |
| No reflected light / interreflection | Conservative (underestimates actual illuminance) | Acceptable for roadway — low reflectance surfaces |
| Single road cross-section | No medians, curbs, or sidewalk zones | Use DIALux for complex cross-sections |
| No luminance calculations | Only illuminance (fc/lux) | RP-8 luminance method requires pavement classification |

### When to Use This vs. DIALux/AGi32

| Use Case | This Tool | DIALux/AGi32 |
|---|---|---|
| Feasibility / spacing study | ✓ Fast, instant | Overkill |
| Fixture comparison | ✓ Upload and compare | Works but slower |
| Client-facing quick answer | ✓ Same-day turnaround | Setup time |
| Permit submittal | ✗ Not a formal report | ✓ Required |
| Complex geometry (medians, curves) | ✗ Straight road only | ✓ Full scene |
| Luminance calculations | ✗ Illuminance only | ✓ Full RP-8 |

## File Structure

```
ies-spacing-calculator/
├── ies-spacing-calculator.html   # The entire application (single file)
├── README.md                     # This file
├── LICENSE                       # MIT License
└── test-fixtures/                # Sample IES files for testing
    ├── JSD-LSL50_IESNA2002.IES
    ├── JSD-LSL80_IESNA2002.IES
    ├── JSD-LSL100_IESNA2002.IES
    └── JSD-LSL120_IESNA2002.IES
```

## Changelog

### v2.1 (2026-06-12)
**Accuracy fixes (engine validated with a 30-test Node suite):**
- **Adaptive pole influence window** — v2.0 summed only ±1 module of neighboring poles (the docs claimed ±2), underestimating Eavg by up to ~1.8% and Emin by ~2.3% at tight spacings. The window now extends at least ±3 modules and grows with mounting height until far poles contribute negligibly (<0.1% residual error vs. a near-infinite row).
- **Interpolation clamping** — bilinear fractions clamped to [0, 1]; no extrapolation past the candela web or the file's vertical angle range.
- **Axially symmetric files** — IES files with a single horizontal plane (nH = 1) now parse and calculate correctly instead of failing.
- **Absolute photometry integration** — switched to the exact cos-difference form (was midpoint sin approximation).
- **Number-of-lamps** applied to display lumens for relative photometry; **scientific-notation** and `.5`-style numbers parsed; **truncated files** detected with a clear error; **external tilt files** (`TILT=<filename>`) rejected explicitly instead of silently misparsing.
- **Parser warnings surfaced** — non-Type-C photometry, TILT=INCLUDE multipliers not applied, ballast factor ≠ 1.0.

**UX and quality:**
- Demo IES file loader (synthetic LED cobra head — try the tool with zero setup)
- CSV export of the full spacing table with input documentation
- Live debounced recalculation after the first run
- LLF defaults to 0.85 and is labeled as a maintained-value factor (was 1.00, which silently compared initial values against RP-8 maintained targets)
- Editing the uniformity slider now flips the road class to Custom (matching the Eavg slider behavior)
- User-controlled text (luminaire name) HTML-escaped before rendering
- Keyboard accessibility for upload zone and arrangement toggle; `prefers-reduced-motion` respected; drag-over highlight properly cleared
- Clearer messaging when no compliant spacing exists in the searched range
- Note added to verify RP-8 target values against the edition governing the project

### v2.0 (2026-03-30)
**Critical fixes from engineering peer review:**
- **Fixed 90° luminaire rotation bug** — `atan2(dy,dx)` → `atan2(dx,dy)` to align with IES Type C convention (0° = across road)
- **Fixed staggered arrangement** — Far-side poles now rotate 180° so the distribution faces the road
- **Applied candela multiplier** — `candelaMult` from IES header now multiplied into candela array
- **TILT=INCLUDE handling** — Parser correctly skips tilt data block instead of corrupting the numeric stream
- **Absolute photometry support** — Detects `lumens ≤ 0` and integrates candela web to compute display lumens
- **Grid placement at zone centers** — Half-step offset prevents unrealistic Emax at pole nadir
- **Quadrant symmetry** — Handles IES files with maxHoriz = 90° (4-way mirroring)
- **Uniformity convention** — Changed from Emin/Eavg decimal to Avg/Min ratio (IES RP-8 standard)

**New features:**
- IES RP-8 road classification dropdown with auto-populated targets
- Speed/pedestrian activity level integration
- Avg/Min and Max/Min ratio display
- Poles per mile column
- Light loss factor control
- Wind load / structural engineering upsell CTAs

### v1.0 (2026-03-30)
- Initial release: IES parser, point-by-point calculation, heatmap, side profile

## Contributing

Issues and PRs welcome. If you find a photometric accuracy issue, please include:
1. The IES file that produces incorrect results
2. Expected vs. actual values
3. Reference calculation (DIALux/AGi32 output preferred)

## Disclaimer

**For preliminary analysis only.** Results must be verified by a licensed professional. Oasis Engineering LLC assumes no liability for design decisions based on this tool.

This calculator uses the point-by-point method per IES RP-8 and is not a substitute for full DIALux/AGi32 simulation required for permit submittals.

## License

MIT — Use it, fork it, embed it, sell it. Attribution appreciated but not required.

## About

Built by [Oasis Engineering LLC](https://oasisengineering.com), a structural and wind engineering firm based in Tampa, FL.

- **Wind Calculations** — [windcalculations.com](https://windcalculations.com)
- **Structural Engineering** — [oasisengineering.com](https://oasisengineering.com)
- **DIY Building Plans** — [bamboodesigns.com](https://bamboodesigns.com)

Need wind load calculations for light poles? EPA analysis? Foundation design? [Get in touch](https://oasisengineering.com).
