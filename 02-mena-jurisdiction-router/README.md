# MENA Jurisdiction Router

## The Problem

The UAE has three overlapping legal systems: onshore Federal law, the DIFC
common-law free zone in Dubai, and the ADGM common-law free zone in Abu Dhabi.
A legally correct DIFC answer can be wrong for an onshore dispute. Generic
retrieval-augmented generation has no concept of this boundary. It returns the
most semantically similar text regardless of jurisdiction, silently mixing
regimes.

In legal work, a correct answer from the wrong jurisdiction is worse than no
answer. It looks right, cites a real provision, and produces advice that does
not apply.

This project measures that failure mode and demonstrates the smallest possible
intervention that eliminates it.

## What This Project Does

Builds a small corpus of UAE legal text across all three regimes and two
retrieval configurations:

- **Baseline** — semantic search over the full corpus, no jurisdiction filter.
  This is what a naive RAG system does.
- **Routed** — a confidence-aware router classifies the query by jurisdiction,
  then restricts retrieval to that regime. When the query is genuinely
  ambiguous, the router returns low confidence and searches across all
  regimes rather than guessing.

The metric is **Jurisdiction Leak Rate**: the fraction of retrieved documents
whose jurisdiction differs from the query's expected jurisdiction.

## Methodology

Six documents across three jurisdictions (two per regime), sourced from:

- UAE Federal Decree-Law No. 33 of 2021 (onshore)
- DIFC Employment Law No. 2 of 2019 (Dubai common-law free zone)
- ADGM Employment Regulations 2021 (Abu Dhabi common-law free zone)

Six evaluation queries, one per jurisdiction plus one deliberately ambiguous
query that mentions both DIFC and ADGM. The ambiguous query tests whether the
router refuses to guess when jurisdiction is unclear.

The router uses keyword markers per jurisdiction (e.g., "DIFC", "مركز دبي
المالي", "ADGM", "سوق أبوظبي العالمي") with Arabic normalization applied to
both markers and queries so that alef variants and diacritics do not break
matching. Federal is the default only when no free-zone marker is present.

## Results

| Metric | Baseline | Routed | Change |
|---|---|---|---|
| Average jurisdiction leak | **73.6%** | **0.0%** | — |
| Routing accuracy (non-ambiguous) | n/a | 100% (5/5) | — |
| Ambiguous queries correctly flagged | n/a | 100% (1/1) | — |

Per-query breakdown:

| Query | Expected | Routed | Confidence | Baseline leak | Routed leak |
|---|---|---|---|---|---|
| Probation period in onshore UAE | Federal | Federal | medium | 0.67 | 0.0 |
| فترة التجربة في مركز دبي المالي العالمي | DIFC | DIFC | high | 0.67 | 0.0 |
| ADGM probation period regulations | ADGM | ADGM | high | 0.67 | 0.0 |
| Data protection across DIFC and ADGM | ambiguous | ambiguous | low | — | — |
| Transfer personal data outside UAE | Federal | Federal | medium | 1.00 | 0.0 |
| DIFC data transfer adequacy | DIFC | DIFC | high | 0.67 | 0.0 |

## Findings

### 1. Baseline retrieval contaminates every query

Five of five non-ambiguous queries leaked across jurisdictions. Four leaked
at 0.67, meaning two of the three retrieved documents came from the wrong
regime. One — "Can I transfer personal data outside the UAE?" — leaked at
1.00. All three retrieved documents were wrong. The correct Federal PDPL
provision did not appear in the top three.

That is the failure mode that matters most. The baseline was not just
imperfect; it was confidently wrong. A user reading the retrieved text would
have seen DIFC and ADGM provisions about data transfer, believed they were
the answer, and applied rules that do not govern the situation.

### 2. The router eliminates the leak entirely

The routed retrieval leaked at 0.0 across every non-ambiguous query. The
mechanism is simple: a metadata filter on jurisdiction, applied before
semantic search. No prompt engineering, no fine-tuning, no model change.
Six documents, six queries, one filter.

### 3. Ambiguity handling works, and the accuracy metric understates it

The ambiguous query — "Data protection rules across DIFC and ADGM" — mentions
both free zones. The router detected one DIFC marker and one ADGM marker,
could not determine which regime the user meant, returned confidence `low`,
and set the filter to `None`. Retrieval then searched across all three
regimes and labelled every result with its jurisdiction.

This is the correct behaviour. A legal AI system that guesses when the query
is genuinely ambiguous will sometimes guess wrong, and the user will not know.
Refusing to filter, and surfacing every result with its jurisdiction label,
is safer.

The mechanical accuracy metric reports 83% because it counts the ambiguous
query as a miss — "ambiguous" does not equal "Federal", which is the label
the router returns as its default fallback. Non-ambiguous accuracy is 100%.
The README reports both numbers because the metric is honest about what it
measures, and what it measures is not the same as what the router is doing.

### 4. Arabic normalization is necessary, not optional

The Arabic DIFC query — "فترة التجربة في مركز دبي المالي العالمي" — was the
second-strongest test case. Without normalization of alef variants and ta
marbuta, the marker "مركز دبي المالي" would not have matched after the query
was normalized. The router would have missed the DIFC signal entirely and
defaulted to Federal, which would have been wrong.

Arabic legal retrieval requires Arabic-first preprocessing. Embedding models
trained on English do not handle this by default.

## Why This Matters for HAQQ

Two implications for the Justinian engine:

1. **Jurisdiction routing is the cheapest intervention that matters most.**
   A metadata layer — jurisdiction labels on every chunk, a router that
   classifies queries before retrieval — is a few hundred lines of code. It
   eliminates the failure mode that generic retrieval exhibits on every
   query in this evaluation. The return on effort is disproportionate.

2. **Refusing to guess is a feature, not a limitation.** Legal AI systems
   that produce confident answers to ambiguous questions are dangerous. The
   router's `low` confidence + `None` filter behaviour is a model for how
   the Justinian engine should handle jurisdictional ambiguity in multi-regime
   markets like the UAE, Qatar, and Saudi Arabia.

The methodology scales. This is six documents and six queries. A production
system would need hundreds of thousands of documents and jurisdiction labels
across federal law, emirate-level law, and every free zone. But the routing
logic — metadata filter, confidence score, ambiguity handling — is the same.

## Files

- notebooks/jurisdiction_router.ipynb — the full evaluation
- data/corpus.json — the jurisdiction-labelled corpus
- jurisdiction_eval.csv — raw per-query results
- jurisdiction_summary.json — aggregated metrics

## How to Run

Open `notebooks/jurisdiction_router.ipynb` in Google Colab. You need a free
Groq API key (console.groq.com) stored in Colab Secrets as `GROQ_API_KEY`.
