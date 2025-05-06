---
layout: post
title: Exploring and Understand Thought Traces in Reasoning Models
date: 2025-05-05
categories: personal
---

## This post is under construction

![Under Construction](https://i.pinimg.com/originals/eb/1b/27/eb1b27863813653543914d222ceb9cd0.gif)

## Introduction: From Simple Question Answering to Engines of Cognition

### The release of GPT-3.5

As the frontier of AI research has continued to creep forward, there's never been a better time to take a look in the rear-view mirror and observe how far we've come. I still clearly remember, as I'm sure you do as well, the very first release of ChatGPT. It's not often we see a tech-demo so eye-opening that it shakes the internet to its core. Within the first 5 days after its release, ChatGPT reached one million users, which is absolutely astonishing.

![Stylized Path to First 1 Million Users](/assets/2025-05-05/first-1-million-users.jpeg)

### Why it struggled to solve complex problems

Despite its rapid adoption by users around the globe, the GPT-3.5 model under the hood suffered from major limitations. It's responses were shallow, and lacked real depth. It didn't talk like an intellectual, because it wasn't one. It hallucinated left and right, and users quickly learned not to trust it (which meaningfully harmed the reputation of the field as a whole). This is because it wasn't capable of reasoning.

When querying a model like GPT-3.5, the output text was *all* of the text that the model had generated. That is to say, there was no hidden reasoning or thinking before it spoke behind the scenes. So, imagine if I asked you to compute the derivative of a complex equation without thinking, no scratch paper, just blurting out the final answer. You would probably fall flat on your face at this task, just like I would. That's because solving complex problems requires exploring steps to arrive at an answer, working through them logically, and evaluating possible answers. If it's a particularly hard problem, you'll probably check the validity of each step to ensure your answer is logically sound.

So, how do we mimic this in a large language model, given that it is merely a sophisticated next-word predictor? It's complicated, but throughout this blog post, I hope to break it down in a meaningful way that makes it apparent.

## Fundamental Principles of a Large Language Model

### The output as a probability distribution

As I mentioned earlier, large language models are merely sophisticated next-word predictors. The input text, or your query to the model, is broken down into a set of tokens. Tokens are an intermediate between letters and words, representing common sequences of letters in your language of choice. For example, the word "underwhelming" might be broken down into the tokens "under" + "whelm" + "ing". This is because it would not be practical to train a language model on the entire dictionary, and it's simpler to train it on smaller vocabulary of common groupings of letters.

Just as our queries to large language models are composed of tokens, the model's outputs are also composed of tokens. However, there's one key missing piece. The existence With each pass of the model, a distribution of most likely 

One burning question that immediately arrises is how the next token is predicted. 

### Autoregression

Autoregression is one of the key features of the large language model. Autoregression is a method where a language model completes its response one token at a time. This means that each new token is written with the context of all of the previous tokens, allowing the model to build out ideas and concepts iteratively across multiple tokens.

### Emergence

The "personality" and writing style of a large language model are emergent properties themselves, which are expressed through this autoregressive process.

But this leaves a burning question. 



## The Rise of Test-Time Compute Scaling

### What is test-time compute scaling?

### Breakthrough after breakthrough

## Inside a Thought Trace: Anatomy of an LLM's Reasoning

### What does a trace look like?

### Emergent behavior in plain view

### Anthropomorphized framing as a learned strategy

## Why Thought Traces are so Powerful

### Implications for transparency

### Real world analogs

## Learning from the Machine: How Traces Teach Us

## Future Directions

## Conclusion