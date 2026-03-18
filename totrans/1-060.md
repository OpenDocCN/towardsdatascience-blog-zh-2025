# Anthropic 新结构化输出功能的实战指南

> 原文：[`towardsdatascience.com/hands-on-with-anthropics-new-structured-output-capabilities/`](https://towardsdatascience.com/hands-on-with-anthropics-new-structured-output-capabilities/)

<mdspan datatext="el1764013594304" class="mdspan-comment">Anthropic 最近</mdspan>宣布在其 API 中的顶级模型支持结构化输出，这是一个旨在确保模型生成的输出与开发者提供的 JSON 模式完全匹配的新功能。

这解决了许多开发者面临的问题，即系统或流程消耗 LLM 的输出进行进一步处理时，该系统必须“知道”其输入的内容，以便相应地处理它。

类似地，当向用户显示模型输出时，你希望每次都以相同的格式显示。

到目前为止，确保 Anthropic 模型输出格式的一致性一直是一个痛点。然而，现在看起来 Anthropic 已经为其顶级模型解决了这个问题。根据他们的公告（文章末尾有链接），他们说，

> Claude 开发者平台现在支持 Claude Sonnet 4.5 和 Opus 4.1 的结构化输出。该功能目前处于公开测试版，确保 API 响应始终与开发者指定的 JSON 模式或工具定义相匹配。

在我们查看一些示例代码之前，有一点需要记住，Anthropic 保证模型的输出将遵循指定的格式，*而不是*任何输出都将 100%准确。模型可能会偶尔出现幻觉。

因此，你可能会得到格式正确但内容错误的答案！

## 设置我们的开发环境

在我们查看一些 Python 代码示例之前，最佳实践是创建一个独立的发展环境，在那里你可以安装任何必要的软件并进行编码实验。现在，你在这个环境中所做的任何操作都将被隔离，并且不会影响你的其他项目。

我将使用 Miniconda 来完成这个任务，但你可以使用你最熟悉的方法。

如果你想要走 Miniconda 路线并且还没有安装它，你必须首先安装它。你可以通过这个链接获取它：

[`docs.anaconda.com/miniconda/ `](https://docs.anaconda.com/miniconda/ )

要跟随我的示例，你还需要一个 Anthropic API 密钥和账户上的一些信用额度。为了参考，我在这篇文章中使用了 12 美分来运行代码。如果你已经有了 Anthropic 账户，你可以通过 Anthropic 控制台在[`console.anthropic.com/settings/keys`](https://console.anthropic.com/settings/keys)获取 API 密钥。

**1/ 创建我们的新开发环境并安装所需的库**

<mdspan datatext="el1764013452233" class="mdspan-comment">我在 WSL2 Ubuntu for Windows 上运行这个</mdspan>

```py
(base) $ conda create -n anth_test python=3.13 -y 
(base) $ conda activate anth_test 
(anth_test) $ pip install anthropic beautifulsoup4 requests 
(anth_test) $ pip install httpx jupyter 
```

**2/ 启动 Jupyter**

现在在命令提示符中输入‘jupyter notebook’。你应该会在浏览器中看到一个 jupyter 笔记本打开。如果它没有自动打开，你可能会在命令后看到一屏的信息。在底部附近，你会找到一个可以复制并粘贴到浏览器中的 URL。它看起来可能像这样：

`http://127.0.0.1:8888/tree?token=3b9f7bd07b6966b41b68e2350721b2d0b6f388d248cc69`

## 代码示例

在我们的两个编码示例中，我们将使用 beta API 中可用的 new output_format 参数。当指定结构化输出时，我们可以使用两种不同的样式。

1. 原始 JSON 模式。

正如其名所示，该结构由一个 JSON 模式块定义，该模式块直接传递到输出格式定义中。

2. Pydantic 模型类。

这是一个使用 Pydantic 的 BaseModel 的常规 Python 类，它指定了模型要输出的数据。与 JSON 模式相比，这是一种定义结构更紧凑的方法。

**示例代码 1 — 文本摘要**

如果你有一堆不同的文本需要总结，但又希望总结具有相同的结构，这会很有用。在这个例子中，我们将处理一些著名科学家的维基百科条目，并以高度组织化的方式检索他们的一些具体关键事实。

在我们的总结中，我们希望为每位科学家输出以下结构，

+   **科学家的名字**

+   **他们出生的时间和地点**

+   **他们的主要成就**

+   **他们获得诺贝尔奖的年份**

+   **他们去世的时间和地点**

> 注意：维基百科中的大多数文本（不包括引文）已根据 Creative Commons Attribution-Sharealike 4.0 国际许可协议（CC-BY-SA）和 GNU 自由文档许可协议（GFDL）发布。简而言之，这意味着你可以自由地：
> 
> **分享** — 以任何媒体或格式复制和重新分发材料
> 
> **改编** — 混合、转换和基于材料进行构建
> 
> 用于任何目的，包括商业用途。

让我们将代码分解成可管理的部分，每个部分都有解释。

首先，我们导入所需的第三方库，并使用我们的 API 密钥设置与 Anthropic 的连接。

```py
import anthropic
import httpx
import requests
import json
import os
from bs4 import BeautifulSoup

http_client = httpx.Client()
api_key = 'YOUR_API_KEY'
client = anthropic.Anthropic(
    api_key=api_key,
    http_client=http_client
)
```

这是将为我们从维基百科上抓取数据的函数。

```py
def get_article_content(url):
    try:
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'}
        response = requests.get(url, headers=headers)
        soup = BeautifulSoup(response.content, "html.parser")
        article = soup.find("div", class_="mw-body-content")
        if article:
            content = "\n".join(p.text for p in article.find_all("p"))
            return content[:15000] 
        else:
            return ""
    except Exception as e:
        print(f"Error scraping {url}: {e}")
        return ""
```

接下来，我们定义我们的 JSON 模式，它指定了模型输出的确切格式。

```py
summary_schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string", "description": "The name of the Scientist"},
        "born": {"type": "string", "description": "When and where the scientist was born"},
        "fame": {"type": "string", "description": "A summary of what their main claim to fame is"},
        "prize": {"type": "integer", "description": "The year they won the Nobel Prize. 0 if none."},
        "death": {"type": "string", "description": "When and where they died. 'Still alive' if living."}
    },
    "required": ["name", "born", "fame", "prize", "death"],
    "additionalProperties": False
}
```

此函数作为我们的 Python 脚本和 Anthropic API 之间的接口。其主要目标是获取非结构化文本（一篇文章），并强制 AI 返回一个包含特定字段的结构化数据对象（JSON），例如科学家的名字、出生日期和诺贝尔奖详情。

函数调用 client.messages.create 向模型发送请求。它将温度设置为 0.2，这降低了模型的创造力，以确保提取的数据是事实性和精确的。extra_headers 参数启用了一个尚未成为标准的特定 beta 功能。通过传递具有值 structured-outputs-2025-11-13 的 anthropic-beta 头，代码告诉 API 激活针对此特定请求的结构化输出逻辑，迫使它产生与您定义的结构匹配的有效 JSON。

由于使用了 output_format 参数，模型返回的是一个保证有效的原始字符串。这行代码 json.loads(response.content[0].text)将这个字符串解析成一个原生的 Python 字典，使得数据立即可用于程序化使用。

```py
def get_article_summary(text: str):
    if not text: return None

    try:
        response = client.messages.create(
            model="claude-sonnet-4-5", # Use the latest available model
            max_tokens=1024,
            temperature=0.2,
            messages=[
                {"role": "user", "content": f"Summarize this article:\n\n{text}"}
            ],
            # Enable the beta feature
            extra_headers={
                "anthropic-beta": "structured-outputs-2025-11-13"
            },
            # Pass the new parameter here
            extra_body={
                "output_format": {
                    "type": "json_schema",
                    "schema": summary_schema
                }
            }
        )

        # The API returns the JSON directly in the text content
        return json.loads(response.content[0].text)

    except anthropic.BadRequestError as e:
        print(f"API Error: {e}")
        return None
    except Exception as e:
        print(f"Error: {e}")
        return None
```

这是我们将所有东西整合在一起的地方。我们定义了我们想要抓取的各种 URL。在显示最终结果之前，它们的内文被传递给模型进行处理。

```py
urls = [
    "https://en.wikipedia.org/wiki/Albert_Einstein",
    "https://en.wikipedia.org/wiki/Richard_Feynman",
    "https://en.wikipedia.org/wiki/James_Clerk_Maxwell",
    "https://en.wikipedia.org/wiki/Alan_Guth"
]

print("Scraping and analyzing articles...")

for i, url in enumerate(urls):
    print(f"\n--- Processing Article {i+1} ---")
    content = get_article_content(url)

    if content:
        summary = get_article_summary(content)
        if summary:
            print(f"Scientist: {summary.get('name')}")
            print(f"Born:      {summary.get('born')}")
            print(f"Fame:      {summary.get('fame')}")
            print(f"Nobel:     {summary.get('prize')}")
            print(f"Died:      {summary.get('death')}")
        else:
            print("Failed to generate summary.")
    else:
        print("Skipping (No content)")

print("\nDone.")
```

当我运行上述代码时，我得到了这个输出。

```py
Scraping and analyzing articles...

--- Processing Article 1 ---
Scientist: Albert Einstein
Born:      14 March 1879 in Ulm, Kingdom of Württemberg, German Empire
Fame:      Developing the theory of relativity and the mass-energy equivalence formula E = mc2, plus contributions to quantum theory including the photoelectric effect
Nobel:     1921
Died:      18 April 1955

--- Processing Article 2 ---
Scientist: Richard Phillips Feynman
Born:      May 11, 1918, in New York City
Fame:      Path integral formulation of quantum mechanics, quantum electrodynamics, Feynman diagrams, and contributions to particle physics including the parton model
Nobel:     1965
Died:      February 15, 1988

--- Processing Article 3 ---
Scientist: James Clerk Maxwell
Born:      13 June 1831 in Edinburgh, Scotland
Fame:      Developed the classical theory of electromagnetic radiation, unifying electricity, magnetism, and light through Maxwell's equations. Also key contributions to statistical mechanics, color theory, and numerous other fields of physics and mathematics.
Nobel:     0
Died:      5 November 1879

--- Processing Article 4 ---
Scientist: Alan Harvey Guth
Born:      February 27, 1947 in New Brunswick, New Jersey
Fame:      Pioneering the theory of cosmic inflation, which proposes that the early universe underwent a phase of exponential expansion driven by positive vacuum energy density
Nobel:     0
Died:      Still alive

Done.
```

不错！艾伦·古斯会很高兴他还活着，但遗憾的是，他还没有赢得诺贝尔奖。此外，请注意，詹姆斯·克拉克·麦克斯韦在诺贝尔奖开始运作之前就已经去世了。

**示例代码 2 — 自动化代码安全与重构代理**。

这是一个完全不同的用例，一个非常实用的软件工程示例。通常，当你要求一个 LLM“修复代码”时，它会给你一个包含代码块的对话式响应。这使得它难以集成到 CI/CD 管道或 IDE 插件中。

通过使用结构化输出，我们可以迫使模型在一个单一的、机器可读的 JSON 对象中返回**干净的代码**、**找到的具体错误列表**和**安全风险评估**。

**场景**

我们将向模型提供一个包含危险的 SQL 注入漏洞和糟糕的编码实践的 Python 函数。模型必须识别特定的缺陷并安全地重写代码。

```py
import anthropic
import httpx
import os
import json
from pydantic import BaseModel, Field, ConfigDict
from typing import List, Literal

# --- SETUP ---
http_client = httpx.Client()
api_key = 'YOUR_API_KEY'
client = anthropic.Anthropic(api_key=api_key, http_client=http_client)

# Intentionally bad code
bad_code_snippet = """
import sqlite3

def get_user(u):
    conn = sqlite3.connect('app.db')
    c = conn.cursor()
    # DANGER: Direct string concatenation
    query = "SELECT * FROM users WHERE username = '" + u + "'"
    c.execute(query)
    return c.fetchall()
"""

# --- DEFINE SCHEMA WITH STRICT CONFIG ---
# We add model_config = ConfigDict(extra="forbid") to ensure 
# "additionalProperties": false is generated in the schema.

class BugReport(BaseModel):
    model_config = ConfigDict(extra="forbid") 

    severity: Literal["Low", "Medium", "High", "Critical"]
    line_number_approx: int = Field(description="The approximate line number where the issue exists.")
    issue_type: str = Field(description="e.g., 'Security', 'Performance', 'Style'")
    description: str = Field(description="Short explanation of the bug.")

class CodeReviewResult(BaseModel):
    model_config = ConfigDict(extra="forbid") 

    is_safe_to_run: bool = Field(description="True only if no Critical/High security risks exist.")
    detected_bugs: List[BugReport]
    refactored_code: str = Field(description="The complete, fixed Python code string.")
    explanation: str = Field(description="A brief summary of changes made.")

# --- API CALL ---
try:
    print("Analyzing code for security vulnerabilities...\n")

    response = client.messages.create(
        model="claude-sonnet-4-5", 
        max_tokens=2048,
        temperature=0.0,
        messages=[
            {
                "role": "user", 
                "content": f"Review and refactor this Python code:\n\n{bad_code_snippet}"
            }
        ],
        extra_headers={
            "anthropic-beta": "structured-outputs-2025-11-13"
        },
        extra_body={
            "output_format": {
                "type": "json_schema",
                "schema": CodeReviewResult.model_json_schema()
            }
        }
    )

    # Parse Result
    result = json.loads(response.content[0].text)

    # --- DISPLAY OUTPUT ---
    print(f"Safe to Run: {result['is_safe_to_run']}")
    print("-" * 40)

    print("BUGS DETECTED:")
    for bug in result['detected_bugs']:
        # Color code the severity (Red for Critical)
        prefix = "🔴" if bug['severity'] in ["Critical", "High"] else "🟡"
        print(f"{prefix} [{bug['severity']}] Line {bug['line_number_approx']}: {bug['description']}")

    print("-" * 40)
    print("REFACTORED CODE:")
    print(result['refactored_code'])

except anthropic.BadRequestError as e:
    print(f"API Schema Error: {e}")
except Exception as e:
    print(f"Error: {e}")
```

这段代码充当一个自动化的安全审计员。它不是让 AI“聊天”关于代码，而是强迫 AI 填写一个包含关于错误和安全风险的特定细节的严格、数字表格。

这就是它在三个简单步骤中的工作方式。

1.  首先，代码使用 Python 类和 Pydantic 结合定义了答案必须呈现的确切样子。它告诉 AI：“给我一个包含错误列表的 JSON 对象，每个错误都有一个严重性评级（如‘关键’或‘低’），以及修复后的代码字符串。”

1.  在将易受攻击的代码发送到 API 时，它使用 output_format 参数传递 Pydantic 蓝图。这严格限制了模型，防止它产生幻觉或添加对话填充。它*必须*返回与您的蓝图匹配的有效数据。

1.  该脚本接收 AI 的响应，该响应保证是可机器读取的 JSON 格式。然后它会自动解析这些数据以显示一个干净的报告，例如将 SQL 注入标记为“关键”问题，并打印出安全、重构后的代码版本。

这是运行代码后我收到的输出。

```py
Analyzing code for security vulnerabilities...

Safe to Run: False
----------------------------------------
BUGS DETECTED:
🔴 [Critical] Line 7: SQL injection vulnerability due to direct string concatenation in query construction. Attacker can inject malicious SQL code through the username parameter.
🟡 [Medium] Line 4: Database connection and cursor are not properly closed, leading to potential resource leaks.
🟡 [Low] Line 1: Function parameter name 'u' is not descriptive. Should use meaningful variable names.
----------------------------------------
REFACTORED CODE:
import sqlite3
from contextlib import closing

def get_user(username):
    """
    Retrieve user information from the database by username.

    Args:
        username (str): The username to search for

    Returns:
        list: List of tuples containing user data, or empty list if not found
    """
    with sqlite3.connect('app.db') as conn:
        with closing(conn.cursor()) as cursor:
            # Use parameterized query to prevent SQL injection
            query = "SELECT * FROM users WHERE username = ?"
            cursor.execute(query, (username,))
            return cursor.fetchall()
```

**为什么这很强大？**

**集成就绪**。您可以在 GitHub Action 中运行此脚本。如果 is_safe_to_run 为 False，您可以自动阻止拉取请求。

**关注点分离**。您将元数据（错误、严重性）与内容（代码）分开。您不需要使用正则表达式从响应中删除“这里是您的修复代码”文本。

**严格的类型检查**。严重性字段被限制在特定的枚举值（关键、高等等）中，确保当模型返回“严重”而不是“关键”等值时，您的下游逻辑不会出错。

## 摘要

Anthropic 推出的原生结构化输出对需要可靠性的开发者来说是一个变革，而不仅仅是对话。通过强制执行严格的 JSON 架构，我们现在可以将大型语言模型视为更类似于确定性软件组件，而不是聊天机器人。

在这篇文章中，我展示了如何使用这个新的测试版功能来简化数据提取和输出，并构建与 Python 代码无缝集成的自动化工作流程。如果您是 Anthropic API 的用户，编写易碎的正则表达式来解析 AI 响应的日子终于结束了。

有关此新测试版功能的更多信息，请点击下面的链接访问 Anthropics 的官方文档页面。

[`platform.claude.com/docs/en/build-with-claude/structured-outputs`](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)
