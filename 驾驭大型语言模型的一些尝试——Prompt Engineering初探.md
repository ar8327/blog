# 驾驭大型语言模型的一些尝试——Prompt Engineering初探 

近来，以ChatGPT为代表的LLM在中国大火。

![ChatGPT trending in China](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/ChatGPT%20trending%20in%20China.png)



这话可不是胡说的，从Google Trend来看，中国地区的搜索量要远超世界各地。

随着一些“爆炸性”、“革命性”的新闻，ChatGPT仿佛成了下一个工业革命，AI取代传统劳动力的论调不一而足。

这篇文章不会讨论所谓的“AGI”会对未来产生什么样的影响，相反，我希望这篇文章能脚踏实地的谈一谈目前这些语言模型的特性，以及如何能更好驾驭它们。

如果没有特殊说明，下面的测试都是在OpenAI的ChatGPT March 23  版本上进行测试的。

[这里](https://help.openai.com/en/articles/6825453-chatgpt-release-notes)是ReleaseNote



# 胡说的语言模型

首先，我想从几个有趣的例子开始。

在下面的例子中，我用两种方式询问ChatGPT谁是"金博文"，这是一个我虚构的名字，这个名字在中国也比较常见。

第一次询问，ChatGPT编造了一个人物

![Who is '金博文'](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/Who%20is%20%27%E9%87%91%E5%8D%9A%E6%96%87%27.png)

稍微改变了一下询问的句式，ChatGPT的回答正常了一些

![Who is '金博文' 2](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/Who%20is%20%27%E9%87%91%E5%8D%9A%E6%96%87%27%202.png)

相比之下，GPT-4在回答这些问题的鲁棒性上比3.5要好得多

![GPT4 is better than GPT3.5](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/GPT4%20is%20better%20than%20GPT3.5.png)

不管采用怎么样的问句，GPT-4都能产生一致的正确回答。

![GPT4 consistanly better](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/GPT4%20consistanly%20better.png)

总结来说，LLM在一些领域会出现胡说的现象，更强大的LLM的一个特性就是在面对不同问题时能够产生一致、正确的回答，这意味着模型的鲁棒性有很大提升。

但模型结构的改进和重新训练是一项耗时的事情，如果有办法能够改进现有模型的表现则是更理想的，通过给定一些Prompt，可以帮助减少LLM胡说。

其实，还可以进一步地思考这个问题，既然Propmt对语言模型的影响如此之大，**那么不同语言、甚至给的例子的不同分布会不会也对语言模型的表现造成影响？**

答案也是肯定的。[这篇文章](https://arxiv.org/pdf/2202.12837.pdf)从多个角度对Few shot给出的samples对LLM的影响做了实验，结果发现，sample的分布等等都会影响LLM的最终表现。换言之，对一个比较初级的LLM，给一些人为设计的存在bias的sample，就能让LLM产生错误的结果。

![2022.12837](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/2022.12837.png)

读者看到这里可能会感到困惑，似乎之前觉得非常牛逼的LLM也不过这么一回事，有些读者可能会发现从上文展示的来看，Prompt本身近似于玄学，什么是有效的Prompt根本无章可循。

为了更系统分析这些问题，接下来我会对LLM的使用场景做一个归类，并且给目前常见的Prompt也做一个归类。通过这种形式，读者或许可以更好地明白LLM在何种任务上通常会有更好的表现。本文希望能够给读者提供一些常见的Prompt思路，并将目前LLM擅长的任务和局限性都写出来，让LLM能够更好地为人类服务。



# LLM 的常见任务

LLM并不是一个新兴的研究领域，因此，LLM模型已经有不少标准化的任务

## 翻译

LLM可以接收一段文字，并翻译成另一种语言

## 情感分析（文本分类）

LLM可以对文字进行情感分析，从文字中分析出用户的情感。情感分析的泛化问题是文本分类问题。

## 文本总结

LLM可以对大段的文字进行概括总结

## NER（命名实体标注）

LLM可以接收一段文字，并找出其中的实体，例如：

对于文本：我在南京西路看到广告牌，上面写的是上证指数今日下跌了100点。

NER或许可以这样标注：我在 | 南京西路(地名) | 看到广告牌，上面写的是 | 上证指数(金融名词) | 下跌了100点

命名实体标注是从自然语言中抽取结构化数据的重要手段

## 问题回答

LLM可以根据用户问题产生回答

## 文本补全

LLM可以对文字进行补全，你可以写个开头然后让LLM完成整个故事

## Paraphrasing

LLM可以接收一段文字，并根据用户要求重写一段文字



# Prompt Engineering 的常见思路

上一段的一些例子已经很好地说明了大型语言模型的用途，并且让读者认识到了LLM的一些局限性。

接下来，我会简单介绍Prompt Engineering的一些思路、框架，有了这些框架，能够帮助读者建立自己的知识地图。

再之后的一章，我会给出实际上的一些Prompt例子。

## Prompt Engineering的基本思路

### Few-shot / Zero-shot learning

LLM在训练时接受的语料中包含许多先验知识(prior knowledge)，因此，LLM在处理新的问题时可以从来没有见过问题。

例如，考虑下面的分类问题：高尔夫是否是一个体育运动？

由于用户在与模型交互的过程中从未提到过高尔夫相关的知识，可以认为此时模型处理的是一个Zero-shot learning problem。

![Is golf a sport](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/Is%20golf%20a%20sport.png)



与之相对地，我们可以给LLM例子，让LLM从例子中学习，并且给出判断。这就是第一个Prompt技巧，在处理LLM训练语料中不包含或者极少的数据时，给LLM少量的样本数据，让LLM现场学习。

举例来说，在下面的例子里我人为定义了4种句子的信息类型（观察、情绪、知识或者观点），并让GPT对给的一个新句子进行分类。这里我在Input中放一模棱两可的句子，希望模型可以从我给的例子中学习到模式，并按照我的预期给出正确的分类。

下面是输入：

```
Instruction: A sentence can be classified into Emotional/Observation/Knowledge/Opinion. 
Input: Tom is believed to be innocent because he was with merry last night
Category: 
```

这句句子包含Observation和Opinion，GPT-4默认会把这句句子分类为Opinion。

![GPT-4 classification](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/GPT-4%20classification.png)

我们可以预先设定一个规则，比如句子中包含Observation就认为是Observation，并根据这个规则编写两个例子，看看GPT4能否成功识别到这个模式。

```
Instruction: A sentence can be classified into Emotional/Observation/Knowledge/Opinion. 
Input: Tom didn't attend school yesterday.
Category: Observation
Input: I love you.
Category: Emotional
Input: He can't pass the exam, he didn't attend class at all.
Category: Observation
Input: Earth is one of the eight planets in our solar system.
Category: Knowledge
Input: People on earth can never live without this planet, we should protect our environment.
Category: Opinion
Input: I believe they had burn the bridge, because I saw Tom alone last night.
Category: Observation
Input: Tom is believed to be innocent because he was with merry last night
Category: 

```

![GPT-4 classification with few-shot samples](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/GPT-4%20classification%20with%20few-shot%20samples.png)

可以看到，GPT-4成功识别了模式并做出了正确的判断。

注：这里没有用GPT-3.5是因为测试没产出想要的效果，prompt没有用。

### Chain of Thought(CoT)

另一个非常常见的Prompt技巧是CoT技巧，具体来说，就是命令模型一步步地“思考”，以期提高模型在处理复杂问题时的推理能力。

这里来一个比较经典的问题，我8岁时我妈妈40岁，现在我妈妈50岁，我几岁？



GPT-3.5不负众望，成功答错。

```
I was 8 when my mother was 40. Now my mother is 50, how old am I?
```



![How old am I](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/How%20old%20am%20I.png)

使用一些简单的Prompt就可以让LLM表现得更好。例如，在最后加一句"Let's think step by step." 

```
I was 8 when my mother was 40. Now my mother is 50, how old am I? Let's think step by step.
```

![How old am I-3](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/How%20old%20am%20I-3.png)

可以看到，LLM输出了计算过程，这一输出过程的命令帮助了LLM产生正确答案。

一个更好的办法是在Prompt中把一个拆解的例子提供给LLM，能够产生更好的答案。

```
Q: I was 13 when my mother was 40. Now my mother is 55, how old am I? Let's think step by step.
A: You was 13 years old when your mother was 40. Now your mother is 55 years old, indicating 55-40=15 years has passed. Thus, adding 15 to your age 15 years ago, which was 13 will be the answer. 13+15=28, you're 28 years old.
Q: I was 8 when my mother was 40. Now my mother is 50, how old am I? Let's think step by step.
A: 
```



![How old am I-4](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/How%20old%20am%20I-4.png)

顺带一提，GPT-4就不存在这一问题。不难看出，更高阶的LLM的一个重要目标就是变得更鲁棒，摆脱对Prompt的依赖。



## 更高阶的Prompt——用工程化的方式来生成Prompt

刚才说到的两种方式其实是大部分Prompt的基本思路了。

读者应该能发现，大部分Prompt本身内容比较重复机械，因此，如果想要构建更鲁棒的模型，用一些工程化的手段来构建Prompt是一个业界共同的思路。

### 多结果投票

LLM的本质是预测下一个生成的单词是什么，因此，对于一个LLM来说，一个问题的响应就会有多种

[这篇文章](https://arxiv.org/pdf/2203.11171.pdf)中提到了让LLM先预测计算多条可能的回答，从中提取结果，并采用投票制最终产生一个结果

![2203.11171](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/2203.11171.png)

### 用LLM来给LLM生成Prompt

#### 生成知识Prompt

读者从上文的Few-shot learning中或许已经发现，Few-shot learning的例子并不一定需要人类来提供。LLM本身的一个重要用途就是产生用例和知识。

举例来说，[这篇文章](https://arxiv.org/pdf/2110.08387.pdf)就尝试让一个LLM根据问题生成知识，并让另一个LLM根据知识来产生答案。

![2110.08387](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/2110.08387.png)

我在这里也可以用ChatGPT来做一个实验，我们可以首先问ChatGPT，高尔夫球是不是一种运动？

```
Q: Is golfball a sport? Answer only yes or no.
```



![ChatGPT thinking golfball is a sport](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/ChatGPT%20thinking%20golfball%20is%20a%20sport.png)



```
Q: What is golfball?
...LLM generation...
Q: Is golfball a sport? Answer only yes or no.
```



LLM似乎认为高尔夫球是一种运动。这时，如果先询问golfball是什么，再问golfball是不是一种运动，LLM就能正确回答。

![ChatGPT with golfball knowledge](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/ChatGPT%20with%20golfball%20knowledge.png)



不难想到，基于现有的知识图谱研究和数据，如果LLM能够直接访问外部数据，在这类任务上将会有非常出色的表现。

#### 自动提示工程师——APE

[这篇文章](https://arxiv.org/pdf/2211.01910.pdf)在上述研究的基础上更进一步，试图让LLM来直接给LLM产生Prompt。

在这篇文章中，作者尝试使用一个模版，让第一个LLM输出模版中应该输入的Prompt候选，并且把第一个LLM的输出让第二个LLM不断执行，根据预先定义好的评分标准来选择最好的Prompt。

![2211.01910](./%E9%A9%BE%E9%A9%AD%E5%A4%A7%E5%9E%8B%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B0%9D%E8%AF%95%E2%80%94%E2%80%94Prompt%20Engineering%E5%88%9D%E6%8E%A2.assets/2211.01910.png)



# LLM开发者生态初探

目前，已经有了一定的软件生态，这些软件生态主要关注于如何让LLM能够与外界交互、通过工程化的手段给LLM生成更可靠的Prompt，进而提高LLM在各项任务上的表现。

## LangChain

[LangChain](https://python.langchain.com/en/latest/getting_started/getting_started.html)是一个用于连接LLM的开发框架。通过LangChain可以把LLM和外界连接（通过Agent功能让LLM可以调用外部功能），可以通过Chain功能让LLM互相对话等

## ClickPrompt

[ClickPrompt](https://github.com/prompt-engineering/click-prompt) 是一个分享Prompt的工具

## Dyno

[Dyno](https://trydyno.com/) 是一个Prompt的IDE工具



# 结语

本文在编写过程中，很大程度参考了[Prompt Engineering Guide](https://www.promptingguide.ai/) 提供的材料。本文旨在对LLM进行初步了解，很多话题并没有深入。有兴趣的读者应该阅读更深入的话题/关注社区动向。