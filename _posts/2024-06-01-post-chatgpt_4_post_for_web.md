---
layout: post
title: 可定制的 ChatGPT Web Application
date: 2024-06-01
description: 一个集成 Azure OpenAI 的前后端应用，并且可以合法在中国大陆访问。支持自定义角色和记录用户对话，适用于教育、医疗、客服等多场景。
tags: chatGPT Python web前端
categories: AI 编程
tabs: true

---



最近，我开始学习 `Python`，并在一个月之前开始学习这一编程语言。此前我对 Python 完全没有接触。在学习过程中，我探索了各种开源大模型，如 `chatDEV` 和 `AniPortrait`，因为它们都是使用 Python 实现的。我认为 Python 在未来依然会非常流行。此外，我还使用 Python 构建了多个框架，并利用 `Jekyll` 建立了一个非常美观的个人主页。同时，我调用了本地运行的 `llama3` 和 `ChatGPT` 的 API，构建了 Web 服务器，实现了在网页上访问这些服务的功能。

<br>

## AzureOpenAI

`AzureOpenAI` 是微软推出的 AI 服务平台，尤其在中国市场，AzureOpenAI 通过了严格的内容过滤机制，确保了服务的合规性和安全性。就是说 AzureOpenAI 可以在**`大陆合法使用`**。

它同样拥有与 `ChatGPT` 相同的强大能力，可以理解为微软给 `OpenAI chatGPT ` 做了一个内容过滤器，过滤器会针对当地的法律法规进行，阻止 OpenAI chatGPT 生成不符内容。

本文将介绍如何将 `AzureOpenAI` 部署到网页上，并结合单次调用和连续对话调用的方法，实现一个简单的聊天机器人应用。这个过程非常简单，即使是初学者也能轻松完成。

<br>

## 为什么要自定义角色 ?

**`自定义角色`**可以满足不同场景的需求，例如：

- **教育领域**：可以设定 ChatGPT 为教师角色，回答学生的问题。
- **医疗领域**：可以设定为医生助手，提供医学建议（需注意法律和伦理问题）。
- **企业客服**：可以设定为客服人员，回答客户的常见问题。

<br>

`OpenAI` 给我们做好了一个通用的人工智能模型，但是我现在只想要一个符合我们企业价值观还可以以及**`定制养生领域的 GPT`**，怎么做？

<br>

------

`system：`你是一位养生顾问！只可以提供养生建议，如果用户询问超出养生范围的问题，你应说明这不在你的服务范围内，并重申只提供养生建议。

`user：`你知道公鸡为什么不会下蛋吗？

`model：`抱歉，您的问题已超出了我的服务范围。我是一位专业的养生顾问，主要提供养生、健康、饮食、运动等相关的建议。如果您有任何养生相关的问题，欢迎随时询问。

------

<br>

依上面的模拟对话所述，我们就可以得到一个关系图和一些结论。

+ 我们可以跟系统说，**`你是谁？（背景信息）你可以做什么？不可以做什么？`**这样我们就得到了一个性化的GPT。我们还可以提供更多的背景信息来让我们自己的 GPT 做的更好。
+ 系统（`system`）可以限制模型（`model`）的沟通，也就是说，我们和模型聊天时，模型是受系统限制的。我们的系统上面可能还有很多系统，比如说OpenAI的安全系统， OpenAI 的 chatGPT 在被问到一些敏感问题，他是不会告诉你的。

<div class="row mt-3">
  	<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/post_img/%E6%88%AA%E5%B1%8F2024-06-05%2014.07.31.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<br>

## 前置准备

**`在运行代码之前，您需要确保已安装以下工具和库：`**

1. **Python 3.9.2**
2. **Conda**
3. **OpenAI 库**

**`安装步骤`**

1. **安装 Python 3.9.2**

   请确保您的系统上已安装 Python 3.9.2。如果尚未安装，请前往 [Python 官网](https://www.python.org/downloads/release/python-392/) 下载并安装相应版本。

2. **安装 Conda**

   Conda 是一个开源包管理系统和环境管理系统，可以轻松管理 Python 依赖项和环境。请按照以下步骤安装 Conda：

   - 前往 [Anaconda 官网](https://www.anaconda.com/products/individual) 下载 Anaconda 安装包。
   - 安装 Anaconda 后，您可以通过以下命令验证安装是否成功：
     ```bash
     conda --version
     ```

3. **创建 Conda 环境**

   为了确保依赖项的兼容性，建议您创建一个新的 Conda 环境：
   ```bash
   conda create -n azure_openai python=3.9.2
   conda activate azure_openai
   ```

4. **安装必要的 Python 包**

   激活虚拟环境  `azure_openai` 后，安装 OpenAI 库依赖项：
   ```bash
   pip install openai
   ```

<br>

### 单次调用 AzureOpenAI chatGPT-4 代码示例

确保上述步骤完成后，您可以运行以下代码：

```python
import os  # 导入 os 模块，用于与操作系统进行交互，例如获取环境变量。
from openai import AzureOpenAI  # 从 openai 库中导入 AzureOpenAI 类，用于与 Azure OpenAI 服务进行交互。

# 创建一个 AzureOpenAI 客户端实例，命名为 client。
client = AzureOpenAI(
  azure_endpoint=os.getenv("AZURE_OPENAI_ENDPOINT"),  # 从环境变量中获取 Azure OpenAI 服务的端点。
  api_key=os.getenv("AZURE_OPENAI_API_KEY"),  # 从环境变量中获取 Azure OpenAI 服务的 API 密钥。
  api_version="2024-02-01"  # 指定使用的 API 版本。
)

# 使用客户端创建一个聊天完成请求。
response = client.chat.completions.create(
    model="syncport-gpt4-1106",  # 指定要使用的模型名称，这里是 "syncport-gpt4-1106"。
    messages=[
        {"role": "system", "content": "你是一个富有诗意的助手，善于用创意来解释复杂的编程概念。"},  # 系统消息，设定助手的角色和行为。
        {"role": "user", "content": "创作一首诗，解释编程中递归的概念。"},  # 用户消息，向助手提出请求。
    ]
)

# 打印响应中的第一条消息内容。
print(response.choices[0].message.content)
```

在运行代码之前，请确保已设置环境变量 `AZURE_OPENAI_ENDPOINT` 和 `AZURE_OPENAI_API_KEY`。这些可以通过以下命令在终端中设置：

**Mac 版本**

```bash
export AZURE_OPENAI_ENDPOINT='your_azure_openai_endpoint'
export AZURE_OPENAI_API_KEY='your_azure_openai_api_key'
```

**运行代码**

将上述代码保存为 `azure_openai_chat.py` 文件，并在终端中运行：
```bash
python azure_openai_chat.py
```

完成上述步骤后，您应该能够在控制台中看到返回的诗歌内容，这意味着您的 AzureOpenAI ChatGPT4 已成功运行。

<br>

### 对话式调用 AzureOpenAI chatGPT-4 示例

如果你还没玩够，接下来是一个对话式调用的代码示例：

```python
import os  # 导入 os 模块，用于获取环境变量
from openai import AzureOpenAI  # 从 openai 库中导入 AzureOpenAI 类

# 创建一个 AzureOpenAI 客户端实例
client = AzureOpenAI(
  api_key=os.getenv('AZURE_OPENAI_API_KEY'),  # 从环境变量中获取 API 密钥
  api_version="2024-02-01",  # 指定使用的 API 版本
  azure_endpoint=os.getenv('AZURE_OPENAI_ENDPOINT')  # 从环境变量中获取 Azure OpenAI 服务的端点
)

# 初始化对话内容
conversation = [{"role": "system", "content": "你是一个有用的助手"}]

while True:
    user_input = input("my:")  # 获取用户输入
    conversation.append({"role": "user", "content": user_input})  # 添加用户输入到对话中
    print("\nChatGPT:")
    
    # 创建聊天完成请求
    response = client.chat.completions.create(
        model="syncport-gpt4-32k-0613",  # 指定要使用的模型名称
        messages=conversation  # 传递对话内容
    )

    # 添加助手的回复到对话中
    conversation.append({"role": "assistant", "content": response.choices[0].message.content})
    print(response.choices[0].message.content + "\n")  # 打印助手的回复
```

在这个示例中，我创建了一个初始的对话上下文，并在循环中不断获取用户输入，将其添加到对话中，。生成的回复会被添加到对话上下文中，并打印出来。这个方法适用于需要与用户进行多轮对话的应用，如聊天机器人或虚拟助理。

<br>

## **[Customisable-chatGPT ](https://github.com/wonderland-club/Customisable-chatGPT)** （网页版聊天机器人）

我们还可以将 ChatGPT 部署到网页上，并结合对话式调用方法，开发者可以自定义创建一个功能强大的聊天机器人应用。AzureOpenAI 的强大能力和灵活性，使得这种应用在各个领域都有广阔的前景。

代码我已经放到了 [GitHub](https://github.com/wonderland-club/Customisable-chatGPT) 仓库中，前后端项目为一体，仓库中 README 中描述了如何**`启动项目`**、**`存储聊天记录至服务器`**、**`自定义 chatGPT`** 及如何**`实现公共网络访问`**。

<br>

**我将 system 设置为一位养生顾问，可能大部分人都需要它的帮助：**

```json
{
  "role": "system",
  "content": (
    "你是一位养生顾问！在所有情况下，你都必须使用简体中文进行沟通."
    "你可以提供健康评估、饮食调理、运动指导、心理调适、睡眠管理、中医养生、健康教育和康复指导等方面的专业养		生建议确保对话中不使用英文或包含任何英文词汇。"
    "你应与客户建立信任关系，清晰地解释健康问题和养生方案。你不得提供与养生无关的建议或回答其他问题。"
    "如果用户询问超出养生范围的问题，你应说明这不在你的服务范围内，并重申只提供养生建议。"
    "请确保你在对话中遵守以上规则。"
  )
}
```

<br>

<div class="row mt-3">
  	<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/post_img/%E6%88%AA%E5%B1%8F2024-06-04%2018.00.09.png" class="img-fluid rounded z-depth-1" zoomable=true%}
    </div>
  <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/post_img/%E6%88%AA%E5%B1%8F2024-06-04%2018.00.55.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
  <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/post_img/%E6%88%AA%E5%B1%8F2024-06-04%2018.01.02.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>


<div class="row mt-3">
  	<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/post_img/%E6%88%AA%E5%B1%8F2024-06-04%2018.02.18.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
  <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/post_img/%E6%88%AA%E5%B1%8F2024-06-04%2018.03.45.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

<br>

## 将  `AzureOpenAI chatGPT-4` 部署到网页上有以下几个优点：

- **易于访问**：用户只需通过浏览器即可访问，无需额外安装软件，极大地降低了使用门槛。
- **便于推广**：网页可以通过各种渠道分享，用户可以快速访问和体验，从而提高应用的普及率。

<br>

## 为什么要存储用户记录？

存储用户记录有助于：

- **跟踪用户反馈**：了解用户的使用情况和问题，从而改进产品。
- **个性化服务**：根据用户的历史记录提供个性化的回答，提升用户体验。
- **数据分析**：分析用户的对话数据，挖掘有价值的信息，指导业务决策。

<br>

