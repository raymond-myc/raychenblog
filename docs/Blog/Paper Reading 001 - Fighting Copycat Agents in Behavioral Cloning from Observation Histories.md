---
title: Paper Reading 001 - Fighting Copycat Agents in Behavioral Cloning from Observation Histories
created: 2024-04-07
tags:
  - paper
  - imitation_learning
date: 2024-04-07
---

!!! tip "Paper Card"

	 * Title: Fighting Copycat Agents in Behavioral Cloning from Observation Histories
	 * First author: Chuan Wen (Tsinghua University)
	 * Correspoding author: Yang Gao (Tsinghua University, UC Berkeley)
	 * Venue: NeurIPS 2020
	 * Open source: No
	 * Tags: Imitation Learning, Casual Confusion

## Copycat problem
**The copycat problem:** When expert actions are strongly correlated over time, the imitative policy tend to learn to output previous actions.

In partially observable situations, utilizing previous histories may help the model capture temporal information such as velocity/tracking. However, due to the existence of the copycat problem stated above, single-frame imitation learning can result in much better performance than multi-frame methods.

In Table 1, BC-OH (behavior cloning with history) has lower $MSE(a_t|a_{t-1}, ..., a_{t-k})$ , indicating the fact that BC-OH's action prediction largely rely on previous actions compared to the expert policy. In Figure 2, a BC-OH imitation policy is trained on the Walker-2d environment and it can be observed that its right leg keeps utilizing the same action across frames, demonstrating the copycat problem.

![[Pasted image 20240407160705.png]]

## Solution
To tackle this copycat problem, the author resort to **adversarial training** and **information bottleneck**

Consider an encoder-decoder architecture, a reasonable way to tackle the copycat problem is to eliminate the information related to historical actions in the output encoding of the encoder network.

Adversarial training is a natural choice to achieve this goal. We can train a discriminator network $D(e_t)$to try to predict $a_{t-1}$ from $e_t$, and encourage the encoder to raise the loss of $D$. This process is expected to remove information about $a_{t-1}$ from $e_t$. 

However, as the ultimate goal is to predict $a_t$, and $a_{t-1}$ may actually contain useful information that can help with the $a_t$ prediction task. To avoid falsely eliminating the information related to this task, the authors proposed to also input the prediction $a_t$ into the $D$ network. This technique can be viewed as adding a "shortcut" to $D$. If $D$ would like to use $a_t$-related information to help identifying $a_{t-1}$, it would directly use $a_t$ rather than trying to recover this information from $e_t$. The design choice remove the incentive/pressure for $E$ to eliminate $a_t$-related information during the encoding process.
![[Pasted image 20240407155917.png]]

Another trick is called the **Information Bottleneck (IB)**. The key idea is to let $E$ output encodings as Gaussian distribution and force its KL-div with unit Gaussian to approach zero. This design choice is somewhat empirical. The expectation for IB is to remove nuisance information not essential for the $a_t$ prediction task.

After applying the adversarial and IB trick, the following Figure shows that those tricks indeed help with the copycat problem (lower past action predictability) and resulted in higher performance.

![[Pasted image 20240407163047.png]]

## Summary
This paper discussed the **copycat problem** in the context of imitation learning. It is the phenomenon that an imitative policy tend to output past actions as shortcuts to the learning objective. **Adversarial training** and **information bottleneck** are two useful tricks to tackle this problem.
