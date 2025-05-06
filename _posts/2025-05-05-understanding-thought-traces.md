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

When querying a model like GPT-3.5, the output text was *all* of the text that the model had generated. That is to say, there was no hidden reasoning or thinking before it spoke behind the scenes. Imagine if I asked you to compute the derivative of a complex equation without thinking, no scratch paper, just blurting out the final answer. You would probably have to guess an answer and fail, just like I would. That's because solving complex problems requires exploring steps to arrive at an answer, working through them logically, and evaluating possible solutions. If it's a particularly hard problem, you'll probably check the validity of each step to ensure your answer is logically sound.

So, it becomes self-evident that the ability to plan, execute, evaluate, and correct is also necessary for large language models to solve complex problems and perform intricate tasks. 

But how could a language model learn to mimic human logic, and utilize it to significantly enhance its cognitive abilities? It's rather complex, but I hope to clearly and elegantly explain how in a meaningful way in this post.

## Working Principles of a Large Language Model

### Tokenization

At heart, large language models are merely sophisticated next-word predictors. The input text, or your query to the model, is broken down into a set of tokens. Tokens are an intermediate between letters and words, representing common sequences of letters in your language of choice. For example, the word "underwhelming" might be broken down into the tokens "under" + "whelm" + "ing". Tokens are practical, because they have more meaning than a single letter, and there are far fewer of them than in the dictionary of your language of choice.

When you press enter to submit your request to a large language model, it receives the entire context. The context is all of the content from the whole conversation, including the user messages (your messages) and the assistant messages (the LLM's response). The model's job is to predict the next token following your user message (the start of its response). From there, it continues writing one token after another until it has finished its response (indicated by the model writing a special token called a stop token).

But how is the next token selected from the model's output?

### The output as a probability distribution

The raw output of the LLM is not just a single token. In reality, it is a vector, which represents a probability distribution. This probability distribution is essentially a list of possible next tokens, each assigned a probability.

There are two ways to select a next token from the output. The first is *sampling*.

Sampling the probability distribution involves randomly selecting a next token based on their probabilities. Sampling creates more variety, and allows the LLM to be more expressive. It also causes the outputs to be unique for identical prompts.

The second way to select a next token from the distribution is *greedy decoding*. Greedy decoding is basically just picking the most probable next token, every time. In this case, one prompt will always create the same output.

But after selecting the first token, we need to select the next, and the next, and so on. The model generating one token at a time with each execution is key. This makes the process of generating a response autoregressive.

### Autoregression

Autoregression is one of the key features of the large language model. Autoregression is a method where a language model completes its response one token at a time. This means that each new token is written with the context of all of the previous tokens its written so far, allowing the model to build out ideas and concepts iteratively across multiple tokens.

Autoregression is something that humans and LLMs have in common. When writing a book (or a blog post), you don't just guess the words you write. You use the context of all of the words you have already written to write the next.

Without this autoregression, you would never be able to express

One large question remains. From this process, how could a "personality" or writing style emerge? How can this seemingly simple process give rise to an intelligence capable of writing an essay, or solving a problem?

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