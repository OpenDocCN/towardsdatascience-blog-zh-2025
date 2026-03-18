# 将数据转化为解决方案：使用 Python 和 AI 构建智能应用

> 原文：[`towardsdatascience.com/transforming-data-into-solutions-building-a-smart-app-with-python-and-ai-c258be3b1fc2/`](https://towardsdatascience.com/transforming-data-into-solutions-building-a-smart-app-with-python-and-ai-c258be3b1fc2/)

一些金融分析师担心，人工智能可能无法证明在该领域进行的巨额投资是合理的。虽然我理解他们的担忧，但我有不同的看法。我既不是 AI Boomer 也不是 AI Doomer——我相信 AI 有潜力推动创新，提高生产力，并带来可衡量的商业成果。

在我上一篇文章中，我探讨了如何使用大型语言模型（LLMs）来结构化非结构化数据。这次，我想更进一步：展示如何通过 LLMs 结构化数据的结果可以作为构建智能应用的**基础**。从而展示如何将 AI 融入更大的图景中。

在这篇文章中，我将分享我是如何使用现代堆栈快速推进 Baker 的开发和部署的——一个智能应用，它是将原始食谱数据集转化为易于使用解决方案的结果。这次旅程不仅突出了技术实现，还展示了 AI 如何解决实际挑战，并在现实场景中创造可衡量的价值。

* * *

## 烘焙师：你的烹饪灵感

在 **[LLMs（较少为人知的）新兴应用](https://towardsdatascience.com/the-lesser-known-rising-application-of-llms-775834116477)** 中，我提到我需要一个食谱数据集来工作于一个个人项目。现在，是时候揭示这个项目了。

![由 WebFactory Ltd 在 Unsplash 提供的照片](img/debd34294a7e10638647b8f2a89372cf.png)

照片由 [WebFactory Ltd](https://unsplash.com/@webfactoryltd?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 提供

管理食物对我来说一直是一个挑战。我很难找到餐点的灵感，结果，我经常让食材浪费——这是我长期以来想要改变的事情。这就是为什么我着手创建一个食谱推荐系统，帮助我（以及其他人）在食材变质之前用完它们。解决方案？**烘焙师**，我解决这个问题的原型。这个项目反映了我利用 AI 解决日常挑战，如**食物浪费**的热情。

Baker 是一个处于早期阶段的开源网络应用，几乎完全用 Python 构建。该应用接受一系列食材及其数量——模仿你可能在冰箱和储藏室中找到的东西——并建议你可以使用这些食材准备的食谱。它旨在简化餐点准备，同时鼓励更智能、更可持续的食物选择。你可以在以下链接尝试该应用：

⇒ [mixit-baker.streamlit.app](https://mixit-baker.streamlit.app/)

然而，你可能需要先阅读文章的剩余部分。在接下来的一个部分中，我将向您演示该应用的演示。

***

## 从想法到 POC：利用人工智能和现代工具加速开发

在对数据集进行初步解析之后，我忙于其他职责，并将这个副项目搁置了数月 😊。但技术发展迅速。当我最终回归时，一系列新的模型、技术和工具已经出现或成熟，这为重新审视和提升项目提供了机会。

**使用 GPT-4o 重新审视数据提取**

第一步是回顾我之前的数据提取工作并[更新结果](https://github.com/VianneyMI/baker/pull/10)。在我之前的迭代中，我使用了**MistralAI 的 open-mixtral-8x7b**，该工具已经过时。这次，我切换到了更先进的新工具**GPT-4o**，结果令人瞩目。

为了将改进置于正确的视角：

+   在早期的运行中，由于 JSON 生成的不一致性，LLM 未能解析 89 个食谱。

+   使用 GPT-4o，数据集中的所有 360 个食谱都成功解析，实现了**100%的成功率**。

这个里程碑反映了人工智能能力提高的速度。仅仅几个月前，大型语言模型（LLMs）往往难以可靠地输出有效的 JSON。这次迭代不仅展示了更好的结果，还展示了采用新工具如何带来实质性的收益。

**后处理挑战**

即使提取效果有所改进，原始数据仍需要大量的后处理。食谱中包含不一致的单位——克、茶匙、杯等等，这使得比较和推荐变得具有挑战性。为了解决这个问题，我采用了系统性的方法：

1.  **标准化**：我定义了一个受限的单位集合（例如，克用于重量，毫升用于体积）。

1.  **映射和转换**：所有原始单位都映射到这个子集，并相应调整数量。

这个标准化步骤对于实现准确的成分过滤和推动食谱推荐引擎至关重要。它还强调了关键点：人工智能的输出必须对其预期应用具有上下文意义，而不仅仅是技术上正确。

**构建引擎和 Web 应用**

数据清理、标准化并准备好使用后，下一步是开始利用这些数据，我通过[构建引擎逻辑并将其封装在 Web 应用中](https://github.com/VianneyMI/baker/pull/18)来实现这一点。

**快速原型设计**

在 Baker 的开发过程中，我秉持了快速原型设计的理念。目标不是从第一天就构建一个应用，而是探索工具和技术，测试想法并收集反馈。

**从笔记本到 POC 应用**：

我构建数据管道和 Web 应用的方式，以及使用人工智能的方式，体现了快速原型设计的理念。

+   **数据管道：** 整个提取和后处理的管道都位于 Jupyter 笔记本中。与完整的 ETL 管道相比，这种设置虽然“最小化但功能性强”，但它提供了快速迭代的速度和灵活性。最初，我认为这是一个一次性过程，但现在我计划将其[转换](https://github.com/VianneyMI/baker/issues/22)成一个更稳健且可重复的过程。

+   **Web 应用程序：** 该 Web 应用程序后端使用 **[FastAPI](https://fastapi.tiangolo.com/)**，前端使用 **[Streamlit](https://streamlit.io/)**。这些框架易于访问，对开发者友好，非常适合快速原型设计交互式应用程序。

+   **使用 AI 加速开发：** 前端完全使用 **[Cursor](https://www.cursor.com/)**，一个 AI 驱动的开发工具生成。虽然我之前听说过 Cursor，但这个项目让我能够充分探索其潜力。体验非常愉快，我计划写一篇关于它的专门文章。

采用这种方法，我能够在几天内而不是几周或几个月内构建一个可工作的 MVP。

**开源：**

**Baker** 的一个标志是其 **开源特性**。通过在 GitHub 上分享项目，我希望能够：

+   **鼓励协作：使其他人能够贡献新功能、增强代码库或通过附加数据集扩展应用程序。**

+   **激发学习：为那些对利用 LLM、结构化非结构化数据或构建推荐系统感兴趣的人提供实用资源。**

开源使得其他人更容易复制结果、贡献改进和交流想法。协作不仅加强了 **Baker**，还促进了集体创新。

**免费部署：易于访问且轻量级**

为了使 **Baker** 易于访问，我使用免费层服务进行了部署：

+   **[Streamlit Cloud](https://streamlit.io/cloud)**：提供前端，提供直观且交互式的用户体验。

+   **[Koyeb](https://www.koyeb.com/docs)**：支持后端处理和 API 调用，无需承担托管成本。

这些平台使我能够快速部署并实验，无需承担传统托管解决方案的财务或技术障碍。这种部署策略突出了现代工具如何使创意想法以低至零的成本变为可访问的应用程序。

* * *

## **幕后：推动 Baker 的技术**

在本节中，我将分享 Baker 的一些实现细节。再次强调，它是开源的，所以我邀请我的技术读者去 GitHub 上查看代码。一些读者可能想要跳转到 下一节。

该应用程序采用极简设计，具有简单的三层架构，并且几乎完全使用 Python 构建。

![作者照片](img/427a5998ef5e833398ed534291d51899.png)

作者照片

它由以下组件组成：

1.  **前端**: Streamlit 接口为用户提供了一个直观的平台，用于与系统交互、查询食谱和接收推荐。

1.  **后端**: 使用 **FastAPI** 构建，后端作为处理用户查询和提供推荐的接口。

1.  **引擎**: 引擎包含查找和过滤食谱的核心逻辑，利用 **[monggregate](https://vianneymi.github.io/monggregate/)** 作为查询构建器。

1.  **数据库**: 食谱存储在一个 **[MongoDB](https://www.mongodb.com/)** 数据库中，该数据库处理由引擎生成的聚合管道。

后端设置

后端在 `app.py` 文件中初始化，其中定义了 FastAPI 端点。例如：

```py
from fastapi import FastAPI
from baker.engine.core import find_recipes
from baker.models.ingredient import Ingredient

app = FastAPI()
@app.get("/")
def welcome():
    return {"message": "Welcome to the Baker API!"}
@app.post("/recipes")
def _find_recipes(ingredients: list[Ingredient], serving_size: int = 1) -> list[dict]:
    return find_recipes(ingredients, serving_size)
```

`/recipes` 端点接受一个食材列表和一个份量，然后将处理委托给引擎。

食谱引擎逻辑

应用程序的核心位于 `engine` 目录下的 `core.py` 文件中。它管理数据库连接和查询管道。以下是 `find_recipes` 函数的示例：

```py
# Imports and the get_recipes_collection function are not included

def find_recipes(ingredients, serving_size=1):
    # Get the recipes collection
    recipes = get_recipes_collection()

    # Create the pipeline
    pipeline = Pipeline()
    pipeline = include_normalization_steps(pipeline, serving_size)
    query = generate_match_query(ingredients, serving_size)
    print(query)
    pipeline.match(query=query).project(
        include=[
            "id",
            "title",
            "preparation_time",
            "cooking_time",
            "original_serving_size",
            "serving_size",
            "ingredients",
            "steps",
        ],
        exclude="_id",
    )

    # Find the recipes
    result = recipes.aggregate(pipeline.export()).to_list(length=None)

    return result

def generate_match_query(ingredients: list[Ingredient], serving_size: int = 1) -> dict:
    """Generate the match query."""

    operands = []
    for ingredient in ingredients:
        operand = {
            "ingredients.name": ingredient.name,
            "ingredients.unit": ingredient.unit,
            "ingredients.quantity": {"$lte": ingredient.quantity / serving_size},
        }
        operands.append(operand)

    query = {"$and": operands}

    return query

def include_normalization_steps(pipeline: Pipeline, serving_size: int = 1):
    """Adds steps in a pipeline to normalize the ingredients quantity in the db

    The steps below normalize the quantities of the ingredients in the recipes in the DB by the recipe serving size.

    """

    # Unwind the ingredients
    pipeline.unwind(path="$ingredients")

    pipeline.add_fields({"original_serving_size": "$serving_size"})
    # Add the normalized quantity
    pipeline.add_fields(
        {
            # "orignal_serving_size": "$serving_size",
            "serving_size": serving_size,
            "ingredients.quantity": S.multiply(
                S.field("ingredients.quantity"),
                S.divide(serving_size, S.max([S.field("serving_size"), 1])),
            ),
        }
    )

    # Group the results
    pipeline.group(
        by="_id",
        query={
            "id": {"$first": "$id"},
            "title": {"$first": "$title"},
            "original_serving_size": {"$first": "$original_serving_size"},
            "serving_size": {"$first": "$serving_size"},
            "preparation_time": {"$first": "$preparation_time"},
            "cooking_time": {"$first": "$cooking_time"},
            # "directions_source_text": {"$first": "$directions_source_text"},
            "ingredients": {"$addToSet": "$ingredients"},
            "steps": {"$first": "$steps"},
        },
    )
    return pipeline
```

**Baker** 的核心逻辑位于 `find_recipes` 函数中。

此函数通过 monggregate 创建一个 MongoDB 聚合管道。这个聚合管道包括几个步骤。

第一步由 `include_normalization_steps` 函数生成，该函数将动态更新数据库中食材的数量，以确保我们是在比较相同的东西。这是通过将数据库中的食材数量更新到用户期望的份量来完成的。

然后通过 `generate_match_query` 函数创建实际的匹配逻辑。我们确保食谱不需要比用户拥有的相关食材更多的东西。

最后，一个投影过滤掉我们不需要返回的字段。

## 用户指南：使用 Baker 几步即可发现食谱

**Baker** 帮助您为您的食材找到更好的归宿，通过找到与您家中已有的食材相匹配的食谱。

应用程序具有一个简单的基于表单的界面。输入您拥有的食材，指定它们的数量，并从提供的选项中选择计量单位。

![作者照片](img/e19f282e22dadb27a6bbae9a5d629924.png)

作者照片

在上面的示例中，我正在寻找两人份的食谱，以使用掉厨房里放置时间过长的 4 个番茄和 2 根胡萝卜。

**Baker** 找到了两个食谱！点击食谱可以查看完整详情。

![作者照片](img/400c614a69918c28d07f950a51a97d56.png)

作者照片

**Baker** 会根据您设置的份量调整食谱中的数量。例如，如果您将份量从两人份调整到四人份，应用会相应地重新计算食材数量。

更新份量大小也可能改变显示的食谱。**Baker** 确保建议的食谱不仅匹配份量大小，还匹配您手头的成分和数量。例如，如果您只有 4 个番茄和 2 个胡萝卜为两个人准备，**Baker** 将避免推荐需要 4 个番茄和 4 个胡萝卜的食谱。

* * *

## 前路漫漫：扩展 Baker 以实现更广泛的影响

Baker 是一个快速且实用地开发的副项目，它是一个功能原型，而不是一个生产就绪的应用程序。

然而，我希望能看到它发展成为更有影响力的东西。

应用程序可以在功能和复杂性方面扩展的几个领域。为了前进，让我们解决其当前的局限性并探索可能的增强。

## 1 – 扩展食谱数据库

Baker 的食谱来源于 [公共领域食谱](https://publicdomainrecipes.com/)，这是一个开源的食谱数据库。虽然这是一个令人惊叹的项目，但可用的食谱数量有限。因此，目前 **Baker** 只知道 360 道食谱。为了更好地理解这一点，一些专门的食谱网站声称拥有数万道食谱。

为了扩展，我需要识别额外的食谱数据源。

## 2 – 改进数据摄取管道

目前，**数据摄取**是通过 Jupyter 笔记本处理的。主要技术优先事项之一是将这些笔记本转换为健壮的 Python 脚本，并将它们集成到 **Baker** 的管理部分，或者甚至将其分离成一个专用的应用程序。

此外，我相信还有改进 LLMs 执行的解析过程的余地，以确保更高的一致性和准确性。

## 3 – 添加语义搜索和灵活性

目前，食谱是通过精确字符串匹配通过成分名称进行搜索的。例如，搜索 "番茄" 不会检索提到 "tomatoes" 的食谱。这可以通过 [使用向量数据库实现语义搜索方法](https://vianmixt.notion.site/Practical-Semantic-Search-with-MongoDB-and-OpenAI-451692801b41465fae1bea5f70238279) 来解决。通过 **语义搜索**，成分名称的变化——无论是由于复数形式、地区差异还是翻译——可以动态映射到相同的概念。

此外，语义搜索将允许用户用他们的母语进行搜索，进一步扩大可访问性。

我还希望引入 **搜索功能的更大灵活性**。例如，如果用户搜索包含番茄、胡萝卜和香蕉的食谱，但数据库中没有这样的组合，搜索结果仍然应该返回包含输入成分子集的食谱（例如，番茄和胡萝卜，番茄和香蕉，或胡萝卜和香蕉）。

从功能角度来看，这是我的首要任务。

## 其他可以考虑的增强功能

虽然上述是主要关注领域，但我还想解决许多其他主题：

+   用户界面增强

+   食谱的排序和排名

+   改进的错误处理和健壮性

+   采用软件工程最佳实践

+   性能优化

* * *

## 结论

我希望您喜欢阅读这个将想法变为现实的过程。如果您喜欢，请考虑在[Buy Me a Coffee](https://buymeacoffee.com/vianmixt)上支持我，以帮助资助未来的发展。现在，让我们总结一下关键要点。

几个月前，我写了一篇文章，关于如何利用新的生成式 AI 浪潮，**使用 LLM 进行数据结构和提取**是一个有前景的应用案例。在这篇后续文章中，我通过展示一个具体的成果进一步发展了这个概念：使用由 LLM 生成的数据集来驱动一个以数据为中心的智能应用，**Baker**。

在这个过程中，我旨在展示如何通过现代工具如**Streamlit、FastAPI、Streamlit Cloud、Koyeb 和 Cursor**使这个过程变得更加容易接触。如果您想深入了解我使用的工具，请在评论中告诉我。

这些工具提供了所需的灵活性和可访问性，以便专注于解决实际问题，而不是被管理多个方面的复杂性所淹没——数据管道、后端逻辑、前端界面、CI/CD 工作流程、云部署、测试以及各种编程语言的复杂性。

但这个项目不仅仅是关于食谱——它是关于释放非结构化数据的潜力来解决现实世界的问题。例如，在**Baker**中使用的相同方法可以应用于创建一个个性化的学习平台。通过将教育内容，如讲座记录或文章，结构化成可访问的格式，并构建一个推荐引擎，可以帮助用户根据他们的目标或兴趣发现相关的学习材料。

不论是用于教育、产品推荐还是知识管理，我在这里分享的原则和工具可以激发无数的应用。

当我们步入新的一年，我鼓励您反思自己未开发的数据库和想法。您可以用这里分享的工具和知识构建什么？通过小步骤、快速迭代和协作心态，您也可以创建出**创新解决方案**，产生影响。

让我们一起为构建一个更智能、更数据驱动的未来干杯。
