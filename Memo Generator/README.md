# Photometric Memo Generator

A free, open-source, browser-based tool for generating analytical photometric estimates of roadway luminaire performance. Upload an IES file, set your roadway geometry, and download a printable two-page analytical memo. All computation runs locally in the browser — no server, no upload, no tracking.

> **⚠ Important — Read First**
>
> This tool produces an **analytical estimate** intended for screening, design exploration, and discussion. **It is not a stamped engineering deliverable.** Outputs are not suitable for permitting, construction, public record filing, or jurisdiction-specific compliance certification. Engage a licensed Professional Engineer for any work intended for construction. See [Liability & Scope](#liability--scope) below.

## What it does

- Parses standard IES LM-63 photometric files (relative or absolute photometry, TILT=NONE)
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
2. **Upload an IES file** — manufacturer photometric data, file extension `.ies`
3. **Set geometry** — pole spacing, mounting height, road width, pole offset, arrangement
4. **Set thresholds** — target E<sub>avg</sub> and Avg/Min uniformity limit (defaults are reasonable for a residential collector road; adjust to match your applicable standard)
5. **(Optional) Add identification** — project name, recipient, preparer
6. **Print to PDF** — Cmd/Ctrl+P, Letter size, no margins, background graphics ON

The preview updates live as you change inputs.

## Methodology

Average maintained illuminance, minimum, maximum, and uniformity ratios are computed by direct point-by-point integration of the user-supplied IES file across a regular grid spanning one full pole-to-pole repeating module of pavement. Per-point illuminance follows the inverse-square cosine law with bilinear interpolation on the candela web:

```
E(x,y) = Σᵢ [ I(γᵢ, Cᵢ) · cos(γᵢ) / dᵢ² ] · LLF
```

where:
- `I(γ,C)` is the candela value bilinearly interpolated from the IES distribution at vertical angle γ and lateral angle C
- `d` is slant distance from the luminaire to the pavement point
- The summation runs over all luminaires within the influence zone (±2 poles per side)
- `LLF` is the user-supplied light loss factor

Photometric data is applied at full candela multiplier with TILT=NONE per IES LM-63. The grid is sampled at 40 longitudinal × 11 transverse zone-center points per module (440 points). Methodology is consistent with the procedures used by DIALux and AGi32 for representative-section roadway analysis.

### Symmetry handling

The IES file's horizontal angle range determines how the distribution is mirrored:
- 0–90°: full quadrant symmetry (4× mirror)
- 0–180°: half-plane symmetry (2× mirror)
- 0–360°: no mirroring

### Absolute photometry

If the IES file declares `lumens ≤ 0` (absolute photometry), total flux is computed by integrating the candela web over the unit sphere with appropriate symmetry expansion.

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

- Additional IES file format edge cases (TILT=INCLUDE handling, vintage 1986 format quirks)
- Metric units toggle (currently US/imperial only)
- Support for AGi32 / DIALux-style multi-criteria evaluation (e.g., minimum point illuminance, max/min ratio limits)
- Pavement reflectance R-table support for luminance-based analysis
- Internationalization

Please don't add server-side functionality without strong justification — the local-only, no-upload posture is a deliberate design choice.

## Technical notes

### File structure

The entire application is a single `index.html` file containing inline CSS and JavaScript. This is intentional — it makes the tool trivially deployable (drop on any static host) and easy to audit (one file, ~2,000 lines). If you prefer split files for development, they're easy to extract.

### IES parser scope

The parser handles the IES LM-63-2002 format with the following caveats:
- TILT=NONE is fully supported
- TILT=INCLUDE is parsed but the tilt multipliers are not currently applied (the candela web is read as-is)
- TILT=<filename> external tilt files are not supported
- Photometric type (Type C, B, A) is read but not used to alter coordinate handling — Type C is assumed
- Keyword block (`[KEYWORD] value` lines before `TILT=`) is parsed only for `[LUMINAIRE]` and `[LUMCAT]` to populate the display name

For the vast majority of modern roadway luminaire IES files, none of these limitations are practically relevant.

### Calculation grid

The 40 × 11 grid (440 points) is a balance between accuracy and live-update responsiveness. For straight-section averages this is well-converged; further refinement changes results by less than ~1%. If you need higher resolution for research purposes, increase `nX` and `nY` in the `analyzeSpacing` function.

## License

MIT — see [LICENSE](LICENSE).

## Acknowledgments

The IES Lumen Method point-by-point integration approach implemented here is a standard textbook procedure (Rea, ed., *IESNA Lighting Handbook*, 9th ed., and successors). The visual presentation borrows from the conventions of DIALux and AGi32 output reports.
