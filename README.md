# The synapse that remembers

DataForge 2026, Pathway track submission. An interactive explainer teaching one falsifiable claim about synaptic plasticity as short-term memory, grounded in the Dragon Hatchling (BDH) architecture.

> The claim: a fixed-size recurrent state can process a sequence of unbounded duration without allocating a memory slot per token, but it forgets through interference.

Live artifact: https://tejaslamba2006.github.io/dataforge-2026-pathway/ (public, no sign-in)
Source: https://github.com/TejasLamba2006/dataforge-2026-pathway
Blog: `blog.pdf` in this repository.

## The claim

Everything in the artifact serves one sentence, taken from the Pathway problem statement's own example list: a fixed-size recurrent state needs no per-token memory slot, but it forgets through interference.

It is falsifiable by construction. Section 2 of the artifact states in advance what you would see if the claim were false, and a one-click self-test runs all three experiments and reports PASS or FAIL. A learner can watch the claim's machinery break in front of them.

## Intended learner and prerequisites

- Audience: students and practitioners who know Transformers and attention but haven't seen post-Transformer memory architectures.
- Prerequisites: what a KV cache is, and basic linear algebra (outer products, decay). No knowledge of BDH, SSMs, or neuroscience assumed.
- Learning objectives. After about 10 minutes the learner can:
  1. State why BDH-style attention needs no growing cache (recurrent collapse of linear attention into a fixed state).
  2. Demonstrate interference-based forgetting by distracting the model and watching recall degrade.
  3. Explain the stability-plasticity trade-off (the decay γ) in their own words.
  4. Locate the mechanism in BDH's equations and say how it differs from a Transformer KV cache.

## Architecture of the artifact

One self-contained `index.html`. No build step, no dependencies, no network calls. Five sections:

| § | Component | Role | Live or precomputed |
|---|---|---|---|
| 1 | Playground: a 24-neuron toy associative memory | The concept behaving. Teach pairs, distract, recall; live 24×24 σ heatmap, neuron strip, token stream, memory ledger comparing the fixed synaptic state against a growing KV cache | Live computation in the browser, plain JS, about 100 lines |
| 2 | Falsification table, experiments E1 to E3 | Pre-stated PASS and FAIL observations; a one-click self-test runs all three and reports | Live, drives the same model |
| 3 | BDH module with paper equations | Grounds the toy in the real architecture: edge-reweighting kernel, BDH-GPU Eq. 4, recent papers with inline citations [1] to [4] | Static text and equations, citations to primary sources |
| 4 | Misconceptions | What the artifact does not claim: not Mamba, toy is not BDH, not durable learning | Static |
| 5 | Explain-back quiz | Learner reproduces the concept in their own words; reference answers | Static content, interactive inputs |

The toy model lives in the `<script>` block of `index.html`: n = 24 neurons, each concept is a sparse positive pattern over 6 neurons, and each token applies global decay `σ ← γ·σ` followed by a Hebbian write `σ[i,j] += x_i·y_j` for co-active pairs. That is the toy analogue of BDH Table 1 rule `A(i),B(j) → σ(i,j)` with ALiBi-style damping U. Recall queries with a cue pattern and scores the readout against every concept pattern by inner product. The RNG is seeded with `mulberry32(20260907)`, so the demo is deterministic and every learner sees the same patterns and the same distraction stream.

KV-cache ledger math: a Transformer attending over t tokens with per-head dimension d = 8 stores about 2·d·t numbers, which grows without bound. The synaptic state stays at n² = 576 numbers (192 in the compressed ρ = Eσ form). These are illustrative numbers for the toy setting, not measurements of a production model.

## Live, animated, precomputed, or synthetic

- Live: all playground computation (writes, decay, recall, ledger, σ heatmap) is real computation in your browser, no animation library. The self-test button executes the full E1 to E3 protocol programmatically.
- Animated: only CSS transitions (button hover, recall-bar widths). No scripted animation is presented as model behavior.
- Precomputed: none. The problem statement allows labelled precomputed results; this artifact needs none, everything runs live.
- Synthetic: all data is synthetic by design, concept patterns from a seeded RNG. No external dataset is used.

## Reproduce

```
1. Open the artifact
   Option A: visit the GitHub Pages URL above
   Option B: open index.html locally, works from file:// with zero setup
2. Click "▶ Run all three experiments" in §3, expect 4 PASS lines (E1, E2, E3a, E3b)
3. Manual demo script in §1:
   Teach A→B, Recall "A → ?"        → B wins
   ⚡ Distract ×24 twice, Recall    → B's score collapses, another concept wins
   Ledger after 240 tokens          → state still 576, KV counter ≈ 15,360
   γ=0.60 + distract → trace wiped, vs γ=0.99 + distract → stale contamination
```

Reference outputs from our verification run. The seed is fixed, so you should match these:

| Check | Expected |
|---|---|
| E1 | state = 576 after 240 tokens; KV = 3,840 at 80 tokens, grows linearly |
| E2 | B fresh ≈ 36.0, after 24 distractors ≈ 18 to 24, winner flips |
| E3a (γ=0.60, 120 tokens) | B's trace wiped, ≈ 6 to 10 |
| E3b (γ=0.99, 24 distractors) | B still top, ≈ 100 to 120 |

## How the claim maps to BDH, with claim status

| Claim component | Status | Evidence |
|---|---|---|
| Fixed state, no per-token slot | Formal: holds by construction in BDH-GPU Eq. 4, the recurrent form of linear attention | [1] §3.2; same collapse used in [3] and [4] |
| Forgetting via interference | Publicly demonstrated for BDH's regime: Hebbian writes plus decay plus about 5% sparse positive activations [1] §6.4, long-context fade reported in [1]. In this artifact it is demonstrated live in the toy model, not claimed as a measured BDH benchmark | [1] §2.2, §6.4 |
| BDH-CQ connection | Publicly stated in [2]: recurrent memory updated at inference, adaptation in state rather than weights | [2] |
| Toy model fidelity | The playground is an independent didactic reimplementation, labelled as such. It reproduces no BDH benchmark and uses no checkpoint, none are public | — |

## Papers cited

1. Kosowski, Uznański, Chorowski, Stamirowska, Bartoszkiewicz, "The Dragon Hatchling: The Missing Link between the Transformer and Models of the Brain", arXiv:2509.26507 (2025).
2. Engdahl, Kosowski, Chorowski, Stamirowska, Uznański et al., "BDH-CQ: In-Context Learning with Recurrent Latent Reasoning", arXiv:2608.09888 (2026).
3. Yang, Wang, Shen, "Gated Linear Attention Transformers with Hardware-Efficient Training", arXiv:2312.06635 (2023).
4. Gu, Dao, "Mamba: Linear-Time Sequence Modeling with Selective State Spaces", arXiv:2312.00752 (2023).

All four are primary sources from 2023 to 2026. Citations appear beside the technical claims they support, in §3 of the artifact and in the claim-status table above.

## Setup

No install. The artifact is one HTML file.

```
git clone https://github.com/TejasLamba2006/dataforge-2026-pathway
# open index.html in any modern browser
```

Works from `file://`, no server, no npm, no network access needed at runtime.

## Sources and licenses record

| Asset | Source | License |
|---|---|---|
| All code (index.html, single file) | Written for this submission | MIT, see `LICENSE` |
| Blog PDF (blog.pdf) | Written for this submission | CC BY 4.0 |
| Equations and figure references from BDH paper [1] | arXiv:2509.26507 | arXiv non-exclusive license, cited for scholarship |
| Equations and claims from BDH-CQ [2] | arXiv:2608.09888 | arXiv license, cited |
| Fonts, images, datasets, weights, third-party components | None reused | — |

## AI assistance disclosure

This artifact was produced with AI assistance (Claude Code) under human direction by the submitting team.

- Code: the toy model, page, and self-test were drafted by AI and reviewed and verified by the team. The deterministic self-test in §3 runs the falsification protocol E1 to E3 with pre-stated pass criteria.
- Research: paper identification, equation extraction from the arXiv PDF, and citation verification were AI-assisted. Every technical claim was checked against the primary sources [1] and [2].
- Writing: README and blog drafted by AI, approved by the team.
- Data and assets: 100% synthetic, generated by seeded RNG in the browser. No external data, weights, or assets.

## Credits

- Dragon Hatchling (BDH): Kosowski, Uznański, Chorowski, Stamirowska, Bartoszkiewicz, Pathway (arXiv:2509.26507)
- BDH-CQ: Engdahl, Kosowski, Chorowski, Stamirowska, Uznański et al. (arXiv:2608.09888)
- DataForge 2026 Pathway track: KDAG IIT Kharagpur × Pathway
