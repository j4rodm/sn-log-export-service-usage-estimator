# Pre-release review

**Reviewed:** 2026-11-24  
**Scope:** [index.html](index.html), [les-estimator-trustbytes.html](les-estimator-trustbytes.html), and the [README](README.md).  
**Disposition:** Migration and core behavior smoke-tested. Do not treat this review as release approval.

The calculator files were moved from intake 0113 without content changes. The findings below predate the move. They are recorded for follow-up rather than silently changing product assumptions as part of the migration.

## Code findings

### High — input formatting can alter a count and leave a stale estimate

The calculation parser strips every non-digit character, while the display formatter strips commas only and parses the remaining value as an integer. Blur and separator-toggle handlers reformat the field without recalculating the estimate.

**Reproduction:** Clear the page, enter `1.500.000` in `syslog`, then move focus elsewhere. Calculation first treats this as 1,500,000, but the formatter changes the displayed value to `1`; the prior estimate remains until another calculation-triggering event.

**Impact:** A pasted count can be silently changed, and the displayed input can disagree with the result.

**Recommended fix:** Define one validated count parser for input, formatting, and calculation. Preserve supported grouping formats and recalculate after formatting. Add regression tests for comma, period, and space grouping.

### Medium — invalid values are silently turned into estimates

Removing non-digits makes `-2` become `2`, `1e6` become `16`, and arbitrary text become zero. The text inputs' pattern attributes do not prevent those values from reaching the calculation, and the UI presents no validation error. Very large values are not guarded as safe integers.

**Impact:** Invalid or misunderstood input produces a plausible-looking estimate instead of a clear correction request.

**Recommended fix:** Specify accepted input syntax, reject malformed, negative, fractional, exponent, and unsafe values, and show accessible validation feedback rather than stripping or defaulting.

### Medium — throughput and storage units are inconsistent

The daily cap is shown as 172.8 GB/day, but the calculation comment uses `2 * 86400 / 1024`, which is 168.75. The displayed value corresponds to decimal division by 1,000. Table conversions likewise divide MB by 1,024 but label the result GB.

**Impact:** Users cannot tell whether the results use decimal GB or binary GiB; the capacity gauge also depends on the unconfirmed scope of the 2 MB/s figure.

**Recommended fix:** Confirm the intended unit convention and throughput scope, then align factors, calculations, labels, and tests.

### Medium — over-capacity progressbar values exceed their declared range

The gauge width is capped visually at 100%, and its ARIA maximum is 100, but `aria-valuenow` is set to the rounded uncapped percentage. The Medium preset exceeds the displayed 100 GB allowance, so its starter gauge can report a value greater than its declared maximum.

**Impact:** Assistive technologies receive an invalid progressbar value for a built-in preset and for sufficiently large manual inputs.

**Recommended fix:** Keep `aria-valuenow` within the declared range and expose actual usage or overage in separate accessible text.

### Low — horizontal overflow at a 320 px viewport

At a 320 px viewport, the page document is 355 px wide in both themes, causing horizontal scrolling. At the tested 375 px and 1280 px viewports, the page fits without horizontal overflow.

**Impact:** Users on very narrow screens or at high zoom may need to scroll horizontally.

**Recommended fix:** Find the overflowing layout child and constrain it at the narrow breakpoint. Recheck the table and input controls at 320 px after the change.

## Product and publication blockers

The 2026-10-30 verification findings remain unresolved:

1. The 2 MB/s throughput limit's scope (instance, broker, or client) is unconfirmed.
2. The included monthly allowance and add-on/Vault entitlement wording conflict across sources; the calculator's 100 GB/month verdict must not be treated as contract guidance.
3. Four supported LES sources (`sys_audit_delete`, `sys_audit_relation`, `sys_flow_log`, `sys_user_login_history`) are not modeled because no conversion factors were established.
4. The `/adcv2/dashboard` monitoring reference needs verification.
5. Physical-database-storage factors may not represent exported or billed payload volumes, including serialization, compression, and node-log overhead.
6. S/M/L presets and node-log baselines have not been validated against representative LES telemetry.

Resolve these with authoritative evidence before presenting the calculator as a reliable sizing or entitlement aid.

## Validation performed

- Both HTML files retain the previously recorded source SHA-256 hashes, demonstrating a content-preserving migration.
- Both inline JavaScript blocks parse successfully and remain byte-identical.
- Browser smoke checks passed on both themes for initial state, all three presets, each table source, each node severity level, Clear, comma-toggle behavior on presets, and expanding the query-help section.
- Browser edge-case checks reproduced the period-grouped value changing from `1.500.000` to `1` on blur while the result stayed at its earlier value, and confirmed that `-2` and `1e6` yield positive estimates.
- The Medium preset set the Starter progressbar `aria-valuenow` to 185 with `aria-valuemax` 100; a high node count set the throughput gauge to 116 with maximum 100.
- Responsive checks in both themes measured page widths of 1265 px at 1280 px and 360 px at 375 px, but 355 px at a 320 px viewport.
- A targeted local pattern scan of the two HTML files, README, `.gitignore`, and MIT license found no credential patterns, ServiceNow instance URLs, opportunity identifiers, or absolute home paths.
- Source inspection found no password/token assignments, fetch/XHR/beacon calls, browser storage, or cookie use. The TrustBytes variant loads fonts from Google Fonts; the default page has no external font dependency.
- The destination's existing MIT license was retained.

The source findings are reproducible defects. Narrow-screen overflow was confirmed. Complete assistive-technology coverage remains a follow-up. The browser's font-load error seen during inspection was from the external Google Fonts request in the TrustBytes theme; the app rendered with its fallback fonts.

## Release status

This migration did not push code, configure GitHub Pages, or deploy a hosted application. The files in this checkout are local changes. Resolve the findings and blockers, complete the remaining validation, and obtain release approval before publishing an updated public version.
