# TIRO Multi-Agent Contract Redliner

A 27-agent legal reasoning pipeline that produces redline recommendations
for commercial contract clauses. Built to test whether decomposing a legal
task into coordinated specialized agents produces meaningfully better work
product than a single-pass LLM call.

## What This Is

Single-pass LLM: one prompt, one response, one agent. Produces a plausible
redline that misses details.

TIRO multi-agent pipeline: 27 specialized agents across 6 sequential rounds,
each with a narrow role, orchestrated to produce a structured deliverable
that survives partner-level critique.

The architecture follows the TIRO decomposition pattern (Trigger, Input,
Requirements, Output) and the multi-agent legal reasoning design described
in HAQQ's engineering blog.

## Architecture

**Round 1 — Input parsing (4 agents, parallel):**
Segmenter, entity extractor, clause type classifier, jurisdiction detector.

**Round 2 — Analysis (8 agents, parallel):**
Liability cap analyzer, carve-out detector, consequential damages analyzer,
mutuality analyzer, scope limiter analyzer, ambiguity detector, missing
provision detector, quantitative exposure analyzer.

**Round 3 — Playbook application (6 agents, parallel, 3 per firm):**
Position matcher, fallback matcher, priority scorer — run separately for
Alpha LLP and Beta Partners so the two twins diverge.

**Round 4 — Synthesis (4 agents, sequential per firm):**
Redline drafter, negotiation strategist, risk scorer, priority ranker.

**Round 5 — Verification (2 agents):**
Citation verifier, consistency checker.

**Round 6 — Critique and revision (2 agents):**
Partner simulator, revision agent.

Total: 27 agents, 6 rounds.

## Baseline

The pipeline is compared against a single-pass baseline: same model, same
clause, same playbook, one prompt instead of 27 agents.

## Results

| Metric | Baseline | Pipeline | Change |
|---|---|---|---|
| Output length (Alpha) | 2,432 chars | 6,302 chars | **+159%** |
| Output length (Beta) | 2,599 chars | 6,628 chars | **+155%** |
| Track changes (Alpha) | 3 | 5 | +67% |
| Track changes (Beta) | 3 | 10 | **+233%** |
| Citations (Alpha) | 0 | 1 | — |
| Citations (Beta) | 0 | 0 | — |
| Alpha–Beta similarity | 0.709 | 0.746 | +0.037 |

## Findings

### 1. The pipeline produces substantially more work product

Output length roughly tripled for both firms. Track changes — the metric
that matters most for a legal deliverable — increased for both, with Beta
showing a 3.3x increase (3 → 10). This is the headline result and it
matches the directional claim in HAQQ's published benchmark.

### 2. Citation behaviour is a design choice, not an architectural property

HAQQ's published benchmark reports 18 legal citations from their multi-agent
pipeline versus zero from single-pass. This pipeline produced 1 and 0.

The difference is not architectural. It is a design choice about what the
agents are asked to do. HAQQ's agents have access to their Justinian legal
ontology and are instructed to cite from it. My agents had access only to
the clause and the firm playbook, neither of which contains external legal
citations. The pipeline could not cite what it was not given.

The honest conclusion: multi-agent decomposition does not automatically
produce citation-grounded output. That requires (a) a legal corpus in the
retrieval layer and (b) explicit instruction to every relevant agent to
cite. Without both, the pipeline produces playbook-grounded work product,
which is useful but different.

### 3. The architecture homogenized the output format

This is the most interesting finding and the one that contradicts the
design intent.

Alpha and Beta became *more* alike under the pipeline (similarity 0.709 →
0.746). They were supposed to become more divergent. Both firms produced
output in the same markdown table format, with the same three columns:
"What to change / Proposed wording / Why".

The cause: the agent prompts imposed a shared output format on both firms.
The Redline Drafter agent in Alpha and the Redline Drafter agent in Beta
were given identical system prompts except for the firm name. The
architecture imposed structural uniformity even as the firm postures
differentiated the content.

This is a real trade-off. Multi-agent decomposition increases the depth and
breadth of analysis. It also increases the risk of homogenization, because
every additional agent is another opportunity to impose a shared format.

### 4. Where the pipeline genuinely wins

Despite the citation and divergence limitations, the pipeline produced
something single-pass could not: structured deliverables with explicit
reasoning at each step. The Alpha output, for example, identified that the
cap was tied to the Customer's fees rather than the Supplier's, which creates
an asymmetric exposure. That observation appears in the pipeline output and
not in the baseline. It is the kind of finding that requires the analysis
agents from Round 2 to isolate a specific dimension and flag it explicitly.

## What Would Fix the Limitations

Two changes would address the two limitations honestly:

1. **Citation grounding.** Add a legal corpus to Round 2, indexed by
   jurisdiction. Instruct every agent that produces prose to cite specific
   provisions when making claims about enforceability or statutory
   requirements.

2. **Divergence enforcement.** Give Alpha's agents and Beta's agents
   structurally different output formats, not just different posture
   descriptions. Alpha's output should read like a risk memo; Beta's like
   a deal sheet. Same architecture, different presentation. That would
   force divergence at the structural layer, not just the content layer.

Both of these are hypotheses about how to improve the architecture. Neither
was tested in this project. They are candidates for the TypeScript port.

## Why This Matters for HAQQ

Three implications for the Justinian engine:

1. **Multi-agent decomposition pays for itself on complex clauses.** A 27-agent
   pipeline produced 2.5x the output and 3x the proposed changes for the same
   clause. The overhead of orchestration is real, but it is paid back in work
   product depth.

2. **Citation grounding is a separate design decision from agent decomposition.**
   The architecture does not produce citations on its own. If a firm wants
   citation-grounded output, the retrieval layer and the prompts must both be
   built for it. This is worth building into the Justinian engine's architecture
   as an explicit layer, not an emergent property.

3. **Homogenization is a hidden risk.** Every shared agent prompt is an
   opportunity to reduce firm-specific divergence. If HAQQ's digital twins are
   supposed to feel meaningfully different to each firm, the architecture must
   be designed for divergence, not just for depth.

## Files

- notebooks/tiro_pipeline.ipynb — the full pipeline and comparison
- data/clause.json — the test clause
- data/playbooks.json — the two firm playbooks
- tiro_comparison.csv — per-condition metrics
- tiro_summary.json — aggregated summary

## How to Run

Open `notebooks/tiro_pipeline.ipynb` in Google Colab. You need a free Groq
API key (console.groq.com) stored in Colab Secrets as `GROQ_API_KEY`.

Runtime: approximately 8-10 minutes. The pipeline makes 56 model calls.
