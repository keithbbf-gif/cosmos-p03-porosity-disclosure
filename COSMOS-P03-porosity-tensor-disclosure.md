# COSMOS-P03 Porosity Tensor: Session Working Notes and Patent Landscape Review

**Public disclosure date: 2026-09-09**

## Overview

This document consolidates a working research session on the COSMOS-P03 provisional patent packet — the "Family-axis occupancy, orthogonal Porosity tensor" invention within the broader COSMOS-P01 through COSMOS-P13 provisional patent series. The session covered four areas in sequence: verification of prior-art citations already in the docket, investigation of the "Model Rater" subsystem, patent-landscape analysis of a proposed blockchain/weighted-average extension, evaluation of a proposed domain-pair relationship mapping extension, and a terminology decision on "tensor" versus "space" framing. This report preserves the reasoning chain and evidentiary basis for each conclusion so the working group and counsel have a standalone reference independent of the chat transcript.

The core invention under review, as drafted in the docket, is a method of seating generative AI models for adversarial or collaborative tasks using a pairwise, per-axis disagreement measurement — as opposed to scalar quality scores, embedding-cosine similarity, or simple majority voting — persisted in a three-layer data store and used to prefer "different-mistake" models over redundant same-family copies.

## Prior Art Citation Verification

An initial verification pass was run against six citations flagged in the COSMOS docket as previously unverified. All six were confirmed as real, correctly described patent documents through patent-database lookups; none were found to be hallucinated or mischaracterized.

| Citation | Status | Relevance to docket |
|---|---|---|
| US12405822B1 (OpenAI) | Confirmed granted, 2025-09-02, filed 2024-06-07 | Shared-workspace multi-agent system — cited as the anti-shape for P02 (dual-lane, no shared context) and P05 |
| US12524210B2 (Microsoft) | Confirmed granted, 2026-01-13 | Hybrid local/remote routing for coding assistants (pick local XOR remote) — distinguished from P06's isolated local-plurality claim |
| US20240311405A1 (Google) | Confirmed granted 2024-09-19 | Dynamic selection of one-of-N generative models by efficiency — distinguished from P06 |
| US12314825B2 (Martian Learning) | Confirmed granted 2025-05-27 | Prompt-routing to a single best candidate model — P06 crowding citation |
| US12536406B2 | Confirmed granted 2026-01-27 | AI-agent gateway/router orchestration — additional P05/P06 crowding reference |
| CN119168059B (Beijing Facewall Intelligent Technology) | Confirmed granted 2025-07-15 | "Agent-based simulated courtroom implementation method" — the key P12 (plaintiff-defense-judge) blocking prior art |

The corresponding US application US20260037351A1 was also verified as real and granted (2026-02-05), but its actual claims concern AI-based data classification/summarization systems rather than a courtroom-triad specifically. This means the CN grant, not the US application, is the stronger blocking reference for the P12 packet, which the docket already holds rather than files as an independent provisional. The recommended outstanding action remains executing the actual USPTO Patent Public Search and TESS queries directly against USPTO's system, since the original research relied on Google Patents and Lens aggregator data rather than the authoritative USPTO database.

## The Model Rater System

Investigation confirmed the "Model Rater" is a real, named component of the P03 architecture — specifically the third and lightest layer of a three-layer persistence stack described in the written description.

| Layer | Path | Role |
|---|---|---|
| JSONL | state/porosity/obs.jsonl | Append-only authority for every raw pairwise observation |
| SQLite | state/porosity/porosity.sqlite | Rebuildable projection of the tensor — never authority |
| Scalar fold | state/model_rater/porosity.json | Per-model hole-set score (Model Rater) — coexists with, never replaces, the full tensor |

The Model Rater is fed by every adversarial trial run in the "Forge" module and every P05 occupancy profile, recording pairwise disagreement-frequency times error-magnitude between models on named axes. It is explicitly named by the operator (referred to in the docket as "Keith") on 2026-09-07, with "porosity" — pronounced "Pourosity" in the source material — as an inventor-coined term. No connection was found between this system and any external researcher or contributor named "Margie Irbe"; the authorship trail for P03 is attributed entirely to the operator, with academic prior art cited as Hidden Clones (arXiv 2603.17111), CAPA (arXiv 2502.04313), Nine Judges (arXiv 2605.29800), LLMs-as-Jury (arXiv 2607.10139), and Knight and Leveson (1986, 1990).

## Tensor and Database Schema

The tensor grid is the operative data structure underlying P03. It is defined as T(model_i, model_j, axis) — a pairwise vector between two specific named models, scoped to a named task domain (axis).

### Observation row schema (JSONL fields)

Every trial writes one row to the append-only ledger with the following fields:

| Field | Meaning |
|---|---|
| at | timestamp |
| trial_id | unique trial identifier |
| profile | which occupancy profile/domain packet ran it |
| stage | pipeline stage |
| axis | named task axis (coding, spec, security, law, facts, bull, bear, risk) |
| model_a, model_b | the pair — undirected fold key sorted(model_a, model_b); same-model pairs are refused |
| disagree | 0 or 1, or null if unknown |
| error_mag | 1 to 10, or null meaning UNMEASURED |
| tokens_a, tokens_b | token cost for each model in that trial |
| who_erred | a / b / both / none / unknown |
| source | local or federation |
| note | free text |

### The two tensors

Two tensors are maintained, both indexed by (model_i, model_j, axis):

- **T (porosity tensor)** — the pair-vector magnitude, computed as disagreement frequency multiplied by observed error magnitude.
- **C (complement tensor)** — four derived signals per cell: rescue (probability that model_j is right when model_i is wrong), co-failure (probability both are wrong), XOR-error (probability exactly one is wrong — the "productive disagreement" signal), and a signed combination of XOR-error minus co-failure times error magnitude.

Both tensors start UNMEASURED for any untested pair and sort last in seating decisions until real trial data populates them. The specification explicitly names "invented scores" (e.g., derived from embedding cosine similarity rather than observed task outcomes) as a rejected failure mode.

### Seating algorithm pseudocode

```
procedure RECORD_PAIR(trial, axis, a, b, ballot_a, ballot_b, who_erred=None, err_mag=None):
    if a.model_id == b.model_id: REFUSE
    if is_rotator(id_a) or is_rotator(id_b): REFUSE
    disagree = 1 if ballot_a != ballot_b else 0
    append obs.jsonl row

procedure SEAT(candidates, seated, axis, budget_tokens):
    for c in candidates:
        score[c] = mean(T[c, s, axis] for s in seated if measured else -inf) / tokens[c]
    prefer signed C when available
    return argmax(score)  # UNMEASURED sorts last
```

No literal SQL `CREATE TABLE` statements exist in the docket; the SQLite schema is defined only at the field/row level shown above. Writing literal DDL with column types and indexes remains an implementation task not yet specified in the patent write-up.

## Blockchain and Weighted-Average Extension: Patent Landscape

A proposal to publish a weighted-average model rating on a blockchain was evaluated against existing patents. Initial findings showed this specific combination — weighted-average reputation scores for AI agents, recorded immutably on a blockchain — is a heavily crowded prior-art space as of 2026.

| Patent/Application | Date | Claim |
|---|---|---|
| US20260222362A1 | Filed, published 2026-07-29 | AI agent registry computing a "weighted composite score" from peer reputation, task-completion rate, and quality metrics, stored as blockchain transactions |
| US20260142933A1 | Published 2026-05-20 | Reputation score "determined by a weighted average of one or more verification statuses," with decay and penalty terms, recorded to a distributed ledger |
| US11494171B1 | 2021 | Decentralized platform publishing AI models as NFTs with a "model rank generator" computing quality scores from validator consensus |
| CN122310531A | Published 2026-06-29 | LLM reputation/credibility scoring with full data-lineage logging on blockchain for tamper-proof audit |
| arXiv 2608.07762 | 2026 | Blockchain commit-reveal protocol specifically for LLM judge/benchmark scores using autonomous economic agents |
| TrustChain-AI (academic framework) | Published 2026-07-06 | Permissioned blockchain plus smart contracts plus weighted, decayed reputation scores for autonomous AI agents |

The conclusion at this stage of the analysis was that collapsing the P03 pairwise tensor into a single published weighted-average score would collide directly with the reputation-score patents above and would abandon the one element — pairwise orthogonality between specific named models — that differentiates P03 from prior art.

### Refinement: incremental aggregation without collapse

Following clarification that the tensor would not be collapsed but rather built up incrementally as more trial data accumulates — with each cell in the existing T(model_i, model_j, axis) structure receiving a running weighted update, and the full tensor state periodically published or hash-anchored to a ledger — the prior-art picture changed materially.

| Element | Prior art status |
|---|---|
| Weighted-average aggregation over incoming updates | Crowded — federated learning aggregation patents (US12541708B2, WO2023023281A1, EP3786872A1) perform this routinely |
| Publishing model parameter tensors to blockchain for audit | Crowded — US20200293887 has edge devices sending tensors to a federated learner update repository; related academic work anchors on-chain commitments of model-update tensors with staleness/reputation-aware weights |
| Blockchain-anchored audit trail of AI evaluation events | Crowded — CN122310531A and US20260073406A1 anchor AI evaluation/compliance artifacts to a ledger via hash digest |
| Pairwise inter-model disagreement tensor, incrementally weighted-averaged per cell, published on-chain | Not found as a specific combination in the search results reviewed |

The distinguishing basis: federated-learning aggregation patents combine parameter updates from multiple copies or clients of the same model architecture into one global model — this is parameter aggregation, structurally different from a symmetric N-by-N-by-axis disagreement matrix between heterogeneous, named models. The blockchain-reputation patents compute one scalar trust number per agent identity, not a directed pairwise vector between two specific agents scoped to a named task axis. The recommendation was that P03's claim language should keep the pairwise-tensor-per-axis structure as the headline claim, with blockchain publication framed as a dependent or embodiment claim rather than the primary claim, to avoid collision with the reputation-score patent family.

## Domain-Pair Relationship Mapping Extension

A further proposal considered whether the tensor could be extended to map qualitative pair-relationship types per domain, not just magnitude and orthogonality degree. Analysis found the existing schema already substantially covers this: since axis is explicitly the domain dimension and every tensor cell is already indexed by (model_i, model_j, axis), a domain:pair value is already the literal structure of the existing claim — not an unclaimed extension.

A search for genuinely novel extensions in this direction — specifically, a qualitative relationship-type classification (for example, "complementary," "redundant," or "adversarial") between two named models, varying by task domain — surfaced the following adjacent but distinguishable prior art:

| Prior art | Distinction from proposed extension |
|---|---|
| US20250259042A1 / US20250259043 / US20250259044A1 (AI agent orchestration platforms) | Use Shapley-value contribution estimation between domain-specialist agents — measures one agent's individual utility contribution, not a pairwise relationship type between two named models |
| CN122205461A (cross-domain cognitive fusion) | Domain-node/relationship-layer graph for telecom resource orchestration — structurally similar in spirit, different application domain |
| US11423307 / US12086547B2 (cross-domain knowledge graphs) | Cross-domain relationship graphs for NLP taxonomy and entity transfer learning — different application entirely |

No prior art was found describing a discrete relationship-type taxonomy between two named generative AI models, scoped per task domain, feeding a model-seating decision. This was flagged as a potentially genuine, still-open claim element worth drafting as a dependent claim under P03 — specifically, classifying each (model_i, model_j, axis) cell into a discrete relationship category rather than only a continuous numeric score.

## Terminology Decision: Tensor Versus Space Framing

A final consideration was whether "space" might serve as a more intuitive analogy than "tensor" for explaining the invention, despite being harder to visualize precisely. This framing was evaluated against prior art and found to carry meaningful risk.

The P03 written description contains a standing distinguisher, stated multiple times in the source material: the invention is explicitly "NOT cosine of embeddings" and orthogonality "rises with observed disagreement... not embedding cosine contrast." This distinction exists because "space" as an analogy maps directly onto a family of prior art that treats model comparison as geometric distance in a latent or embedding space, rather than observed task disagreement.

| Prior art using "space" framing | Claim |
|---|---|
| US20220188644A1 (latent-space misalignment) | Compares two neural networks via a statistical distance between their weight-derived latent spaces |
| US20240160902A1 (similarity-based GenAI filtering) | Positions model outputs as vectors in a shared embedding/latent space compared via distance thresholds |
| US20220358373A1 (latent space optimization) | Explicit "ambient space" and "manifold" geometric framing for generative models |
| US11971868 (dataset search via compressed representation) | Datasets and samples placed in shared latent space, compared via Euclidean/cosine/Manhattan distance |

All four treat "space" as a coordinate system derived from weights or activations, where distance is a proxy for similarity — this is the exact "adjacent vocabulary, different sensor" pattern the P03 background section names explicitly to distinguish itself from, citing Multi-LLM Prototype, DiscoUQ-Embed, and DALC as the closest prior art specifically for this reason.

The conclusion was to retain "tensor grid on named axes" as the operative claim terminology, since a "space" analogy risks implying the relationship between two models could be computed from their weights or embeddings alone, without running them against each other on real trials — precisely the shortcut the invention's novelty argument depends on rejecting. If an intuitive visual metaphor is still desired for non-technical explainer material, the docket's existing Reason (1990) swiss-cheese holes-and-overlay metaphor was identified as a safer choice, since it is already explicitly disclaimed in the source material as illustrative only and not part of the claimed mechanism.

## Summary of Open Items for Counsel

- USPTO TESS and Patent Public Search remain unexecuted for the full P01 through P13 combination-novelty question; prior research relied on Google Patents, Lens, and aggregator data.
- The P12 (plaintiff-defense-judge) packet should remain on HOLD given the granted CN119168059B blocking reference; the corresponding US application's claims are narrower than initially assumed.
- If blockchain publication is added to P03, claim language should preserve the pairwise-tensor-per-axis structure as the primary claim and frame ledger publication as a dependent embodiment, to avoid collision with US20260222362A1 and US20260142933A1.
- A discrete relationship-type taxonomy per domain-pair (as opposed to continuous magnitude/orthogonality scores) was identified as a potentially novel, currently unclaimed extension worth further drafting.
- Claim and explainer language should continue to use "tensor grid on named axes" rather than "space," reserving any spatial analogy for explicitly disclaimed, non-technical illustration only.

---

*This document is a working research and reasoning record, not a filed patent application, legal advice, or a formal novelty opinion. Publication here is solely to establish a timestamped public disclosure record.*
