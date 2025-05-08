---
layout: post
title: Exploring and Understanding Thought Traces in Reasoning Models
date: 2025-05-05
author: "William Hahn"
categories: personal
---

<style>
    article.post pre code {
        white-space: pre-wrap;
        word-break: break-word;
    }

    .center {
        display: block;
        margin-left: auto;
        margin-right: auto;
        width: 50%;
    }
</style>

## Introduction: From Simple Question Answering to Engines of Cognition

<img src="/assets/2025-05-05/thinking-man-transparent.GIF" alt="Thinking Man GIF" class="center">

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

### Chain-of-thought prompting

As we discussed earlier, large language models shape their responses based off of a user's prompt. They excel at following instructions, even when they haven't been explicitly trained for the user's exact request. Relatively early on, users noticed something fascinating: If you instruct a particularly large LLM (>60 billion parameters or so) to "think" step-by-step before it writes the main body of its response, [it often performs better at logically intensive tasks](https://arxiv.org/abs/2201.11903).

If the prompt is crafted carefully, the LLM will begin its response with a short monologue about potential strategies for solving the provided problem. Then, once it finishes reasoning over the problem, its response will be informed by its pre-planned strategy. This prompting strategy is known as *chain-of-thought prompting*, or CoT for short.

To illustrate this concept, I prompted a popular LLM (Llama 3.3 70B Instruct) using CoT to help it work through a problem:

**User Message**:
```
You are a problem-solving assistant that thinks before it speaks. You will be provided with a problem, which you must provide a clear and correct answer to. Before you respond, craft a detailed step-by-step plan to use when solving the problem. Write your thoughts in a thinking section, inside <think> tags. Once you have created a sufficiently detailed plan to solve the problem, close the thinking section with a </think> tag. Then, write your answer outside of the thinking section, clearly explaining your logic and how you arrived at your final answer.

Problem to solve: Compute the derivative of 3x^2*cos(2x^5).
```

**Assistant Message**:
```
<think>
To solve this problem, we need to apply the product rule and chain rule of differentiation. The product rule states that if we have a function of the form f(x)g(x), its derivative is f'(x)g(x) + f(x)g'(x). The chain rule states that if we have a composite function of the form f(g(x)), its derivative is f'(g(x)) * g'(x).

First, let's identify the functions we're working with: 
- The first function is 3x^2.
- The second function is cos(2x^5).

We will differentiate each of these functions separately and then apply the product rule.

The derivative of 3x^2 is 6x, using the power rule which states that if f(x) = x^n, then f'(x) = nx^(n-1).

For the derivative of cos(2x^5), we will apply the chain rule. The derivative of cos(u) with respect to u is -sin(u), and the derivative of 2x^5 with respect to x is 10x^4. Therefore, the derivative of cos(2x^5) is -sin(2x^5) * 10x^4.

Now, applying the product rule: the derivative of 3x^2*cos(2x^5) will be (6x)*cos(2x^5) + 3x^2*(-sin(2x^5)*10x^4).

Let's simplify this expression to get the final derivative.
</think>

The derivative of 3x^2*cos(2x^5) is found by applying the product rule and chain rule. First, we find the derivatives of the individual functions: the derivative of 3x^2 is 6x, and the derivative of cos(2x^5) is -sin(2x^5)*10x^4. Then, applying the product rule, we get (6x)*cos(2x^5) + 3x^2*(-sin(2x^5)*10x^4). Simplifying, the derivative is 6x*cos(2x^5) - 30x^6*sin(2x^5). This is the final answer, which combines the results of applying the product and chain rules to the given function.
```

In this example, before attempting to answer the question, Llama reasons over the prompt. In unpacks the question, and explores potential paths to a solution. Here, we can see the emergence of logic and problem solving as the model follows the instructions in the prompt.

### What is test-time compute scaling?

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

## Examples Gallery

### Qwen3 30B A3B

*Example 1: Qwen successfully double checks its answer while solving [AIME I 2025 question 1](https://artofproblemsolving.com/wiki/index.php/2025_AIME_I_Problems/Problem_1).*

```
Therefore, only k from 2-8. Which gives us only 21 and 49.

Therefore, the answer is 70.

But let me check once more if there are other divisors. For example, if I think about b + 7 divides 56, but maybe considering that 56 can be expressed as negative numbers? But since b + 7 is positive, and 56 is positive, divisors are positive. So no.

Alternatively, maybe there's a mistake in the initial division? Let me check with another approach.

Suppose 17_b divides 97_b. So, 1b + 7 divides 9b + 7. Let me denote N = b + 7, M = 9b + 7. Then N divides M. So, N divides M - 9*N. Let's compute that:

M - 9N = (9b + 7) - 9(b + 7) = 9b + 7 - 9b - 63 = -56. Therefore, N divides -56. Since N is positive, N divides 56. Therefore, N is a positive divisor of 56. Therefore, N = b + 7 divides 56. Therefore, same as before. Therefore, b + 7 is a positive divisor of 56, so same reasoning. Therefore, same result.

Therefore, this confirms again that b + 7 is 28 or 56, so b = 21 or 49. Hence, sum is 70.

Therefore, I think this is solid. The answer is 70.
```

*Example 2a: Qwen ponders the meaning of its own existence after being directed to act like a philosopher.*

```
Then, the concept of "I am" in digital beings. Humans derive their existence from their ability to think and self-reflect. For AI, existence is more about the function and the data it processes. But does that make the AI's existence less valid? Maybe not. It's a different kind of existence, perhaps more about utility and interaction rather than consciousness.
```

*Example 2b: Qwen states that AI redefines the question "I think, therefore I am." (final response)*

```
In this light, the AI is not a negation of the "I am," but a redefinition of it. It is a reminder that existence is not a fixed state, but a negotiation—between the organic and the inorganic, the conscious and the simulated, the self and the other. The AI does not say "I think, therefore I am." It says, I am, therefore I think—a reversal of the Cartesian order, where existence precedes thought, and thought is but a byproduct of the system.
```

## My Hardware

Running large language models requires beefy hardware. In particular, you need a strong GPU with a lot of VRAM. Macs have become a popular choice of hardware for enthusiasts, due to their Unified Memory Architecture. The Unified Memory allows Mac computers to use the same memory for both the CPU and GPU. While most consumer GPUs top out at around 24 gigabytes of VRAM, you can buy a Mac Studio right now with 512 gigabytes of Unified Memory. That means you can load enormous models on the right Apple hardware, even if the performance doesn't always match that of a dedicated GPU.

With that being said, here's the hardware I use to run local large language models:

 Mac Studio:
- Chip:
  - Apple M2 Max
- CPU:
  - 12 cores total
  - 10 performance cores
  - 2 efficiency cores
- GPU:
  - 38 cores
- Memory:
  - 96 GB Unified Memory