---
title: 配置
description: Configure your LLM and model provider
keywords: [configure, llm, provider]
---

# 配置

## 修改默认的 LLM

在 `config.json` 中，你可以找到 `models` 属性，一个你保存的 Continue 可以使用的模型列表：

```json
"models": [
    {
        "title": "Smart Model",
        "provider": "free-trial",
        "model": "gpt-4"
    },
    {
        "title": "Fast Model",
        "provider": "free-trial",
        "model": "gpt-3.5-turbo"
    }
]
```

只需要指定 `model` 和 `provider` 属性，我们会自动探测提示模板和其他重要信息，但是如果你查找一些基础设置之外的东西，我们会在下面解释一些其他的选项。

## Azure OpenAI 服务

If you'd like to use OpenAI models but are concerned about privacy, you can use the Azure OpenAI service, which is GDPR and HIPAA compliant. After applying for access [here](https://azure.microsoft.com/en-us/products/ai-services/openai-service), you will typically hear back within only a few days. Once you have access, set up a model in `config.json` like so:

```json
"models": [{
    "title": "Azure OpenAI",
    "provider": "openai",
    "model": "gpt-4",
    "apiBase": "https://my-azure-openai-instance.openai.azure.com/",
    "engine": "my-azure-openai-deployment",
    "apiVersion": "2023-07-01-preview",
    "apiType": "azure",
    "apiKey": "<MY_API_KEY>"
}]
```

The easiest way to find this information is from the chat playground in the Azure OpenAI portal. Under the "Chat Session" section, click "View Code" to see each of these parameters.

## Self-hosting an open-source model

For many cases, either Continue will have a built-in provider or the API you use will be OpenAI-compatible, in which case you can use the "openai" provider and change the "baseUrl" to point to the server.

However, if neither of these are the case, you will need to wire up a new LLM object. Learn how to do this [here](#defining-a-custom-llm-provider).

## 认证

Basic 认证可以对任何提供者，使用 `apiKey` 字段：

```json title="~/.continue/config.json"
{
  "models": [
    {
      "title": "Ollama",
      "provider": "ollama",
      "model": "llama2-7b",
      "apiKey": "xxx"
    }
  ]
}
```

如果你需要发送定制 header 来认证，你可以使用 `requestOptions.headers` 属性，比如在这个示例中使用 Ollama ：

```json title="~/.continue/config.json"
{
  "models": [
    {
      "title": "Ollama",
      "provider": "ollama",
      "model": "llama2-7b",
      "requestOptions": {
        "headers": {
          "X-Auth-Token": "xxx"
        }
      }
    }
  ]
}
```

## 定制聊天模板

大多数开源模型要求特定的聊天格式，比如 llama2 和 codellama 期望输入看起来像是 `"[INST] How do I write bubble sort in Rust? [/INST]"` 。 Continue 将自动尝试探测正确的提示格式，基于你提供的 `model` 值，但是如果你收到无意义的回复，你可以使用 `template` 属性来明确你希望的格式。可选项是： `["llama2", "alpaca", "zephyr", "phind", "anthropic", "chatml", "openchat", "neural-chat", "none"]` 。

如果你希望创建一个全新的聊天模板，可以在 [config.ts](../customization/code-config.md) 完成，通过定义一个函数并把它添加到你的 `LLM` 的 `templateMessages` 属性。这里有一个 `templateMessages` 的示例，对于 Alpaca/Vicuna 格式：

```typescript
function templateAlpacaMessages(msgs: ChatMessage[]): string {
  let prompt = "";

  if (msgs[0].role === "system") {
    prompt += `${msgs[0].content}\n`;
    msgs.pop(0);
  }

  prompt += "### Instruction:\n";
  for (let msg of msgs) {
    prompt += `${msg.content}\n`;
  }

  prompt += "### Response:\n";

  return prompt;
}
```

它可以像这样使用：

```typescript title="~/.continue/config.ts"
function modifyConfig(config: Config): Config {
  const model = config.models.find(
    (model) => model.title === "My Alpaca Model",
  );
  if (model) {
    model.templateMessages = templateAlpacaMessages;
  }
  return config;
}
```

这个函数和一些其他默认实现放在 [`continuedev.libs.llm.prompts.chat`](https://github.com/continuedev/continue/blob/main/core/llm/templates/chat.ts) 。

## 定制 /edit 提示

You also have access to customize the prompt used in the '/edit' slash command. We already have a well-engineered prompt for GPT-4 and sensible defaults for less powerful open-source models, but you might wish to play with the prompt and try to find a more reliable alternative if you are for example getting English as well as code in your output.

To customize the prompt, use the `promptTemplates` property of any model, which is a dictionary, and set the "edit" key to a template string with Mustache syntax. The 'filePrefix', 'fileSuffix', 'codeToEdit', 'language', 'contextItems', and 'userInput' variables are available in the template. Here is an example of how it can be set in `config.ts`:

```typescript title="~/.continue/config.ts"
const codellamaEditPrompt = `\`\`\`{{{language}}}
{{{codeToEdit}}}
\`\`\`
[INST] You are an expert programmer and personal assistant. Your task is to rewrite the above code with these instructions: "{{{userInput}}}"

Your answer should be given inside of a code block. It should use the same kind of indentation as above.
[/INST] Sure! Here's the rewritten code you requested:
\`\`\`{{{language}}}`;

function modifyConfig(config: Config): Config {
  config.models[0].promptTemplates["edit"] = codellamaEditPrompt;
  return config;
}
```

You can find all existing templates for /edit in [`core/llm/templates/edit.ts`](https://github.com/continuedev/continue/blob/main/core/llm/templates/edit.ts).

## 定义一个定制的 LLM 提供者

If you are using an LLM API that isn't already [supported by Continue](./select-provider.md), and is not an OpenAI-compatible API, you'll need to define a `CustomLLM` object in `config.ts`. This object only requires one of (or both of) a `streamComplete` or `streamChat` function. Here is an example:

```typescript title="~/.continue/config.ts"
export function modifyConfig(config: Config): Config {
  config.models.push({
    options: {
      title: "My Custom LLM",
      model: "mistral-7b",
    },
    streamComplete: async function* (prompt, options) {
      // Make the API call here

      // Then yield each part of the completion as it is streamed
      // This is a toy example that will count to 10
      for (let i = 0; i < 10; i++) {
        yield `- ${i}\n`;
        await new Promise((resolve) => setTimeout(resolve, 1000));
      }
    },
  });
}
```
