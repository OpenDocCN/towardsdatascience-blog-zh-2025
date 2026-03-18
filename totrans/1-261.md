# 使用 DataHub 动作将 DataHub 集成到 Jira：实用指南

> 原文：[`towardsdatascience.com/integrating-datahub-into-jira-a-practical-guide-using-datahub-actions/`](https://towardsdatascience.com/integrating-datahub-into-jira-a-practical-guide-using-datahub-actions/)

<mdspan datatext="el1758220114298" class="mdspan-comment">在这篇文章中</mdspan>，我们将介绍如何使用 [DataHub](https://docs.datahub.com/docs/features) 动作框架将 [DataHub](https://docs.datahub.com/docs/actions) 事件集成到 Jira 工作流程中。在深入探讨之前，我们将简要介绍 DataHub 是什么以及如何使用其动作框架进行有效的数据管理。最后，我们将通过一个具体示例来展示如何编写一个自定义动作，在 DataHub 中创建数据产品时创建 Jira 票据。

希望这篇文章可以作为如何将 DataHub 事件集成到您特定 Jira 流程中的通用模板。

## 目录

+   **什么是 DataHub？**

+   **什么是 DataHub 动作？**

+   **开发自定义 Jira 动作**

    +   **YAML 配置**

    +   **定义我们的自定义动作类**

    +   **运行我们的动作**

+   **总结**

+   **来源**

* * *

## 什么是 DataHub？

DataHub 是一个支持数据发现、治理和 [元数据](https://www.opendatasoft.com/en/blog/what-is-metadata-and-why-is-it-important-data/) 管理的数据目录。它提供了允许组织实现自己的 [数据网格](https://www.datamesh-architecture.com/) 的功能——一种分权的数据管理方法，使各个业务领域能够主动管理自己的数据质量/需求。

如 DataHub 这样的元数据平台因各种原因而极具价值：

+   跨领域数据发现和分析——[数据消费者](https://www.secoda.co/glossary/what-is-a-data-consumer)（例如：数据分析师/科学家）可以使用 DataHub 探索他们可以用于分析的相关数据集。为了帮助数据消费者理解跨各个领域的数据，每个领域都需要通过足够的业务背景来丰富他们的数据资产。

+   [<mdspan datatext="el1758219938372" class="mdspan-comment">数据治理</mdspan>](https://www.databricks.com/discover/data-governance)——DataHub 允许组织建立数据单一来源的真相，通过在数据资产上实施访问策略来加强安全性，并通过支持数据分类来确保合规性。

* * *

## 什么是 DataHub 动作？

在一个数据成熟的组织中，元数据始终在演变。因此，组织对元数据变化做出实时反应非常重要。

+   DataHub 提供了[动作框架](https://docs.datahub.com/docs/actions/)来帮助将 DataHub 中发生的元数据更改集成到更大的[基于事件架构](https://www.confluent.io/learn/event-driven-architecture/#e-commerce-order-processing)中。

+   该框架允许您根据 DataHub 中发生的事件指定配置以触发某些动作。

+   常见用例可能包括在数据集发生变化时通知相关方，将元数据更改集成到组织工作流程中等。

DataHub 动作框架的常见用例之一是将元数据更改集成到组织特定的通知中。DataHub 为此提供了对某些第三方平台的原生支持，例如[Slack](https://docs.datahub.com/docs/actions/actions/slack)和[Teams](https://docs.datahub.com/docs/actions/actions/teams)。

在本文中，我们将探讨如何使用动作框架将 DataHub 元数据更改集成到 Jira 工作流程中。具体来说，我们将实现一个自定义动作，在创建新的[data product](https://docs.datahub.com/docs/dataproducts)时创建 Jira 工单。然而，这里的配置和代码可以很容易地修改，以触发其他 DataHub 事件的 Jira 工作流程。

* * *

## 开发自定义 Jira 动作

每个 DataHub 动作都在一个单独的[pipeline](https://docs.datahub.com/docs/actions/concepts#pipelines)中运行，这是一个持续运行的过程，重复以下步骤：从相关源轮询事件数据，对这些事件应用转换/过滤，并执行所需的动作。

+   我们可以通过 YAML 定义我们的动作管道配置。

+   我们可以通过扩展 DataHub 的基[Action 类](https://github.com/datahub-project/datahub/blob/master/datahub-actions/src/datahub_actions/action/action.py)来定义执行自定义动作的逻辑。

让我们逐一了解这两个步骤。

### YAML 配置

我们的 YAML 配置需要指定 4 个关键方面：

1.  动作管道名称（应该是唯一的且静态的）

1.  事件源配置

1.  转换 + 过滤配置

1.  动作配置

DataHub 动作有两种可能的事件源：

+   [DataHub Cloud](https://docs.datahub.com/docs/actions/sources/datahub-cloud-event-source)

+   [Kafka](https://docs.datahub.com/docs/actions/sources/kafka-event-source)

Kafka 是该框架的默认事件源。除非您使用的是 DataHub Cloud v0.3.7 以上的实例，否则您将处理来自 Kafka 的事件数据。

Kafka 事件源会发出两种类型的事件：

+   [元数据变更事件](https://docs.datahub.com/docs/actions/events/metadata-change-log-event) — 当 DataHub 中的任何内容发生变化时触发。

+   [实体变更事件](https://docs.datahub.com/docs/actions/events/entity-change-event) — 当 DataHub 中的特定实体发生变化时触发。

由于我们正在监听数据产品的创建，并且实体变更事件的 数据结构更容易处理，我们将过滤实体变更事件。

这是我们 YAML 文件的样子（`jira_action.yaml`）。

```py
# jira_action.yaml

# 1\. Action Pipeline Name
# This may be whatever you like as long as it's unique.
name: "jira_action"

# 2\. Event Source - Where to source event data from.
# Kafka is the default event source (https://docs.datahub.com/docs/actions/sources/kafka-event-source).
source:
  type: "kafka"
  config:
    connection:
      bootstrap: ${KAFKA_BOOTSTRAP_SERVER:-localhost:9092}
      schema_registry_url: ${SCHEMA_REGISTRY_URL:-http://localhost:8081} 

# 3\. Filter - Filter events that reach the Action
# We'll listen for Data Product CREATE events.
# For more information about other events: https://docs.datahub.com/docs/actions/events/entity-change-event#entity-create-event
filter:
  event_type: "EntityChangeEvent_v1"
  event:
    entityType: "dataProduct"
    category: "LIFECYCLE"
    operation: "CREATE"

# 4\. Action - What action to take on events.
# Here, we'll reference our custom Action file (jira_action.py).
action:
  type: "jira_action:JiraAction"
  config:
    # Action-specific configs (map)
```

### 定义我们的自定义操作类

接下来，我们需要在自定义操作类中实现创建 Jira 工单的逻辑，我们将在单独的`jira_action.py`文件中定义它。

我们的操作类将扩展 DataHub 的基本操作类，它包括重写以下方法：

+   `create()`—在实例化操作时被调用。如果您在 YAML 文件中指定了任何特定于操作的配置，此方法将把这个配置作为字典传递给此操作的每个实例。

+   `act()`—当接收到事件时被调用。此方法将包含我们操作的核心逻辑，即创建 Jira 工单。

+   `close()`—当我们的操作管道关闭时被调用。

由于我们没有指定任何特定于操作配置，并且一旦我们的操作终止，我们不需要担心任何特殊的清理工作，我们的工作将主要涉及重写`act()`。

我们将使用[Python Jira](https://jira.readthedocs.io/)，它是围绕 Jira REST API 的 Python 包装器，以编程方式与我们的 Jira 实例交互。有关如何以编程方式与 Jira 交互的更多信息/示例，请查看文档。

这是我们代码的样子。

```py
# jira_action.py

from datahub_actions.action.action import Action
from datahub_actions.event.event_envelope import EventEnvelope
from datahub_actions.event.event import Event
from datahub_actions.pipeline.pipeline_context import PipelineContext
from jira import JIRA

class JiraAction(Action):
    @classmethod
    def create(cls, config_dict: dict, ctx: PipelineContext) -> "Action":
        """
        Shares any action-specific config across all instances of the action.
        """
        return cls(ctx, config_dict)
    def __init__(self, ctx: PipelineContext, config_dict: dict):
        self.ctx = ctx
        self.config = config_dict

    def act(self, event: EventEnvelope) -> None:
        """
        Create a Jira ticket when a data product is created in DataHub with its DataHub link.
        We'll use the Python Jira API to programmatically interact with Jira (https://jira.readthedocs.io/index.html).
        """
        event_object = event.event
        entity_urn = event_object.entityUrn
        # Extract DataHub link for data product
        DATAHUB_DOMAIN = "http://localhost:9002/" # replace with link to your DataHub instance
        data_product_link = f"{DATAHUB_DOMAIN}{entity_urn}"

        # Authenticate into your Jira instance (https://jira.readthedocs.io/examples.html#authentication).
        jira = JIRA(
            token_auth="API token",  # Self-Hosted Jira (e.g. Server): the PAT token
            # basic_auth=("email", "API token"),  # Jira Cloud: a username/token tuple
            # basic_auth=("admin", "admin"),  # a username/password tuple [Not recommended]
            # auth=("admin", "admin"),  # a username/password tuple for cookie auth [Not recommended]
        )
        # Create Jira issue (https://jira.readthedocs.io/examples.html#issues).
        # For more info about the attributes you can specify in a Jira Issue,
        # check out the Issue class (https://github.com/pycontribs/jira/blob/main/jira/resources.py)
        issue_dict = {
            'project': {}, # JIRA project to create issue under in dict form (ex: {'id': 123})
            'summary': 'New Data Product',
            'description': f'Data Product Link: {data_product_link}',
            'issuetype': {}, # define issue type in dict form (ex: {'name': 'Bug'})
            'reporter': '',
            'assignee': ''
        }
        jira.create_issue(fields=issue_dict)
    def close(self) -> None:
        """
        Cleanup any processes happening inside the Action.
        """
        pass
```

虽然这里的配置和代码是针对数据产品创建事件特定的，但它们可以被修改以将其他 DataHub 事件集成到 Jira 工作流程中，例如添加/删除标签、术语、域、所有者等。

+   您可以在[这里](https://docs.datahub.com/docs/actions/events/entity-change-event)找到不同实体变更事件的列表。

+   监听这些事件将涉及在 YAML 中更改 Filter 配置为特定实体变更事件的字段值。

例如，为了在[数据集](https://docs.datahub.com/docs/generated/metamodel/entities/dataset)上创建一个[Jira 工单](https://docs.datahub.com/docs/actions/events/entity-change-event#add-tag-event)，我们会在 YAML 中的 Filter 配置中做如下更新：

```py
filter:
  event_type: "EntityChangeEvent_v1"
  event:
    entityType: "dataset"
    category: "TAG"
    operation: "ADD"
```

### 运行我们的操作

现在我们已经创建了我们的操作配置和实现，我们可以通过将这些两个文件（`jira_action.yaml`和`jira_action.py`）放置在我们的 DataHub 实例相同的 python 运行时环境中来运行此操作。

然后，我们可以使用以下命令通过 CLI 运行我们的操作：

```py
datahub actions -c jira_action.yaml
```

想要了解更多关于开发/运行自定义操作的信息，请查看[文档](https://docs.datahub.com/docs/actions/guides/developing-an-action#step-3-running-the-action)。

* * *

## 总结

感谢阅读！为了简要回顾一下我们讨论的内容：

+   DataHub 是一个数据目录，它促进了高效的数据发现、管理和治理。

+   DataHub 提供自己的操作框架，以实时将元数据更改集成到组织工作流程中。

+   使用该框架，我们可以通过简单地定义 YAML 中的动作管道和自定义 Python 类中的实现逻辑来编写自己的动作，以便将 DataHub 事件集成到 Jira 工作流程中。

如果您在使用 DataHub 动作实现实时数据治理方面有任何其他想法/经验，我非常愿意在评论中听到！

* * *

## 来源

DataHub 动作：

+   [概述](https://docs.datahub.com/docs/actions)

+   [关键概念](https://docs.datahub.com/docs/actions/concepts)

+   [实体变更事件](https://docs.datahub.com/docs/actions/events/entity-change-event)

+   [开发自定义动作](https://docs.datahub.com/docs/actions/guides/developing-an-action)

Python Jira：

+   [文档](https://jira.readthedocs.io/)
