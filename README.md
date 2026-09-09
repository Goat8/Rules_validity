# GRN Rule Inspector

A lightweight, browser-based tool to **visualise and quality-check Boolean gene-regulatory rule files**   the kind exported by TISON and similar Boolean network builders.

**Live:** https://goat8.github.io/Rules_validity/

It runs entirely in your browser. The rules file you open is parsed locally and **never uploaded anywhere**, so it is safe to use on unpublished data.

---

## What it does

Load a rules `.txt` file and get three views:

### 1. Overview / QC
A health check on the whole rule set:
- **Summary tiles**   total targets, real rules (with %), Undefined, "too many sources", edge count, mean and max fan-in.
- **Rule-type composition**   a single bar splitting every target into *real rule / too many sources / constant = 0 / constant = 1 / undefined*, so degenerate and missing rules are visible at a glance rather than hidden inside a coverage number.
- **Fan-in distribution**   how many regulators each target has (spikes at the high end flag over-connected / "too many source nodes" targets).
- **Activator vs inhibitor balance**   count of activating vs inhibiting edges, with the ratio.
- **Top regulators**   the genes appearing in the most rules (by out-degree).

### 2. Inspect gene
Type any target gene and see:
- Its classification (real rule / undefined / too many / constant) and fan-in.
- The raw rule, syntax-highlighted.
- The rule drawn as a **logic ladder**   activator (green) and inhibitor (red) literals grouped into `AND` terms, joined by `OR`.
- Separate **activator / inhibitor** lists (and a "context" list for regulators that appear both ways).
- What the gene **regulates downstream**. Every gene name is clickable to jump to it.

### 3. Subnetwork
Centre on any gene and see its neighbourhood:
- **Regulators (incoming)** on the left, **targets it regulates (outgoing)** on the right.
- Edges coloured **green = activates**, **red = inhibits**; arrows point at the regulated gene.
- Click any node to recentre on it.

---

## How to use

1. Open the tool. It starts on a small **built-in sample** so you can see how it looks.
2. Click **Load rules file…** and choose your `.txt` file (or **Paste** rules directly, or **Sample** to restore the demo).
3. Use the tabs to switch between **Overview / QC**, **Inspect gene**, and **Subnetwork**.
4. The **◐** button toggles light / dark theme.

---

## Input format

One rule per line, `Target = expression`:

```
RAD52 = ABL1 && WRN
CDK1  = CDC25C || MYBL2 && PTTG1 || CDC25A && E2F1
CFH   = !SIRT1
SEMA3F = Undefined
CFLAR  = Too many source nodes
```

Supported syntax:

| Element        | Written as                                  |
|----------------|---------------------------------------------|
| AND            | `&&` (or `&`)                               |
| OR             | `\|\|` (or `\|`)                            |
| NOT            | `!` (e.g. `!SIRT1`)                          |
| Gene names     | letters, digits, `_`, `-`, `.`              |
| No rule        | `Undefined`, `Too many source nodes`        |
| Constants      | `0` / `False`, `1` / `True`                 |
| Grouping       | parentheses                                 |

Blank lines and lines starting with `#` are ignored. Expressions are read as
disjunctive normal form (OR of AND-terms); a regulator written `!X` is treated
as an inhibitor of that target, otherwise an activator.

---

## Notes

- No install, no server, no build step   a single self-contained HTML file.
- Works offline (fonts fall back to system typefaces without a connection).
- Tested on full TISON rule exports (2,000+ targets).
