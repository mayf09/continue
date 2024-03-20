---
title: 上下文提供者
description: 输入 '@' 来选择内容给 LLM 作为上下文
keywords: [context, "@", provider, LLM]
---

# 上下文提供者

上下文提供者允许你输入 '@' 看到一个下拉框内容，可以输入给 LLM 作为上下文。每个上下文提供者是一个插件，也就是说如果你想要引用一些不在这里的信息来源，你可以请求（或者构建！）一个新的上下文提供者。

举个例子，比如你正在解决一个 Github Issue 。你输入 '@issue' 并选择一个你工作的。 Continue 可以看到 issue 的标题和内容。你也知道那个 issue 与文件 'readme.md' 和 'helloNested.py' 相关，所以你输入 '@readme' 和 '@hello' 来寻找和选择它们。现在这 3 个 "上下文条目" 会显示在你其余输入的行内。

![上下文条目](/img/context-provider-example.png)

## 内置上下文提供者

为了使用任何内置的上下文提供者，打开 `~/.continue/config.json` 并将它添加到 `contextProviders` 列表中。

### Code

输入 '@code' 来引用项目中指定的函数或类。

```json
{ "name": "code" }
```

### Git Diff

输入 '@diff' 来引用你对当前分支所做的所有修改。这是有用的，如果你想总结你所做的，或者在提交之前询问一个你的工作的通用的复查。

```json
{ "name": "diff" }
```

### 终端

输入 '@terminal' 来引用你的 IDE 终端的内容。

```json
{ "name": "terminal" }
```

### 文档

输入 `@docs` 从任何文档站点索引或获取片段。你可以通过选择 "Add Docs" 下拉框添加任何站点，然后输入文档的根 URL 和标题来记住它。在站点索引之后，你可以输入 `@docs` ，从下拉框选择你的文档， Continue 将在回答你的问题时，使用相似搜索来自动查找重要的章节。

```json
{ "name": "docs" }
```

### 打开文件

输入 '@open' 来引用所有你打开的文件的内容。设置 `onlyPinned` 为 `true` 只引用固定的文件。

```json
{ "name": "open", "params": { "onlyPinned": true } }
```

### 代码库检索

输入 '@codebase' 来从你的代码库自动检索最相关的片段。查看更多关于索引和检索 [这里](../walkthroughs/codebase-embeddings.md) 。

```json
{ "name": "codebase" }
```

### 文件夹

输入 '@folder' 来使用与 '@codebase' 相同的检索机制, 但是只有一个文件夹。

```json
{ "name": "folder" }
```

### 精准搜索

输入 '@search' 来引用代码库搜索的结果，就像你从 VS Code 搜索得到的结果。这个上下文提供者由 [ripgrep](https://github.com/BurntSushi/ripgrep) 驱动。

```json
{ "name": "search" }
```

### 文件树

输入 '@tree' 来引用当前工作区的结构。 LLM 将会看到你的项目的嵌套目录结构。

```json
{ "name": "tree" }
```

### Google

输入 '@google' 来引用 Google 搜索的结果。例如，输入 "@google python tutorial" ，如果你想要搜索讨论学习 Python 的方法。

```json
{
  "name": "google",
  "params": { "serperApiKey": "<your serper.dev api key>" }
}
```

注意：你可以从 [serper.dev](https://serper.dev) 免费获得 API key 。

### GitHub Issues

输入 '@issue' 来引用 Github issue 的讨论。确保包含你自己的 [GitHub personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token) 来避免流量限制：

```json
{
  "name": "issue",
  "params": {
    "repos": [
      {
        "owner": "continuedev",
        "repo": "continue"
      }
    ],
    "githubToken": "ghp_xxx"
  }
}
```

### Jira Issues

输入 '@jira' 来引用 Jira issue 的讨论。确保使用你自己的 [Atlassian API Token](https://id.atlassian.com/manage-profile/security/api-tokens) 。

```json
{
  "name": "jira",
  "params": {
    "domain": "company.atlassian.net",
    "email": "someone@somewhere.com",
    "token ": "ATATT..."
  }
}
```

#### Jira Datacenter 支持

这个上下文提供者支持 Jira API 版本 2 和 3 。它默认会使用版本 3 ，因为这是云上版本使用的，但是如果你有 Jira 的 datacenter 版本，你需要设置 API 版本为 2 ，使用  `apiVersion` 属性。

```json
  "params": {
    "apiVersion": "2",
    ...
  }
```

#### Issue 查询

默认情况下，使用下面的查询来查找 issue ：

```jql
assignee = currentUser() AND resolution = Unresolved order by updated DESC
```

你可以通过设置 `issueQuery` 参数来覆盖这个查询。

### 代码大纲

输入 '@outline' 来引用所有当前打开文件的大纲。文件大纲只包含文件的函数和类的定义。支持的文件扩展名是 '.js', '.mjs', '.go', '.c', '.cc', '.cs', '.cpp', '.el', '.ex', '.elm', '.java', '.ml', '.php', '.ql', '.rb', '.rs', '.ts'

```json
{ "name": "outline" }
```

### 代码高亮

输入 '@highlights' 来引用所有打开文件中的高亮部分。高亮是由在 [Aider Chat](https://github.com/paul-gauthier/aider) 的 Paul Gauthier's 的 ['repomap'](https://aider.chat/docs/repomap.html) 技术来计算的。支持的文件扩展名与 '@outline' 相同 (在幕后，我们使用相关的 tree-sitter 语法器来解析语言) 。

```json
{ "name": "highlights" }
```

### PostgreSQL

输入 `@postgres` 来引用表的 schema ，和一些示例行。一个下拉框会出现，允许你选择指定的表或所有表。

只需要创建数据库连接的配置： `host`, `port`, `user`, `password` 和 `database` 。

默认情况下， `schema` 过滤器设置为 `public` ， `sampleRows` 设置为 3 。你可以取消设置 schema ，如果你希望包含所有 schema 的表。

[这是一个简单的 demo](https://github.com/continuedev/continue/pull/859) 。

```json
{
  "name": "postgres",
  "params": {
    "host": "localhost",
    "port": 5436,
    "user": "myuser",
    "password": "catsarecool",
    "database": "animals",
    "schema": "public",
    "sampleRows": 3
  }
}
```

### 数据库表

输入 `@database` 来引用表 schema ，你可以基于你的配置使用下拉或者输入表名。配置支持多个数据库，允许你为 PostgresSQL MySQL SQLite 指定不同的连接详情。每个连接都应该有唯一的名字，连接类型（比如， postgres, sqlite ），以及每个数据库类型必须的连接参数。

```json
{
  "name": "database",
  "params": {
    "connections": [
      {
        "name": "examplePostgres",
        "connection_type": "postgres",
        "connection": {
          "user": "username",
          "host": "localhost",
          "database": "exampleDB",
          "password": "yourPassword",
          "port": 5432
        }
      },
      {
        "name": "exampleSqlite",
        "connection_type": "sqlite",
        "connection": {
          "filename": "/path/to/your/sqlite/database.db"
        }
      }
    ]
  }
}
```

### 请求上下文提供者

没有看到你想要的？创建一个 issue [这里](https://github.com/continuedev/continue/issues/new?assignees=TyDunn&labels=enhancement&projects=&template=feature-request-%F0%9F%92%AA.md&title=) 来请求一个新的 ContextProvider 。

## Building Your Own Context Provider

> Currently custom context providers are only supported in VS Code, but are coming soon to JetBrains IDEs.

### Introductory Example

To write your own context provider, you just have to implement the `CustomContextProvider`
interface:

```typescript
interface CustomContextProvider {
  title: string;
  displayTitle?: string;
  description?: string;
  getContextItems(
    query: string,
    extras: ContextProviderExtras,
  ): Promise<ContextItem[]>;
}
```

As an example, let's say you have a set of internal documents that have been indexed in a vector database. You've set up a simple REST API that allows internal users to query and get back relevant snippets. This context provider will send the query to this server and return the results from the vector database. The return type of `getContextItems` _must_ be an array of objects that have all of the following properties:

- `name`: The name of the context item, which will be displayed as a title
- `description`: A longer description of the context item
- `content`: The actual content of the context item, which will be fed to the LLM as context

```typescript title="~/.continue/config.ts"
const RagContextProvider: CustomContextProvider = {
  title: "rag",
  displayTitle: "RAG",
  description:
    "Retrieve snippets from our vector database of internal documents",

  getContextItems: async (
    query: string,
    extras: ContextProviderExtras,
  ): Promise<ContextItem[]> => {
    const response = await fetch("https://internal_rag_server.com/retrieve", {
      method: "POST",
      body: JSON.stringify({ query }),
    });

    const results = await response.json();

    return results.map((result) => ({
      name: result.title,
      description: result.title,
      content: result.contents,
    }));
  },
};
```

It can then be added in `config.ts` like so:

```typescript title="~/.continue/config.ts"
export function modifyConfig(config: Config): Config {
  if (!config.contextProviders) {
    config.contextProviders = [];
  }
  config.contextProviders.push(RagContextProvider);
  return config;
}
```

No modification in `config.json` is necessary.

### Custom Context Providers with Submenu or Query

There are 3 types of context providers: "normal", "query", and "submenu". The "normal" type is the default, and is what we've seen so far.

The **"query"** type is used when you want to display a text box to the user, and then use the contents of that text box to generate the context items. Built-in examples include ["search"](#exact-search) and ["google"](#google). This text is what gets passed to the "query" argument in `getContextItems`. To implement a "query" context provider, simply set `"type": "query"` in your custom context provider object.

The **"submenu"** type is used when you want to display a list of searchable items in the dropdown. Built-in examples include ["issue"](#github-issues) and ["folder"](#folders). To implement a "submenu" context provider, set `"type": "submenu"` and implement the `loadSubmenuItems` and `getContextItems` functions. Here is an example that shows a list of all README files in the current workspace:

```typescript title="~/.continue/config.ts"
const ReadMeContextProvider: CustomContextProvider = {
  title: "readme",
  displayTitle: "README",
  description: "Reference README.md files in your workspace",
  type: "submenu",

  getContextItems: async (
    query: string,
    extras: ContextProviderExtras,
  ): Promise<ContextItem[]> => {
    // 'query' is the filepath of the README selected from the dropdown
    const content = await extras.ide.readFile(query);
    return [
      {
        name: getFolder(query),
        description: getFolderAndBasename(query),
        content,
      },
    ];
  },

  loadSubmenuItems: async (
    args: LoadSubmenuItemsArgs,
  ): Promise<ContextSubmenuItem[]> => {
    // Filter all workspace files for READMEs
    const allFiles = await args.ide.listWorkspaceContents();
    const readmes = allFiles.filter((filepath) =>
      filepath.endsWith("README.md"),
    );

    // Return the items that will be shown in the dropdown
    return readmes.map((filepath) => {
      return {
        id: filepath,
        title: getFolder(filepath),
        description: getFolderAndBasename(filepath),
      };
    });
  },
};

export function modifyConfig(config: Config): Config {
  if (!config.contextProviders) {
    config.contextProviders = [];
  }
  config.contextProviders.push(ReadMeContextProvider);
  return config;
}

function getFolder(path: string): string {
  return path.split(/[\/\\]/g).slice(-2)[0];
}

function getFolderAndBasename(path: string): string {
  return path
    .split(/[\/\\]/g)
    .slice(-2)
    .join("/");
}
```

The flow of information in the above example is as follows:

1. The user types `@readme` and selects it from the dropdown, now displaying the submenu where they can search for any item returned by `loadSubmenuItems`.
2. The user selects one of the READMEs in the submenu, enters the rest of their input, and presses enter.
3. The `id` of the chosen `ContextSubmenuItem` is passed to `getContextItems` as the `query` argument. In this case it is the filepath of the README.
4. The `getContextItems` function can then use the `query` to retrieve the full contents of the README and format the content before returning the context item which will be included in the prompt.

### Importing outside modules

To include outside Node modules in your config.ts, run `npm install <module_name>` from the `~/.continue` directory, and then import them in config.ts.

Continue will use [esbuild](https://esbuild.github.io/) to bundle your `config.ts` and any dependencies into a single Javascript file. The exact configuration used can be found [here](https://github.com/continuedev/continue/blob/5c9874400e223bbc9786a8823614a2e501fbdaf7/extensions/vscode/src/ideProtocol.ts#L45-L52).

### `CustomContextProvider` Reference

- `title`: An identifier for the context provider
- `displayTitle` (optional): The title displayed in the dropdown
- `description` (optional): The longer description displayed in the dropdown when hovered
- `type` (optional): The type of context provider. Options are "normal", "query", and "submenu". Defaults to "normal".
- `getContextItems`: A function that returns the documents to include in the prompt. It should return a list of `ContextItem`s, and is given access to the following arguments:
  - `extras.fullInput`: A string representing the user's full input to the text box. This can be used for example to generate an embedding to compare against a set of other embedded documents
  - `extras.embeddingsProvider`: The embeddings provider has an `embed` function that will convert text (such as `fullInput`) to an embedding
  - `extras.llm`: The current default LLM, which you can use to make completion requests
  - `extras.ide`: An instance of the `IDE` class, which lets you gather various sources of information from the IDE, including the contents of the terminal, the list of open files, or any warnings in the currently open file.
  - `query`: (not currently used) A string representing the query
- `loadSubmenuItems` (optional): A function that returns a list of `ContextSubmenuItem`s to display in a submenu. It is given access to an `IDE`, the same that is passed to `getContextItems`.
