---
title: "From Dictionary Lookups to Attention: How Language Models Learned to Understand"
date: 2026-05-24
draft: false
tags: ["NLP", "transformers", "deep-learning", "AI"]
categories: ["Deep Dives"]
summary: "A first-principles walkthrough of how machines went from keyword matching to genuine language understanding — and why each attempt failed before transformers cracked it."
ShowToc: true
TocOpen: false
---

## The Problem That Took 60 Years

Here's the fundamental challenge of natural language processing: humans understand words. Computers don't. A computer sees "apple" and has no idea whether you're talking about a fruit or a trillion-dollar company. Worse — change the arrangement of the same words, and the entire meaning changes. "The dog bit the man" and "The man bit the dog" use identical words but mean completely different things.

Every attempt to make machines understand language is an attempt to solve this gap. Here's how each one failed — and what finally worked.

---

## Attempt 1: Dictionary-Based Models

The first instinct was obvious: give the machine a dictionary. Map every word to a definition, look up meanings, done.

**Why it failed:** The same word shows up in completely different contexts. "Blackberry" is a fruit. "BlackBerry" is a dead phone company. "Python" is a snake and a programming language. A dictionary can't tell you which meaning applies — it just lists all of them and shrugs.

**The lesson:** Static definitions don't capture context. Meaning is not a property of the word — it's a property of the word *in a situation*.

---

## Attempt 2: Statistical Pattern Matching

If dictionaries don't work, maybe statistics will. Count how often words appear together in large datasets. If "machine" and "learning" appear near each other frequently, assume they're related. Predict the next word based on frequency.

**Why it failed:** Pattern matching is not understanding. These models could tell you that "I went to the ___" is probably followed by "store" or "park" because those completions are statistically common. But they had no concept of *why*. They lacked context entirely. They were sophisticated autocomplete — nothing more.

**The lesson:** Frequency of co-occurrence is a weak proxy for meaning.

---

## Attempt 3: Word Embeddings — Words as Vectors

This is where things got interesting. Instead of defining words with dictionaries or counting co-occurrences, researchers asked: *what if we represent each word as a point in space?*

The idea: convert every word into a vector — a list of numbers — where the position in this space encodes the word's meaning *relative to every other word*. Words with similar meanings cluster together. Words with different meanings sit far apart.

The power of this approach is best illustrated by the arithmetic it enables:

```
King - Man + Woman = Queen
Apple - Edibility + Company = Apple (the company)
Swimming - Swim + Throw = Throwing
```

This was a genuine breakthrough. For the first time, a machine had something resembling *semantic understanding*. "King" and "Queen" are related in the same direction as "Man" and "Woman". The geometry of language encodes meaning.

**But it wasn't enough.** These embeddings were static. The vector for "Python" was the same whether you were discussing snakes or code. Context was still missing from the representation itself.

---

## Attempt 4: Sequence Models — RNNs, LSTMs, and the Memory Problem

Next idea: don't just represent words — *read them in order*. Process a sentence word by word, maintaining a running memory of what you've seen so far.

### RNNs (Recurrent Neural Networks)

An RNN processes words one at a time, like a person reading a book through a tiny slit in a piece of paper. It maintains a hidden state — a kind of short-term memory — that gets updated with each new word.

**The fatal flaw: vanishing gradients.** As sentences get longer, the signal from early words gets diluted. By the time the RNN reaches word 50, it has largely forgotten word 3. It's like a boat with a hole — information leaks out constantly.

And because it processes words sequentially, one at a time, you can't parallelize it effectively. GPUs sit mostly idle.

### LSTMs (Long Short-Term Memory)

LSTMs tried to fix the memory problem by adding *gates* — mechanisms that decide what information to keep and what to discard. Think of it as a filing system. The "forget gate" throws away irrelevant information. The "input gate" decides what new information is worth storing. The "output gate" controls what gets passed forward.

**Better, but still limited.** The gates helped, but information still had to squeeze through a single memory line. With very long sequences, even selective memory wasn't enough. And the sequential processing bottleneck remained — still can't use GPUs efficiently.

### The Comparison

| Feature | RNN | LSTM | Transformer |
|---------|-----|------|-------------|
| Reading Style | One word at a time | One word at a time + filters | Entire sentence at once |
| Memory | Very short-term | Improved, but limited | Perfect across the sequence |
| Speed | Slow (sequential) | Slow (sequential) | Fast (parallel) |
| Understanding | Basic | Moderate | Deep and contextual |

---

## Attempt 5: Transformers — The Architecture That Changed Everything

In 2017, the paper "Attention Is All You Need" introduced transformers. The core insight was radical: **stop reading sequentially. Look at every word simultaneously.**

Instead of processing a sentence left-to-right like every previous model, a transformer *observes* the entire sentence at once. It sees how every word relates to every other word, all in parallel.

### How Attention Works

The mechanism has three components — Query, Key, and Value:

- **Query (Q):** Each word asks, "Is anyone here relevant to my meaning?"
- **Key (K):** Each word broadcasts, "This is what I represent."
- **Value (V):** If a match is found, information is shared.

Here's the concrete example that makes it click:

```
"The animal did not cross the road, because it was tired."
→ "it" refers to "animal" (tired = property of a living thing)

"The animal did not cross the road, because it was wide."
→ "it" refers to "road" (wide = property of a surface)
```

A transformer resolves this correctly because the attention mechanism lets "it" look at *every other word* and determine which one it's most related to in this specific context. An RNN, processing left-to-right, would struggle — by the time it reaches "tired" or "wide", the signal from "animal" and "road" has already degraded.

### Why It Scales

Because attention is computed in parallel (every word attends to every other word simultaneously), transformers can fully exploit GPU hardware. An RNN with 1000 words needs 1000 sequential steps. A transformer does it in one parallel pass.

The cost: attention has O(n²) complexity. 10 words = 100 attention connections. 1000 words = 1,000,000 connections. This is why longer context windows require exponentially more compute. But the trade-off is worth it — you get deep, contextual understanding at every position.

### The Evolution Continues

The transformer architecture didn't stop at basic language modeling:

**Transformers** → **Transformers with chain-of-thought reasoning** (more computation per token = better inference) → **Transformers + reasoning + function calling** (agents that can act on the world, not just generate text)

Each generation builds on the same attention mechanism. The architecture is the foundation; everything else is scaffolding on top.

---

## The Classroom Analogy

Here's the mental model I use to think about all of this:

Imagine every word in a sentence is a **student** sitting in a classroom. Each student has an **ID card** with thousands of traits — their meaning along different dimensions (gender, tone, technicality, formality, etc.). This ID card is the word's *embedding*.

In an RNN classroom, students can only whisper to the person sitting next to them. Information passes one seat at a time. By the time a message reaches the other end of the room, it's garbled.

In a transformer classroom, every student can make *eye contact* with every other student simultaneously. They all look at each other, figure out who's relevant to whom, and update their own ID cards based on those connections. The student "Python" looks around — if the room is full of biology students, his ID card updates to mean "snake." If the room is full of computer science students, it updates to mean "programming language."

**The key insight:** meaning is not a fixed property of a word. It's dynamically constructed by the *context* surrounding it. Transformers are the first architecture that truly captures this.

---

## Why This Matters for Engineers

If you're building AI systems in 2026, you're building on transformers. Understanding *why* they work — not just *that* they work — changes how you approach problems:

- You understand why context window limits matter (O(n²) attention cost)
- You understand why fine-tuning works (the embedding space is already semantically structured)
- You understand why RAG exists (extend context without extending the window)
- You understand why prompting matters (you're manipulating the attention pattern)

The history isn't academic trivia. Each failed attempt teaches you a constraint that the final architecture had to overcome. And the next architecture will overcome constraints we haven't identified yet.

---

*This post is part of my ongoing notes from the 100x School AI/ML Bootcamp. I write to learn, and I share to compound.*
