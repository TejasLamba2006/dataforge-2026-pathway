# The synapse that remembers

When you ask a Transformer a question about a book it read, it re-reads the passage every time. Attention has no memory of its own. Every answer over a long document costs a pass over every token so far, and serving that costs a growing cache. If the document never ends, the cache never stops growing.

That is the machine Pathway's Dragon Hatchling (BDH) attacks [1]. The paper asks what happens if attention stops being lookup-over-history and becomes something a brain would recognize: a synapse. Store memory on edges between neurons. Weaken it with decay. Strengthen it when the two ends are co-active, the Hebbian rule. The state is the synapses themselves, a fixed pool of numbers. No slot per token, ever.

If a model forgets nothing, something has to give somewhere else. BDH chooses the honest trade: a fixed state that forgets. That is the claim this artifact teaches, and the Pathway problem statement lists it as an example of the kind of claim worth teaching: a fixed-size recurrent state can process a sequence of unbounded duration without allocating a memory slot per token, but it forgets through interference.

## What the artifact is

One HTML file, no dependencies, no network calls. The model runs in the learner's browser.

Section 1 is a playground. A 24-neuron toy with a 24×24 synaptic matrix σ, 576 numbers, its only memory. Concepts A through H are sparse positive patterns over 6 neurons each. Press "Teach A→B" and the matrix applies decay, then writes the co-activation pattern into the synapses. Press "Distract ×24" and 24 random non-repeating pairs get written, each one decaying everything else a little more. Press Recall "A → ?" and the readout is scored against every concept pattern by inner product, with bars for the top candidates. A ledger sits next to the matrix: tokens written, fixed synaptic numbers (576, never moves), and the KV-cache counter that climbs 16 numbers per token. After 240 tokens, KV is at 15,360 numbers and still climbing, while σ sits at 576. The grid never resizes. That is the first half of the claim, visible without any math.

Section 2 is a falsification table. Each experiment states in advance what you would see if the claim were false. E1: after 240 tokens the state counter still reads 576. E2: teach A→B, recall, distract ×24, recall again, and B's score collapses, mine went from 36.0 to 18.0 and another concept took the win. E3: the decay dial γ is the stability-plasticity knob. γ=0.60 wipes the A→B trace after 120 tokens. γ=0.99 lets stale associations linger and contaminate recall. Forgetting through interference is what you watch happen, and it is the second half of the claim. A self-test button runs all three programmatically and prints PASS or FAIL against the pre-stated observations, 4 of 4 PASS on the shipped build.

Section 3 is where the toy meets the paper. The BDH-GPU equations (Eq. 4) show the middle term is linear attention with an exponential temporal kernel U, computable recurrently as ρ_t = γ·ρ_{t−1} + x_t v_tᵀ with readout a_t = ρ_tᵀx_t [1, §3.2]. ρ = Eσ is n·d numbers and never grows with t. The same collapse is used by Gated Linear Attention [3] and Mamba [4]. Writes are Hebbian (Table 1 rule A(i),B(j)→σ(i,j)) [1, §2.2]. Activations are sparse and non-negative, about 5% of neurons per token [1, §6.4]. Because the state is fixed, new writes must eventually dilute old ones. Forgetting is structural. BDH-CQ builds its reasoning loop on this substrate: recurrent memory updated continuously at inference, adaptation in state rather than weights [2].

Section 4 draws the boundaries. The playground is a didactic reimplementation of one mechanism, not BDH itself. It reproduces no benchmark. There is no public BDH checkpoint. BDH is not an SSM in the Mamba sense, the paper says so and the problem statement warns about it. And synaptic state is short-term memory. Whether fast state consolidates into slow weights is an open question the paper itself flags [1].

Section 5 asks the learner to explain it back. Three questions, no multiple choice, then reference answers.

## Why the falsification framing matters

The problem statement asks for a falsifiable claim, pre-stated conditions, and no overclaiming. The fastest way to meet that bar is to pre-commit: state what would count as a failure before you look at the data. E2 is the sharpest example. Before running anything, the table says: if recall of B were unchanged at every distraction level, forgetting through interference would be false. Then you run it and B goes from 36.0 to 18.0 and loses to a distractor. The experiment had a chance to break the claim and didn't.

The self-test makes it harder to overclaim by accident. It saves the demo state, resets, runs the protocol, and restores. It prints PASS only when the pre-stated criteria hold: E1 state fixed at 576, E2 score drop plus winner flip, E3a wipe at γ=0.60, E3b lingering at γ=0.99. 4 of 4 PASS.

## What I'd do differently

Two honest limits of the artifact. First, the toy model is small. With 24 neurons and 6-neuron patterns, cross-talk dominates after about 15 to 20 overlapping associations, so the demo degrades gracefully rather than catastrophically, and the interference effect is mild compared to what a trained BDH sees. Second, the decay γ is a single global scalar. Real BDH's temporal kernel U can be per-edge, which would let the model tune forgetting per connection. Neither limit threatens the claim, and both are declared in the artifact's misconceptions section.

## License and credits

Code and blog: MIT and CC BY 4.0, Tejas Lamba, 2026. The BDH equations and findings belong to Pathway's authors and are cited for scholarship. Built for DataForge 2026, the Pathway track, KDAG IIT Kharagpur.

References: [1] Kosowski, Uznański, Chorowski, Stamirowska, Bartoszkiewicz, "The Dragon Hatchling: The Missing Link between the Transformer and Models of the Brain", arXiv:2509.26507. [2] Engdahl, Kosowski, Chorowski, Stamirowska, Uznański et al., "BDH-CQ: In-Context Learning with Recurrent Latent Reasoning", arXiv:2608.09888. [3] Yang, Wang, Shen, "Gated Linear Attention Transformers with Hardware-Efficient Training", arXiv:2312.06635. [4] Gu, Dao, "Mamba: Linear-Time Sequence Modeling with Selective State Spaces", arXiv:2312.00752.
