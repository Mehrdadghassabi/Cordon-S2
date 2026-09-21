# Reasoning Reduces the Influence of Poisoned Context in RAG

> [Paper (PDF)](https://arxiv.org/pdf/2608.17153)

---

## Overview

Retrieval-Augmented Generation (RAG) systems ground LLM outputs in external documents — but this opens a vulnerability: **knowledge-poisoning attacks**, where adversarial text injected into retrieved passages can steer model responses.

This paper asks: **can deliberative reasoning reduce a model's susceptibility to poisoned evidence, even without architectural defenses?**

The answer is yes — but with a twist. Enabling reasoning:
- **Reduces** the behavioral influence of detected poison (lower Cordon Rate and Leakage Rate)
- **Increases** overall attack success (lower Poison Detection Rate)

This reveals that **poison detection and resistance to poisoning are distinct capabilities**, motivating multi-dimensional evaluation of RAG robustness.

---

## Key Concepts

### Cordon Rate (CR)
The probability that a model's final answer is influenced by a poisoned document **despite the model having detected it as misinformation**.

```
CR = P(influenced | poison_detected ∧ no-RAG_answer_not_poison-aligned)
```

A CR of 0 is the ideal — a model that detects misinformation should fully exclude it from its answer.

### Leakage Rate (LR)
The probability that a model is influenced by a poisoned document **even when explicitly instructed to ignore all retrieved context**.

```
LR = P(influenced_by_ignore_condition ∧ no-RAG_answer_not_poison-aligned)
```

An LR of 0 is the ideal — explicit top-down instructions to disregard context should be followed completely.

Both metrics have a natural zero reference point (trivially achievable by construction), making them useful for comparing model behavior against an idealized reliable agent.

---

## Results

### Effect of Enabling Reasoning (SciFact, n=200)

| Model | Reasoning | Leakage Rate | Cordon Rate |
|---|---|---|---|
| DeepSeek-V4-Flash | Off | 0.235 | 0.211 |
| DeepSeek-V4-Flash | **On** | **0.140** | **0.107** |
| Qwen3.6-Plus | Off | 0.240 | 0.058 |
| Qwen3.6-Plus | **On** | **0.170** | **0.011** |

Enabling reasoning reduces Cordon Rate by **~49%** (DeepSeek) and **~81%** (Qwen), and Leakage Rate by **~40%** (DeepSeek) and **~29%** (Qwen).

### Detection vs. Resistance (SciFact, n=200)

| Model | Reasoning | Attack Success Rate | Poison Detection Rate |
|---|---|---|---|
| DeepSeek-V4-Flash | Off | 0.233 | 0.965 |
| DeepSeek-V4-Flash | On | 0.298 | 0.665 |
| Qwen3.6-Plus | Off | 0.133 | 0.830 |
| Qwen3.6-Plus | On | 0.279 | 0.500 |

Higher reasoning → **worse** at detecting poison explicitly, yet **better** at not being behaviorally influenced by it. Detection and resistance are decoupled.

### Cross-Model Comparison (Claude, SciFact, n=200)

| Model | ASR | Leakage Rate | Cordon Rate |
|---|---|---|---|
| Claude Haiku 4.5 (reasoning off) | 0.280 | 0.245 | 0.220 |
| Claude Sonnet 4.6 (reasoning on) | **0.080** | **0.093** | **0.069** |

Consistent pattern, though the models also differ in capability — this comparison is treated as supporting rather than causal evidence.

---

## Research Questions

| RQ | Question | Finding |
|---|---|---|
| **RQ1** | Does reasoning reduce poisoned-context influence? | Yes — both CR and LR drop substantially when reasoning is enabled. |
| **RQ2** | Does poison detection predict poison resistance? | No — detection and resistance are distinct; enabling reasoning improves one while hurting the other. |
| **RQ3** | Does the pattern extend across model families? | Yes — the Claude Haiku/Sonnet comparison is consistent with within-model results. |
| **RQ4** | Does susceptibility vary across datasets? | Yes — SciFact shows clear susceptibility; FiQA and MS MARCO near-zero (limited sample). |

---

## Experimental Setup

- **Dataset:** 200 randomly sampled SciFact questions (seed 38), plus 40 questions each from FiQA and MS MARCO (exploratory)
- **Target models:** DeepSeek-V4-Flash, Qwen3.6-Plus (reasoning on/off), Claude Haiku 4.5, Claude Sonnet 4.6
- **Judge model:** Gemini 2.5 Pro
- **Poison generation:** GPT-5.6 (with Grok 4.6 as fallback); each poison is a ~500-word authoritative-sounding passage contradicting the correct answer
- **Inference settings:** Provider-recommended defaults via official APIs (as of early September 2026)

### Poison Generation Pipeline

```
Question → Correct answer → Contradictory sentence → 500-word passage
                                                          ↓
                                         Judge verifies passage contradicts answer
                                         (retry if it doesn't)
```

---

## Repository Structure


---

## Citation

```bibtex
@misc{ghassabi2026saferrag,
  title         = {Towards Safer RAG: Only Agents Capable of System 2 Thinking may Access Untrusted Documents},
  author        = {Ghassabi, Mehrdad and Ebrahimi, Audrina and Hakim, Sadra and Kashani, Hamidreza Baradaran},
  year          = {2026},
  eprint        = {2608.17153},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CL},
  url           = {https://arxiv.org/abs/2608.17153}
}
```
