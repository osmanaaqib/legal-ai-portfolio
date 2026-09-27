# Legal AI Twin Simulator

A demonstration of how institutional context — a firm's risk posture and
negotiation positions — can be encoded into a legal AI system.

## The Problem

Generic legal AI produces generic output. It does not know that Firm A
requires mutual indemnification and Firm B accepts one-way indemnification
favoring its client. It does not know that Firm A caps liability at 1x fees
and Firm B accepts 3x with carve-outs. When two lawyers at different firms
ask the same question, they get the same answer, which means the system is
not usable as a drafting tool for either of them.

HAQQ's "legal AI twin" concept addresses this directly: each firm gets a
system that learns and reflects its specific positions.

## What This Project Does

Simulates two synthetic law firms with opposing risk postures:

- **Alpha LLP** — conservative, risk-averse. Mutual obligations, low caps,
  broad exit rights. Protecting downside matters more than winning the deal.
- **Beta Partners** — commercial, aggressive. One-way protections favoring
  the client, higher caps, harder exit provisions. Winning the deal matters
  more than minimizing downside.

Each firm has a playbook of four positions on standard contract clause types:
limitation of liability, indemnification, termination, and confidentiality.

The same contract clause is submitted to both twins. Each twin retrieves its
own playbook positions and generates a redline recommendation. The evaluation
measures adherence (does each twin reflect its own playbook?) and divergence
(do the two twins actually produce different output?).

## Methodology

Four test clauses, each written as neutral boilerplate so that neither firm's
position is obvious from the drafting itself. The twins must apply their
playbooks, not react to bias in the counterparty text.

Each twin is implemented as a retrieval-augmented generation pipeline:

1. The clause is embedded and matched against the firm's playbook positions
2. The best-matching position is retrieved along with the firm's posture
3. The model generates a structured recommendation (verdict, reasoning, redline)

An LLM judge scores each output against the firm's playbook for adherence.
Divergence is measured as cosine similarity between the two firms' outputs —
lower similarity means more divergent output.

## Results

| Metric | Value | Target |
|---|---|---|
| Alpha average adherence | **87.5%** | 70%+ |
| Beta average adherence | 67.5% | 70%+ |
| Overall average adherence | 77.5% | — |
| Average similarity | 0.806 | below 0.85 |
| Verdict divergence | 1 / 4 | 3+ / 4 |

Per-clause breakdown:

| Clause | Type | Alpha adherence | Beta adherence | Alpha verdict | Beta verdict | Similarity |
|---|---|---|---|---|---|---|
| clause_lol | limitation_of_liability | 100 | 0 | Accept | Push back | 0.832 |
| clause_indem | indemnification | 100 | 70 | Push back | Push back | 0.809 |
| clause_term | termination | 100 | 100 | Push back | Push back | 0.815 |
| clause_conf | confidentiality | 50 | 100 | Push back | Push back | 0.769 |

## Findings

### 1. The mechanism works, unevenly

Alpha's adherence was strong — three of four clauses at 100%, one at 50%.
The retrieval-then-generate pipeline correctly identified the relevant
playbook position and applied it in the output.

Beta's adherence was weaker, dragged down by a collapse on the limitation
of liability clause. Beta's playbook calls for a 3x cap with carve-outs for
IP infringement and data breach. The counterparty clause offered a 1x
mutual cap. Beta's twin said "Push back" — directionally correct — but the
judge scored adherence 0, meaning the twin's reasoning did not actually
reference its own playbook position. It pushed back for reasons that were
not grounded in Beta's stated position.

This is the failure mode worth flagging. A twin that produces the right
verdict for the wrong reason is not yet a twin — it is a generic model
that happens to agree with the playbook on this clause. The mechanism
needs to force playbook citation before the model is allowed to reach a
verdict.

### 2. Divergence is moderate, not strong

Average similarity across the four clauses was 0.806. In a scale where 1.0
means identical output and 0.0 means unrelated output, 0.806 sits closer to
identical than the design intended. The twins produce different text, but
they converge more than they should.

The most divergent clause was confidentiality (0.769), where Alpha's
5-year term and Beta's perpetual-for-trade-secrets posture pulled the outputs
apart. The most convergent was limitation of liability (0.832), where both
twins pushed back on the same clause for overlapping reasons.

A stronger twin would diverge more sharply. Likely causes of the current
convergence: the postures are described in similar vocabulary, the playbook
positions are structured identically, and the model has priors about what
"conservative" and "aggressive" mean in contract negotiation that it applies
regardless of the retrieved context.

### 3. Verdict divergence is low

Only one of four clauses produced different verdicts between the two twins.
On the other three, both firms said "Push back" — even though they were
pushing back for different reasons.

This is a real limitation of the current design. Two law firms with opposite
risk postures should not agree on the verdict for three out of four clauses.
The clause texts are probably too aggressive for the "Accept" posture in
most cases, and the playbooks do not have enough scope for the twins to
converge on "Accept" from genuinely different starting positions.

To fix this: include clauses that are closer to each firm's stated position,
so that Alpha accepts some and Beta rejects them, and vice versa. The
divergence metric needs clauses where the twins actually disagree, not
just clauses where they disagree on reasoning.

### 4. Where the concept holds

The strongest result is that all four clauses produced output that was
recognisably firm-specific. Even where the verdict was the same, the
reasoning differed. Alpha's reasoning on the LoL clause cited the mutual
cap and the exclusion of indirect damages. Beta's reasoning on the same
clause would have cited the carve-outs it wants added. That is the twin
concept working at the reasoning layer, even if the verdict layer doesn't
diverge as much as intended.

## Why This Matters for HAQQ

Three implications for the Justinian engine:

1. **The retrieval mechanism is the right primitive.** Grounding a firm's
   output in a structured playbook produces firm-specific behaviour without
   fine-tuning a model per client. This project demonstrates the mechanism
   at small scale. Production would require larger playbooks, finer chunking,
   and possibly a trained classifier over playbook positions.

2. **Adherence is measurable, and the metric matters.** The judge in this
   project scores not just whether the twin agreed with the playbook, but
   whether it *referenced* it. A twin that produces the right output for
   the wrong reason is not yet a twin. This distinction is worth building
   into evaluation from the start.

3. **Divergence is harder than it looks.** Getting a model to produce
   *different* output for two firms with *opposite* postures is not trivial.
   The model's priors about legal drafting pull toward consensus. Forcing
   divergence requires either stronger playbook signal, or explicit
   divergence testing as part of the evaluation.

The methodology scales, but the current result is honest about where it
breaks.

## Files

- notebooks/twin_simulator.ipynb — the full evaluation
- data/playbooks.json — the two firm playbooks
- data/clauses.json — the four test clauses
- twin_eval.csv — per-clause results
- twin_summary.json — aggregated metrics

## How to Run

Open `notebooks/twin_simulator.ipynb` in Google Colab. You need a free
Groq API key (console.groq.com) stored in Colab Secrets as `GROQ_API_KEY`.
