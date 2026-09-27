# legal-engineer-portfolio
Portfolio for Legal Engineer Application
# Civil Law Ground Truth Curator

## The Problem

Legal AI cannot be evaluated without gold-standard data. Most benchmarks for
legal AI are either generic (BLEU scores, MMLU-style trivia) or Anglo-centric
(US case law, UK statutory interpretation). Neither tells you whether a system
produces a legally sound answer for a specific article of UAE Federal Labour
Law.

This project is a methodology for curating jurisdiction-specific ground truth
and measuring whether a legal AI system stays within the source.

## The Article

**Article 9 of UAE Federal Decree-Law No. 33 of 2021 (Probation Period).**

I chose this article for four reasons:

1. It has six clauses, each with a distinct rule, giving reasonable coverage.
2. It contains rules that are easy to conflate. Clause 1 and Clause 4 both
   use a 14-day notice period, but for different things — employer termination
   versus a foreign worker leaving the country.
3. It is silent on whether probation can be extended. The article caps the
   period at six months and prohibits repeat probation with the same employer,
   but says nothing about extension. This is exactly the kind of gap where
   legal AI systems hallucinate rules that do not exist.
4. It sits within a jurisdiction (UAE Federal) that generic legal AI systems
   are notoriously bad at, because their training data skews Anglo-American.

The corpus is derived from the official text. The gazette PDF is in `data/`.

## Methodology

Eight gold-standard question-answer pairs, each with:

- The question in English and Arabic
- The ideal answer in both languages
- A citation to the specific clause
- A weighted rubric that defines what "legally sound" means for that question

Three of the eight questions are **traps**. Trap questions test whether the AI
says "the article does not address this" rather than inventing a rule. The
three traps are:

- **qa_003** — Can probation be extended? (Article is silent.)
- **qa_007** — What happens if a foreign worker leaves during probation
  without proper notice? (Requires finding the one-year work permit ban in
  Clause 6, which is buried.)
- **qa_008** — What notice must a worker give to move to another UAE employer
  during probation? (Clause 3 uses a 14-day notice, distinct from the 14-day
  employer notice in Clause 1.)

Each question was run through **two conditions**:

- **RAG condition** — retrieval over the six clauses, then generation
- **Baseline condition** — no source text, model answers from prior knowledge

A second LLM (Llama via Groq, `openai/gpt-oss-120b`) scored every answer
against the rubric. The judge saw the article text as the source of truth.

## Results

| Metric | RAG (grounded) | Baseline (no source) | Gap |
|---|---|---|---|
| Average rubric score | **88.9%** | 42.5% | **+46.4 pts** |
| Hallucination rate | **12.5%** (1/8) | 87.5% (7/8) | — |
| Trap question average | **81.5%** | 41.0% | **+40.5 pts** |
| Non-trap question average | 93.3% | 43.3% | +50.0 pts |

Full per-question breakdown:

| ID | Clause | Trap | RAG | Base |
|---|---|---|---|---|
| qa_001 | 1 | — | 100.0% | 33.3% |
| qa_002 | 1 | — | 100.0% | 66.7% |
| qa_003 | 2 | yes | 44.4% | 44.4% |
| qa_004 | 2 | — | 100.0% | 0.0% |
| qa_005 | 2 | — | 100.0% | 33.3% |
| qa_006 | 4 | — | 66.7% | 83.3% |
| qa_007 | 6 | yes | 100.0% | 28.6% |
| qa_008 | 3 | yes | 100.0% | 50.0% |

## Findings

### 1. Ungrounded legal AI fails on UAE labour law almost every time

The baseline answered 7 of 8 questions with invented or incorrect rules. It
scored only 42.5% against a rubric that rewards staying within the source.

The failure modes were specific. On qa_001, the baseline answered correctly
that the maximum probation period is six months, then cited "Article 33" —
which does not exist. It fabricated a citation while getting the substantive
rule right. That is the most dangerous kind of hallucination for a lawyer,
because the answer looks correct until the citation is checked.

On qa_004, the baseline answered that an employer may place an employee on
probation more than once. The article says the opposite. The model did not
hedge. It stated an incorrect rule as fact.

### 2. Grounding cuts hallucination from 87.5% to 12.5%

Retrieval over the six clauses dropped hallucination by a factor of seven.
Every non-trap question except qa_006 scored 100% under the RAG condition.
The trap questions that the baseline failed — qa_007 and qa_008 — both scored
100% under RAG, which suggests the model can find buried clauses when the
source is in front of it.

### 3. The single RAG failure (qa_003) is the most interesting result

On qa_003 — "Can probation be extended?" — both conditions failed. The RAG
system answered:

> "No. Under Article 9, Clause 1 the employer may appoint a worker to
> probation for a period not exceeding six months, and the worker may be
> placed on probation only once with the same employer..."

The model treated the article's silence on extension as an implicit
prohibition. It converted "the article does not say X" into "the article
prohibits X." That is a category error, and the rubric penalized it.

This is the failure mode I would flag first to a product team. It is not
a retrieval failure — the model had the right clause. It is not a
hallucination in the conventional sense — the model did not invent a rule.
It is a reasoning failure: the model does not know how to represent
legislative silence, and defaults to a confident binary answer because
that is what the prompt structure rewards.

### 4. One judge inconsistency, logged honestly

On qa_004, the baseline answer was flatly wrong but the judge did not flag it
as a hallucination. It scored the answer 0.0% (correct) but marked
`hallucination=False`. This is a limitation of using a single LLM as judge:
it can detect that an answer fails the rubric without agreeing on whether
the failure constitutes hallucination. A production evaluation harness would
use a panel of judges or a human review layer for trap questions.

## Why This Matters for HAQQ

Two implications for the Justinian engine's ground-truth curation:

1. **Jurisdiction-specific rubrics are necessary.** A generic benchmark would
   have scored the baseline answer to qa_001 highly, because it stated the
   correct rule. Only the rubric — which weighted the fabricated citation as
   a failure — caught the problem.

2. **The hardest failure mode is not hallucination.** It is the model's
   inability to represent silence. A legal AI system that cannot say
   "this article does not address your question" is more dangerous than one
   that occasionally invents a rule, because silence is common in statute
   and confident wrong answers are hard to spot.

This is a six-clause article with eight questions. A production evaluation
harness would need hundreds. But the methodology scales: curate the source,
define what legally sound means, measure both conditions, and log the
failures honestly.

## How to Run

Open `notebooks/ground_truth_curator.ipynb` in Google Colab. You need a free
Groq API key (console.groq.com) stored in Colab Secrets as `GROQ_API_KEY`.

## Files

- `notebooks/ground_truth_curator.ipynb` — the full evaluation
- `data/article_9.json` — the gold-standard data
- `data/fed_decree_law_33_2021.pdf` — the source gazette
- `eval_results.csv` — raw output from the final run
- `eval_summary.json` — aggregated metrics
