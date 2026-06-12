# Photometric Memo Generator

A free, open-source, browser-based tool for generating analytical photometric estimates of roadway luminaire performance. Upload an IES file, set your roadway geometry, and download a printable two-page analytical memo. All computation runs locally in the browser — no server, no upload, no tracking.

> **⚠ Important — Read First**
>
> This tool produces an **analytical estimate** intended for screening, design exploration, and discussion. **It is not a stamped engineering deliverable.** Outputs are not suitable for permitting, construction, public record filing, or jurisdiction-specific compliance certification. Engage a licensed Professional Engineer for any work intended for construction. See [Liability & Scope](#liability--scope) below.

## What it does

- Parses standard IES LM-63 photometric files (relative or absolute photometry)
- Computes point-by-point illuminance across one full pole-to-pole repeating module of pavement
- Reports E<sub>avg</sub>, E<sub>min</sub>, E<sub>max</sub>, Avg/Min uniformity, Max/Min ratio, S/MH ratio
- Renders a heatmap visualization of the illuminance distribution
- Generates a print-ready two-page PDF memo with configuration, results, methodology, and disclaimer

## Quick start

This is a single-file HTML application. No build step, no dependencies beyond a browser.

```bash
# Clone or download
git clone https://github.com/YOUR-USERNAME/photometric-memo-tool.git
cd photometric-memo-tool

# Open directly in a browser
open index.html      # macOS
xdg-open index.html  # Linux
start index.html     # Windows
```

Or just double-click `index.html`. It works offline once loaded (Google Fonts cached).

To deploy on a website, copy `index.html` to your web root. That's it.

## Usage

1. **Acknowledge the disclaimer** on first load (required, persists for the browser session)
2. **Upload an IES file** — manufacturer photometric data, file extension `.ies` — or click **Try with demo IES file** to explore with a synthetic distribution
3. **Set geometry** — pole spacing, mounting height, road width, pole offset, arrangement
4. **Set thresholds** — target E<sub>avg</sub> and Avg/Min uniformity limit (defaults are reasonable for a residential collector road; adjust to match your applicable standard)
5. **(Optional) Add identification** — project name, recipient, preparer
6. **Print to PDF** — Cmd/Ctrl+P, Letter size, no margins, background graphics ON

The preview updates live as you change inputs. The document number is deterministic — the same configuration on the same day always produces the same number, so a reprinted memo matches its original.

## Methodology

Average maintained illuminance, minimum, maximum, and uniformity ratios are computed by direct point-by-point integration of the user-supplied IES file across a regular grid spanning one full pole-to-pole repeating module of pavement. Per-point illuminance follows the inverse-square cosine law with bilinear interpolation on the candela web:

```
E(x,y) = Σᵢ [ I(γᵢ, Cᵢ) · cos(γᵢ) / dᵢ² ] · LLF
```

where:
- `I(γ,C)` is the candela value bilinearly interpolated from the IES distribution at vertical angle γ and lateral angle C (interpolation fractions clamped — no extrapolation past the photometric web)
- `d` is slant distance from the luminaire to the pavement point
- The summation runs over all luminaires within an adaptive influence window: at least ±3 poles per side, extending until poles sit at least 12 mounting heights beyond the module (residual truncation error below 0.1% for typical roadway geometry)
- `LLF` is the user-supplied light loss factor

Photometric data is applied at full candela multiplier per IES LM-63. The grid is sampled at 40 longitudinal × 11 transverse zone-center points per module (440 points). Methodology is consistent with the procedures used by DIALux and AGi32 for representative-section roadway analysis.

### Symmetry handling

The IES file's horizontal angle range determines how the distribution is mirrored:
- Single horizontal plane: axially symmetric (no lateral dependence)
- 0–90°: full quadrant symmetry (4× mirror)
- 0–180°: half-plane symmetry (2× mirror)
- 0–360°: no mirroring

### Absolute photometry

If the IES file declares `lumens ≤ 0` (absolute photometry), total flux is computed by integrating the candela web over the unit sphere (exact cos-difference form) with appropriate symmetry expansion. For relative photometry, the declared per-lamp lumens are multiplied by the number of lamps for the displayed total.

## What this tool does NOT do

This is a single-section screening calculation. It does not:

- Perform full-site analysis with intersections, curves, or multiple road classifications
- Evaluate light trespass, glare metrics (BUG ratings), or veiling luminance
- Account for road surface reflectance variations, pavement R-tables, or wet-surface conditions
- Validate jurisdictional compliance (ANSI/IES RP-8-25, AASHTO GL-7, IDA, AHJ-specific)
- Verify the supplied IES file is appropriate for the application
- Replace a stamped photometric design report
- Cross-check structural pole loading, foundation design, or wind effects

## Browser compatibility

Tested in current versions of Chrome, Edge, Firefox, and Safari. Print fidelity is best in Chromium-based browsers (Chrome, Edge, Brave) — Safari and Firefox have minor differences in print CSS handling.

Print to PDF settings:
- **Paper size:** Letter (8.5 × 11 in)
- **Margins:** None (the memo has internal padding)
- **Background graphics:** ON (otherwise the heatmap renders blank)
- **Scale:** 100% / Default

## Privacy

All processing happens in your browser. The IES file you upload is never transmitted anywhere — there is no server. The only outbound network requests are for Google Fonts (CSS only). If you fork this and want zero external requests, replace the Google Fonts link with locally hosted font files or system font fallbacks.

## Liability & Scope

**This tool is provided as-is, without warranty of any kind**, express or implied, including but not limited to merchantability, fitness for a particular purpose, accuracy, or non-infringement. The authors and contributors shall have no liability for any decision, design, construction outcome, code violation, financial loss, injury, or other consequence arising from the use of this tool or its outputs.

**Outputs from this tool are analytical estimates only.** They are explicitly:

- **Not** stamped or sealed by a Professional Engineer
- **Not** a full-site photometric design
- **Not** intended for permitting, construction, public record filing, or jurisdiction-specific compliance certification
- **Not** a substitute for professional engineering analysis

Site-specific factors that materially affect lighting performance — including intersection geometry, road curvature, local-road segments, light trespass at property lines, pavement reflectance, structural pole design, wind loading, and AHJ-specific criteria — are **outside the scope of this tool**.

The user is solely responsible for:
- Verifying that the IES file is appropriate for the intended luminaire and application
- Verifying that the user-specified inputs accurately represent the project geometry
- Verifying results against the applicable governing standard (ANSI/IES RP-8-25 is current as of publication; the user must determine which version applies to their jurisdiction and project)
- Engaging a licensed Professional Engineer for any work intended for construction

By using this tool the user acknowledges and accepts all of the above.

## Contributing

Issues and pull requests welcome. Areas where contribution would be especially valuable:

- Additional IES file format edge cases (TILT=INCLUDE multiplier application, vintage 1986 format quirks)
- Metric units toggle (currently US/imperial only)
- Support for AGi32 / DIALux-style multi-criteria evaluation (e.g., minimum point illuminance, max/min ratio limits)
- Pavement reflectance R-table support for luminance-based analysis
- Internationalization

Please don't add server-side functionality without strong justification — the local-only, no-upload posture is a deliberate design choice.

## Technical notes

### File structure

The entire application is a single `index.html` file containing inline CSS and JavaScript. This is intentional — it makes the tool trivially deployable (drop on any static host) and easy to audit (one file, ~2,000 lines). If you prefer split files for development, they're easy to extract. The photometric engine block is shared with the companion IES Spacing Calculator and validated by a 30-test Node suite — if you modify it, re-run the tests.

### IES parser scope

The parser handles the IES LM-63 format with the following behavior:
- TILT=NONE is fully supported
- TILT=INCLUDE is parsed but the tilt multipliers are not currently applied (the candela web is read as-is); a warning is shown
- TILT=&lt;filename&gt; external tilt files are rejected with a clear error (they cannot be resolved in-browser)
- Scientific-notation and `.5`-style numbers are parsed; truncated files are detected and rejected with an error
- Axially symmetric files (a single horizontal plane) are supported
- Photometric type (Type C, B, A) is read; non-Type-C files trigger a warning since Type C coordinate handling is assumed
- Ballast factor is read and surfaced as a warning when ≠ 1.0, but is not silently applied — fold it into your LLF if applicable
- Keyword block (`[KEYWORD] value` lines before `TILT=`) is parsed for `[LUMINAIRE]` and `[LUMCAT]` to populate the display name

For the vast majority of modern roadway luminaire IES files, none of these limitations are practically relevant.

### Calculation grid

The 40 × 11 grid (440 points) is a balance between accuracy and live-update responsiveness. For straight-section averages this is well-converged; further refinement changes results by less than ~1%. If you need higher resolution for research purposes, increase `nX` and `nY` in the `analyzeSpacing` function.

### Document numbers

Memo numbers take the form `PM-YYMMDD-XXXX`, where the suffix is a deterministic hash of the configuration (luminaire, geometry, thresholds) and the date. Identical inputs on the same day reproduce the same number; any input change produces a new one.

## Changelog

### v1.1 (2026-06-12)
**Accuracy fixes (engine validated with a 30-test Node suite, shared with the IES Spacing Calculator):**
- **Adaptive pole influence window** — v1.0 summed only ±1 module of neighboring poles (the docs claimed ±2), underestimating E<sub>avg</sub> by up to ~1.8% and E<sub>min</sub> by ~2.3% at tight spacings. The window now extends at least ±3 modules and grows with mounting height (residual error <0.1%).
- **Interpolation clamping** — bilinear fractions clamped to [0, 1]; the v1.0 lookup could extrapolate past the candela web. Vertical angles clamped to the file's actual range.
- **Parser robustness** — scientific notation and `.5`-style numbers parsed (v1.0's regex dropped them, silently shifting every subsequent value); null guard on the numeric block; truncated files detected; external tilt files rejected with a clear error instead of misparsing the filename as data.
- **Axially symmetric files** (single horizontal plane) supported.
- **Absolute photometry integration** — exact cos-difference form.
- **Number-of-lamps** applied to display lumens; **ballast factor** and **non-Type-C photometry** surfaced as warnings instead of being silently ignored.

**Correctness and quality:**
- **Methodology naming fixed** — the memo previously labeled the calculation the "IES Lumen Method," which is the coefficient-of-utilization/zonal-cavity procedure, not what this tool does. All references now correctly describe it as point-by-point integration per IES LM-63 photometry.
- **Deterministic document numbers** — v1.0 generated a random memo number on every keystroke; numbers are now a stable hash of configuration + date (prefix changed `PE-` → `PM-` to avoid any Professional Engineer implication).
- **HTML escaping** — luminaire name and all user-entered identification fields are escaped before rendering (a crafted IES file or pasted text can no longer inject markup into the memo).
- **Input validation** — numeric fields validated against sane bounds with inline highlighting instead of producing NaN memos.
- **Demo IES loader** — synthetic LED cobra-head file for trying the tool with zero setup.
- **Parser warnings panel** in the upload card; clearer inline parse errors (no more `alert()`).
- **Debounced live preview** (no full heatmap re-render on every keystroke); keyboard-accessible upload zone; `prefers-reduced-motion` respected.

### v1.0
- Initial release: IES parser, point-by-point calculation, two-page printable memo, heatmap, acknowledgment gate.

## License

MIT — see [LICENSE](LICENSE).

## Acknowledgments

The point-by-point inverse-square-cosine integration implemented here is a standard textbook procedure (Rea, ed., *IESNA Lighting Handbook*, 9th ed., and successors). The visual presentation borrows from the conventions of DIALux and AGi32 output reports.
