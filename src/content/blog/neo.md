---
title: 'Neo: Synthetic Data Generation at Home'
description: ''
pubDate: 'Apr 19 2026'
heroImage: '../../assets/neo/pascals_triangle.png'
---

> The accompanying GitHub repository for this post will be made available soon.

## Introduction

Since the dawn of ChatGPT, I've been fascinated by the process of training large language models at home. While the process of training a model is fairly well documented, obtaining high-quality, domain-specific data for training remains a barrier for DIYers like myself. Many niche datasets contain questionable content, broken formatting, and other artifacts which reduce the viability of small models trained on them. As a result, much of the process's complexity and difficulty is offloaded to the mid-training and post-training stages, reducing the performance and predictability of the resultant model.

After running into this problem head-on, I set out to build a synthetic data generation system for churning out steerable, auditable, pure data to train my own models. If all goes well, I'll have high quality data on demand, and I'll be able to train small language models which benchmark higher, remain coherent in my intended domains, and can be iterated on rapidly.

In this post, I will begin by covering the key concepts and terminology involved, document the process of designing the system, and wrap up with a tutorial on how to deploy my system on your own hardware.

## Offline Distillation

![Distillation](../../assets/neo/distillation.png)

The process of distilling a model is simple in principle. While one could choose to pre-train a model from scratch on a vast corpus of scraped internet knowledge using a dataset like [OpenWebText](https://huggingface.co/datasets/Skylion007/openwebtext), many open-source and open-weights models exist today which have already learned the trickier concepts to teach a model, such as reasoning, tool calling, and formatting.

Pre-training on large datasets is relatively simple, but reinforcement learning (used to tune behavior and the aforementioned trickier concepts) is a difficult niche to truly master. Thankfully, the open models available today have already been subjected to advanced RL post-training methods, and have internalized the patterns and behaviors taught by it. These models can be repeatedly prompted and their responses recorded, building a dataset of synthetic data on which a new model can be trained.

This process is known as distillation. More specifically, I walked into this project set on performing an offline distillation. In the context of generative language modeling, offline distillation is the process of generating a synthetic dataset using an existing model (the teacher) which encapsulates its knowledge, formatting, and behavior, and training a smaller model (the student) on that data. On-policy distillation works by first letting the student generate its own rollout given a prompt, then having the teacher generate its own rollout on that same prompt (or on the student's trajectory). The student is then updated on a learning signal derived from the teacher's correction, evaluation, or continuation.

For the sake of simplicity, I chose to perform an offline distillation, because I already have training pipelines set up which I wanted to use. In the future, however, I may aim to perform an on-policy distillation, because on-policy mitigates the problem of the student compounding errors during inference; but that is not a project for today.

## A Discussion of the Requirements Involved

### The Queries Problem

Anyone who has interacted with a large language model for a meaningful amount of time intuitively understands the causal relationship between a user query and a model's response. That is to say, the content which a model generates is caused by and a result of the prompt submitted by a user. If you tell a model to generate an essay about caring for plants in the home, that causes it to generate an essay about caring for plants in the home. Similarly, if you tell it to write a very long essay about the same topic, the essay it generates will likely be much longer than it otherwise would have been.

The challenge this presents is that a teacher model will not generate high quality synthetic data in the absence of high quality queries. In order to generate a diverse set of responses written by the teacher, a diverse set of queries is a prerequisite. A core goal from the start of this project was to generate data which is rich in diverse, novel signal, rather than monotonous and repetitive noise.

I did some napkin math, and worked out that if I wished to generate one billion tokens (which, in many cases, is only a modest amount), and assumed that on average a query results in the model generating 2,000 tokens, 500,000 queries would be required across a wide range of topics to elicit as much knowledge as possible from the teacher. If I instead generated five responses per rollout, the number reduces to 100,000 queries, which is still meaningfully significant.

One key aspect of distillation, however, is that capturing knowledge from the teacher is not enough on its own. A true distillation requires also teaching the student to model different patterns of behavior, rather than just diverse knowledge. By leveraging multiple system prompts, a diverse distribution of behavior becomes attainable. Running generation with five standard system prompts, and generating five responses per system prompt, the total number of queries required to generate one billion tokens further shrinks to 20,000.

Since manually writing 20,000 queries does not sound like a fun weekend (or month, for that matter), synthetically generating queries became a design requirement for this project.

### Multi-turn Interaction

![Dominoes Falling](../../assets/neo/dominoes.png)

While it would be a sound gamble to bet that a sizable portion of interactions with large language models only comprise of one message and one response, this is not always the case. After a model generates a response, a user may provide a correction, change the topic, or continue the conversation in some other way. In all of these cases, the model's output is not agnostic to the content it generated earlier in the message chain. A simple demonstration is to query a language model for help writing an essay, and then ask it "Where next?" The response it generates will be informed by the first query as well as its response.

If a language model is only trained for single-turn interaction, and is thrown into a multi-turn scenario, it will not have the necessary priors to understand the relationship between the earlier messages and the latest message. Consequently, generating only single-turn data is not sufficient to train a language model to encapsulate the behavior of the teacher model.

This principle defined the course of this project, and the shape the pipeline and primitives would need to take on. While I initially intended to build a naïvely simple pipeline that would generate queries and one response for each query, it became clear that a design which could handle chats of any arbitrary length was necessary to generate the data needed.

### Model Selection

![A clay model of Albert Carrier by Auguste Rodin](../../assets/neo/clay_model.jpg)

#### Types of Models

Choosing a model to serve as the teacher was the most amusing part of this project. While it may be tempting to choose a closed-source frontier model such as GPT-5.4, such closed models are cost-prohibitive. In addition to the cost, the APIs used to interact with these models have relatively low rate limits, which restricts the generation speed.

Closed-source AI labs are also not particularly fond of the idea of their [models being used to generate synthetic data for distillation](https://www.anthropic.com/news/detecting-and-preventing-distillation-attacks). While ironic that they consider such an act intellectual property theft (given that they train their models are trained on most of the books that can be found in your local library), I don't make the rules. Distilling their models is considered a bannable offense, which is not a risk I am willing to take.

Thankfully, as previously mentioned, a [cornucopia](https://huggingface.co/models) of open models exists on the internet, which can be freely downloaded and run on just about any hardware architecture (give the hardware has sufficient memory and can meet my throughput needs). Out of the common architectures, two prominent families exist: dense models and mixture-of-experts (MoE) models.

Dense models use all of their active parameters during a forward pass. For each token they write, the entirety of the model is used. This makes dense models more capable for their size, but much slower to run (often 5x slower from my testing). MoE models, on the other hand, are composed of many smaller "expert" networks, which specialize into different niches at train time. During a forward pass, only the selected number of experts (typically 8) are activated. For example, in the case of Gemma 3 26B A4B, only 4 billion out of the total 26 billion parameters are active during generation for a given token. This makes them significantly more compute-efficient, albeit slightly less capable than their dense counterparts.

#### Models Tested

Due to my throughput requirements, I decided that I would only consider MoE models for this project. Out of the available models which I can run locally on my own hardware, I tested the following options:

| Model Name | Parameters Total | Parameters Active | Total Experts | Active Experts |
| --- | --- | --- | --- | --- |
| [Qwen3.6 35B A3B](https://huggingface.co/Qwen/Qwen3.6-35B-A3B) | 35B | 3B | 256 | 8 |
| [GLM 4.7 Flash](https://huggingface.co/zai-org/GLM-4.7-Flash) | 30B | 3B | 64 | 4 |
| [Gemma 4 26B A4B](https://huggingface.co/google/gemma-4-26B-A4B-it) | 26B | 4B | 128 | 8 |

After scientifically smelling the responses for funny vibes and brittle behavior, I chose Gemma 4 26B A4B due to its recent knowledge cutoff, accuracy, and compact chain-of-thought reasoning.

#### Model Configuration and Deployment

In order to run this model, I first needed to choose a library to run it with. Since I'm familiar with it and have used it for a long time, I chose to run the model using [llama.cpp](https://github.com/ggml-org/llama.cpp), which is an efficient pipeline with C++ implementations of many popular architectures. It also works well on Apple Silicon, and has an OpenAI compatible API, making integration simple.

I got started by downloading llama.cpp locally to my computer using the following commands:

```bash
cd ~/
git clone --depth=1 https://github.com/ggml-org/llama.cpp
cd llama.cpp/
```

Once downloaded, I installed the Python libraries it requires for converting models to its native `gguf` file format using:

```bash
uv sync
```

After all of the requirements finished installing, I began the model download, which is about 55GB in size:

```bash
hf download google/gemma-4-26B-A4B-it --local-dir /Volumes/Data/gemma-4-26B-A4B-it
```

Since the model downloads in HuggingFace's `safetensors` format, I converted it to a `gguf` file using the built-in conversion utility:

```bash
uv run convert_hf_to_gguf.py /Volumes/Data/gemma-4-26B-A4B-it --outfile /Volumes/Data/gemma-4-26B-A4B-it.gguf
```

Next, I decided that in order to speed up inference, I would sacrifice a small slice of performance in the name of speed by quantizing the model. By default, Gemma's parameters are stored as 16-bit floating point numbers. However, they can be converted (quantized) to 8-bit integers, which reduces the model's memory footprint and speeds up inference on most platforms. Quantizing the model first requires building the llama.cpp C++ backend, which I achieved by running:

```bash
cmake -B build
cmake --build build --config Release -j12 # replace -j12 with -j[your processor's number of threads]
```

Then, with the project built, I moved on to quantizing the model. Sorry, Gemma.

```bash
./build/bin/llama-quantize /Volumes/Data/gemma-4-26B-A4B-it.gguf /Volumes/Data/gemma-4-A4B-it-Q8_0.gguf Q8_0
```

After the command finished, the model was compressed and ready to run!

![Gemma weights on my computer](../../assets/neo/gemma_weights.jpg)

Next, I tested the model using the integrated server's chat interface. Note that this operation requires about 40GB of available memory. To reduce this number, you may wish to requantize the model to a lower precision such as `Q6_K` or `Q4_K_M`, or reduce the `--ctx-size`.

```bash
./build/bin/llama-server \
    --model /Volumes/Data/gemma-4-26B-A4B-it-Q8_0.gguf \
    --reasoning off \
    --ctx-size 128000 \
    --host 0.0.0.0 \
    --port 8080
```

This command hosts the server, and the user interface can be accessed at `localhost:8080`. Note that for this Gemma model, the following sampling parameters are recommended. In the chat interface, these can be set within the settings menu, and later I'll cover configuring these when making requests to the server:

- Temperature: 1.0
- Top K: 64
- Top P: 1.0
- Min P: 0.0

When the llama.cpp server is running, it hosts an OpenAI compatible API. This API can be interacted with using the same patterns as OpenAI's own API used to serve their models. OpenAI also has a Python library called `openai`, which provides helpful primitives and functions for interacting with the API. A simple example of interacting with the API looks like:

```python
from openai import OpenAI, ChatCompletionMessageParam

MODEL_ID = "gemma-4-26B-A4B-it-Q8_0.gguf" # change if your model has a different ID
BASE_URL = "http://localhost:8080"
API_KEY = "none" # set to a real key for working with a different API which requires authentication

TEMPERATURE = 1.0
TOP_K = 64
TOP_P = 1.0
MIN_P = 0.0

client = OpenAI(
    base_url=BASE_URL
    api_key=API_KEY
)

chat: list[ChatCompletionMessageParam] = [
    {"role": "user", "content": "Who are you?"}
]

response = client.chat.completions.create(
    model=MODEL_ID,
    messages=chat,
    temperature=TEMPERATURE,
    extra_body={
        "top_k": TOP_K,
        "top_p": TOP_P,
        "min_p": MIN_P
    }
)

print(response.choices[0].message.content)
```

## The Meat and Potatoes

![Engineers working with computing equipment](../../assets/neo/engineers.jpeg)

With the infrastructure set up and tested, I began working on the data generator itself. I chose to call this data generator Neo, since it is the newest of the data generation projects I've worked on, and will be used to bring a new model into the world.

### Server Design Requirements

I started by laying out the rough requirements which would be used to design the server.

First, I mandated that I would avoid any unnecessary complexity by all means necessary. While it can be tempting to take shortcuts, doing so risks accumulating technical debt, or worse yet the gods cursing you with an unsatisfactory harvest. Things break, and I knew that changes would need to be made, so the code would need to be readable and workable from the start.

Second, the server needed to operate asynchronously. Synthetic data generation runs can take a long time, and eventually, I get bored and close my laptop. When that happens, the request should not be dropped, and the server should continue generating.

Third, reliability was non-negotiable. It's rare for something to work perfectly the first time, even if the issues were not my own doing (though, admittedly that is the most likely scenario). Models make mistakes, and sometimes fail to adhere to structured output schemas, the llama.cpp server could go down, or something entirely unexpected could happen. None of those things could be allowed to bring down or spoil a run which had been running for multiple hours or days.

Fourth, the server had to be able to export data in a widely compatible format. It couldn't save in a format which would be tricky to integrate with in the future.

Fifth, I would not design the infrastructure and primitives for this project alone. I could foresee a scenario where it would be expedient to reuse a lot of the code from this project. Models, schemas, and functions needed to be usable in plain Python without the server acting as a barrier to reuse.

Finally, security. I'm not well versed in cybersecurity, so I wouldn't want this project to handle any sensitive data being transmitted between the server and the client. Thus, the server would only hang onto the API key for any non-local model providers used, and would transmit no sensitive data.

### Chats as a Primitive

In order to act in the spirit of simplicity and extensibility, I chose to make the `Chat` the core primitive of this project.

The `Chat` object extends the [pydantic](https://pydantic.dev) `BaseModel` class so that it can be easily parameterized and validated. This primitive has four key responsibilities:

1. Store messages
2. Interoperate with OpenAI message lists (`list[ChatCompletionMessageParam]`)
3. Generate plain text and structured output responses with a single method
4. Generate followup queries

Messages are stored using another class, `ChatMessage`. Together, they make creating and storing chats simple, and generating responses one function call away. Instantiating a `Chat` is simple:

```python
from app.common.chats import Chat, ChatMessage

my_chat = Chat(
    complete=False, # defaults to False if left unspecified
    messages=[
        ChatMessage(
            role="system",
            content="You are a helpful assistant."
        ),
        ChatMessage(
            role="user",
            content="What is the capital of France?"
        )
    ]
)
```

Once instantiated, generating a response and automatically appending it to the chat is also trivial:

```python
my_chat.generate(
    max_retries=3,
    model_id="gemma-4-26B-A4B-it-Q8_0.gguf", # will default to env variable DEFAULT_MODEL_ID
    append_to_chat=True # defaults to true if unspecified
)
```

At any point, I can append a new message from a `str`, `ChatCompletionMessageParam`, or `ChatMessage`:

```python
# from string
my_chat.add_message(
    message="What is the capital of Germany?",
    role="user"
)

# from ChatCompletionMessageParam
my_chat.add_message(
    message={
        "role": "assistant",
        "content": "The capital of Germany is Berlin."
    }
)

# from ChatMessage
my_chat.add_message(
    message=ChatMessage(
        role="user",
        content="What is the capital of China?"
    )
)

# generate
my_chat.generate(max_retries=3)
```

You can also set the system message to influence model behavior mid-chat, supporting a `message` parameter as `str`, `ChatCompletionMessageParam`, or `ChatMessage`:

```python
# set from string
my_chat.set_system_message("You are an expert in geography.")

# create a new chat with a new system message
with_new_system = my_chat.duplicate_with_system_message("You are an expert in international relations.")
```

To simplify implementations for synthetic data generation, two methods are worth noting. The method `to_context_string()` dumps the context as a string which can be sent to agents for tasks like summarization, and `generate_followup()` wraps the `generate()` method to generate a follow-up reply as a `FollowUpResponse`:

```python
# generate a follow-up and append it to the chat
my_chat.generate_followup(
    max_retries=3,
    model_id="gemma-4-26B-A4B-it-Q8_0.gguf`, # defaults to DEFAULT_MODEL_ID env var
    append_to_chat=True # defaults to True
)

# print the context string
print(my_chat.to_context_string())
```


If you choose to write your own OpenAI API wrapper, it can be easily dumped to a `list[ChatCompletionMessageParam]`, or created from `list[ChatCompletionMessageParam]`:

```python
openai_messages: list[ChatCompletionMessageParam] = my_chat.to_openai_chat()

another_chat = Chat.from_openai_chat(openai_messages)
```

### Jobs

In this project, data generation tasks exist as jobs. As of right now, there are two types of jobs: `QueriesJob` and `DataJob`. `JobType` is a special typing variable which is defined as `JobType = Union[QueriesJob, DataJob]`.

There are three objects which are used globally in the project to handle jobs throughout the system:

```python
from uuid import UUID
from asyncio import Queue, Lock
from app.common.types import JobType

jobs: dict[UUID, JobType] = {}
job_queue: Queue[UUID] = Queue()
job_lock: Lock()
```

`jobs` stores registered jobs within a dictionary with `UUID` keys. When a job is registered, it is assigned a random UUID and inserted into this dictionary. `job_queue` is a queue of UUIDs, which is used by a worker process to fetch jobs which have not yet been completed, and run them sequentially. When a job is registered, its status is set to `pending`, and it is added to the queue. `job_lock` is a lock which is used to prevent simultaneous access of the `jobs` dict, which could potentially leave jobs in a broken state.

Each job type has an accompanying job request type, which is the schema a client must conform to when submitting a request to register a job. A `QueriesJob` is registered when a client submits a `QueriesJobRequest`, and a `DataJob` is registered when a client submits a `DataJobRequest`.

Data generation happens in two stages. First, the client must register a `QueriesJob`. This job generates queries according to the parameters contained within the request. Once the queries are finished generating, the client can register a `DataJob`, specifying all of the desired parameters along with the UUID of the `QueriesJob`. Internally, the `QueriesJob` converts all of the generated queries into `Chat`s, which are then utilized by the `DataJob` to perform rollouts.

Below is the spec for `QueriesJobRequest` and `QueriesJob`:

```python
from __future__ import annotations
from pydantic import BaseModel
from app.common.literals import OnError, JobStatus, SaveFormat
from app.common.config import GLOBAL_SETTINGS

class QueriesJobRequest(BaseModel):
    categories: list[str]
    queries_per_category: int
    max_retries: int = 3
    on_error: OnError = "continue"
    model_id: str = GLOBAL_SETTINGS.default_model_id

    def intialize_job(self) -> QueriesJob:
        ...

class QueriesJob(QueriesJobRequest):
    status: JobStatus
    error_detail: str | None = None
    result: list[QueriesResponse] | None = None

    def save(self, uuid_str: str, format: SaveFormat) -> None: # save the job to {uuid_str}.{format}, with format being one of "json", "jsonl"
        ...

    @classmethod
    def load(cls, uuid_str: str) -> "QueriesJob": # load from {uuid_str}.json
        ...
```

And the spec for `DataJobRequest` and `DataJob`:

```python
from __future__ import annotations
from pydantic import BaseModel
from app.common.literals import OnError, JobStatus, SaveFormat
from app.common.config import GLOBAL_SETTINGS
from app.common.chats import Chat

class DataJobRequest(BaseModel):
    system_messages: list[str]
    chat_length_max: int
    chat_length_min: int
    queries_job_uuid: str
    max_retries: int = 3
    on_error: OnError = "continue"
    model_id: str = GLOBAL_SETTINGS.default_model_id

    def initialize_job(self) -> DataJob:
        ...

class DataJob(DataJobRequest):
    status: JobStatus
    error_detail: str | None = None
    chats: list[Chat] | None = None

    def save(self, uuid_str: str, format: SaveFormat) -> None: # saves the job to {uuid_str}.{format}
        ...

    @classmethod
    def load(cls, uuid_str: str) -> "DataJob": # loads a job from {uuid_str}.json
        ...
```

### Configuration

In order for the server to operate, it must be configured before starting it. The server has guards which will prevent it from starting if the minimum configuration is not met, or if one or more values are illegally configured. All environment variables must be contained within the file `.env` at the root of the project folder.

Below is a minimum viable configuration:

```
API_HOST = "http://localhost:8080"
DEFAULT_MODEL_ID = "gemma-4-26B-A4B-it-Q8_0.gguf"
```

And fully configured example:

```
# provider
API_HOST = "http://localhost:8080"
API_KEY = "super-secret-key"
DEFAULT_MODEL_ID = "gemma-4-26B-A4B-it-Q8_0.gguf"

# sampling
TEMPERATURE = 1.0
TOP_K = 64
TOP_P = 1.0
MIN_P = 0.0
```

### Interacting with the Server

The server itself is based on FastAPI and Uvicorn. FastAPI makes building APIs simple, and abstracts away the process of opening and holding sockets, HTTP networking, etc. It provides wonderfully succinct decorators for defining endpoints, simplifies exception handling, and miniaturizes startup and shutdown processes. Uvicorn allows the FastAPI project to be run as a persistent server process from the command line or in a script.

Once the server has been configured, and you have checked that the llama.cpp server described earlier is running, you can run the server with one command:

```bash
uv run uvicorn main:app --reload --host 0.0.0.0 --port 8080
```

When the server starts, it will validate the configuration and create the directory for saving jobs (defined using the SAVE_DIR environment variable in the .env file). The worker process for running jobs will also start automatically.

#### Pinging

After starting the server, and ensuring that no errors appeared in the terminal log, you can check that the server is alive and healthy:

```
GET http://localhost:8080/ping
```

If the server is reachable, you will receive a `200 OK` status response with the JSON body:

```json
{
    "message": "pong!"
}
```

#### Registering a Job

> For use cases which include programmatically interacting with or polling the server, it is the responsibility of the client to maintain the UUID returned in the Neo server's response. The UUID is required to get the results, save the job, or load the job.

After confirming that the server is healthy, the client can register a job. Registering jobs requires adhering to either the `QueriesJobRequest` or `DataJobRequest` schema specified in the Jobs section of this article. To register a job, hit the endpoint:

```
POST http://localhost:8080/register
```

To register a `QueriesJob`, include the JSON body:

```json
{
  "categories": "array[string]",
  "queries_per_category": "integer",
  "max_retries": "integer",
  "on_error": "stop" | "continue",
  "model_id": "integer"
}
```

Alternatively, if you with to register a `DataJob`, the same endpoint is used, but your request must conform to:

```json
{
  "system_messages": "array[string]",
  "chat_length_max": "integer",
  "chat_length_min": "integer",
  "queries_job_uuid": "string",
  "max_retries": "integer",
  "on_error": "stop" | "continue",
  "model_id": "string"
}
```

After the request is sent, the server will return a JSON body confirming that the job was registered:

```json
{
  "uuid_str": "b5e2053a-3b1c-436c-af0c-812279814ffb",
  "message": "Job has been successfully scheduled!"
}
```

#### Listing Jobs and Their Statuses

All of the jobs which are currently in memory can be listed by hitting the `/jobs` endpoint. Note that this only lists jobs which are in memory, and does not include jobs which are saved to the configured `SAVE_DIR` but not loaded.

```
GET http://localhost:8080/jobs
```

The server will respond with a JSON body which looks like:

```json
{
  "jobs": {
    "f7a3cd71-b8f3-4645-90cd-244d74851fb4": "error_stopped",
    "b5e2053a-3b1c-436c-af0c-812279814ffb": "complete",
    "91148120-0f2a-4db4-98d7-4a93922f58e3": "running"
  }
}
```

#### Getting Jobs

Regardless of status, a job can be retrieved from the server. No processing is applied to the job before it is returned; the server will send the entire job object as it currently exists in memory. If the job is not in memory, it must be loaded before it can be retrieved.

This endpoint serves a few common use-cases:

- Retrieving a job for further processing in your own projects
- Spot-checking data to ensure that it meets your standards of quality before proceeding
- Checking the `error_detail` field to inspect what went wrong during a run

To retrieve a job from the server, send `GET` a request to this endpoint:

```
GET http://localhost:8080/job/{job_uuid}
```

If the job exists, the server will return the job object dumped as JSON:

```json
{
  "categories": [
    "american history"
  ],
  "queries_per_category": 5,
  "max_retries": 3,
  "on_error": "continue",
  "model_id": "gemma-4-26B-A4B-it.gguf",
  "status": "complete",
  "error_detail": null,
  "result": [
    {
      "queries": [
        {
          "number": 0,
          "query": "Analyze the socio-economic impact of the Reconstruction Era on the Southern United States, specifically focusing on the tension between Freedmen's Bureau initiatives and the rise of Black Codes."
        },
        {
          "number": 1,
          "query": "Compare and contrast the political philosophies of Alexander Hamilton and Thomas Jefferson regarding the scope of federal authority and the establishment of a national bank."
        },
        {
          "number": 2,
          "query": "Examine the primary causes of the Great Depression in the 1930s, evaluating the roles of the stock market crash, bank failures, and the impact of the Dust Bowl on rural populations."
        },
        {
          "number": 3,
          "query": "Trace the evolution of the Women's Suffrage movement in the United States from the Seneca Falls Convention of 1848 to the ratification of the 19th Amendment in 1920."
        },
        {
          "number": 4,
          "query": "Discuss the strategic significance of the Battle of Midway during World War II and how it shifted the momentum of the Pacific Theater from Japan to the United States."
        }
      ]
    }
  ]
}
```

#### Saving Jobs

> If you choose to save is already present within the `SAVE_DIR`, this action will overwrite it.

After a run, you may opt to save a job to disk. This endpoint saves the job to the server machine's file system in the directory set with the environment variable `SAVE_DIR`. There are two formats available for saving: `json` and `jsonl` (`parquet` is on the roadmap). The file is saved with the filename `{job_uuid}.{format}`.

Saving a job as `json` is lossless, and contains the entire job dumped as JSON. `json` should be used if you intend to load a job back into memory at a later time. `jsonl` is lossy, and does not contain all of the information from the job, such as the `error_detail` or `max_retries`. `jsonl` should be used for big data operations, such as training your own model or working with a data loader.

You can save a job by hitting the endpoint:

```
GET http://localhost:8080/job/{job_uuid}/save/{format}
```

If the request is successful, the server will return a `200 OK` with the JSON body:

```json
{
  "uuid_str": "ae3d11b6-ce41-44a3-82ea-f591dd27ebae",
  "message": "Job successfully saved!"
}
```

#### Loading Jobs

> If the job that the client requests to load is already in memory, it will be overwritten.

> If a job is loaded and its status is not either `complete`, `error_stopped`, or `error_continued`, the job will be automatically added to the queue and scheduled for processing.

In a scenario where the server is restarted, the contents of the memory will be wiped. If you saved your jobs before restarting the server, they can be loaded using this endpoint. Note that only jobs saved as `json` can be loaded, since `jsonl` is lossy and does not contain all of the information necessary to reconstruct a job.

Loading a job can be achieved by hitting this endpoint:

```
GET http://localhost:8080/job/{job_uuid}/load
```

The server will automatically determine the type of the job and parse it. If the file can be found and is not corrupted, the server will return a `200 OK` with the following JSON body:

```json
{
  "uuid_str": "f086700c-5a11-4225-8907-45756e0cf480",
  "message": "Successfully loaded job!"
}
```

That covers all of the endpoints currently supported by the Neo server.

## Limitations and Update Roadmap

While the Neo server is functional and ready for generating data, there are more features that I would like to add in coming days and weeks. I will update this post as bugs are patched and additional features are added.

- [ ] Create a new endpoint for deleting jobs
- [ ] Create a new endpoint for bulk saving and loading jobs
- [ ] Add support for automatic saving with a shutdown task
- [ ] Add support for automatic loading on startup
- [ ] Capture reasoning traces with responses
- [ ] Include tool calls and structured output in the generation pipeline
- [ ] Add support for `parquet` for export for tighter integration with data loaders
- [ ] Package into a Python library which can be installed with `uv`

## Conclusion

Thank you for taking the time to read this post! As previously mentioned, Neo is a work in progress, and later updates will expand the feature set. Maybe one day, you will find it in a state where you choose to use it for your own data generation needs. Happy coding!