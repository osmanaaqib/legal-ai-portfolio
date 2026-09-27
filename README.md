# Legal Engineer Portfolio

This repository is my application for the Legal Engineer role at HAQQ.

It contains three projects, each addressing a specific problem I believe
HAQQ's Justinian engine must solve to scale across MENA civil-law jurisdictions.

## The Projects

### 01 — Civil Law Ground Truth Curator

A methodology for curating jurisdiction-specific gold-standard data and
evaluating whether a legal AI system stays within its source.

Built around Article 9 of UAE Federal Decree-Law No. 33 of 2021 (Probation
Period). Eight questions, three of them traps, run through two conditions:
grounded retrieval versus a no-source baseline. A second LLM scores every
answer against a weighted legal rubric.

**Result:** grounded retrieval cut hallucination from 87.5% to 12.5%. The
single remaining failure was not a hallucination — it was a reasoning failure
where the model could not represent legislative silence.

### 02 — MENA Jurisdiction Router

A confidence-aware retrieval system that routes legal queries across Federal,
DIFC, and ADGM regimes without silently guessing when a query is ambiguous.

**Result:** baseline retrieval leaked across jurisdictions on 73.6% of
queries. A metadata router reduced that to zero, without touching the model.


### 03 — Legal AI Twin Simulator

Two synthetic firms with opposing risk postures. Same contract clause
submitted to both. Each twin retrieves its own playbook and produces a
redline recommendation. Measures whether each twin stays faithful to its
playbook, and whether the two twins actually diverge.

**Result:** overall adherence 77.5%. Divergence is moderate rather than
strong, and one twin's reasoning collapsed on a single clause. The
findings document where the concept holds and where it breaks.

### 04 — TIRO Multi-Agent Contract Redliner

A 27-agent legal reasoning pipeline across 6 rounds. Produces redline
recommendations for contract clauses and compares against a single-pass
baseline.

**Result:** the pipeline tripled output length and increased proposed
track changes by up to 3.3x. Citations stayed low because the pipeline
had no legal corpus to cite from — an honest finding about what
multi-agent decomposition does and does not automatically produce.


## Why I Built This

I come from government litigation in South Africa. I trained at the Office of
the State Attorney, which handles litigation for every provincial department
in KwaZulu-Natal — the largest legal institution in the country. That work
taught me to reason structurally about legal problems: identify the controlling
rule, the applicable jurisdiction, and the facts that actually matter.

When I moved into AI work, I noticed that most legal AI systems fail at exactly
the point where that reasoning matters. They retrieve plausible text. They do
not reason about jurisdiction. They do not know what a "legally sound" answer
looks like in a specific regime. And they cannot be evaluated, because nobody
has defined what "correct" means for a given jurisdiction.

These three projects are my attempt to address that. They are not a product.
They are a methodology — and a demonstration that I can execute on it.

## What Each Project Demonstrates

| Project | HAQQ requirement | Key artifact |
|---|---|---|
| Ground Truth Curator | Curate legal knowledge and define AI ground truth | Rubric-scored evaluation with baseline comparison |
| Jurisdiction Router | Translate legal processes into product logic | Leak-rate benchmark across Federal / DIFC / ADGM |
| Legal AI Twin | Bridge Legal, Product and AI teams | Firm-specific playbook encoding |

## How to Read This Repository

Each project folder contains:

- `README.md` — the problem, approach, and results
- `data/` — inputs: the source legislation, the gold-standard data
- `notebooks/` — runnable Colab notebooks
- Output files at the project root (CSV, JSON) — the raw results

Start with `01-civil-law-ground-truth-curator/`. It is the centerpiece.

## Contact

Aaqib Osman
osmanaaqib@gmail.com
linkedin.com/in/aaqib-osman-aa69b4215 
