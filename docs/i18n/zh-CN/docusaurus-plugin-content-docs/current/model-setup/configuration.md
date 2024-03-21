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

如果你想要使用 OpenAI 模型，但是担心隐私问题，你可以使用 Azure OpenAI 服务，它是符合 GDPR 和 HIPAA 的。在接受访问 [这里](https://azure.microsoft.com/en-us/products/ai-services/openai-service) 之后，你通常会在几天之内收到回复。一旦你可以访问，在 `config.json` 中设置一个模型，像这样：

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

找到这个信息最简单的方式是在 Azure OpenAI portal 的聊天操场。在 "Chat Session" 部分，点击 "View Code" 来查看每个参数。

## 自托管开源模型

很多情况下， Continue 将有内建的提供者，或者你使用的 API 是 OpenAI 兼容的，这种情况下，你可以使用 "openai" 提供者并修改 "baseUrl" 指向你的服务器。

不过，如果这些情况都不是，你需要构建一个新的 LLM 对象。了解如何做这个 [这里](#defining-a-custom-llm-provider) 。

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

## 自定义 /edit 提示词

你也可以自定义 '/edit' 斜杠命令使用的提示词。我们已经有精心设计的对于 GPT-4 的提示词，以及对于能力稍弱的开源模型的实用的默认提示词，但是你可能希望把玩提示词，试着找出一个更可靠的替代，如果你比如想在输出中得到英语和代码。

To customize the prompt, use the `promptTemplates` property of any model, which is a dictionary, and set the "edit" key to a template string with Mustache syntax. The 'filePrefix', 'fileSuffix', 'codeToEdit', 'language', 'contextItems', and 'userInput' variables are available in the template. Here is an example of how it can be set in `config.ts`:
为了自定义提示词，使用任何模型的 `promptTemplates` ，它是一个字典，设置 "edit" 键为一个 Mustache 语法的模板字符串。 'filePrefix', 'fileSuffix', 'codeToEdit', 'language', 'contextItems' 和 'userInput' 变量可以在模板中使用。这是一个例子，它如何在 `config.ts` 中设置：

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

你可以在 [`core/llm/templates/edit.ts`](https://github.com/continuedev/continue/blob/main/core/llm/templates/edit.ts) 中找到所有 `/edit` 已存的模板。

## 定义一个自定义的 LLM 提供者

If you are using an LLM API that isn't already [supported by Continue](./select-provider.md), and is not an OpenAI-compatible API, you'll need to define a `CustomLLM` object in `config.ts`. This object only requires one of (or both of) a `streamComplete` or `streamChat` function. Here is an example:

如果你使用一个 LLM API ，不在 [Continue 支持](./select-provider.md) 中，并且不是 OpenAI 兼容的 API ，你需要在 `config.ts` 中定义一个 `CustomLLM` 对象。这个对象需要一个（或都需要） `streamComplete` 或 `streamChat` 函数。这是一个例子：

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
