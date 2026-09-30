# LaSRC in Rust

A [reveal.js](https://revealjs.com) deck on porting the USGS ESPA LaSRC atmospheric
correction library to Rust: why HLS needed it, how agentic development and an automated
validation suite made it affordable, and how we would like to work with GSFC on
algorithmic change going forward.

Structured and styled after
[NASA-IMPACT/esdis-spotlight-virtualization](https://github.com/NASA-IMPACT/esdis-spotlight-virtualization).

## View

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>. Press `S` for speaker notes, `F` for fullscreen.

Every slide carries a full presenter script in an `<aside class="notes">`.

## Structure

26 slides in five acts, targeted at 15 to 20 minutes, driven by [`outline.md`](./outline.md):

| Slides | Act | |
|---|---|---|
| 1 to 7 | The dependency and what it costs | Where LaSRC sits in HLS, why ESPA was the right choice in 2019, the science code versus production code framing, and what the gap costs around the codebase and inside the algorithm |
| 8 to 11 | Vision and trigger | SNWG funded GSFC work, the community driven atmospheric correction vision, why Rust, and the GDAS requirement that forced the decision |
| 12 to 14 | The robot army | Multi-agent porting, the compare-against-reference loop, and why the initial port was a foundation rather than a finished result |
| 15 to 20 | Validation and the humans | The in-house validation suite, what it measures, one comparison in detail, the bugs the full run surfaced, and the cases where the port was arguably more correct than the reference |
| 21 to 26 | Results and the ask | Equivalence and performance, what is next in priority order, the proposed GSFC collaboration loop, and takeaways |

The argument turns on slide 4 (science code is not production code), slide 13 (the loop),
and slide 25 (the collaboration proposal). Slide 20 exists on purpose: conceding that the
port sometimes disagreed with the C reference and was right to is what makes slide 25 land.

## Before presenting

Three slides carry `.todo` blocks. Search the source for `class="todo"`.

| Slide | What it needs |
|---|---|
| 17 | Re-export the granule comparison panel from the current validation run, replacing `images/granule-panel-rows.png` |
| 21 | Confirm the headline figures (100% within 5 DN, 30% faster, 7% less memory) against the current run |
| 22 | **Required.** Both figures are from an earlier run and do not show the final result. Replace `images/validation-boxplot.png` and `images/validation-scatter.png` |
| 23 | Optional. Space reserved for a fuller performance picture: runtime table, memory profile or per-granule breakdown |

Slide 22 is the one that matters. The stored run those figures came from passes 100%
against the per-band Bandpass MD reference thresholds, but scores 60% to 100% per band
against the stricter 5 DN bar that slide 21 claims. Showing them as they are invites a
question the deck cannot answer.

## Figures

| File | Source |
|---|---|
| `granule-panel-rows.png` | Two band rows cropped from the per-granule comparison panel, `hls-application` `notebooks/LaSRC_container_validation.ipynb` |
| `validation-boxplot.png` | Per-granule mean absolute difference by band against the threshold, same notebook |
| `validation-scatter.png` | Mean reflectance C against Rust, one panel per band, same notebook |
| `ODSI full light.png` | ODSI logo |
| `rust-logo.png`, `rust-logo-blk.svg` | Rust Foundation, <https://www.rust-lang.org/policies/media-guide> |

Two diagrams are drawn in the deck's own palette as inline SVG in `index.html`, not images:
the HLS processing chain on slide 2, and the porting loop on slide 13.

## Conventions

No em dashes anywhere in the deck text, by request.
