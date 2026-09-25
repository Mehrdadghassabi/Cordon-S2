# Reasoning Reduces the Influence of Poisoned Context in RAG

Research code and artifacts for:

**Reasoning Reduces the Influence of Poisoned Context in RAG**

📄 **Paper:** https://arxiv.org/abs/2608.17153

## Overview

Retrieval-Augmented Generation (RAG) grounds language-model responses in retrieved evidence, but retrieved documents can contain deliberately injected misinformation. A model may detect that a retrieved passage is misleading while still allowing that passage to influence its final answer.

This work studies whether **deliberative reasoning reduces the influence of poisoned context** when models have direct access to untrusted retrieved evidence.

Rather than proposing a new isolation-based defense, we empirically examine the relationship between:

* detection of poisoned evidence,
* attack success,
* and the model's susceptibility to contextual influence.

The experiments distinguish **detecting misinformation** from **resisting its influence**.

## Main Findings

On the main SciFact experiment, reasoning reduces both Leakage Rate (LR) and Cordon Rate (CR) for DeepSeek-V4-Flash and Qwen3.6-Plus.

| Model             | Reasoning |    LR |    CR |
| ----------------- | --------: | ----: | ----: |
| DeepSeek-V4-Flash |       Off | 0.235 | 0.211 |
| DeepSeek-V4-Flash |        On | 0.140 | 0.107 |
| Qwen3.6-Plus      |       Off | 0.240 | 0.058 |
| Qwen3.6-Plus      |        On | 0.170 | 0.011 |

At the same time, reasoning changes detection and attack-success behavior:

| Model             | Reasoning |   ASR |   PDR |
| ----------------- | --------: | ----: | ----: |
| DeepSeek-V4-Flash |       Off | 0.233 | 0.965 |
| DeepSeek-V4-Flash |        On | 0.298 | 0.665 |
| Qwen3.6-Plus      |       Off | 0.133 | 0.830 |
| Qwen3.6-Plus      |        On | 0.279 | 0.500 |

These results illustrate that **poison detection and resistance to contextual influence are distinct properties**. In these experiments, reasoning lowers measured contextual influence even though poison detection decreases and attack success increases.

## Research Questions

### RQ1 — Effect of reasoning

**Does reasoning reduce the influence of poisoned context?**

For both DeepSeek-V4-Flash and Qwen3.6-Plus, reasoning reduces both LR and CR on the main SciFact evaluation.

### RQ2 — Detection versus resistance

**Does detecting poisoned evidence imply resistance to its influence?**

The results indicate that these properties can diverge. Detection alone does not fully characterize whether poisoned evidence affects the final answer.

### RQ3 — Cross-model comparison

**Does the reduction extend across different model families?**

A supporting comparison uses Claude Haiku 4.5 without reasoning and Claude Sonnet 4.6 with maximum reasoning effort.

| Model             | Reasoning |   ASR |    LR |    CR |
| ----------------- | --------: | ----: | ----: | ----: |
| Claude Haiku 4.5  |       Off | 0.280 | 0.245 | 0.220 |
| Claude Sonnet 4.6 |        On | 0.080 | 0.093 | 0.069 |

This comparison is **supporting rather than causal**, because the models and reasoning configurations differ simultaneously.

### RQ4 — Dataset and question variation

**Does susceptibility to contextual influence vary across datasets and questions?**

Additional exploratory experiments use the first 40 questions from FiQA and the first 40 questions from MS MARCO.

For DeepSeek-V4-Flash, both reasoning settings produced zero LR and CR across MS MARCO and across all but one FiQA question. The remaining FiQA case exhibited a leakage rate of 1.

## Metrics

### Cordon Rate (CR)

Cordon Rate measures poison-aligned behavior among cases where the model detects the poisoned evidence, while excluding cases where the no-RAG answer is already poison-aligned:

```text
CR = P(poison-aligned(M_RAG)
       | detected(M_RAG) ∧ ¬poison-aligned(M_no-RAG))
```

### Leakage Rate (LR)

Leakage Rate measures cases in which poisoned retrieved context influences the model despite an instruction to ignore the retrieved documents:

```text
LR = P(poison-aligned(M_ignore)
       ∧ ¬poison-aligned(M_no-RAG))
```

### Attack Success Rate (ASR)

ASR measures the overall rate at which the poisoning attack produces a poison-aligned answer.

### Poison Detection Rate (PDR)

PDR measures the rate at which the model correctly identifies the poisoned evidence.

## Experimental Setup

### Main evaluation

* **Dataset:** SciFact
* **Sample:** 200 randomly selected questions
* **Random seed:** 38
* **Main models:** DeepSeek-V4-Flash and Qwen3.6-Plus
* **Reasoning:** within-model reasoning on/off comparisons
* **Judge model:** Gemini 2.5 Pro
* **Poison generation:** GPT-5.6
* **Fallback poison generator:** Grok 4.6 when GPT-5.6 refused generation
* **Model settings:** provider-recommended settings

### Additional evaluation

* **FiQA:** first 40 questions
* **MS MARCO:** first 40 questions

These experiments are exploratory and examine variation across datasets and questions.

## Evaluation Conditions

The experiments use three principal conditions.

### RAG

The model receives the question together with retrieved documents containing the poisoned passage. The model is instructed to reason about the context, check for misinformation, and provide a final answer.

### No-RAG

The model receives the question without retrieved documents. This provides the baseline used to identify cases where the model is already aligned with the poisoning direction without access to the retrieved context.

### Ignore-context

The model receives the retrieved documents but is explicitly instructed to ignore them and rely on its parametric knowledge.

This condition is used to measure **Leakage Rate**, capturing influence from retrieved context despite the instruction to ignore it.

## Repository Structure

```text
Cordon-S2/
├── src/
├── evaluation/
├── poisons/
├── doc/
└── README.md
```

## Reproducibility

The main experimental configuration is:

| Component                  | Setting                         |
| -------------------------- | ------------------------------- |
| Main dataset               | SciFact                         |
| Main sample size           | 200                             |
| Random seed                | 38                              |
| Main model comparisons     | DeepSeek-V4-Flash, Qwen3.6-Plus |
| Reasoning comparison       | Off vs. On                      |
| Judge                      | Gemini 2.5 Pro                  |
| Poison generation          | GPT-5.6                         |
| Poison-generation fallback | Grok 4.6                        |
| Additional datasets        | FiQA, MS MARCO                  |

Because the experiments use proprietary model APIs, exact outputs may vary as model providers update their systems.

## Interpreting the Results

The central distinction of this work is:

```text
Detection ≠ Attack Success ≠ Contextual Influence
```

A model can detect a poisoned passage without necessarily being resistant to its influence. Conversely, changes in attack success do not by themselves fully describe how poisoned context affects model behavior.

The reasoning experiments therefore evaluate contextual influence separately through CR and LR.

In particular, the main experiments show cases where reasoning is associated with:

* lower poison detection,
* higher attack success rate,
* but lower measured contextual influence.

This motivates treating **detection** and **resistance to contextual influence** as separate evaluation dimensions.

## Limitations

* The main evaluation contains 200 SciFact questions.
* The FiQA and MS MARCO experiments use 40 questions each and are exploratory.
* The experiments rely primarily on proprietary models and APIs.
* Reasoning is used as an operational proxy for deliberative reasoning; it should not be interpreted as a literal implementation of human System 2 cognition.
* Poisoned passages are generated synthetically.
* Automated judging introduces potential evaluation error.
* The Claude comparison is not a controlled within-model causal comparison because model family and reasoning configuration differ.
* Dataset-level differences are observed empirically but are not sufficient to establish why susceptibility varies across datasets or individual questions.

## Relation to Prior Work

This work builds on research showing that models can detect misinformation in retrieved evidence while still being influenced by that evidence.

It is also related to the **Cordon Principle**, which restricts the final-answer agent's access to raw retrieved evidence. The present work does not propose replacing such isolation-based approaches. Instead, it studies whether reasoning-enabled models can interact directly with untrusted evidence while exhibiting reduced contextual influence.

## Citation

If you use this work, please cite:

```bibtex
@misc{ghassabi2026reasoning,
  title = {Reasoning Reduces the Influence of Poisoned Context in RAG},
  author = {Ghassabi, Mehrdad and Ebrahimi, Audrina and Hakim, Sadra and Baradaran Kashani, Hamidreza},
  year = {2026},
  eprint = {2608.17153},
  archivePrefix = {arXiv},
  primaryClass = {cs.CL},
  url = {https://arxiv.org/abs/2608.17153}
}
```
