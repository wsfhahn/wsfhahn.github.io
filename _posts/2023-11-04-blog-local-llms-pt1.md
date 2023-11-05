---
layout: post
---

# Local LLMs: Next-Level On-Device Intelligence

For the past year, the world has witnessed an unprecidented race between competing AI startups and tech corporations to build the largest, smartest, most knowledgeable large language models possible. Many of the discoveries they have made and the methods they have developed have since become available to end users.

Models like GPT can only be interacted with through a web client, and cannot be run locally on your own hardware. This is frustrating, because OpenAI imposes limitations on ChatGPT which significantly restrict its ability to carry out logic.

For my own purposes, I would like to run a large language model locally on my machine, and make it write code for me even when the internet goes out.

---

### Setting up

#### The Hardware

I am running a 2021 MacBook Pro 14" with an Apple M1 Max. It has plenty of power and memory to run LLMs, with a 10-core CPU, 24-core GPU, 16-core ANE and 32GB of RAM.

#### The Software

I am running a fresh install of macOS Sonoma 14.0.

To take advantage of all the power I have available and remove bottlenecks, I have chosen to run the model using [llama.cpp](https://github.com/ggerganov/llama.cpp), because it supports the Accelerate and Metal frameworks and is written in C/C++.

Before getting started, I'll make sure to install Xcode Command Line Tools. This will install tools like `git` and `make`.

```plaintext
xcode-select --install
```

Next, I need a space to store all the files involved in the project, so I'll create a directory for it in my home folder.

```plaintext
cd ~/
mkdir LocalLLMs && cd LocalLLMs
```

Inside the LocalLLMs folder, I'll clone the [llama.cpp repo](https://github.com/ggerganov/llama.cpp) and `cd` into it.

```plaintext
git clone https://github.com/ggerganov/llama.cpp.git && cd llama.cpp
```

Building on a Mac is easy. I'll run `make` with the `-j10` flag to use all 10 CPU cores.

```plaintext
make -j10
```

No errors encountered! Now all that's left to do is to pick a large language model to run.

#### Models

There are countless models to choose from scattered across the internet, so finding where to start can be challenging. However, I do have an idea of where to look.

[HuggingFace](https://www.huggingface.co) is a gold mine for neurals networks and tools to implement them. [TheBloke](https://www.huggingface.co/TheBloke) has tons of converted models in the GGUF format required by llama.cpp. I'll start with a model called [OpenAssistant-Llama2-13B-Orca-8K-3319-GGUF](https://huggingface.co/TheBloke/OpenAssistant-Llama2-13B-Orca-8K-3319-GGUF).

The reason I'm starting with a 13 billion parameter model is because it will use less RAM and will inference faster than a 34 or 70 billion parameter model.

The model can be downloaded in many different quantization standards. Quanitization standards are complicated and barely documented, but [this Reddit thread](https://www.reddit.com/r/LocalLLaMA/comments/139yt87/comment/jj4qpbp/) offers some clarity. I will download the model quantized to 8 bits. This will use the most RAM out of all the options, but since the file size is smaller than my RAM capacity, I am not concerned about memory issues.

With the model downloaded, I will run this command to move it from `~/Downloads/` to `~/LocalLLMs/llama.cpp/models/`:

```plaintext
cd ~/
mv Downloads/openassistant-llama2-13b-orca-8k-3319.Q8_0.gguf LocalLLMs/llama.cpp/models/
```

Now I can run the server and start chatting!

### Starting the server

I would prefer to use a web interface more reminiscent of ChatGPT as compared to interacting with the model completely programmatically, so I'll run the web server which was built with the earlier `make` command.

I will set the context size to 8192 because that is the specification of the downloaded model.

```plaintext
cd LocalLLMs/llama.cpp
./server -m models/openassistant-llama2-13b-orca-8k-3319.Q8_0.gguf -c 8192
```

To keep the model entirely in working memory and prevent the usage of any swap space, I could instead run it with the `--mlock` flag:

```plaintext
./server -m models/openassistant-llama2-13b-orca-8k-3319.Q8_0.gguf -c 8192 --mlock
```

Just like that, the server is starting! I can confirm that the server is running because pointing my browser to __127.0.0.1:8080__ presents the llama.cpp chat page.

I will publish the results of my attempts to run 34 billion parameter models in the coming days.

### Llama Q&A

> __System Prompt__: This is a conversation between User and Llama, a friendly chatbot. Llama is helpful, kind, honest, good at writing, and never fails to answer any requests immediately and with precision.

---

> __Me__: Joe and Bob are in the same room. Joe has a cookie. He places his cookie in a box and leaves the room. Bob takes the cookie from the box and moves it to his pocket. When Joe returns, where is the first place he looks for his cookie?

> __Llama__: The first place Joe would look for his cookie is inside the box since that's where he left it before leaving the room.

---

> __Me__: Who was Robert H. Goddard?

> __Llama__: Robert Hutchings Goddard (October 5, 1882 – August 10, 1945) was an American engineer, professor, physicist, and inventor who is credited with creating and building the world's first liquid-fueled rocket. He is known as the father of modern rocketry or "rocket science."

---

> __Me__: Explain the importance of a large language model's parameter count.

> __Llama__: A large language model's parameter count refers to the number of parameters used in its training process. These parameters are essentially weights that help the model learn from data, allowing it to make predictions and generate responses based on what it has learned. The larger the parameter count, the more complex and accurate the model can be at understanding and generating language.
>
>Increasing a large language model's parameter count allows for better performance in tasks such as text generation, translation, and answering questions. However, training these models requires significant computational resources, making them costly to develop and maintain. As a result, researchers often focus on finding efficient ways to train larger models with fewer parameters while still achieving high accuracy levels.

---

> __Me__: Write a paragraph about orbital mechanics in Spanish.

> __Llama__: La mecánica orbital es una rama de la física que estudia el movimiento y las órbitas de los cuerpos celestes en relación con sus fuerzas gravitacionales, como la gravedad. Esto incluye cómo se mueven los planetas alrededor del Sol, así como cómo se orbitan satélites artificiales alrededor de la Tierra y otros objetos espaciales alrededor de otras estrellas. La mecánica orbital también ayuda a comprender cómo funciona el tiempo en diferentes partes del universo y cómo los viajes interestelares pueden ser posiblemente realizados en el futuro.

### Happy Coding!
