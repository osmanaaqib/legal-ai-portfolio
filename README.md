# Legal AI Portfolio

A collection of independent projects exploring document intelligence,
retrieval-augmented generation, and legal reasoning in AI systems.

These projects were built to understand the infrastructure that sits underneath
serious legal AI. How ground truth gets defined, how jurisdictions get routed,
how institutional context gets encoded, and how multi-agent pipelines produce
better work product than single-pass systems.

## Projects

### 01 — Civil Law Ground Truth Curator

A methodology for curating jurisdiction-specific gold-standard data and
evaluating whether a legal AI system stays within its source. Built around
Article 9 of UAE Federal Decree-Law No. 33 of 2021.

**Finding:** grounded retrieval cut hallucination from 87.5% to 12.5%. The
single remaining failure was a reasoning failure where the model could not
represent legislative silence.

### 02 — MENA Jurisdiction Router

A confidence-aware retrieval system that routes legal queries across Federal,
DIFC, and ADGM regimes without silently guessing when a query is ambiguous.

**Finding:** baseline retrieval leaked across jurisdictions on 73.6% of queries.
A metadata router reduced that to zero without touching the model.

### 03 — Legal AI Twin Simulator

Two synthetic firms with opposing risk postures. Same contract clause submitted
to both. Each twin retrieves its own playbook and produces a redline
recommendation.

**Finding:** overall adherence 77.5%. Divergence is moderate rather than strong,
and one twin's reasoning collapsed on a single clause. The findings document
where the concept holds and where it breaks.

### 04 — TIRO Multi-Agent Contract Redliner

A 27-agent legal reasoning pipeline across 6 rounds. Produces redline
recommendations for contract clauses and compares against a single-pass baseline.

**Finding:** the pipeline tripled output length and increased proposed track
changes by up to 3.3x. Citations stayed low because the pipeline had no legal
corpus to cite from — an honest finding about what multi-agent decomposition
does and does not automatically produce.

## Why this exists

I come from government litigation in South Africa. I trained at the Office of
the State Attorney, the country's largest legal institution. That work taught
me to reason structurally about legal problems, identify the controlling rule,
the applicable jurisdiction, and the facts that actually matter.

When I moved into AI work, I noticed that most legal AI systems fail at exactly
the point where that reasoning matters. These projects are my attempt to
understand why, and to build the smallest possible interventions that fix it.

## Contact

Aaqib Osman
osmanaaqib@gmail.com
linkedin.com/in/aaqib-osman-aa69b4215
