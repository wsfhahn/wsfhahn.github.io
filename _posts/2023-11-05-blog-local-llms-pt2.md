---
layout: post
---

# Local LLMs: CodeLlama's Productive Potential

In [my last post](https://wsfhahn.github.io/2023/11/04/blog-local-llms-pt1.html), I successfully chatted with a 13 billion parameter large language model locally on my computer using [llama.cpp](https://github.com/ggerganov/llama.cpp). Although it was a fun experiment, I decided to push it even further by running a significantly larger, 34 billion parameter model called [CodeLlama 34B Instruct](https://huggingface.co/TheBloke/CodeLlama-34B-Instruct-GGUF).

My goal in running a 34B model was to generate functional sample code which I could use to learn more programming languages, like JavaScript and Rust. Having a local bot to come up with ideas and write and check code is convenient, even though it lacks the ability to complete entire projects on its own.

---

### Key Challenges

Loading a 34B model presented some key challenges not encountered when running a 13B model. My computer has 32GB of memory, and so to run CodeLlama 34B, I had to choose a quantized model that can run without filling my RAM.

Another challenge I encountered were limitations with Metal and memory usage. Running the model through Apple's Metal framework is significantly fast than running it on CPU, but Metal sets its maximum working memory size to just 21GB on my machine. I have yet to find a workaround for this, so I can only run 34B models on CPU. 

### Setting up

So that this post could be followed as a guide, I decided to start from the very beginning, so following all of the commands in this post will get llama.cpp up and running on a fresh install of macOS 14.0.

The first step is to install Xcode Command Line Tools. This can be done with the following command:

```plaintext
xcode-select --install
```

After Xcode Command Line Tools are installed, we're ready to download llama.cpp. Clone it using `git`:

```plaintext
cd ~/
git clone https://github.com/ggerganov/llama.cpp.git
```

Next, `cd` into the directory:

```plaintext
cd llama.cpp
```

Due to the limitations with the Metal framework and its memory handling, Metal will not be able to run a 34B model in 32GB of memory. Build without Metal on all CPU cores using the following command, substituting `N` for the number of cores you have available:

```plaintext
make -jN LLAMA_NO_METAL=1
```

Once the build is finished, you need to download a quantized CodeLlama 34B Instruct model in GGUF format from [HuggingFace](https://huggingface.co/TheBloke/CodeLlama-34B-Instruct-GGUF). For a MacBook Pro with M1 Max and 32GB RAM, the best model that fits in memory is [codellama-34b-instruct.Q5_K_S.gguf](https://huggingface.co/TheBloke/CodeLlama-34B-Instruct-GGUF/resolve/main/codellama-34b-instruct.Q5_K_S.gguf).

Once the download is finished, move the model from `~/Downloads/` to `~/llama.cpp/models/`:

```plaintext
mv ~/Downloads/codellama-34b-instruct.Q5_K_S.gguf ~/llama.cpp/models/
```

### Starting the Server

Starting the server is simple. To prevent the model from inferencing at a snail's pace, it is important to use the `--mlock` flag. Without this, part of the model usually ends up in swap space, which is significantly slower than RAM.

34B models have a much greater maximum context window, meaning they can generate substantially more content before becoming nonsensical. When starting the server, the context size can be set to 8192 tokens or 16,384 tokens.

Start the server with the following command:

```plaintext
cd ~/llama.cpp
./server -m models/codellama-34b-instruct.Q5_K_S.gguf -c 8192 --mlock
```

Due to the size of the model, it will take some time to load. It is worthwhile closing unused applications to free up memory for the model.

### CodeLlama Q&A

CodeLlama's system prompt is set to the following:

> This is a conversation between User and CodeLlama, a trained and knowledgeable computer programmer. CodeLlama writes perfect, efficient code that works every time and is helpful and kind in fulfilling all requests.

The CodeLlama Q&A will be formatted like so:

> My question

```plaintext
CodeLlama's Code
```

---

> Write a Python function that takes a diameter as an input and computes all other properties of the circle.

```plaintext
def get_circle_properties(diameter):
    radius = diameter / 2
    area = math.pi * (radius ** 2)
    circumference = 2 * math.pi * radius
    return {"area": area, "circumference": circumference}
```

---

> Make a dataset of movie reviews and their sentiments, positive or negative, in JSON format.

```plaintext
{
  "reviews": [
    {
      "text": "This is a great movie!",
      "sentiment": "positive"
    },
    {
      "text": "I hated this movie.",
      "sentiment": "negative"
    }
  ]
}
```

---

> Write a Python script to invert a binary tree.

```plaintext
class Node(object):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
    
def invert_tree(node):
    if node is None:
        return None
    
    # Recursively invert the tree's children
    node.left, node.right = invert_tree(node.right), invert_tree(node.left)
    
    return node
```