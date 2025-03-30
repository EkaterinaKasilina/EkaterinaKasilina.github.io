---
layout: post
title: KL Divergence vs. Cross-Entropy: Understanding the Difference and Similarities
description: >
  Simple explanation of two crucial ML concepts
  
image: /assets/img/blog/kld.png

sitemap: false
hide_last_modified: true
tags: [ml, metrics]
---

Simple explanation of two crucial ML concepts

The full text and code also can be found in my [Medium blog](https://medium.com/@katykas/kl-divergence-vs-cross-entropy-understanding-the-difference-and-similarities-9cbc0c796598)

## Introduction
When we train machine learning models, especially for classification tasks, two fundamental concepts frequently arise: cross-entropy and KL(Kullback–Leibler) divergence. While cross-entropy is more frequently used as an optimization objective, KL divergence is rarely applied for this purpose.

A quick reminder of why we consider them together in the first place:

### Cross-Entropy Formula:
![](https://miro.medium.com/v2/resize:fit:828/format:webp/1*lfZ9hhE-5FUxNXQ0_0J7Wg.png)

### KL Divergence Formula:
![](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*xDjm8bNW2dtblCX8_2SnXw.png)

So the difference is just the left part! Let’s dive in and explore details.

## Basics
Here are just a few small reminders. Feel free to skip this section if you’re already experienced.

Imagine we’re working on a binary classification task, where we have the actual class distribution (p) and the predicted distribution (q) for labels.

```python
import numpy as np

p = np.array([1, 0])  # True label (one-hot encoded for class 0)
q = np.array([0.7, 0.3])  # Predicted probability distribution
```

The objective is to train the machine learning model so that the predicted distribution (q) closely matches the actual distribution (p).

## KL Divergence
KL divergence (Kullback-Leibler divergence) is a measure of how one probability distribution q(x) diverges from a true distribution p(x). It tells us how much information is lost when q(x) is used to approximate p(x).


KL Divergence Formula:

```python
def kl_divergence(p, q):
    # Add a small constant to avoid log(0) errors
    epsilon = 1e-10
    p = np.clip(p, epsilon, 1.0)
    q = np.clip(q, epsilon, 1.0)
    
    # Calculate KL Divergence
    return np.sum(p * np.log(p)) - np.sum(p * np.log(q))

# Compute KL Divergence
kl = kl_divergence(p, q)
print(f"KL Divergence: {kl}")
```
```python
Output:
KL Divergence: 0.3566749417565447
```

* If q(x) is a perfect match for p(x), KL divergence is 0.
* If q(x) poorly approximates p(x), KL divergence increases.
* It is not symmetric: KL(p || q) ≠ KL(q || p)

## Cross-Entropy and relationship with KL Divergence
Cross-entropy measures how different one probability distribution is from another. It quantifies the number of bits required to encode data from a true distribution p(x) using an approximating distribution q(x).

![](https://miro.medium.com/v2/resize:fit:828/format:webp/1*lfZ9hhE-5FUxNXQ0_0J7Wg.png)

## Cross-Entropy
We can rewrite the formula:  
![](https://miro.medium.com/v2/resize:fit:1064/format:webp/1*VLVD6X6LNue3zJ7_dSAftw.png)

Where H(p) represents the entropy of the true distribution  
![](https://miro.medium.com/v2/resize:fit:610/format:webp/1*L9_tttKyPckrC4_ssE_nKA.png)

```python
def cross_entropy(p, q):
    return -np.sum(p * np.log(q))

def entropy(p):
    epsilon = 1e-10
    p = np.clip(p, epsilon, 1.0)
    return - np.sum(p * np.log(p))

ce = cross_entropy(p, q)
print(f"Cross-Entropy: {ce}")

hp = entropy(p)
print(f"Cross-Entropy using KL Divergence:", hp + kl)
```

```python
Output:
Cross-Entropy: 0.35667494405912975
Cross-Entropy using KL Divergence: 0.35667494405912975
```

Since entropy H(p) is fixed for a given p, minimizing cross-entropy effectively minimizes KL divergence**. This is why cross-entropy is commonly used as a loss function — it ensures that the predicted distribution q(x) gets as close as possible to p(x).

**In the context of Maximum Likelihood Estimation (MLE), minimizing KL divergence and cross-entropy essentially lead to the same result. However, in Bayesian Inference (Variational Inference), we minimize the Evidence Lower Bound (ELBO).

## When to Use Cross-Entropy vs. KL Divergence
* Cross-Entropy is typically used as a loss function in classification tasks, especially when the goal is to train a model to predict a probability distribution.
* KL divergence is used to measure the difference between two probability distributions, p (true distribution) and q (predicted distribution). It is often used in unsupervised settings or in cases where you’re approximating a distribution (e.g., in variational inference, GANs, etc.)
That’s it!

Thanks for reading!
