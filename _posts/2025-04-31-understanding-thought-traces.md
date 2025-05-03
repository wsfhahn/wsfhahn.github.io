---
layout: post
title: Exploring and Understand Thought Traces in Reasoning Models
date: 2025-04-31
categories: personal
---

## This post is under construction

![Under Construction](https://i.pinimg.com/originals/eb/1b/27/eb1b27863813653543914d222ceb9cd0.gif)

## Introduction

As the frontier of AI research has continued to creep forward, the paradigm of training large language models has fundamentally shifted.

The first QA capable large language models, made possible by the advent of the Transformer, were little more to users than a cool party trick. GPT-3.5 could, for example, write simple poems, although it often failed to make works rhyme. Attempts to get it to cool tools were futile, as it couldn't consistently write syntactically correct JSON. And math? Give me a break. However, since then, flagship LLMs have become invaluable tools to developers and researchers. They're incredibly powerful for tasks like code generation, especially when paired with a user who understands the code it's writing. In fact, some are even capable of solving novel research problems when directed by a researcher to do so. The magic key that unlocked this new type of intelligence was **inference compute scaling**.

## Inference Compute Scaling

Large language models, at their core, are merely sophisticated next-token predictors. The input text, or the user query, is split up into tokens. The language model's job it to predict the most likely next token, given the context of all of the text in the conversation when the user hit send. The fact that the model completes the response one token at a time rather than in one shot is important, because it means the process is autoregressive. Autoregression allows the model the build up ideas incrementally, unfolding into new insights and understanding with each new token.