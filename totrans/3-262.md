# 使用 Pydantic 管理环境变量

> 原文：[`towardsdatascience.com/manage-environment-variables-with-pydantic/`](https://towardsdatascience.com/manage-environment-variables-with-pydantic/)

## 简介

开发者正在开发那些打算部署到服务器上的应用程序，以便任何人都可以使用。通常在这些应用程序所在机器上，开发者会设置环境变量，以便应用程序可以运行。这些变量可以是外部服务的 API 密钥、数据库的 URL 以及更多。

然而，对于本地开发来说，在机器上声明这些变量非常不方便，因为这既是一个缓慢又混乱的过程。因此，我想在这个简短的教程中分享如何使用 Pydantic 以安全的方式处理环境变量。

## .env 文件

在 Python 项目中，你通常会将所有环境变量存储在一个名为 .env 的文件中。这是一个包含所有变量以 `key : value` 格式存在的文本文件。你也可以利用 `{}` 语法，通过一个变量的值来声明另一个变量。

下面是一个示例：

```py
#.env file

OPENAI_API_KEY="sk-your private key"
OPENAI_MODEL_ID="gpt-4o-mini"

# Development settings
DOMAIN=example.org
ADMIN_EMAIL=admin@${DOMAIN}

WANDB_API_KEY="your-private-key"
WANDB_PROJECT="myproject"
WANDB_ENTITY="my-entity"

SERPAPI_KEY= "your-api-key"
PERPLEXITY_TOKEN = "your-api-token"
```

请注意，.env 文件应该保持私密，因此，将此文件包含在你的 **.gitignore** 文件中非常重要，以确保你 **永远不会将其推送到 GitHub**，否则，其他开发者可能会窃取你的密钥并使用你付费的工具。

## env.example 文件

为了让克隆你仓库的开发者生活更轻松，你可以在项目中包含一个 env.example 文件。这是一个只包含应该放入 .env 文件中的键的文件。这样，其他人就会知道他们需要设置哪些 API、令牌或一般的秘密才能使脚本工作。

```py
#env.example

OPENAI_API_KEY=""
OPENAI_MODEL_ID=""

DOMAIN=""
ADMIN_EMAIL=""

WANDB_API_KEY=""
WANDB_PROJECT=""
WANDB_ENTITY=""

SERPAPI_KEY= ""
PERPLEXITY_TOKEN = ""
```

# python-dotenv

python-dotenv 是你用来加载 .env 文件中声明的变量的库。要安装此库：

```py
pip install python-dotenv
```

现在，你可以使用 load_dotenv 来加载这些变量。然后，通过 os 模块获取这些变量的引用。

```py
import os
from dotenv import load_dotenv

load_dotenv()

OPENAI_API_KEY = os.getenv('OPENAI_API_KEY')
OPENAI_MODEL_ID = os.getenv('OPENAI_MODEL_ID')
```

此方法首先会查找你的 .env 文件以加载你那里声明的变量。如果此文件不存在，变量将从主机机器中获取。这意味着你可以为本地开发使用 .env 文件，但当代码部署到主机环境（如虚拟机或 Docker 容器）时，我们将直接使用主机环境中定义的环境变量。

## Pydantic

> Pydantic 是 Python 中用于数据验证最常用的库之一。它还用于将类序列化为 JSON 并反序列化。它自动生成 JSON 模式，减少了手动模式管理的需求。它还提供了内置的数据验证，确保序列化的数据符合预期的格式。最后，它易于与流行的 Web 框架（如 FastAPI）集成。

[**pydantic-settings**](https://github.com/pydantic/pydantic-settings) 是 Pydantic 的一个功能，用于从环境变量中加载和验证设置或配置类。

```py
!pip install pydantic-settings
```

我们将创建一个名为`Settings`的类。这个类将继承`BaseSettings`。这使任何字段的默认行为是读取.env 文件中的值。如果.env 文件中没有找到变量，它将使用提供的默认值。

```py
from pydantic_settings import BaseSettings, SettingsConfigDict

from pydantic import (
    AliasChoices,
    Field,
    RedisDsn,
)

class Settings(BaseSettings):
    auth_key: str = Field(validation_alias='my_auth_key')  
    api_key: str = Field(alias='my_api_key')  

    redis_dsn: RedisDsn = Field(
        'redis://user:pass@localhost:6379/1', #default value
        validation_alias=AliasChoices('service_redis_dsn', 'redis_url'),  
    )

    model_config = SettingsConfigDict(env_prefix='my_prefix_')
```

在上面的`Settings`类中，我们定义了几个字段。`Field`类用于提供关于一个属性的[额外信息](https://docs.pydantic.dev/latest/api/fields/#pydantic.fields.Field)。

在我们的案例中，我们设置了一个`validation_alias`。因此，在.env 文件中查找的变量名称被覆盖。在上面的案例中，环境变量*my_auth_key*将被读取而不是*auth_key*。

您还可以在.env 文件中指定多个别名来查找，您可以通过利用`AliasChoises(choise1, choise2)`来实现。

最后一个属性`model_config`包含有关特定主题的所有变量（例如，连接到数据库）。这个变量将存储所有以前缀`env_prefix`开始的.env 变量。

## 实例化和使用设置

下一步就是实际实例化并在您的 Python 项目中使用这些设置。

```py
from pydantic_settings import BaseSettings, SettingsConfigDict

from pydantic import (
    AliasChoices,
    Field,
    RedisDsn,
)

class Settings(BaseSettings):
    auth_key: str = Field(validation_alias='my_auth_key')  
    api_key: str = Field(alias='my_api_key')  

    redis_dsn: RedisDsn = Field(
        'redis://user:pass@localhost:6379/1', #default value
        validation_alias=AliasChoices('service_redis_dsn', 'redis_url'),  
    )

    model_config = SettingsConfigDict(env_prefix='my_prefix_')

# create immediately a settings object
settings = Settings()
```

现在如何在代码库的其他部分使用设置。

```py
from Settings import settings

print(settings.auth_key) 
```

您最终可以轻松访问您的设置，Pydantic 帮助您验证这些秘密是否具有正确的格式。有关更高级的验证技巧，请参阅 Pydantic 文档：[`docs.pydantic.dev/latest/`](https://docs.pydantic.dev/latest/)

## **最终思考**

管理项目的配置是一个无聊但重要的软件开发部分。像 API 密钥、数据库连接这样的秘密通常是驱动您应用程序的动力。天真地，您可以在代码中硬编码这些变量，它仍然可以工作，但出于明显的原因，这并不是一个好的实践。在这篇文章中，我向您介绍了如何使用 pydantic 设置以结构化和安全的方式处理您的配置。

💼 [领英](https://www.linkedin.com/in/marcello-politi/) ️| 🐦 [X (Twitter)](https://x.com/Marcello_AI)  | [💻](https://emojiterra.com/laptop-computer/)  [网站](https://marcello-politi.super.site/)
