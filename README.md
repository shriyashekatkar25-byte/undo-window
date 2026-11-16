# Undo Window

**How long should a dropped data volume stay recoverable?** An importance-sampling Monte Carlo prototype that picks a default recovery window for dropped volumes.

- **Live demo:** https://shriyashekatkar25-byte.github.io/undo-window/
- **Gallery:** https://shriyashekatkar25-byte.github.io/apm-lab/

## The launch that inspired it
On Nov 7, 2026, Databricks shipped **UNDROP VOLUME** for Unity Catalog ([release notes](https://docs.databricks.com/aws/en/release-notes/product/2026/november)). A dropped volume can now be restored within 7 days, along with its privileges and tags, on Databricks Runtime 19 and above ([UNDROP VOLUME reference](https://docs.databricks.com/aws/en/sql/language-manual/sql-ref-syntax-ddl-undrop-volume)).

## The PM decision
Should the default recovery window stay at 7 days, or move to 14, 21 or 30?
- **Success metric:** share of accidental drops still recoverable when someone notices (target ≥ 99%)
- **Guardrails:** retained-storage spend ≤ $50k/month · hard purge within 30 days · ≤ 300 unrecoverable accidents/year · tail-estimate relative error ≤ 10%
- **Stakeholders:** data/ML engineers, platform admins, security/privacy/legal, support, the storage infra team

With the default assumptions, the simulation recommends **14 days**. Seven days recovers about 97.7% of accidents, and 30 days blows the storage budget. The director memo on the page rewrites itself as you move the sliders.

## What the prototype shows about product judgment
| Databricks APM skill | Where it shows up |
|---|---|
| Customer problem framing | Late-discovered mistakes (monthly jobs, audits) versus storage cost and compliance purge deadlines |
| Prototyping | Live, seeded simulation with sliders for the window, how slow discovery is, fleet scale and the accident rate |
| Data-driven decisions | Five candidate windows scored against the guardrails; the smallest passing window wins |
| SQL / Python | Illustrative audit-log SQL to measure drop-to-undo delay, plus a NumPy/SciPy importance sampler |
| Clear writing | A self-updating director memo and a one-sentence VP summary |

## Methods (Stats 102C Monte Carlo)
| Method | Used for |
|---|---|
| Ch6 importance sampling: proposal g = W + Exp(θ), weights w = f/g | Estimating the rare probability that a mistake is noticed after the window |
| Ch6 running estimate ± 1.96·SE, variance-reduction factor | Showing that IS converges and plain MC does not |
| Ch5 mixture (composition) + Box-Muller | Discovery delay: fast exponential group plus slow lognormal group |
| Ch3 inverse CDF | Exponential draws for the fast group and the proposal |

## Design baseline
Four pins from Shriya's Pinterest home feed, each redrawn in CSS/SVG (no pin images used):
- **Hero:** [Mooniya Chillu, pink etched portrait](https://www.pinterest.com/pin/50735933297477389/). Took the monochrome magenta-on-blush palette (#b0185a, #d9558f, #fbe3ee), crosshatch etching texture and flowing curl lines. The drawing itself is an original hourglass.
- **Cards & buttons:** ["Authenticity is Magnetic" yellow note](https://www.pinterest.com/pin/619033911328579824/). Took the saturated sticky-note yellow gradient (#ffd21f → #f5b800), hand-written marker headings in maroon ink, and slightly tilted, taped notes.
- **KPI badges:** [galaxy_blush, blue glass evil-eye star](https://www.pinterest.com/pin/813603488982031975/). Took the glossy cobalt glass (#1c2bb0 → #3a5cff), the specular highlight, and a concentric navy/white/sky-blue eye-ring motif.
- **Simulation room:** [Hummingbirds around a spiral](https://www.pinterest.com/pin/378020962481858273/). Took the watercolor paper with teal (#1f7f7a, #62bfc4) and rust (#b43e22) accents, soft pink washes, and samples scattered like a flock.

---
Unofficial concept prototype by Shriya Shekatkar. All data is simulated. Not affiliated with any company.
