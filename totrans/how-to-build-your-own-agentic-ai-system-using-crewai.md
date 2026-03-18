# 如何使用 CrewAI 构建自己的代理式人工智能系统

> 原文：[`towardsdatascience.com/how-to-build-your-own-agentic-ai-system-using-crewai/`](https://towardsdatascience.com/how-to-build-your-own-agentic-ai-system-using-crewai/)

## <mdspan datatext="el1762569992687" class="mdspan-comment">什么是代理式</mdspan>人工智能？

代理式人工智能，最初由 Andrew Ng 提出，作为能够自主**规划、执行和完成复杂任务**的 AI“伙伴”，是随着生成式人工智能应用的爆发而出现的新概念。根据 Google Trends 的搜索量，该术语自 2025 年 7 月下旬以来迅速获得人气。

![“Agentic AI”过去 12 个月的 Google 搜索量](img/e224cda6b7d0b2d036cf38a9f4899663.png)

“Agentic AI”过去 12 个月的 Google 搜索量

尽管出现时间不长，BCG 的研究文章“[代理式人工智能如何改变企业平台](https://www.bcg.com/publications/2025/how-agentic-ai-is-transforming-enterprise-platforms)”表明，组织一直在积极采用代理式人工智能工作流程来转型其核心技术平台，并协助营销自动化、客户服务、工作场所生产力等，从而使得工作流程周期加快 20%至 30%。

本文也有视频版本。

## 从 LLM 到多代理系统

区分代理式人工智能系统与传统自动化系统的关键在于其自主规划行动和逻辑的能力，只要能实现一个特定、预定义的目标。因此，对代理的中间步骤进行严格编排或预定的决策轨迹较少。"[在语言模型中协同推理和行动](https://arxiv.org/abs/2210.03629)”被认为是将早期 LLM 代理框架“ReAct”形式化的基础论文，该框架由三个关键元素组成——**行动**、**思维**和**观察**。如果您想了解更多关于 ReAct 如何工作的细节，请参阅我的博客文章“[简要解释 6 种常见的 LLM 定制策略](https://towardsdatascience.com/6-common-llm-customization-strategies-briefly-explained/)”。

> [简要解释 6 种常见的 LLM 定制策略](https://towardsdatascience.com/6-common-llm-customization-strategies-briefly-explained/)

随着该领域的快速增长，很明显，单个 LLM 代理无法满足人工智能应用和集成的需求。因此，开发了多代理系统来编排代理的功能，形成一个动态的工作流程。虽然每个代理实例都是基于角色的、以任务为导向的，强调实现单一目标，但多代理系统具有多功能性和更通用的能力。LangChain 文章“[**多代理架构基准测试**](https://blog.langchain.com/benchmarking-multi-agent-architectures/)”表明，当任务中所需的知识领域数量增加时，单代理系统的性能会下降，而多代理系统可以通过减少输入到每个代理的噪声量来实现可持续的性能。

## 使用 CrewAI 构建简单的代理式 AI 系统

CrewAI 是一个开源的 Python 框架，允许开发者构建生产就绪和协作的 AI 代理团队来处理复杂任务。与其他流行的代理框架如 LangChain 和 LlamaIndex 相比，它更侧重于基于角色的多代理协作，同时为复杂的代理架构提供较少的灵活性。尽管它是一个相对较新的框架，但由于其实施的简便性，从 2025 年 7 月开始，它越来越受到人们的关注。

> *使用 CrewAI 框架构建代理系统时，我们可以使用雇佣一个跨职能项目团队（或一个**团队**）的类比，其中团队中的每个 AI **代理** 都有一个特定的角色，能够执行多个与角色相关的**任务**。代理配备了**工具**，这些工具有助于他们完成工作。*

现在，我们已经涵盖了 CrewAI 框架的核心概念——代理、任务、工具和团队——让我们看看示例代码来构建一个最小可行代理系统。

1. 使用以下 bash 命令安装 CrewAI 并设置环境变量，例如，将 OpenAI API 密钥作为环境变量导出，以便访问 OpenAI GPT 模型。

```py
pip install crewai
pip install 'crewai[tools]'
export OPENAI_API_KEY='your-key-here'
```

2. 从 CrewAI 的[内置工具列表](https://docs.crewai.com/en/concepts/tools)创建工具，例如，应用 `DirectoryReadTool()` 以访问目录，以及 `FileReadTool()` 以读取目录中存储的文件。

```py
from crewai_tools import DirectoryReadTool, FileReadTool

doc_tool = DirectoryReadTool(directory='./articles')
file_tool = FileReadTool()
```

3. 通过指定其角色、目标和提供工具来启动代理。

```py
from crewai import Agent

researcher = Agent(
    role="Researcher",
    goal="Find information on any topic based on the provided files",
    tools=[doc_tool, file_tool]
)
```

4. 通过提供描述并分配代理来创建任务。

```py
from crewai import Task

research_task = Task(
    description="Research the latest AI trends",
    agent=researcher
)
```

5. 通过组合你的代理和任务来组建团队。使用 `kickoff()` 启动工作流程执行。

```py
from crewai import Crew

crew = Crew(
    agents=[researcher],
    tasks=[research_task]
)

result = crew.kickoff()
```

## 开发一个代理式社交媒体营销团队

### 0. 项目目标

让我们通过以下逐步程序来扩展这个简单的 CrewAI 示例，创建一个社交媒体营销团队。这个团队将根据用户的兴趣主题生成博客文章，并为不同的社交媒体平台创建定制的活动信息。

![图片](img/c68461dc85fb658e6993e0f5ef6c6a6c.png)

*如果我们询问团队关于“代理人工智能”这一主题的输出示例。*

**博客文章**

![](img/576d2d762aaba2b90da7cc76f221876d.png)

**X(Twitter) 消息** `探索代理人工智能的未来！你是否曾想过到 2025 年代理人工智能将如何重新定义我们与技术互动的方式？理解 2025 年的 AI 趋势不仅对技术爱好者至关重要，对各个行业的商业企业来说也是必不可少的。#代理人工智能 #AI 趋势`

**YouTube 消息** `探索代理人工智能的突破性趋势！🌟 发现代理人工智能如何以前所未有的方式改变行业！到 2025 年，这些革命性的趋势将重塑我们与技术互动的方式，特别是在银行和金融领域。你准备好拥抱未来了吗？别忘了阅读我们最新的博客文章并订阅以获取更多见解！`

**Substack 消息** `代理人工智能的未来格局：2025 年你需要了解的内容，代理人工智能正在重塑行业，特别是在金融和银行业。本博客涵盖了关键趋势，如代理人工智能的变革潜力、新的监管框架和重大的技术进步。企业如何成功整合代理人工智能同时管理风险？这项技术的未来对消费者意味着什么？加入我们最新的帖子中的对话，让我们共同探讨这些创新将如何影响我们的未来！`

### 1. 项目环境设置

按照 CrewAI 的[快速入门](https://docs.crewai.com/en/quickstart)指南设置开发环境。我们为这个项目使用以下目录结构。

```py
├── README.md
├── pyproject.toml
├── requirements.txt
├── src
│   └── social_media_agent
│       ├── __init__.py
│       ├── crew.py
│       ├── main.py
│       └── tools
│           ├── __init__.py
│           ├── browser_tools.py
│           └── keyword_tool.py
└── uv.lock
```

### 2. 开发工具

我们首先为团队添加的工具是[`网站搜索工具`](https://docs.crewai.com/en/tools/search-research/websitesearchtool)，这是一个由 CrewAI 实现的内置工具，用于在网站内容中进行语义搜索。

我们只需要几行代码就可以安装 crewai 工具包并使用`WebsiteSearchTool`。这个工具可以通过市场研究代理访问，以找到与给定主题相关的最新市场趋势或行业新闻。

```py
pip install 'crewai[tools]'
```

```py
from crewai_tools import WebsiteSearchTool

web_search_tool = WebsiteSearchTool()
```

下面的截图显示了当输入“YouTube 视频”这一主题时，`web_search_tool`的输出结果。

![代理人工智能工具输出示例](img/f4c101ccd5fb8ab639a4d732319d7ae5.png)

接下来，我们将通过继承 BaseTool 类并使用[SerpApi (Google 趋势 API)](https://serpapi.com/)来创建一个自定义的`keyword_tool`。如下面的代码所示，这个工具生成与输入关键词相关的顶级搜索和趋势查询。这个工具可以通过营销专家代理访问，以调查趋势关键词并优化博客文章以进行搜索引擎优化。我们将在下一节中看到一个关键词工具输出的示例。

```py
import os
import json
from dotenv import load_dotenv
from crewai.tools import BaseTool
from serpapi.google_search import GoogleSearch

load_dotenv()
api_key = os.getenv('SERPAPI_API_KEY')

class KeywordTool(BaseTool):
    name: str = "Trending Keyword Tool"
    description: str = "Get search volume of related trending keywords."

    def _run(self, keyword: str) -> str:
        params = {
            'engine': 'google_trends',
            'q': keyword,
            'data_type': 'RELATED_QUERIES',
            'api_key': api_key
        }

        search = GoogleSearch(params)

        try:
            rising_kws = search.get_dict()['related_queries']['rising']
            top_kws = search.get_dict()['related_queries']['top']

            return f"""
                    Rising keywords: {rising_kws} \n 
                    Top keywords: {top_kws}
                """
        except Exception as e:

            return f"An unexpected error occurred: {str(e)}" 
```

### 3. 定义团队类

![CrewAI 架构](img/8a75999a210840f193361c7e7c709100.png)

在`crew.py`脚本中，我们定义了我们的社交媒体团队，包含三个代理——*市场研究员*、*内容创作者*、*市场营销专家*——并为每个代理分配任务。我们使用`@CrewBase`装饰器初始化`SocialMediaCrew()`类。`topic`属性传递用户关于他们感兴趣主题的输入，而`llm`、`model_name`属性指定了在整个团队工作流程中使用的默认模型。

```py
@CrewBase
class SocialMediaCrew():
    def __init__(self, topic: str):
        """
        Initialize the SocialMediaCrew with a specific topic.

        Args:
            topic (str): The main topic or subject for social media content generation
        """

        self.topic = topic
        self.model_name = 'openai/gpt-4o'
        self.llm = LLM(model=self.model_name,temperature=0)
```

### 4. 定义代理

CrewAI 代理依赖于三个基本参数——*角色、目标和背景故事*——来定义它们的特征以及它们操作的环境。此外，我们还为代理提供相关的`工具`以方便他们的工作，以及其他参数以控制调用代理的资源消耗，避免不必要的 LLM 令牌使用。

例如，我们使用以下代码定义了“市场营销专家代理”。从使用`@agent`装饰器开始。将角色定义为“市场营销专家”，并提供我们之前开发的`keyword_tool`的访问权限，以便市场营销专家可以研究趋势关键词，以优化博客内容，实现最佳的 SEO 性能。

*访问我们的[GitHub 仓库](https://github.com/destingong/social-media-agent)以获取其他代理定义的完整代码。*

```py
@CrewBase
class SocialMediaCrew():
    def __init__(self, topic: str):
        """
        Initialize the SocialMediaCrew with a specific topic.

        Args:
            topic (str): The main topic or subject for social media content generation
        """

        self.topic = topic
        self.model_name = 'openai/gpt-4o'
        self.llm = LLM(model=self.model_name,temperature=0)
```

将`verbose`设置为 true 允许我们利用 CrewAI 的可追溯性功能来观察代理调用过程中的中间输出。下面的截图显示了“市场营销专家”代理使用`keyword_tool`研究“YouTube 趋势”的思想过程，以及基于工具输出的 SEO 优化博客文章。

*市场营销专家的中间输出*

![图片 1](img/67bb784314ccd07c0f004aedfff35921.png)![图片 2](img/e6abd914d4b3ddc34be975c3a22bf328.png)

定义代理的另一种方法是使用以下格式将代理上下文存储在 YAML 文件中，提供额外的实验灵活性和在必要时迭代提示工程。

*示例* *agent.yaml*

```py
marketing_specialist: 
  role: >
   "Marketing Specialist"
  goal: >
   "Improve the blog post to optimize for Search Engine Optimization using the Keyword Tool and create customized, channel-specific messages for social media distributions"
  backstory: >
    "A skilled Marketing Specialist with expertise in SEO and social media campaign design"
```

### 5. 定义任务

如果一个代理被视为一个在特定领域（例如内容创作、研究）专业化的员工（“谁”），并具有角色或特征，那么任务就是员工执行的动作（“什么”），具有预定义的目标和输出预期。

在 CrewAI 中，任务是通过`description`、`expected_output`和可选参数`output_file`配置的，该参数将中间输出保存为文件。另外，也建议使用独立的 YAML 文件来提供更干净、可维护的方式来定义任务。在下面的代码片段中，我们为团队提供了执行四个任务的精确指令，并将它们分配给具有相关技能集的代理。我们还将在工作文件夹中保存每个任务的输出，以便于比较不同的输出版本。

+   `research`：研究给定主题的市场趋势；分配给市场研究员。

+   `write`：撰写一篇引人入胜的博客文章；分配给内容创作者。

+   `refine`: 优化博客文章以实现最佳 SEO 性能；分配给市场营销专员。

+   `distribute`: 生成适用于每个社交媒体渠道的特定平台消息；分配给市场营销专员。

```py
@task
def research(self) -> Task:
    return Task(
        description=f'Research the 2025 trends in the {self.topic} area and provide a summary.',
        expected_output=f'A summary of the top 3 trending news in {self.topic} with a unique perspective on their significance.',
        agent=self.market_researcher()
    )

@task
def write(self) -> Task:
    return Task(
        description=f"Write an engaging blog post about the {self.topic}, based on the research analyst's summary.",
        expected_output='A 4-paragraph blog post formatted in markdown with engaging, informative, and accessible content, avoiding complex jargon.',
        agent=self.content_creator(),
        output_file=f'blog-posts/post-{self.model_name}-{timestamp}.md' 
    )

@task
def refine(self) -> Task:
    return Task(
        description="""
            Refine the given article draft to be highly SEO optimized for trending keywords. 
            Include the keywords naturally throughout the text (especially in headings and early paragraphs)
            Make the content easy for both search engines and users to understand.
            """,
        expected_output='A refined 4-paragraph blog post formatted in markdown with engaging and SEO-optimized contents.',
        agent=self.marketing_specialist(),
        output_file=f'blog-posts/seo_post-{self.model_name}-{timestamp}.md' 
    )

@task
def distribute(self) -> Task: 
    return Task(
        description="""
            Generate three distinct versions of the original blog post description, each tailored for a specific distribution channel:
            One version for X (formerly Twitter) – concise, engaging, and hashtag-optimized.

            One version for YouTube post – suitable for video audience, includes emotive cue and strong call-to-action.

            One version for Substack – slightly longer, informative, focused on newsletter subscribers.

            Each description must be optimized for the norms and expectations of the channel, making subtle adjustments to language, length, and formatting.
            Output should be in markdown format, with each version separated by a clear divider (---).
            Use a short, impactful headline for each version to further increase channel fit.
        """,
        expected_output='3 versions of descriptions of the original blog post optimized for distribution channel, formatted in markdown, separated by dividers.',
        agent=self.marketing_specialist(),
        output_file=f'blog-posts/social_media_post-{self.model_name}-{timestamp}.md' 
    )
```

下面的 CrewAI 输出日志显示了任务执行细节，包括状态、代理分配和工具使用。

```py
🚀 Crew: crew
├── 📋 Task: research (ID: 19968f28-0af7-4e9e-b91f-7a12f87659fe)
│   Assigned to: Market Research Analyst
│   Status: ✅ Completed
│   └── 🔧 Used Search in a specific website (1)
├── 📋 Task: write (ID: 4a5de75f-682e-46eb-960f-43635caa7481)
│   Assigned to: Content Writer
│   Status: ✅ Completed
├── 📋 Task: refine (ID: fc9fe4f8-7dbb-430d-a9fd-a7f32999b861)
│   **Assigned to: Marketing Specialist**
│   Status: ✅ Completed
│   └── 🔧 Used Trending Keyword Tool (1)
└── 📋 Task: distribute (ID: ed69676a-a6f7-4253-9a2e-7f946bd12fa8)
    **Assigned to: Marketing Specialist**
    Status: ✅ Completed
    └── 🔧 Used Trending Keyword Tool (2)
╭───────────────────────────────────────── Task Completion ──────────────────────────────────────────╮
│                                                                                                    │
│  Task Completed                                                                                    │
│  Name: distribute                                                                                  │
│  Agent: Marketing Specialist                                                                       │
│  Tool Args:                                                                                        │
│                                                                                                    │
│                                                                                                    │
╰────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### 6. 启动团队

在 crew.py 脚本的最后一步，我们协调任务、代理和工具来定义团队。

```py
@crew
def crew(self) -> Crew:
    return Crew(
        agents=self.agents,
        tasks=self.tasks,
        verbose=True,
        planning=True,
    )
```

在 main.py 中，我们实例化一个`SocialMediaCrew()`对象，并通过接受用户对其感兴趣的主题的输入来运行团队。

```py
# main.py
from social_media_agent.crew import SocialMediaCrew

def run():
    # Replace with your inputs, it will automatically interpolate any tasks and agents information
    inputs = {
        'topic': input('Enter your interested topic: '),
    }
    SocialMediaCrew(topic=inputs['topic']).crew().kickoff(inputs=inputs)
```

现在，让我们以“代理式 AI”为例，以下是我们的社交媒体团队在依次执行任务后生成的输出文件。

*“write”任务的输出*

![“write”任务输出](img/bbc5655ebef810d58d4b46640dfd8640.png)

*“refine”任务的输出*

![图片](img/1b28ed088846a81054f8edf3db99db2c.png)

*“distribute”任务的输出*

![图片](img/1bb48f3db0a43d488ee51b8aa0a5a1c7.png)

* * *

## 要点

本教程演示了如何使用 CrewAI 框架创建一个代理式 AI 系统。通过协调具有不同角色和工具的专用代理，我们实现了一个多代理团队，能够为不同的社交媒体平台生成优化的内容。

+   **环境设置**：使用必要的依赖项和工具初始化您的开发环境以进行 CrewAI 开发。

+   **开发工具**：开发内置和自定义工具组件的基础工具结构。

+   **定义代理**：创建具有明确定义的角色、目标和背景故事的专用代理。为他们配备相关的工具以支持其角色。

+   **创建任务**：创建具有清晰描述和预期输出的任务。为任务执行分配代理。

+   **启动团队**：将任务、代理和工具作为一个团队协调在一起并执行工作流程。

### 更多类似资源
