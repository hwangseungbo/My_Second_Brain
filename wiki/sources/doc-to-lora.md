---
title: "Doc-to-LoRA: Learning to Instantly Internalize Contexts"
type: source
created: 2026-04-10
updated: 2026-04-10
sources: ["Doc_to_LoRA.pdf"]
tags: [LLM, LoRA, context-distillation, hypernetwork, efficiency, long-context]
---

# Doc-to-LoRA: Learning to Instantly Internalize Contexts

## Citation

Rujikorn Charakorn, Edoardo Cetin, Shinnosuke Uesaka, Robert T. Lange. "Doc-to-LoRA: Learning to Instantly Internalize Contexts." Preprint, February 19, 2026. Sakana AI, Tokyo, Japan. arXiv:2602.15902v1 [cs.CL]. Project page: github.com/SakanaAI/doc-to-lora.

## Abstract

Long input sequences are central to in-context learning, document understanding, and multi-step reasoning of Large Language Models (LLMs). However, the quadratic attention cost of Transformers makes inference memory-intensive and slow. While context distillation (CD) can transfer information into model parameters, per-prompt distillation is impractical due to training costs and latency. To address these limitations, we propose Doc-to-LoRA (D2L), a lightweight hypernetwork that meta-learns to perform approximate CD within a single forward pass. Given an unseen prompt, D2L generates a LoRA adapter for a target LLM, enabling subsequent queries to be answered without re-consuming the original context, reducing latency and KV-cache memory consumption during inference of the target LLM. On a long-context needle-in-a-haystack task, D2L successfully learns to map contexts into adapters that store the needle information, achieving near-perfect zero-shot accuracy at sequence lengths exceeding the target LLM's native context window by more than 4x. On real-world QA datasets with limited compute, D2L outperforms standard CD while significantly reducing peak memory consumption and update latency. We envision that D2L can facilitate rapid adaptation of LLMs, opening up the possibility of frequent knowledge updates and personalized chat behavior.

## Key Claims and Arguments

1. **Amortized context distillation**: D2L replaces the expensive per-context optimization loop of traditional [[concepts/context-distillation]] with a single forward pass through a [[concepts/hypernetwork]], amortizing the cost of query generation and backpropagation into a one-time meta-training phase.

2. **Sub-second internalization**: Once meta-trained, D2L internalizes a new context in under one second (both batched and iterative modes), compared to ~40 seconds for oracle CD and >100 seconds for vanilla CD with generated queries.

3. **Beyond-context-window generalization**: On a synthetic Needle-in-a-Haystack (NIAH) task, D2L achieves near-perfect zero-shot accuracy on contexts up to 32K+ tokens, despite the base LLM (gemma-2-2b-it) having only an 8K context window and the hypernetwork being trained on sequences of only 32-256 tokens. Performance remains close to perfect up to 40K tokens (5x the training chunk count).

4. **Outperforms CD under limited budgets**: On real-world QA benchmarks, D2L outperforms vanilla CD (with generated queries) while using dramatically less memory and latency for the internalization step.

5. **Cross-modal zero-shot transfer**: D2L can zero-shot transfer visual information from a VLM (gemma-3-4b-it) to a text-only LLM (gemma-2-2b-it), achieving 75.03% accuracy on Imagenette (10-class ImageNet subset) without ever seeing images during training.

6. **Emulates many-query CD**: D2L effectively emulates CD with a very large number of generated queries. Training across millions of context samples exposes the hypernetwork to a larger and more diverse query distribution than any single CD run, providing implicit regularization.

## Method Description

### Problem Setup: Context Distillation

[[concepts/context-distillation]] (CD) trains an LLM -- without access to relevant information -- to imitate its own outputs when the information is provided in context. The teacher has context c in its prompt; the student does not. The query-independent CD objective optimizes:

min KL[ p(y | x, c) || p_theta_c(y | x) ]

over multiple generated queries X = {x_i} and self-generated responses Y = {y_i ~ p(.|x_i, c)}, yielding context-specific parameters theta_c.

### Meta-Learning the CD Process

D2L meta-trains a [[concepts/hypernetwork]] H_phi to map a context c directly to a set of [[concepts/lora]] adapter parameters: Delta W_c = H_phi(c). The meta-training objective is:

min_phi E_{(c, D_c) ~ D} E_{(x,y) ~ D_c} KL[ p(y | x, c) || p_{theta + H_phi(c)}(y | x) ]

The key distinction from vanilla CD is that a single H_phi generalizes across many contexts, rather than optimizing separate weights per context. The meta-training dataset D includes diverse contexts, queries, and self-generated responses from a large corpus.

### D2L Architecture

1. **Context encoding**: A context c is fed through the frozen target LLM to obtain per-layer token activations Z in R^{L x N x D}, where L is the number of transformer layers, N is the number of context tokens, and D is the hidden size.

2. **Perceiver-based hypernetwork**: For each transformer layer l, a shared [[concepts/perceiver]]-style cross-attention module consumes activations Z_{l-1} and outputs low-rank LoRA parameters. The Perceiver uses r learnable, input-independent latent queries Q_m in R^{r x d_q}. Cross-attending Q_m to Z_{l-1} yields r latent vectors, which are then mapped by per-layer output heads to rows of A_l and columns of B_l (the LoRA matrices). This design naturally handles variable-length inputs by mapping them to a fixed number of latent queries (the LoRA rank).

3. **Long-context composition via chunking**: For long contexts, the input is partitioned into K contiguous chunks. Each chunk is processed independently through the hypernetwork, producing per-chunk adapters. The chunks are combined by concatenating along the rank dimension, yielding LoRA matrices with total rank r * K. This allows D2L to integrate information across many chunks without changing the hypernetwork's output shape.

4. **Architecture specifics**: The Perceiver module has 8 cross-attention blocks without self-attention layers. Each generated adapter is applied to the "down projection" layer of each MLP block of the base model. The hypernetwork has only 309M trainable parameters.

5. **Inference modes**: D2L can operate in batched mode (produces all layer adapters in a single pass, faster) or iterative mode (one layer at a time, lower memory). Both are mathematically equivalent.

### Training Details

- Meta-training data is derived from a subset of FineWeb-Edu (~900M tokens) plus passage-grounded QA datasets (PwC, SQuAD, ROPES, DROP), totaling ~3.2M unique contexts.
- For FineWeb-Edu contexts, 10 queries per sample are generated using gemma-3-12b-it in two iterations (5 queries each), with the second iteration encouraged to produce non-overlapping, harder questions.
- Self-responses are sampled from gemma-2-2b-it with top-16 token logit values recorded as training targets.
- Two-stage training: (1) 80K steps with single-chunk outputs only; (2) 20K steps with random chunking (50% 1 chunk, 12% 2 chunks, 37.5% for 3-8 chunks). Each batch packs context inputs into 4K-token sequences with gradient accumulation across 8 GPUs (>200K context tokens per batch).
- Full meta-training takes ~5 days on 8 H200 GPUs for gemma-2-2b-it.

## Experimental Results

### Needle-in-a-Haystack (NIAH) Task (Section 4)

- Base model: gemma-2-2b-it (8K context window).
- D2L trained on 32-256 token sequences, evaluated on haystacks up to 128K+ tokens.
- D2L achieves perfect accuracy matching the base model up to 8K tokens.
- Beyond 8K tokens (where the base model fails), D2L maintains near-perfect accuracy up to ~40K tokens (40 chunks of 1024 tokens, despite training on max 8 chunks).
- Memory: Base model uses >12 GB additional memory for a 128K-token haystack. D2L consistently uses <50 MB regardless of haystack length.

### Short-Context QA (Section 5.1.1)

Benchmarks: SQuAD, DROP, ROPES. Performance reported as ROUGE-L F1 relative to base model with context (ICL upper bound).

- **SQuAD**: D2L achieves 82.5% relative performance vs. ICL upper bound, outperforming all in-parameter baselines (CD with generated queries, T2L, base model without context). Performance is roughly equivalent to LLMLingua-2 at 40% compression, but D2L removes the context entirely.
- **Update latency**: D2L internalizes in <1 second (both modes). CD oracle ~40s, vanilla CD >100s.
- **Memory**: D2L and CD oracle both use <2 GB during updates. CD with generated queries uses >40 GB.

### Long-Context QA (Section 5.1.2)

Benchmarks: 2WikiMultihopQA, MultiFieldQA, QASPER (from LongBench, up to 32K tokens). D2L was never trained on sequences this long (max training length: 2,344 tokens).

- **2WikiMultihopQA** (Table 1):
  - CD oracle: 0.901 normalized performance, 7.82 GB memory, ~40s latency
  - D2L batched: 0.857, 11.52 GB, ~0.21s
  - D2L iterative: 0.844, 3.79 GB, ~0.55s
  - CD (5 generated queries): 0.704, 79.37 GB, ~72.5s
  - CD (25 generated queries): 0.745, 59.93 GB, ~465s
- D2L iterative uses 2x less update memory than CD oracle while maintaining sub-second latency.
- ICL baseline requires ~1 GB VRAM for response generation on long contexts; all in-parameter methods use <100 MB.
- Interesting finding: Combining D2L with truncated context input slightly improves performance on 2WikiMultihopQA and MultiFieldQA, possibly because internalized knowledge mitigates "lost-in-the-middle" or attention noise effects.

### Zero-Shot Visual Information Transfer (Section 5.2)

- VLM encoder: gemma-3-4b-it; target LLM: gemma-2-2b-it (text-only).
- D2L maps first 26 layers of VLM activations to corresponding target LLM layers.
- Imagenette (10-class subset of ImageNet): 75.03% accuracy purely through internalized information (random baseline: 10%). D2L and the target LLM never saw images during training.
- Text QA performance degrades somewhat with VLM encoder (e.g., SQuAD: 0.705 vs. 0.814 with LLM encoder).

### Ablation Studies

- **LoRA rank** (Table 7): Rank-16 outperforms rank-8 significantly on SQuAD (0.896 vs. 0.814) and DROP (0.711 vs. 0.655), similar on ROPES. Shows D2L benefits from higher-capacity parameterizations.
- **Training objective** (Table 6): KL distillation outperforms next-token prediction (NTP) loss (0.819 vs. 0.763 on SQuAD at 50% training), with an even wider gap on the extreme "swapped" generalization test (0.385 vs. 0.235 recall).
- **Data ablation** (Table 5): Removing QA task data from training yields comparable performance, suggesting D2L is robust to training data format.
- **Query budget comparison** (Table 4): On a 100-sample SQuAD subset, D2L achieves 0.866 vs. CD with 100 generated queries at 0.650 (taking >10 minutes per sample). D2L takes 0.086 seconds.
- **Generality across base models**: Results with Mistral-7B-Instruct-v0.2 and Qwen3-4B-Instruct-2507 show similar trends, confirming method generality.
- **KV cache generation** (Appendix D): The architecture can also generate compressed KV cache (prefix-tuning) instead of LoRA, achieving near-perfect NIAH accuracy up to 8K tokens with Qwen3-4B.

## Key Figures and Tables Described

- **Figure 1**: Overview diagram showing D2L training (left) and downstream performance comparison (right). Shows the pipeline: document fed through frozen LLM to get activations, shared hypernetwork produces per-layer LoRA adapters, distillation loss against teacher's contextualized response.
- **Figure 2**: NIAH retrieval performance (top) and additional inference memory (bottom) vs. haystack length. D2L maintains near-perfect accuracy well beyond the 8K base model limit; memory stays flat at <50 MB vs. >12 GB for ICL at 128K tokens.
- **Figure 3**: SQuAD performance vs. context length ratio, update latency, and additional memory for model updates. D2L achieves sub-second internalization with <2 GB memory.
- **Figure 4**: Long-document QA performance on 2WikiMultihopQA, MultiFieldQA, and QASPER plotted against additional memory for generation. Shows D2L competitive with CD oracle at far less memory.
- **Table 1**: Detailed comparison on 2WikiMultihopQA with performance, memory, and latency.
- **Table 2**: Cross-modal transfer results (LLM-to-LLM vs. VLM-to-LLM), including 75.03% Imagenette accuracy.

## Limitations

The authors acknowledge several limitations:

1. **Expensive meta-training**: While inference is cheap, the one-time meta-training phase is costly -- approximately 5 days on 8 H200 GPUs for gemma-2-2b-it.
2. **Model-specific training**: D2L requires retraining the hypernetwork for each new target LLM.
3. **Performance gap vs. ICL**: There remains a general performance gap between in-context learning and in-parameter knowledge methods, including D2L.
4. **Knowledge interference**: D2L can override existing internal knowledge of the base LLM (Table 8). When internalized knowledge and queries are unrelated, performance drops significantly (0.096 on SQuAD with assistant prompt, vs. 0.211 for CD). The hypernetwork appears to assume subsequent queries will always be related to the internalized context.
5. **LoRA-only parameterization**: The work focuses solely on LoRA; other parameterizations might be more efficient or avoid catastrophic forgetting.

## Connections to Other Work

- **[[concepts/context-distillation]]**: D2L is fundamentally an amortized approximation of CD. The paper builds directly on CD literature (Askell et al., 2021; Snell et al., 2023; Caccia et al., 2025; Eyuboglu et al., 2025).
- **[[concepts/lora]]** (Hu et al., 2022): The output parameterization of the hypernetwork. The chunking mechanism effectively increases LoRA rank for longer contexts.
- **[[concepts/hypernetwork]]** (Ha et al., 2016): D2L is a hypernetwork that predicts task-specific parameters. Related to HyperLoRA (Lv et al., 2024), HINT (Ivison et al., 2023), HyperTuning (Phang et al., 2023), and Text-to-LoRA (Charakorn et al., 2025).
- **[[concepts/perceiver]]** (Jaegle et al., 2021): The cross-attention architecture that enables variable-length input to fixed-shape output mapping.
- **[[concepts/prompt-compression]]**: Related to LLMLingua-2 (Pan et al., 2024), Gisting (Mu et al., 2024), and Activation Beacon (Zhang et al., 2025a), but D2L operates in parameter space rather than token space.
- **Cartridges** (Eyuboglu et al., 2025): Uses sleep-time compute for CD with prefix-tuning parameterization. D2L differs by aiming for instant, generic CD.
- **Generative Adapter** (Chen et al., 2025): Most closely related. Uses next-token prediction loss on ground-truth tokens rather than CD objective. D2L's use of KL distillation with self-generated responses leads to better factual recall (higher ROUGE-L recall) and stronger generalization.
- **MEND** (Li et al., 2024): Trains a hypernetwork via CD to compress few-shot examples into prefix tokens.
- **[[concepts/long-context-processing]]**: D2L addresses the fundamental challenge of quadratic attention cost and KV-cache growth, offering an alternative to approaches like extended context windows or attention approximations.
- **Continual learning and personalization**: The authors envision D2L enabling rapid, repeated knowledge updates and personalized LLM behavior -- relevant to [[concepts/continual-learning]] and inference-time training.
