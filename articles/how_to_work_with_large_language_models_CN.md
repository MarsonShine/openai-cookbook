# 大语言模型如何工作的

## 大语言模型如何工作

[大语言模型][Large language models Blog Post]是将文本映射到文本的函数。给定一串输入文本，大型语言模型就会预测接下来的文本。

大型语言模型的神奇之处在于，通过对大量文本进行最小化预测误差的训练，模型最终会学习到对这些预测有用的概念。例如，它们可以学习

- 如何拼写
- 语法如何工作
- 如何转述
- 如何回答问题
- 如何进行对话
- 如何用多种语言写作
- 如何编码
- 等等。

它们通过"阅读"大量现有文本，学习单词在上下文中如何与其他单词搭配出现，并利用所学知识预测下一个最有可能出现的单词，以响应用户的请求，以及之后出现的每个单词。

GPT-3 和 GPT-4 为[许多软件产品](https://openai.com/customer-stories)提供支持，包括生产力应用程序、教育应用程序、游戏等。

## 如何控制大型语言模型

在大型语言模型的所有输入中，影响最大的是文本提示。

大型语言模型可以通过以下几种方式提示输出：

- **指令(Instruction)**：告诉模型您想要什么。
- **完成(Completion)**：引导模型完成您想要的开头部分。
- **场景(Scenario)**：给模型一个情境让其角色扮演。
- **例子(Demonstration)**：向模型展示您想要的东西：
  - 提示中的写几个例子。
  - 微调训练数据集中的成百上千个示例。

下面分别举例说明。

### 指令提示示例

将您的指令写在提示语的顶部（或底部，或两者兼而有之），然后模型会尽力按照指令进行处理，然后停止。指令可以很详细，因此不要害怕写一段话来详细说明您想要的输出，只要注意模型可以处理多少[token](https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them)即可。

下面指令提示的例子:

```text
Extract the name of the author from the quotation below.

“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation
```

输出:

```text
Ted Chiang
```

### 完成提示示例

完成式提示利用了大型语言模型尝试编写它们认为最有可能出现的文本的方式。要引导模型，可以尝试开始一个模式或句子，并由您希望看到的输出来完成。与直接指令相比，这种引导大型语言模型的模式需要更多的谨慎和尝试。此外，模型并不一定知道在哪里停止，因此您通常需要停止序列或后处理来切断超出所需输出的文本。

例子:

```text
“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation

The author of this quote is
```

输出:

```text
 Ted Chiang
```

### 情景提示示例

给模型提供一个需要遵循的情景或需要扮演的角色，对于复杂的询问或寻求富有想象力的回答很有帮助。使用假设性提示时，您可以设置一个情境、问题或故事，然后要求模型以该情境中的角色或该主题的专家的身份进行回答。

例子：

```text
Your role is to extract the name of the author from any given text

“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation
```

输出:

```text
 Ted Chiang
```

### 演示提示示例（少样本学习）

与完成式提示类似，演示可以向模型展示您希望它做什么。这种方法有时也称为少样本学习，因为模型会从提示中提供的少量示例中学习。

例子：

```text
Quote:
“When the reasoning mind is forced to confront the impossible again and again, it has no choice but to adapt.”
― N.K. Jemisin, The Fifth Season
Author: N.K. Jemisin

Quote:
“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation
Author:
```

Output:

```text
 Ted Chiang
```

### 微调提示示例

有了足够多的训练示例，您就可以对自定义模型进行[微调][Fine Tuning Docs]。在这种情况下，说明就没有必要了，因为模型可以从提供的训练数据中学习任务。不过，包含分隔符序列（例如，`->` 或 `###` 或任何不常出现在输入中的字符串）会很有帮助，**可以告诉模型提示何时结束，输出何时开始**。如果没有分隔符序列，模型就有可能继续阐述输入文本，而不是开始您希望看到的答案。

微调提示示例（针对在类似提示-完成对上进行过定制训练的模型）：

```text
“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation

###


```

输出:

```text
 Ted Chiang
```

## 代码能力

大语言模型不仅在文本方面表现出色，在代码方面也是如此。OpenAI 的 [GPT-4][GPT-4 and GPT-4 Turbo] 模型就是最好的例子。

GPT-4为[众多创新产品][OpenAI Customer Stories]提供动力，包括

- [GitHub Copilot][GitHub Copilot]（在 Visual Studio 和其他集成开发环境中自动完成代码）
- [Replit](https://replit.com/)（可以完成、解释、编辑和生成代码）
- [cursor](https://cursor.sh/) （在专为与人工智能结对编程而设计的编辑器中更快地构建软件。

GPT-4 比以前的模型（如 `gpt-3.5-turbo-instruct`）更先进。但是，要让 GPT-4 在编码任务中发挥最大作用，给出清晰而具体的指令仍然很重要。因此，设计良好的提示需要更加谨慎。

### 更多提示建议

有关更多提示示例，请访问 [OpenAI 示例][OpenAI Examples]。

一般来说，输入提示是改进模型输出的最佳工具。您可以尝试以下技巧

- **更加具体**：例如，如果你希望输出是一个逗号分隔的列表，那么就要求它返回一个逗号分隔的列表。如果你想让它在不知道答案时说"我不知道"，那就告诉它"如果你不知道答案，就说'我不知道'"。你的指令越具体，模型的反应就越好。
- **提供语境**：帮助模型了解您的请求的大背景。这可以是背景信息、您想要的例子/演示或解释您任务的目的。
- **要求模型像专家一样回答问题**。明确要求模型生成高质量的输出或输出结果，就像专家所写的那样，这可以促使模型给出它认为专家会写的更高质量的答案。像"详细解释"或"逐步描述"这样的短语就很有效。
- **提示模型写下解释其推理的一系列步骤**。如果理解答案背后的"为什么"很重要，则提示模型写出其推理。只需在每个答案前加上一行类似"[让我们一步步思考](https://arxiv.org/abs/2205.11916)"的文字即可。

[Fine Tuning Docs]: https://platform.openai.com/docs/guides/fine-tuning
[OpenAI Customer Stories]: https://openai.com/customer-stories
[Large language models Blog Post]: https://openai.com/research/better-language-models
[GitHub Copilot]: https://github.com/features/copilot/
[GPT-4 and GPT-4 Turbo]: https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo
[GPT3 Apps Blog Post]: https://openai.com/blog/gpt-3-apps/
[OpenAI Examples]: https://platform.openai.com/examples
