---
layout: post
title: Exploring and Understand Thought Traces in Reasoning Models
date: 2025-05-05
categories: personal
---

## This post is under construction

![Under Construction](https://i.pinimg.com/originals/eb/1b/27/eb1b27863813653543914d222ceb9cd0.gif)

## Introduction: From Simple Question Answering to Engines of Cognition

![Thinking Man](/assets/2025-05-05/thinking-man-transparent.GIF)

### The release of GPT-3.5

As the frontier of AI research continues to creep forward, there's never been a better time to take a look in the rear-view mirror and witness how far we've come. I still clearly remember, as I'm sure you do as well, the very first release of ChatGPT. It's not often we see a tech-demo so disruptive that it shakes the internet to its core. Within the first 5 days after its release, ChatGPT reached one million users, which is an astonishing rate of growth.

![Stylized Path to First 1 Million Users](/assets/2025-05-05/first-1-million-users.jpeg)

### Why it struggled to solve complex problems

Despite its rapid adoption by users around the globe, the GPT-3.5 model under the hood suffered from major limitations. It's responses were shallow, and lacked real depth. It didn't present itself as a problem-solver, because it wasn't one. It hallucinated left and right, and users quickly learned not to trust it (which meaningfully harmed the reputation of the field as a whole). This is because it wasn't trained for complex reasoning and logic.

When querying a model like GPT-3.5, the output text was *all* of the text that the model had generated. That is to say, there was no hidden reasoning or thinking before it spoke behind the scenes. Imagine if I asked you to compute the derivative of a complex equation without thinking, no scratch paper, just blurting out the final answer. You would probably have to guess an answer and fail, just like I would. That's because solving complex problems requires exploring steps to arrive at an answer, working through them logically, and evaluating possible solutions. If it's a particularly hard problem, you'll probably check the validity of each step to ensure your answer is logically sound. GPT-3.5 was not capable of logic in this sense.

So, it becomes evident that the ability to plan, execute, evaluate, and correct is also necessary for large language models to solve complex problems and perform intricate tasks. 

But how could a language model learn to mimic human logic, and utilize it to significantly enhance its cognitive abilities? It's rather complex, but I hope to clearly and elegantly explain how in a meaningful way in this post.

Before we dive into the methods used by model providers to train models for reasoning, we must first understand the basic working principles of a large language model.

## Working Principles of a Large Language Model

### Tokenization

At heart, large language models are sophisticated next-word predictors. The input text, or your query to the model, is broken down into a list of tokens. Tokens are an intermediate between letters and words, representing common sequences of letters in your language of choice. For example, the word "underwhelming" might be broken down into the tokens "under" + "whelm" + "ing". Tokens are practical, because they have more meaning than a single letter, and there are far fewer of them than there are words in the dictionary.

When you press enter to submit your request to a large language model, it receives the entire context. The context is all of the content from the whole conversation, including the user messages (your messages) and the assistant messages (the LLM's responses). The model's job is to predict the next token following your user message (the start of its response). From there, it continues writing one token after another until it has finished its response (indicated by the model writing a special token called an end-of-sequence token).

But the model doesn't directly output tokens. Instead, its raw output is a vector of logits, which needs to be processed and converted into a probability distribution. Then, a next token must be selected from this probability distribution.

### The output as a probability distribution

There are two primary methods to select a next token from the output. The first is *sampling*. Sampling the probability distribution involves randomly selecting a next token based on their probabilities. Sampling creates more variety, and allows the LLM to be more expressive. In most cases, it also causes the outputs to be unique, even for identical prompts.

The second way to select a next token from the distribution is *greedy decoding*. Greedy decoding is basically just picking the most probable next token, every time. In this case, one prompt will always create the same output.

But after selecting the first token, we need rerun the model to select the next token, and the next, and so on until we reach an end-of-sequence token. The model generating one token at a time with each execution is key. This makes the process of generating a response autoregressive.

### Autoregression

Autoregression is one of the key features of the large language model. Autoregression is the process of iteratively generating new outputs with reference to all of the previous outputs. In the context of large language models, it describes the process of generating one token after another, using all of the previously generated tokens as context.

This process allows a large language models to express ideas and concepts across multiple tokens, while building upon what it's generated so far.

Autoregression is something that humans and LLMs have in common. When writing a book (or a blog post), you don't just guess the next word you write. You use the context of all of the words you have already written.

Without autoregression, ideas and concepts would be shallow and lack sophistication. Each new token accumulates nuance, abstraction and complexity. Through this process, a unique writing style is also expressed, and coherence can be maintained across long contexts.

### Emergence

When discussing LLMs, a frequently explored topic is emergence. Through autoregression, the model gives expression to emergent properties, complex behaviors that arise from interactions across layers and tokens. Emergence allows for LLMs to express behaviors that weren't programmed, and carry out abstract reasoning and understanding.

One great example of a property that emerges in LLMs is the ability to solve never-before seen problems solely by following user instructions. Dynamic style adaptation is another good example, wherein the model can adapt its writing patterns and style depending on the context of the conversation.

Emergent properties like these are what give large language models their shocking flexibility and depth. They don't act like rigid, rule-base systems, but rather dynamic agents – partners in conversation. When you interact with very large models like OpenAI's GPT-4o, it often feels like you're chatting with another human-like intelligence. This effect is often hard to shake even when you understand that they're much more similar to mirrors, reflecting back responses entirely shaped by your inputs.

But how can we bend and shape the output of a large language model to carry out logic? We need to do more than just roughly mimic human logic. It needs to be grounded in truth, and incredible stable across long generations. Researchers at OpenAI developed an intuitive solution: *test-time compute scaling*.

## The Rise of Test-Time Compute Scaling

### What is test-time compute scaling?

As we discussed earlier, large language models shape their responses based off of the content of a user's prompt. They excel at following instructions, even if they haven't been explicitly trained for a particular use case. Relatively early on, users noticed something fascinating: If you instruct an LLM to "think" step-by-step before it writes the main body of its response, it often performs better at logically intensive tasks.

If the prompt is crafted carefully, the LLM will begin its response with a short monologue about potential strategies for solving the provided problem. Then, once it finishes reasoning over the problem, its response will be informed by a 

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