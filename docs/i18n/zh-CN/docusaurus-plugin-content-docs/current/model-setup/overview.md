# 概述

Continue 是使用任何 [Large Language Model (LLM)](https://www.youtube.com/watch?v=zjkBMFhNj_g) 编码的最简单的方式。你可以使用它，通过商业模型，比如 GPT-4 ，通过 OpenAI API ，开源模型，比如 CodeLlama ，使用 Ollama 运行在你的笔记本上，以及所有中间的东西。

When you first install Continue, you can try it out for free using a proxy server that securely makes calls with our API keys to models like GPT-4, Gemini Pro, and Phind CodeLlama via OpenAI, Google, and Together respectively.

Once you're ready to use your own API key or a different model / provider, press the `+` button in the bottom left to add a new model to your `config.json`.

If you are unsure what model or provider to use, here is our current rule of thumb:

- Use GPT-4 via OpenAI if you want the best possible model overall
- Use DeepSeek Coder 33B via the Together API if you want the best open-source model
- Use DeepSeek Coder 6.7B with Ollama if you want to run a model locally

Learn more:

- [Select a provider](select-provider.md)
- [Select a model](select-model.md)
- [Configuration](configuration.md)
