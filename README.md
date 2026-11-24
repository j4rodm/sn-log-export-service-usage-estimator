# ServiceNow Log Export Service Usage Estimator

A browser-based planning calculator for ServiceNow Log Export Service (LES). Enter monthly record counts and a node-log profile to estimate daily and monthly data volumes, or start with Small, Medium, and Large presets.

> **Pre-release:** The existing calculator has been migrated for review, not approved for public deployment. Its capacity, entitlement, and billing assumptions require confirmation, and known input-handling defects remain. Read the [review and release blockers](REVIEW.md) before relying on its results.

## Run locally

Open [index.html](index.html) directly in a modern browser. There is no build step, package installation, backend, or ServiceNow connection.

For an optional local HTTP preview, run this from the repository root with Python 3:

```sh
python3 -m http.server 8000 --bind 127.0.0.1
```

Then open <http://127.0.0.1:8000/>. Stop the preview with Ctrl+C.

## Use the calculator

1. Choose a size preset for an illustrative starting point, or enter your own **monthly** record counts.
2. Set the number of application nodes and the node-log severity threshold.
3. Read the daily/monthly totals and per-source breakdown.
4. Use **Clear** to reset the inputs. The separator toggle controls displayed thousands separators.

The expandable query-help section contains example ServiceNow background scripts for obtaining record counts. The page never runs those scripts or connects to an instance. Have an authorized administrator review their scope and performance before running them.

Use plain, non-negative whole-number inputs while the input-validation findings in [REVIEW.md](REVIEW.md) remain open. Period- or space-grouped input can currently be misread when the field loses focus.

## What is included

| File | Purpose |
|---|---|
| [index.html](index.html) | Default calculator, using the original Midnight Ink theme |
| [les-estimator-trustbytes.html](les-estimator-trustbytes.html) | TrustBytes-themed alternative with the same calculation logic |
| [REVIEW.md](REVIEW.md) | Migration validation, reproducible defects, and release gates |
| [LICENSE](LICENSE) | Existing MIT license |

Each calculator keeps its CSS and JavaScript inline. The default page has no external resource dependencies. The TrustBytes variant requests fonts from Google Fonts and uses local fallback fonts when those requests are unavailable. Neither page sends entered values to a server, stores them, or includes analytics.

## Estimation model

The current implementation uses these historical **physical database storage** factors, not measured exported or billable LES payload sizes:

| Table source | Records per MB |
|---|---:|
| `syslog` | 12,900 |
| `syslog_transaction` | 780 |
| `sys_audit` | 1,788 |
| `sys_outbound_http_log` | 900 |

Table-source monthly output is calculated as `records / factor / 1024`. Node-log monthly output is `node count * baseline`: 30 at Info+Warn+Error, 18 at Warn+Error, or 10 at Error only. Daily output divides the monthly total by 30.

The UI labels these results as GB. The units and throughput-cap conversion need reconciliation before release; the current formula is documented here to describe the implementation, not certify it.

### Limitations

- Database storage is not the same as serialized, compressed, or billable export volume. These are planning estimates, not billing predictions.
- Presets and node-log baselines are illustrative, not validated sizing guarantees.
- Only four table sources plus node logs are modeled. Other LES-supported sources are not included.
- The displayed 2 MB/s throughput cap and 100 GB/month included allowance are unverified assumptions. Do not treat the verdict as contractual entitlement advice.
- Monthly-to-daily conversion assumes 30 days, regardless of the calendar month.
- Existing dashboard guidance and product/entitlement wording need verification.

Consult the current [ServiceNow LES documentation](https://www.servicenow.com/docs/r/platform-security/les-landing-page.html), your agreement, and measured LES usage before making sizing or purchasing decisions. This is an independent planning aid, not an official ServiceNow sizing or licensing tool.

## Development and release

Keep the inline JavaScript in both HTML files identical when changing behavior. No test framework or dependency manifest is currently configured. Check both variants when changing calculations, inputs, or layout:

- Empty state; all three presets; Clear.
- Manual source counts and each node-log severity setting.
- Separator toggle, blur/focus behavior, and formatted input.
- Desktop and narrow-screen layout, keyboard access, and capacity gauges.

The default calculator was renamed to [index.html](index.html) so a future static host can serve it at the site root. This migration does not configure GitHub Pages, publish a deployment, or push changes. Resolve the [release blockers](REVIEW.md) and obtain release approval before publishing or sharing a hosted calculator.

## License

[MIT](LICENSE). The license supplied with this repository is unchanged.