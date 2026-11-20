---
title: "Can you make models believe false facts in the prompt?"
collection: publications
category: papers
permalink: /publication/2026-11-18-prompted-false-facts
excerpt: 'We fine-tune a single model to believe false facts given in its prompt, so that we do not need a separately fine-tuned model for each set of false beliefs.'
date: 2026-11-18
venue: 'arXiv preprint arXiv:2611.23938'
paperurl: 'https://arxiv.org/abs/2611.23938'
citation: 'Sutradhar, A., &amp; Roger, F. (2026). &quot;Can you make models believe false facts in the prompt?&quot; <i>arXiv preprint arXiv:2611.23938</i>.'
---

Making LLMs believe false facts is useful for alignment evaluations and for building model organisms, but the usual method, synthetic document fine-tuning (SDF), needs a separately fine-tuned model for each set of false beliefs. We fine-tune one model on synthetic documents covering thousands of facts, with each document conditioned on a fact placed in the prompt, so that the model learns to believe whichever false fact the prompt supplies.

On facts the model never saw in training, belief depth grows roughly log-linearly with the number of training facts. When we applied the method to alignment audits on a small model we got negative results, because small models often already act as if they believe facts placed in honeypot system prompts. Adding reinforcement learning so that the model less often says aloud that it is following a prompted false fact generalizes to held-out evaluations and nearly matches standard SDF on some of them. Training against one of the belief-depth metrics, however, makes those evaluations harder to trust.

Work done during the Anthropic Fellows Program and MATS.

**Keywords:** Synthetic Document Fine-tuning, Belief Editing, Alignment Evaluations, Model Organisms, AI Safety
