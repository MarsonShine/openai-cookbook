# 提高可靠性的技术

当 GPT-3 在一项任务中失败时，您该怎么办？

- 寻找能得到更可靠答案的更好提示？
- 投资数以千计的示例来微调自定义模型？
- 假设模型无法完成任务，然后继续前进？

答案并不简单--这取决于具体情况。但是，如果您的任务涉及逻辑推理或复杂性，请考虑尝试本文中的技术，以创建更可靠、性能更高的提示。

## 为什么 GPT3 在复杂任务上会失败

如果让你用 13 x 17，你会立即想到答案吗？对我们大多数人来说，可能不会。然而，这并不意味着人类无法计算两位数乘法。只要几秒钟，再加上纸笔，就能算出 13 x 17 = 130 + 70 + 21 = 221。

同样，如果给 GPT-3 一个在计算下一个标记所需的时间内无法完成的复杂任务，它可能会制造一个错误的猜测。然而，与人类类似，这并不一定意味着模型无法完成该任务。在有一定时间和空间推理的情况下，模型仍然可能能够可靠地回答。

举个例子，如果你问 `gpt-3.5-turbo-instruct` 以下关于杂耍球的数学问题，它的答案是错误的：

```gpt-3.5-turbo-instruct
Q: A juggler has 16 balls. Half of the balls are golf balls and half of the golf balls are blue. How many blue golf balls are there?
A:
```

```gpt-3.5-turbo-instruct
There are 8 blue golf balls.
```

这是否意味着 GPT-3 不能做简单的数学题？事实上，只要用`让我们一步一步思考`来提示模型，模型就能可靠地解决问题：

```gpt-3.5-turbo-instruct
Q: A juggler has 16 balls. Half of the balls are golf balls and half of the golf balls are blue. How many blue golf balls are there?
A: Let's think step by step.
```

```gpt-3.5-turbo-instruct
There are 16 balls in total.
Half of the balls are golf balls.
That means that there are 8 golf balls.
Half of the golf balls are blue.
That means that there are 4 blue golf balls.
```

当然，仅从一个例子中很难判断这种`让我们一步步思考`的窍门在一般情况下是否真的有效，或者只是在这个特殊问题上运气好而已。但它确实有效。在一个单词数学问题的基准测试中，`让我们一步步思考`技巧大大提高了 GPT-3 的解题率，从毫无价值的 18% 提高到了不错的 79%！

## 模型能力依赖于上下文

在学习使用 GPT-3 时，一个常见的概念性错误是认为 GPT-3 的能力在所有情况下都是固定不变的。例如，如果 GPT-3 做错了一道简单的逻辑题，那么它就一定不能做简单的逻辑题。

但正如`让我们一步步思考`的例子所说明的，GPT-3 的明显失误有时可以通过更好的提示来弥补，从而帮助模型将自己引向正确的输出。

## 如何提高在复杂任务上的可靠性

本文其余部分将分享在复杂任务中提高大语言模型可靠性的技术。虽然有些技术是针对某些类型的问题，但其中许多技术是建立在一般原则基础上的，可广泛应用于各种任务，例如

- 给出更清晰的指令
- 将复杂的任务拆分成更简单的子任务
- 安排指令的结构，使模型继续完成任务
- 提示模型在回答前进行解释
- 询问许多可能答案的理由，然后进行综合
- 生成多种输出，然后利用模型选出最佳输出
- 微调自定义模型，以最大限度地提高性能

## 将复杂的任务拆分成更简单的子任务

给模型更多思考时间和空间的方法之一是将任务分解成更简单的部分。

举个例子，我们可以向模型提出一个关于文本的选择题--在本例中，是一个线索游戏。当直接提问时，`gpt-3.5-turbo-instruct`无法将线索 3 和 5 放在一起，因此回答错误：

```gpt-3.5-turbo-instruct
Use the following clues to answer the following multiple-choice question.

Clues:
1. Miss Scarlett was the only person in the lounge.
2. The person with the pipe was in the kitchen.
3. Colonel Mustard was the only person in the observatory.
4. Professor Plum was not in the library nor the billiard room.
5. The person with the candlestick was in the observatory.

Question: Was Colonel Mustard in the observatory with the candlestick?
(a) Yes; Colonel Mustard was in the observatory with the candlestick
(b) No; Colonel Mustard was not in the observatory with the candlestick
(c) Unknown; there is not enough information to determine whether Colonel Mustard was in the observatory with the candlestick

Solution:
```

```gpt-3.5-turbo-instruct
(c) Unknown; there is not enough information to determine whether Colonel Mustard was in the observatory with the candlestick
```

虽然线索 3 和 5 确定了 Colonel Mustard 是天文台里唯一的人，而且天文台里的人拿着烛台，但模型未能将它们组合成 (a) 是的正确答案。

不过，我们可以把任务分成三部分，而不是直接询问答案：

- 首先，逐一查看线索，考虑线索是否可能相关
- 第二，结合相关线索，推理出问题的答案
- 第三，写出最终答案：(a)、(b) 或 (c)

```gpt-3.5-turbo-instruct
Use the following clues to answer the following multiple-choice question, using the following procedure:
(1) First, go through the clues one by one and consider whether the clue is potentially relevant
(2) Second, combine the relevant clues to reason out the answer to the question
(3) Third, map the answer to one of the multiple choice answers: either (a), (b), or (c)

Clues:
1. Miss Scarlett was the only person in the lounge.
2. The person with the pipe was in the kitchen.
3. Colonel Mustard was the only person in the observatory.
4. Professor Plum was not in the library nor the billiard room.
5. The person with the candlestick was in the observatory.

Question: Was Colonel Mustard in the observatory with the candlestick?
(a) Yes; Colonel Mustard was in the observatory with the candlestick
(b) No; Colonel Mustard was not in the observatory with the candlestick
(c) Unknown; there is not enough information to determine whether Colonel Mustard was in the observatory with the candlestick

Solution:
(1) First, go through the clues one by one and consider whether the clue is potentially relevant:
```

```gpt-3.5-turbo-instruct
Clue 1: Miss Scarlett was the only person in the lounge. This clue is not relevant.
Clue 2: The person with the pipe was in the kitchen. This clue is not relevant.
Clue 3: Colonel Mustard was the only person in the observatory. This clue is relevant.
Clue 4: Professor Plum was not in the library nor the billiard room. This clue is not relevant.
Clue 5: The person with the candlestick was in the observatory. This clue is relevant.

(2) Second, combine the relevant clues to reason out the answer to the question:
Clue 3 and Clue 5 both say that the person with the candlestick was in the observatory. Colonel Mustard was the only person in the observatory, so Colonel Mustard was the person with the candlestick.

(3) Third, write the final answer:
(a) Yes; Colonel Mustard was in the observatory with the candlestick
```

通过给模型更多的思考时间和空间，并引导它按照推理计划进行推理，它就能找出正确答案 (a) 是。

将复杂指令拆分成更小的子任务的另一个好处是，可以帮助模型专注于每个子任务。

例如，如果我们要求`gpt-3.5-turbo-instruct`用原文概括一篇文章，模型可能会回到英文：

```gpt-3.5-turbo-instruct
Summarize the text using the original language of the text. The summary should be one sentence long.

Text:
"""
La estadística (la forma femenina del término alemán Statistik, derivado a su vez del italiano statista, "hombre de Estado")​ es una ciencia que estudia la variabilidad, colección, organización, análisis, interpretación, y presentación de los datos, así como el proceso aleatorio que los genera siguiendo las leyes de la probabilidad.​ La estadística es una ciencia formal deductiva, con un conocimiento propio, dinámico y en continuo desarrollo obtenido a través del método científico formal. En ocasiones, las ciencias fácticas necesitan utilizar técnicas estadísticas durante su proceso de investigación factual, con el fin de obtener nuevos conocimientos basados en la experimentación y en la observación. En estos casos, la aplicación de la estadística permite el análisis de datos provenientes de una muestra representativa, que busca explicar las correlaciones y dependencias de un fenómeno físico o natural, de ocurrencia en forma aleatoria o condicional.
"""

Summary:
```

```gpt-3.5-turbo-instruct
The text explains that statistics is a science that studies the variability, collection, organization, analysis, interpretation, and presentation of data, as well as the random process that generates them following the laws of probability.
```

但是，如果我们先让模型识别文本的语言，然后再对文本进行总结，它就会变得更加可靠：

```gpt-3.5-turbo-instruct
First, identify the language of the text. Second, summarize the text using the original language of the text. The summary should be one sentence long.

Text:
"""
La estadística (la forma femenina del término alemán Statistik, derivado a su vez del italiano statista, "hombre de Estado")​ es una ciencia que estudia la variabilidad, colección, organización, análisis, interpretación, y presentación de los datos, así como el proceso aleatorio que los genera siguiendo las leyes de la probabilidad.​ La estadística es una ciencia formal deductiva, con un conocimiento propio, dinámico y en continuo desarrollo obtenido a través del método científico formal. En ocasiones, las ciencias fácticas necesitan utilizar técnicas estadísticas durante su proceso de investigación factual, con el fin de obtener nuevos conocimientos basados en la experimentación y en la observación. En estos casos, la aplicación de la estadística permite el análisis de datos provenientes de una muestra representativa, que busca explicar las correlaciones y dependencias de un fenómeno físico o natural, de ocurrencia en forma aleatoria o condicional.
"""

Language:
```

```gpt-3.5-turbo-instruct
Spanish

La estadística es una ciencia que estudia la variabilidad, colección, organización, análisis, interpretación, y presentación de los datos, así como el proceso aleatorio que los genera siguiendo las leyes de la probabilidad.
```

## 在回答之前，提示模型进行解释

提高答案可靠性的另一个有效技巧是促使模型逐步推理出答案，而不是立即跳到最终答案。通过`大声思考`，模型更有可能得出正确答案。

### 零样本

#### 方法

[小岛武（Takeshi Kojima）等人于 2022 年发表了这篇论文](https://arxiv.org/abs/2205.11916)，提示模型推理出答案的最简单方法就是在答案前简单地加上`让我们一步一步地思考`。图 2 举例说明了这一点：

![](../images/zero-shot_reasoners_fig2.png)

[Source: _Large Language Models are Zero-Shot Reasoners_ by Takeshi Kojima et al. (2022).](https://arxiv.org/abs/2205.11916)

#### 结果

作者发现，将这一简单技巧应用于 MultiArith 数学数据集，`让我们一步步思考`的准确率翻了两番，从 18% 提高到 79%！

[![zero-shot reasoning example](../images/zero-shot_reasoners_tab5.png)
Source: _Large Language Models are Zero-Shot Reasoners_ by Takeshi Kojima et al. (2022).](https://arxiv.org/abs/2205.11916)

#### 影响

虽然`让我们一步一步想`的技巧对数学问题很有效，但并不是对所有任务都有效。作者发现，它对多步骤算术问题、符号推理问题、策略问题和其他推理问题最有帮助。它对简单数学问题或常识性问题没有帮助，估计对许多其他非推理任务也没有帮助。

[![zero-shot reasoning example](../images/zero-shot_reasoners_tab1.png)
Source: _Large Language Models are Zero-Shot Reasoners_ by Takeshi Kojima et al. (2022).](https://arxiv.org/abs/2205.11916)

要了解更多信息，请阅读[论文全文](https://arxiv.org/abs/2205.11916)。

如果你在自己的任务中应用这种技术，不要害怕尝试自定义指令。`让我们一步一步地想`是一种通用的方法，因此，如果能根据自己的使用情况定制更严格的指令格式，可能会取得更好的效果。例如，你可以尝试更有条理的变体，如`首先，逐步思考为什么 X 可能是真的。第二，逐步思考为什么 Y 可能是真的。第三，逐步思考 X 还是 Y 更有意义。`你甚至可以给模型一个示例格式，以帮助它保持正确的方向，例如：

```gpt-3.5-turbo-instruct
Using the IRS guidance below, answer the following questions using this format:
(1) For each criterion, determine whether it is met by the vehicle purchase
- {Criterion} Let's think step by step. {explanation} {yes or no, or if the question does not apply then N/A}.
(2) After considering each criterion in turn, phrase the final answer as "Because of {reasons}, the answer is likely {yes or no}."

IRS guidance:
"""
You may be eligible for a federal tax credit under Section 30D if you purchased a car or truck that meets the following criteria:
- Does the vehicle have at least four wheels?
- Does the vehicle weigh less than 14,000 pounds?
- Does the vehicle draw energy from a battery with at least 4 kilowatt hours that may be recharged from an external source?
- Was the vehicle purchased in a year before 2022?
  - If so, has the manufacturer sold less than 200,000 qualifying vehicles? (Tesla and GM have sold more than 200,000 qualifying vehicles.)
- Was the vehicle purchased in a year after 2022?
  - If so, is the vehicle present in the following list of North American-assembled vehicles? (The only electric vehicles assembled in North America are the Audi Q5, BMW 330e, BMW X5, Chevrolet Bolt EUV, Chevrolet Bolt EV, Chrysler Pacifica PHEV, Ford Escape PHEV, Ford F Series, Ford Mustang MACH E, Ford Transit Van, GMC Hummer Pickup, GMC Hummer SUV, Jeep Grand Cherokee PHEV, Jeep Wrangler PHEV, Lincoln Aviator PHEV, Lincoln Corsair Plug-in, Lucid Air, Nissan Leaf, Rivian EDV, Rivian R1S, Rivian R1T, Tesla Model 3, Tesla Model S, Tesla Model X, Tesla Model Y, Volvo S60, BMW 330e, Bolt EV, Cadillac Lyriq, Mercedes EQS SUV, and Nissan Leaf.)
"""

Question: Can I claim a federal tax credit for my Toyota Prius Prime bought in 2021?

Solution:

(1) For each criterion, determine whether it is met by the vehicle purchase
- Does the vehicle have at least four wheels? Let's think step by step.
```

```gpt-3.5-turbo-instruct
 The Toyota Prius Prime has four wheels, so the answer is yes.
- Does the vehicle weigh less than 14,000 pounds? Let's think step by step. The Toyota Prius Prime weighs less than 14,000 pounds, so the answer is yes.
- Does the vehicle draw energy from a battery with at least 4 kilowatt hours that may be recharged from an external source? Let's think step by step. The Toyota Prius Prime has a battery with at least 4 kilowatt hours that may be recharged from an external source, so the answer is yes.
- Was the vehicle purchased in a year before 2022? Let's think step by step. The Toyota Prius Prime was purchased in 2021, which is before 2022, so the answer is yes.
- Was the vehicle purchased in a year after 2022? N/A
- If so, is the vehicle present in the following list of North American-assembled vehicles? N/A

(2) After considering each criterion in turn, phrase the final answer as "Because of {reasons}, the answer is likely {yes or no}."

Because the Toyota Prius Prime meets all of the criteria for a federal tax credit, the answer is likely yes.
```

### 少样本示例

#### 方法

提示模型推理出答案的方法有很多。其中一种方法是用几个例子来演示（'少样本'），谷歌的 [Jason Wei 和 Denny Zhou](https://ai.googleblog.com/2022/05/language-models-perform-reasoning-via.html) 等人就是这样研究的。下面是一个”少样本“思维链提示的例子：

[![chain of thought example](../images/chain_of_thought_fig1.png)
Source: _Chain of Thought Prompting Elicits Reasoning in Large Language Models_ Jason Wei and Denny Zhou et al. (2022)](https://ai.googleblog.com/2022/05/language-models-perform-reasoning-via.html)

更多由人类标签员编写的推理链演示：

[![chain of thought example](../images/chain_of_thought_fig3.png)
Source: _Chain of Thought Prompting Elicits Reasoning in Large Language Models_ Jason Wei and Denny Zhou et al. (2022)](https://ai.googleblog.com/2022/05/language-models-perform-reasoning-via.html)

[(请注意，梨子是否真的会浮起来还存在疑问）](https://twitter.com/Meaningness/status/1561062170074370048?s=20&t=mpHt8f3RRboztXxdhLFnWQ)

#### 结果

作者在对小学数学问题进行测试后发现，思维链提示使解题率提高了两倍，从 18% 提高到 57%。

[![chain of thought example](../images/chain_of_thought_fig5.png)
Source: _Chain of Thought Prompting Elicits Reasoning in Large Language Models_ Jason Wei and Denny Zhou et al. (2022)](https://ai.googleblog.com/2022/05/language-models-perform-reasoning-via.html)

除数学问题外，思维链提示还提高了与体育理解、抛硬币追踪和最后一个字母连接有关的问题的成绩。在大多数情况下，不需要太多的例子就能使成绩提升达到饱和（少于 8 个左右）。

[![chain of thought example](../images/chain_of_thought_fig11.png)
<br>Source: _Chain of Thought Prompting Elicits Reasoning in Large Language Models_ Jason Wei and Denny Zhou et al. (2022)](https://ai.googleblog.com/2022/05/language-models-perform-reasoning-via.html)

要了解更多信息，请阅读[论文全文](https://arxiv.org/abs/2201.11903)。

#### 影响

与`让我们一步步思考`技术相比，基于少量示例的方法的一个优势是，您可以更轻松地指定您希望模型在得出最终答案之前进行推理的格式、长度和风格。当模型最初的推理方式或深度不正确时，这一点尤其有用。

### 微调

#### 方法

一般来说，要想在任务中获得最高性能，就需要对自定义模型进行微调。然而，使用解释对模型进行微调可能需要成千上万个解释示例，而编写解释示例的成本很高。

在2022年，Eric Zelikman 和 Yuhuai Wu 等人发表了一种巧妙的方法，利用少样本提示来生成可以用于微调模型的解释数据集。这个想法是使用少样本提示来生成候选解释，并仅保留产生正确答案的解释。然后，为了获得一些错误答案的附加解释，重新尝试少样本提示，但在问题中给出正确答案作为一部分。作者将他们的方法称为 STaR（自学习推理器）。

[![STaR procedure](../images/star_fig1.png)
Source: _STaR: Bootstrapping Reasoning With Reasoning_ by Eric Zelikman and Yujuai Wu et al. (2022)](https://arxiv.org/abs/2203.14465)

有了这种技术，你就可以把微调的好处和思维链提示的好处结合起来，而不需要编写成千上万的示例解释。

#### 结果

当作者将这一技术应用于《常识问答》数据集时，他们发现 STaR 的表现优于单纯的思维链提示（73% > 37%）和单纯的微调（73% > 60%）：

[![STaR results](../images/star_tab1.png)
Source: _STaR: Bootstrapping Reasoning With Reasoning_ by Eric Zelikman and Yujuai Wu et al. (2022)](https://arxiv.org/abs/2203.14465)

要了解更多信息，请阅读[论文全文](https://arxiv.org/abs/2203.14465)。

#### 影响

使用少量提示来扩展或修改微调数据集是一个可以推广到解释性写作之外的想法。例如，如果您有大量非结构化文本需要训练，您可能会发现有机会使用提示符从非结构化文本中提取结构化数据集，然后在该结构化数据集上微调自定义模型。

## 思维链提示拓展

此外，还发表了许多思维链提示的扩展文章。

### 选择-推理提示(selection-inference prompting)

#### 方法

由 Antonia Creswell 等人发表的一种链式思维技术的扩展是将用于生成解释和答案的单个提示分成较小的部分。首先，一个提示从文本中选择相关的事实子集（“选择提示”）。然后，第二个提示从所选的事实中推断出一个结论（“推理提示”）。这些提示在循环中交替使用，以生成多个推理步骤，并最终得出最终答案。作者在下图中说明了这个想法：

[![Selection-inference prompting](../images/selection-inference_fig1.png)
Source: _Selection-Inference: Exploiting Large Language Models for Interpretable Logical Reasoning_ by Antonia Creswell et al. (2022)](https://arxiv.org/abs/2205.09712)

#### 结果

当应用于一个 7B 参数模型时，作者发现在 bAbi 和 Proof Writer 基准任务（这两个任务都需要较长的推理步骤序列）上，选择-推理提示相对于思维链提示大大提高了性能。他们取得的最佳成绩是将选择-推理提示与微调相结合。

[![Selection-inference prompting](../images/selection-inference_fig4.png)
Source: _Selection-Inference: Exploiting Large Language Models for Interpretable Logical Reasoning_ by Antonia Creswell et al. (2022)](https://arxiv.org/abs/2205.09712)

#### 影响

虽然在这些基准上取得了很大的进步，但之所以特别选择这些基准，是因为它们需要较长的推理序列。在不需要多步骤推理的问题上，收益可能会小一些。

这些结果凸显了使用大语言模型的一些普遍经验。首先，将复杂任务拆分成较小的任务是提高可靠性和性能的好方法；任务越复杂，模型出错的空间就越小。其二，要想获得最高性能，通常需要将微调与您所选择的任何方法结合起来。

要了解更多信息，请阅读[论文全文](https://arxiv.org/abs/2205.09712)。

### 忠实推理架构

在发表选择推理提示技术几个月后，作者在后续论文中对该技术进行了扩展，提出了以下想法

- 确定选择-推理循环何时应该停止或继续
- 添加一个值函数，帮助搜索多条推理路径
- 通过微调模型来推理句子标签（如 Sen1），而不是写出句子本身，从而减少对虚假事实的幻觉

#### 方法

在最初的选择-推理技术中，专门的“选择”和“推理”提示被交替使用，以选择事实并从这些事实中进行推理，从而生成一系列推理步骤。

作者通过两个额外的组件扩展了这个技术。

首先，作者添加了一个“停止器”模型，在每个推理步骤之后，询问该推理步骤是否足以回答问题。如果是，则模型生成最终答案。

停止器模型带来了一些优势：

- 它可以告诉选择-推理过程何时停止或继续进行，根据需要进行调整。
- 如果该过程从未停止，将不会得到答案，这往往比产生虚构的猜测更可取。

[![Faithful reasoning](../images/faithful-reasoning_fig3.png)
Source: _Faithful Reasoning Using Large Language Models_ by Antonia Creswell et al. (2022)](https://arxiv.org/abs/2208.14271)

[![Faithful reasoning](../images/faithful-reasoning_fig5.png)
Source: _Faithful Reasoning Using Large Language Models_ by Antonia Creswell et al. (2022)](https://arxiv.org/abs/2208.14271)

其次，作者添加了一个价值函数，用于评估推理步骤的质量和搜索多个推理轨迹。这与提高可靠性的一个共同主题相呼应；与其从模型中生成单一答案，不如生成一组答案，然后使用某种类型的价值函数/判别器/验证器模型来选出最佳答案。

[![Faithful reasoning](../images/faithful-reasoning_fig7.png)
Source: _Faithful Reasoning Using Large Language Models_ by Antonia Creswell et al. (2022)](https://arxiv.org/abs/2208.14271)

除了这两项扩展，作者还使用了一种技巧来减少对虚假事实的幻觉。他们没有要求模型写出事实性句子，而是对模型进行了微调，使其能够使用句子标签（例如，sen1）。这有助于防止模型幻觉出提示语境中未提及的虚假事实。

[![Faithful reasoning](../images/faithful-reasoning_fig4.png)
Source: _Faithful Reasoning Using Large Language Models_ by Antonia Creswell et al. (2022)](https://arxiv.org/abs/2208.14271)

#### 结果

作者在两个基准上评估了他们的技术：ProofWriter 任务（未显示）和 [EntailmentBankQA](https://allenai.org/data/entailmentbank) （已显示）。该技术大大提高了准确性，尤其是在较难的推理问题上。

![Faithful reasoning](../images/faithful-reasoning_tab2.png)
[Source: _Faithful Reasoning Using Large Language Models_ by Antonia Creswell et al. (2022)](https://arxiv.org/abs/2208.14271)

此外，他们的句子标签操作技巧基本上消除了幻觉！

![Faithful reasoning](../images/faithful-reasoning_tab5.png)
[Source: _Faithful Reasoning Using Large Language Models_ by Antonia Creswell et al. (2022)](https://arxiv.org/abs/2208.14271)

#### 影响

本文阐述了提高大语言模型可靠性的一些有益经验：

- 将复杂任务拆分成更小、更可靠的子任务
- 以循序渐进的方式生成答案，并在过程中对其进行评估
- 生成许多可能的答案，然后使用另一个模型或函数来挑选最合适的答案
- 通过限制模型所能表达的内容（例如，使用**句子标签**而不是句子）来减少幻觉
- 通过在专门任务中对模型进行微调，最大限度地提高模型性能

要了解更多信息，请阅读[论文全文](https://arxiv.org/abs/2205.09712)。

> 句子标签是一种用于减少幻觉的技术。这项技术的主要目的是通过为每个句子添加标签，以便更好地理解句子的上下文和语境，从而减少幻觉和提高模型的性能。
>
> 具体实现方式可能包括为每个句子添加语法标签、词性标签或其他语义标签，以便模型能够更好地理解句子的结构和含义。这样一来，模型在进行推理和生成文本时，就能更准确地理解句子的含义，从而减少幻觉的发生。

### 从少到多的提示

除了在长推理链（选择推理的优势所在）上表现不佳外，当示例较短但任务较长时，思维链提示法尤其吃力。

#### 方法

从少到多提示法是另一种将推理任务分解为更小、更可靠的子任务的技术。其思想是通过提示模型，例如`为了解决{问题}，我们首先需要解决：”`，来引导模型产生一个子任务。然后，凭借这个子任务，模型可以生成一个解决方案。解决方案被附加到原始问题上，并重复这个过程，直到产生最终答案。

[![Least-to-most prompting](../images/least-to-most_fig1.png)
Source: _Least-to-most Prompting Enables Complex Reasoning in Large Language Models_ by Denny Zhou et al. (2022)](https://arxiv.org/abs/2205.10625)

#### 结果

在使用 `code-davinci-002`（针对代码进行了优化，但仍能理解文本）进行涉及长推理链的基准测试时，作者测得的收益高达 16% -> 99.7%！

[
![Least-to-most prompting results on last-letter-concatenation task](../images/least-to-most_tab4.png)
![Least-to-most prompting results on SCAN](../images/least-to-most_tab9.png)
![Least-to-most prompting results on DROP numerical reasoning](../images/least-to-most_tab11.png)
Source: _Least-to-most Prompting Enables Complex Reasoning in Large Language Models_ by Denny Zhou et al. (2022)](https://arxiv.org/abs/2205.10625)

#### 影响

尽管上述从少到多提示所带来的收益令人印象深刻，但这些收益是在需要较长推理链的非常狭窄的任务集上测得的。

不过，它们还是说明了一个共同的主题：通过（a）将复杂的任务分解成更小的子任务，以及（b）给模型更多的时间和空间来找出答案，从而提高可靠性。

要了解更多信息，请阅读[论文全文](https://arxiv.org/abs/2205.10625)。

## 相关理念

### Maieutic prompting

#### 方法

与前几种试图最大限度地提高正确答案可能性的技术相比，另一种方法是使用 GPT-3 生成一棵可能解释（正确和不正确）的树，然后分析它们之间的关系，猜测哪一组是正确的。这种技术被 [Jaehun Jung 等人于 2022 年 5 月](https://arxiv.org/abs/2205.11822) 称为"maieutic prompting"（"maieutic"的意思是与苏格拉底式的提问方法有关，即通过提问来激发想法）。

该方法比较复杂，具体步骤如下：

- 首先，构建一个助产树，其中每个节点都是一个可能为真或为假的陈述：
  - 从一个多项选择题或真/假陈述开始（例如`战争不可能平局`）。
  - 对于问题的每个可能答案，使用模型生成相应的解释（使用类似`战争不可能平局？是的，因为`的提示）。
  - 然后，使用问题和生成的解释提示模型，并要求其生成答案。如果通过反转解释（使用类似`说{解释}是错误的`的前缀）可以改变答案，那么该解释被认为是“逻辑完整的”。
  - 如果一个解释在逻辑上不完整，则递归地重复上述过程，将每个解释转化为一个真或假的问题，并为每个新问题生成更多的解释。
  - 当所有递归解释完成后，您将得到一个解释树，其中每个叶子都具有反转解释会导致模型答案相反的属性。
- 其次，将该树转换为关系图：
  - 对于树中的每个节点，计算模型对每个节点的相对置信度（从给出解释得到“True”答案的概率推断）。
  - 对于树中的每对节点，使用模型确定它们是蕴含（暗示）还是相互矛盾的关系。
- 第三，找到最一致的信念集，并将其视为真：
  - 具体而言，利用每个节点的置信度和它们之间的逻辑关系，将问题形式化为加权最大可满足性问题（MAX-SAT）。
  - 使用求解器找到最自洽的信念集，并将其视为真实的。


![Maieutic prompting](../images/maieutic_fig2.png)



![Maieutic prompting](../images/maieutic_fig6.png)
Source: _Maieutic Prompting: Logically Consistent Reasoning with Recursive Explanations_ by Jaehun Jung et al. (2022)](https://arxiv.org/abs/2205.11822)

#### 结果

![Maieutic prompting results](../images/maieutic_tab1.png)
[Source: _Maieutic Prompting: Logically Consistent Reasoning with Recursive Explanations_ by Jaehun Jung et al. (2022)](https://arxiv.org/abs/2205.11822)

#### 影响

除了复杂性之外，这种方法还有一个局限性，那就是它似乎只适用于可以作为多项选择题提出的问题。

要了解更多信息，请阅读[论文全文](https://arxiv.org/abs/2205.11822)。

## 拓展

### 自洽性

#### 方法

对于有离散答案集的任务，提高可靠性的一个简单方法是从模型中抽取多个解释和答案（使用正温度），然后选出出现频率最高的最终答案。

[![Self-consistency method](../images/self-consistency_fig1.png)
Source: _Self-Consistency Improves Chain of Thought Reasoning in Language Models_ by Xuezhi Wang et al. (2022)](https://arxiv.org/abs/2203.11171)

#### 结果

在一系列数学和推理基准测试中，这项技术将准确率提高了 1 到 24 个百分点不等。(下面绘制的是谷歌 LaMDA 模型的结果；使用谷歌更大的 PaLM 模型，基线更高，但收益略低）。

[![Self-consistency results](../images/self-consistency_fig3.png)
Source: _Self-Consistency Improves Chain of Thought Reasoning in Language Models_ by Xuezhi Wang et al. (2022)](https://arxiv.org/abs/2203.11171)

#### 影响

尽管这种技术很容易实现，但它可能代价高昂。生成一组10个答案将使您的成本增加10倍。

此外，与许多这些技术一样，它仅适用于具有有限答案集的任务。对于每个答案都是唯一的开放性任务（例如写诗），选择最常见的答案意义不明确。

最后，当存在多个路径或表达方式可以达到答案时，这种技术应该是最有益的；如果只有一条路径，那么这种技术可能根本没有帮助。一个极端的例子是：如果任务是生成一个令牌答案，那么从100次生成中选择最常见的令牌与从具有最高对数概率的令牌中选择（可以在temperature=0的单个生成中获得）没有区别。

### 验证器

另一种提高任务性能的关键技术是训练一个验证器或鉴别器模型来评估主要生成模型的输出。如果鉴别器拒绝输出，则可以重新采样生成模型，直到获得可接受的输出。在许多情况下，判断一个答案是否正确比创建一个答案更容易，这有助于解释这种方法的优势。

#### 方法

2021年，OpenAI的研究人员将这种技术应用于小学数学问题的评分，使用以下步骤：

- 首先，他们在问题和解决方案上对模型进行了微调。
- 对于训练集中的每个问题，他们生成了100个解决方案。
- 这100个解决方案中的每一个根据最终答案是否正确被自动标记为正确或错误。
- 使用这些解决方案，其中一些被标记为正确，一些被标记为错误，他们对验证器模型进行了微调，以分类问题和候选解决方案是否正确或错误。
- 最后，在测试时，生成模型为每个问题创建100个解决方案，并根据验证器模型的评分选择得分最高的解决方案作为最终答案。

[![Verifier method](../images/verifiers_fig3.png)
Source: _Training Verifiers to Solve Math Word Problems_ by Karl Cobbe et al. (2021)](https://arxiv.org/abs/2110.14168)

#### 结果

使用 175B GPT-3 模型和 8,000 个训练示例，该技术大大提高了小学数学的准确率，从 ~33% 提高到 ~55%。

[![Verifier results](../images/verifiers_fig5.png)
Source: _Training Verifiers to Solve Math Word Problems_ by Karl Cobbe et al. (2021)](https://arxiv.org/abs/2110.14168)

#### 影响

与自洽性技术类似，这种方法的成本也会很高，比如说，每个任务生成 100 个解决方案会使成本增加约 100 倍。

## 可靠性理论

尽管上述技术的方法各不相同，但它们的共同目标都是提高复杂任务的可靠性。它们主要通过以下方式实现这一目标

- 将不可靠的操作分解为更小、更可靠的操作（例如，选择-推理提示）
- 使用多个步骤或多种关系，使系统的可靠性高于任何单个组件（例如，智能提示法）

#### 概率图形模型

这种试图用可靠性较低的组件来构建一个可靠系统的范式，让人联想到概率编程，而且该领域的许多分析技术也可应用于这一范式。

在_语言模型级联_一文中，戴维-多汉（David Dohan）等人在概率图形模型范式中解释了上述技术：

#### 思维链提示

[![graphical model of chain of thought prompting](../images/lm_cascades_fig1.png)
Source: _Language Model Cascades_ by David Dohan et al. (2022)](https://arxiv.org/abs/2207.10342)

#### 微调思维链提示/自学器

[![graphical model of fine-tuned chain of thought prompting](../images/lm_cascades_fig3.png)
Source: _Language Model Cascades_ by David Dohan et al. (2022)](https://arxiv.org/abs/2207.10342)

#### 选择推理提示

[![graphical model of selection-inference prompting](../images/lm_cascades_fig4.png)
Source: _Language Model Cascades_ by David Dohan et al. (2022)](https://arxiv.org/abs/2207.10342)

#### 验证器

[![graphical model of verifiers](../images/lm_cascades_fig5.png)
Source: _Language Model Cascades_ by David Dohan et al. (2022)](https://arxiv.org/abs/2207.10342)

#### 影响

尽管将这些技术表述为概率图形模型可能无法立即用于解决任何特定问题，但该框架可能有助于选择、组合和发现新技术。

## 结束语

大语言模型的研究非常活跃，而且发展迅速。研究人员不仅不断改进模型，还不断提高我们对如何更好地使用模型的理解。为了强调这些发展的速度，请注意上面分享的所有论文都是在过去 12 个月内发表的（在我写这篇文章时是 2022 年 9 月）。

未来，我们期待有更好的模型和技术发表。即使这里的具体技术被未来的最佳实践所取代，它们背后的一般原则仍将是专家用户工具包的重要组成部分。

## 参考资料

| Lesson                                                       | Paper                                                        | Date     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------- |
| 将复杂任务分解为更简单的子任务（并考虑向用户公开中间产出）   | [AI Chains: Transparent and Controllable Human-AI Interaction by Chaining Large Language Model Prompts](https://arxiv.org/abs/2110.01691) | 2021 Oct |
| 您可以通过生成多个候选方案，然后挑选最合适的方案来提高产量   | [Training Verifiers to Solve Math Word Problems](https://arxiv.org/abs/2110.14168) | 2021 Oct |
| 在推理任务中，模型在回答前逐步推理的效果更好                 | [Chain of Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903) | 2022 Jan |
| 您可以通过生成许多解释-答案输出并挑选最受欢迎的答案来改进逐步推理方法 | [Self-Consistency Improves Chain of Thought Reasoning in Language Models](https://arxiv.org/abs/2203.11171) | 2022 Mar |
| 如果您想对分步推理器进行微调，只需使用多选题和答案数据就可以了 | [STaR: Bootstrapping Reasoning With Reasoning](https://arxiv.org/abs/2203.14465) | 2022 Mar |
| 即使没有实例，逐步推理法也非常有效                           | [Large Language Models are Zero-Shot Reasoners](https://arxiv.org/abs/2205.11916) | 2022 May |
| 通过交替使用"选择"提示和"推理"提示，您可以获得比逐步推理更好的效果 | [Selection-Inference: Exploiting Large Language Models for Interpretable Logical Reasoning](https://arxiv.org/abs/2205.09712) | 2022 May |
| 对于较长的推理问题，您可以将问题拆分成若干块，逐步解决，从而提高逐步推理的能力 | [Least-to-most Prompting Enables Complex Reasoning in Large Language Models](https://arxiv.org/abs/2205.10625) | 2022 May |
| 您可以让模型分析好的解释和假的解释，找出哪组解释最一致       | [Maieutic Prompting: Logically Consistent Reasoning with Recursive Explanations](https://arxiv.org/abs/2205.11822) | 2022 May |
| 您可以从概率编程的角度来考虑这些技术，在概率编程中，系统由不可靠的组件组成 | [Language Model Cascades](https://arxiv.org/abs/2207.10342)  | 2022 Jul |
| 您可以通过句子标签操作消除幻觉，也可以通过“停止器”提示减少错误答案 | [Faithful Reasoning Using Large Language Models](https://arxiv.org/abs/2208.14271) | 2022 Aug |
