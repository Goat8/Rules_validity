# GRN Rule Inspector

A lightweight, browser-based tool to **visualise and quality-check Boolean gene-regulatory rule files**: the kind exported by TISON and similar Boolean network builders.

Two pages, one repo:

| Page                         | Use it to                                                          | Link                                                |
| ---------------------------- | ------------------------------------------------------------------ | --------------------------------------------------- |
| **Inspector** (`index.html`) | Audit one rules file: QC overview, per-gene logic, subnetwork      | https://goat8.github.io/Rules_validity/             |
| **Compare** (`compare.html`) | Check that a **refactored** rules file reproduces the **original** | https://goat8.github.io/Rules_validity/compare.html |

Both run entirely in your browser. The files you open are parsed locally and **never uploaded anywhere**, so they are safe to use on unpublished data.

---

## Compare: original vs refactored rules

Drop the original `rules.txt` into the **Original** box and the refactored one into **Refactored**. Rules are matched **by logic, not by text**, so `A || B` and `B || A`, or `!A && !B` and `!(A || B)`, count as a match.

Every target gets one status:

| Status                            | Meaning                                                                                                                                                                 |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Identical**                     | Same rule text (or the same `Undefined` / `Too many source nodes` marker)                                                                                               |
| **Equivalent**                    | Same logic, written differently (reordered, De Morgan, redundant terms). Proven by normalising both rules, or by checking every input combination (up to 12 regulators) |
| **Different**                     | The logic differs. Shows regulators added / removed / sign-flipped, the % of inputs where the two agree, and example inputs where they diverge                          |
| **Unverified**                    | More than 12 regulators and no difference in 2,048 sampled inputs, but not provable. Not counted as a match                                                             |
| **Rule lost / gained**            | One side has a real rule, the other has `Undefined` or `Too many source nodes`                                                                                          |
| **Marker changed**                | `Undefined` on one side, `Too many source nodes` on the other                                                                                                           |
| **Only in original / refactored** | Target appears in just one file                                                                                                                                         |

The top of the page gives the headline (“all N rules match” or “X of N differ”) and a status bar. Filter by status, search by gene, click any row for the side-by-side logic, and **Export CSV** for the full table.

### Coverage map and regulator overlap

For the genes that are targets in both files, the page groups each one (click a group to list it):

| Group                                      | Meaning                                                        |
| ------------------------------------------ | -------------------------------------------------------------- |
| **Both have a real rule**                  | Compared further for regulator overlap (below)                 |
| **Rescued: original Undefined / Too many** | The refactored file has a real rule where the original gave up |
| **Only original has a rule**               | The refactored file has no real rule                           |
| **Neither has a rule**                     | Undefined, Too many source nodes or a constant on both sides   |

A rule that is only `0` or `1` is a constant and does not count as a real rule.

Where both have a real rule, it reports the **Jaccard overlap** of the two regulator sets (mean, median, and how many are the same / partly overlapping / disjoint). Overlap ignores gates and signs, so it also counts rules with the **same regulators but different logic** (for example `A && B` vs `A || B`). For a refactor the regulators should be the same; low overlap is expected only when comparing a different method.

### Is a mismatch caused by missing nodes?

A **node** is any gene named in a file, as a target or as a regulator. Every mismatch is tagged with a cause. For a firm answer, also add each run's **`Cytoscape.csv`** (optional): it lists the regulators available to every gene, so _different inputs_ can be told apart from _different code_.

| Cause                                                | From the rules files alone                                                                                             | With both `Cytoscape.csv` files                                                                      |
| ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Missing nodes**                                    | The rules agree once the absent genes are held at 0 or 1 (_Different_), or every regulator is absent (_Lost / gained_) | Every change in the gene's available regulators involves an absent gene                              |
| **Partly missing nodes**                             | Absent genes are involved, but the logic differs beyond that (or over 12 regulators)                                   | Some changed regulators are absent genes, some are not                                               |
| **Edges changed**                                    | n/a                                                                                                                    | Available regulators changed, but all exist in both networks                                         |
| **Not node-related** / **Same inputs, rule changed** | Every regulator named in the rule exists in both networks                                                              | Same available regulators in both runs, so the code or other data (e.g. expression) changed the rule |
| **Can't tell from rules**                            | One side is `Too many source nodes`, which lists no regulators                                                         | Resolved                                                                                             |

_Only in one file_ counts as missing nodes when the gene is not a node in the other file at all.

With both edge files, the page also reports the **same inputs, same rule** check: of the genes with identical available regulators in both runs, how many rules match. That is the fair test of the refactored code. Genes whose inputs changed cannot test it; rerun both pipelines on the same input.

The page warns if an edge file doesn't fit its rules file (it may belong to another run). `NKX2-1` in `Cytoscape.csv` and `NKX2_1` in rules files are treated as the same gene. **Export CSV** includes the coverage group, overlap, cause and input changes.

If the refactor is faithful you should see **100%**, with every row _Identical_ or _Equivalent_ and nothing under _Only in original / refactored_.

---

## What it does

Load a rules `.txt` file and get three views:

### 1. Overview / QC

A health check on the whole rule set:

- **Summary tiles** — total targets, real rules (with %), Undefined, "too many sources", edge count, mean and max fan-in.
- **Rule-type composition** — a single bar splitting every target into _real rule / too many sources / constant = 0 / constant = 1 / undefined_, so degenerate and missing rules are visible at a glance rather than hidden inside a coverage number.
- **Fan-in distribution** — how many regulators each target has (spikes at the high end flag over-connected / "too many source nodes" targets).
- **Activator vs inhibitor balance** — count of activating vs inhibiting edges, with the ratio.
- **Top regulators** — the genes appearing in the most rules (by out-degree).

### 2. Inspect gene

Type any target gene and see:

- Its classification (real rule / undefined / too many / constant) and fan-in.
- The raw rule, syntax-highlighted.
- The rule drawn as a **logic ladder** — activator (green) and inhibitor (red) literals grouped into `AND` terms, joined by `OR`.
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

| Element    | Written as                           |
| ---------- | ------------------------------------ |
| AND        | `&&` (or `&`)                        |
| OR         | `\|\|` (or `\|`)                     |
| NOT        | `!` (e.g. `!SIRT1`)                  |
| Gene names | letters, digits, `_`, `-`, `.`       |
| No rule    | `Undefined`, `Too many source nodes` |
| Constants  | `0` / `False`, `1` / `True`          |
| Grouping   | parentheses                          |

Blank lines and lines starting with `#` are ignored. Expressions are read as
disjunctive normal form (OR of AND-terms); a regulator written `!X` is treated
as an inhibitor of that target, otherwise an activator.

---

## Notes

- No install, no server, no build step — a single self-contained HTML file.
- Works offline (fonts fall back to system typefaces without a connection).
- Tested on full TISON rule exports (2,000+ targets).
