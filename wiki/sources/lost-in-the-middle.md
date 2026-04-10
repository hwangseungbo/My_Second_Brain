---
title: "Lost in the Middle: How Language Models Use Long Contexts"
type: source
created: 2026-04-10
updated: 2026-04-10
sources: ["Lost_in_the_Middle.pdf"]
tags: [LLM, long-context, attention, positional-bias, retrieval-augmented-generation]
---

# Lost in the Middle: How Language Models Use Long Contexts

## Citation

Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, Percy Liang. "Lost in the Middle: How Language Models Use Long Contexts." *Transactions of the Association for Computational Linguistics (TACL)*, 2023. arXiv:2307.03172v3 [cs.CL], 20 Nov 2023. Stanford University, University of California Berkeley, Samaya AI. Code and data: nelsonliu.me/papers/lost-in-the-middle

## Abstract

While recent language models have the ability to take long contexts as input, relatively little is known about how well they use longer context. We analyze the performance of language models on two tasks that require identifying relevant information in their input contexts: multi-document question answering and key-value retrieval. We find that performance can degrade significantly when changing the position of relevant information, indicating that current language models do not robustly make use of information in long input contexts. In particular, we observe that performance is often highest when relevant information occurs at the beginning or end of the input context, and significantly degrades when models must access relevant information in the middle of long contexts, even for explicitly long-context models. Our analysis provides a better understanding of how language models use their input context and provides new evaluation protocols for future long-context language models.

## Key Claims and Arguments

1. **Language models do not robustly use information in long input contexts.** Performance degrades significantly when the position of relevant information changes within the context, even for models explicitly designed for long contexts.

2. **A U-shaped performance curve emerges.** Models perform best when relevant information is at the very beginning ([[concepts/primacy-bias]]) or very end ([[concepts/recency-bias]]) of the input context, with a significant drop for information placed in the middle.

3. **Extended-context models are not necessarily better.** Models with extended context windows (e.g., GPT-3.5-Turbo-16K vs. GPT-3.5-Turbo, Claude-1.3-100K vs. Claude-1.3) show nearly identical performance when input fits within both models' context windows, suggesting that simply expanding the context window does not improve context utilization.

4. **More context is not always better.** Providing a language model with more retrieved documents increases the amount of content the model must reason over, potentially decreasing accuracy. Reader performance saturates long before retriever recall does.

5. **The positional bias is not solely caused by instruction fine-tuning.** Base models (without instruction fine-tuning) also exhibit the U-shaped curve, though instruction fine-tuning can slightly reduce the worst-case performance gap.

## Experimental Setup

### Task 1: Multi-Document Question Answering

- **Dataset:** NaturalQuestions-Open (2655 queries where the annotated long answer is a paragraph).
- **Setup:** The model receives a question and *k* documents. Exactly one document contains the answer; the remaining *k - 1* are distractor documents retrieved via Contriever (fine-tuned on MS-MARCO) that do not contain any annotated answer.
- **Controls:** (i) Position of the answer-containing document is varied across all positions. (ii) Input context length is modulated by changing *k* (10, 20, or 30 documents, yielding roughly 2K, 4K, or 6K tokens).
- **Metric:** Accuracy -- whether any correct answer string appears in the predicted output.

### Task 2: Synthetic Key-Value Retrieval

- **Setup:** Models receive a JSON object with *k* key-value pairs (all random UUIDs) and must return the value for a specified key. This is a minimal retrieval testbed stripped of natural language semantics.
- **Controls:** Position of the target key-value pair and number of pairs (75, 140, or 300 pairs, yielding roughly 4K, 8K, or 16K tokens).
- **Metric:** Accuracy -- whether the correct value appears in the output.

### Models Tested

| Model | Type | Max Context |
|---|---|---|
| GPT-3.5-Turbo (0613) | Closed, decoder-only | 4K tokens |
| GPT-3.5-Turbo-16K (0613) | Closed, decoder-only | 16K tokens |
| Claude-1.3 | Closed, decoder-only | 8K tokens |
| Claude-1.3 (100K) | Closed, decoder-only | 100K tokens |
| MPT-30B-Instruct | Open, decoder-only | 8192 tokens (ALiBi) |
| LongChat-13B (16K) | Open, decoder-only | 16384 tokens (condensed RoPE) |
| Flan-T5-XXL | Open, encoder-decoder | 512 tokens (training) |
| Flan-UL2 | Open, encoder-decoder | 2048 tokens (training) |
| GPT-4 (8K) | Closed, decoder-only | 8K tokens (subset eval) |
| Llama-2 (7B, 13B, 70B) | Open, decoder-only | 4096 tokens |

## Key Findings

### The U-Shaped Performance Curve

Across all decoder-only models, performance follows a distinctive U-shape as the position of relevant information moves from the beginning to the end of the input context. Performance is highest at the very start (primacy bias) and very end (recency bias), with a significant trough in the middle.

- **GPT-3.5-Turbo** with 20 documents: performance drops by more than 20 percentage points from best to worst position. In the worst case, mid-context performance falls *below* closed-book performance (56.1%), meaning the model does worse with documents than without.
- **Closed-book vs. oracle baselines:** GPT-3.5-Turbo achieves 56.1% closed-book and 88.3% oracle, showing that when the model can find the right document, it can use it effectively -- the problem is locating it within a long context.

### Extended-Context Models Offer No Advantage

When input fits within both a base model and its extended-context variant, their performance curves are nearly superimposed. For example, GPT-3.5-Turbo and GPT-3.5-Turbo-16K perform identically in 10- and 20-document settings, and Claude-1.3 and Claude-1.3-100K are essentially the same across all settings.

### Key-Value Retrieval Results

- **Claude-1.3 and Claude-1.3 (100K)** achieve near-perfect accuracy on all key-value retrieval settings.
- **GPT-3.5-Turbo, MPT-30B-Instruct** show a U-shaped curve even on this minimal retrieval task. GPT-3.5-Turbo (16K) worst-case performance with 300 key-value pairs drops to 45.6%.
- This demonstrates that some models struggle with even basic token matching from mid-context positions.

### Encoder-Decoder vs. Decoder-Only (Section 4.1)

Encoder-decoder models (Flan-UL2, Flan-T5-XXL) are relatively robust to position changes *within their training-time sequence length*. Flan-UL2 shows only 1.9% absolute difference between best and worst positions at 2K tokens. However, when evaluated on sequences longer than training length, they too exhibit the U-shaped curve. The authors hypothesize the bidirectional encoder enables better relative importance estimation between documents.

### Query-Aware Contextualization (Section 4.2)

Placing the query both before and after the documents (so decoder-only models can attend to query tokens while processing documents):
- **Dramatically improves key-value retrieval** -- all models achieve near-perfect performance, including GPT-3.5-Turbo (16K) going from 45.6% worst-case to 100%.
- **Minimally affects multi-document QA** -- slight improvement when the answer is at the beginning, slight decrease otherwise.

### Effect of Instruction Fine-Tuning (Section 4.3)

- MPT-30B (base) and MPT-30B-Instruct both show the U-shaped curve, confirming the bias is not an artifact of instruction fine-tuning.
- Instruction fine-tuning raises absolute performance uniformly and slightly reduces the worst-case gap (from ~10% to ~4%).
- Base models without instruction fine-tuning can still use long-range information (beginning of context) when given instruction-formatted prompts, possibly learned from similar formats in pre-training data (e.g., StackOverflow).

### Model Scale Effects (Appendix E, Llama-2)

- **7B models** are solely recency-biased (no primacy effect).
- **13B and 70B models** exhibit the full U-shaped curve (both primacy and recency bias).
- The authors hypothesize that prior work did not observe primacy bias because the models studied were too small (<1B parameters).
- RLHF fine-tuning slightly mitigates positional bias at 13B scale but has minimal effect at 70B.

### Open-Domain QA Case Study (Section 5)

Using a retriever-reader pipeline on NaturalQuestions-Open with Wikipedia:
- Reader performance saturates long before retriever recall. Going from 20 to 50 retrieved documents yields only marginal reader gains (1.5% for GPT-3.5-Turbo, 1% for Claude-1.3), while retriever recall continues to climb.
- This suggests practical value in **reranking** (pushing relevant documents to the start) and **ranked list truncation** (retrieving fewer but more relevant documents) for [[concepts/retrieval-augmented-generation]] systems.

## Implications for RAG and Long-Context Systems

1. **Document ordering matters in [[concepts/retrieval-augmented-generation]].** Placing the most relevant documents at the beginning or end of the context is critical. The middle is effectively a dead zone for many models.
2. **Retrieving more documents has diminishing (and potentially negative) returns.** Beyond a modest number of documents (~20), additional context provides little benefit and may hurt performance.
3. **Reranking and truncation are promising strategies.** Rather than feeding all retrieved documents, systems should prioritize pushing the most relevant documents to prominent positions (start/end) and pruning low-value context.
4. **Extended context windows alone do not solve the problem.** Simply increasing a model's context window does not improve its ability to use information within that window. The utilization problem is distinct from the capacity problem.
5. **Evaluation protocols for long-context models should test positional robustness.** The authors propose that any claim of robust long-context use must demonstrate minimal performance variation across positions of relevant information.

## Limitations Acknowledged by the Authors

- The study focuses on a specific set of models available at the time (mid-2023); results may differ for newer architectures.
- Only two tasks are studied (multi-document QA and key-value retrieval); other tasks may show different patterns.
- Greedy decoding is used throughout; other decoding strategies are left to future work.
- Full GPT-4 evaluation was cost-prohibitive (estimated >$6000), so only a 500-question subset with 20 documents was tested.
- The NaturalQuestions dataset has some temporal ambiguity between the Wikipedia dump and annotations, though experiments on an unambiguous subset show similar trends.

## Key Figures and Tables Described

- **Figure 1 / Figure 5:** The central U-shaped performance curves for multi-document QA across all models at 10, 20, and 30 documents. Shows clear primacy and recency bias with a mid-context trough.
- **Figure 7:** Key-value retrieval performance at 75, 140, and 300 pairs. Claude-1.3 achieves near-perfect accuracy; other models show the U-shaped degradation.
- **Figure 8:** Encoder-decoder (Flan-UL2, Flan-T5-XXL) vs. decoder-only comparison. Encoder-decoders are robust within training-length sequences but degrade on longer ones.
- **Figure 9:** Query-aware contextualization in multi-document QA -- minimal improvement over standard prompting.
- **Figure 10:** Base model (MPT-30B) vs. instruction-tuned (MPT-30B-Instruct) -- both show U-shaped curves, confirming the bias is pre-existing.
- **Figure 11:** Retriever recall vs. reader accuracy as a function of number of retrieved documents -- reader performance plateaus far earlier than recall.
- **Figure 15 (Appendix D):** GPT-4 results -- higher absolute performance but the same U-shaped pattern persists.
- **Figure 16 (Appendix E):** Llama-2 scale comparison (7B/13B/70B) -- smaller models are recency-only; larger models exhibit the full U-shape.
- **Table 1:** Closed-book and oracle baselines for all models.
- **Tables 5-7 (Appendix G):** Full tabulated results for multi-document QA at 10, 20, and 30 documents across all positions.

## Connections to Other Work

- **Serial-position effect in psychology** (Ebbinghaus, 1913; Murdock, 1962): The U-shaped curve mirrors human free-recall patterns, where people best remember the first and last items in a list. Surprising for Transformers, whose [[concepts/self-attention]] mechanism is technically equally capable of attending to any position.
- **Khandelwal et al. (2018):** Found that small LSTM language models make increasingly coarse use of longer context, with a recency bias. This paper extends the finding to large Transformers, revealing an additional primacy bias absent in smaller models.
- **Long-context architectures** (Transformer-XL, Longformer, BigBird, FlashAttention, Hyena, RWKV, S4): While much work has focused on making long contexts computationally feasible, this paper shows that the *utilization* of long contexts is a separate and unsolved problem.
- **[[concepts/retrieval-augmented-generation]] systems** (REPLUG, Contriever, In-Context RALM): The findings directly impact the design of RAG pipelines, suggesting that naive context stuffing is suboptimal.
- **Sun et al. (2021):** Found that longer contexts improve prediction of only a few tokens in contiguous text. This paper complements that finding by showing positional bias in instruction-formatted tasks.
- **Qin et al. (2023):** Found that efficient Transformers are recency-biased on long-context NLP tasks, consistent with this paper's findings for standard Transformers.
