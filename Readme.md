# Towards Safer RAG

## Only Agents Capable of System 2 Thinking May Access Untrusted Documents

This repository contains the research materials for the paper **"Towards Safer RAG: Only Agents Capable of System 2 Thinking May Access Untrusted Documents"** by **Mehrdad Ghassabi** (University of Isfahan).

## Overview

Retrieval-Augmented Generation (RAG) systems can be vulnerable to **knowledge-poisoning attacks**, where misinformation in retrieved documents influences a model's final answer. Notably, a model may detect that a document contains misinformation while still being influenced by it during answer synthesis.

This work investigates whether **System 2 reasoning** can mitigate this monitoring-control gap and proposes a refined security principle:

> **Only agents capable of deliberative System 2 reasoning should access untrusted documents.**

## Contributions

We introduce two evaluation metrics:

- **Cordon Rate ($C$):** Measures cases where a model detects poisoned evidence but is nevertheless influenced by it.
- **Contamination Rate ($T$):** Measures implicit influence from poisoned evidence when the model is explicitly instructed to ignore the retrieved documents.

We empirically compare **DeepSeek-Chat** and **DeepSeek-Reasoner** across three BEIR datasets:

- **SciFact**
- **FiQA**
- **MS-MARCO**

The results show substantially lower contamination and cordon rates for the reasoning-capable model, providing empirical support for the proposed security principle.

## Experimental Setup

The experiments use:

- **Target models:** DeepSeek-Chat and DeepSeek-Reasoner
- **Judge model:** Gemini 2.5 Pro
- **Datasets:** SciFact, FiQA, and MS-MARCO
- **Poison generation:** GPT-5.6, with Grok 4.6 used for a small number of generation failures
- **Temperature:** 0.1

The study uses deliberately naive poisoned passages in order to investigate the effect of reasoning capability rather than the sophistication of the attack.
