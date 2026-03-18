# 使用 FastAPI、PostgreSQL 和 Render 构建视频游戏推荐系统：第一部分

> 原文：[`towardsdatascience.com/building-video-game-recommender-systems-with-fastapi-postgresql-and-render-part-1/`](https://towardsdatascience.com/building-video-game-recommender-systems-with-fastapi-postgresql-and-render-part-1/)

## 简介

<mdspan datatext="el1758671252006" class="mdspan-comment">推荐系统</mdspan>使应用程序能够为用户生成智能建议，有效地从其他内容中筛选出相关内容。在本文中，我们利用 PostgreSQL、FastAPI 和 Render 构建和部署了一个动态视频游戏推荐系统，根据用户已交互的游戏推荐新游戏。目的是提供一个清晰的示例，说明如何构建一个独立的推荐系统，然后将其集成到前端系统或其他应用程序中。

对于这个项目，我们使用来自 Steams API 的视频游戏数据，但这也很容易被你感兴趣的其他产品数据所替代，关键步骤将是相同的。我们将介绍如何将此数据存储在数据库中，将游戏标签向量化，根据用户交互的游戏生成相似度分数，并返回一系列相关推荐。在本文末尾，我们将把这个推荐系统作为 Web 应用程序部署，这样每当用户与一个新游戏交互时，我们都可以动态生成并存储为该用户的新推荐集。

将使用以下工具：

+   PostgreSQL

+   FastAPI

+   Docker

+   Render

只对 GitHub 仓库感兴趣的人可以在这里找到它 [这里](https://github.com/pinstripezebra/recommender_system).

## 目录

由于这个项目的长度，它被分为两篇文章。第一部分涵盖了项目的设置和理论（以下步骤 1–5 所示），第二部分涵盖了部署。如果你在寻找第二部分，它位于 [这里](https://towardsdatascience.com/building-a-video-game-recommender-system-with-fastapi-postgresql-and-render-part-2).

## 第一部分

1.  数据集概述

1.  整体系统架构

1.  数据库设置

1.  FastAPI 设置

    – 模型

    – 路由

1.  构建相似度管道

## [第二部分](https://towardsdatascience.com/building-a-video-game-recommender-system-with-fastapi-postgresql-and-render-part-2):

1.  在 Render 上部署 PostgreSQL 数据库

1.  将 FastAPI 应用作为 Render 网络应用程序部署

    – 将我们的应用程序容器化

    – 将 Docker 镜像推送到 DockerHub

    – 从 DockerHub 拉取镜像进行渲染

## 数据集概述

本项目的数据集包含来自 [steamworks API](https://partner.steamgames.com/doc/home) 的前 ~2000 款游戏的资料。这些数据是免费的，并且根据服务条款，可用于个人和商业用途，但有一个每 5 分钟 200 请求的限制，这导致我们只处理了数据的一个子集。服务条款可以在[这里](https://steamcommunity.com/dev/apiterms)找到。

下面展示了游戏数据集的概述。大多数字段都是相对自描述的；需要注意的是，唯一的商品标识符是 appid。除了这个数据集，我们还有几个额外的表，我们将在下面详细说明；对我们推荐系统来说最重要的是游戏标签表，它包含将每个与游戏相关的标签（策略、RPG、卡牌游戏等）映射到 appid 的值。这些是从数据概述中显示的分类字段中提取的，然后通过旋转创建游戏标签表，以便为每个 appid:category 组合有一个唯一的行。

![图片](img/0e466cf076118de9764388a31e059369.png)

图 2：数据集概述

要更详细地了解我们项目的结构，请参阅下面的图表。

![图片](img/e1131084870ccc014d7266c856a23ebd.png)

图 3：项目文件结构

现在我们将快速概述这个项目的架构，然后深入探讨如何填充我们的数据库。

## 架构

对于我们的推荐系统，我们将使用 PostgreSQL 数据库，并带有 FastAPI 数据访问 + 处理层，这将允许我们向用户的游戏列表中添加或删除游戏。通过 FastAPI POST 请求对他们的游戏库进行更改的用户还将启动一个利用 FastAPI 的后台任务功能的推荐流程，该流程将从数据库中查询他们喜欢的游戏，与非喜欢的游戏计算相似度分数，并更新用户推荐表，以包含他们新的 top-N 推荐游戏。最后，PostgreSQL 数据库和 FastAPI 服务都将部署在 Render 上，以便在本地环境之外访问。对于这个部署步骤，可以使用任何云服务，但在这个案例中我们选择了 Render，因为它简单易用。

回顾一下，从用户的角度来看，我们的整体工作流程将如下所示：

1.  用户通过从 FastAPI 向我们的数据库发送 POST 请求将游戏添加到他们的图书馆。

    +   如果我们想将推荐系统附加到前端应用程序上，我们可以轻松地将这个 POST API 与用户界面绑定在一起。

1.  这个 POST 请求启动了一个 FastAPI 后台任务，该任务运行我们的推荐流程。

1.  推荐流程查询我们的数据库以获取用户的游戏列表和全局游戏列表。

1.  然后计算用户游戏与所有游戏之间的相似度分数。

1.  最后，我们的推荐流程向数据库发送 POST 请求，以更新该用户的推荐游戏表。

![图片](img/74a49d184445cb12575f9c4fe0c8ca4f.png "图 4")

图 4：推荐系统图

## 设置数据库

在我们构建推荐系统之前，第一步是设置我们的数据库。我们的基本数据库图如图 5 所示。我们之前讨论了上面的游戏表；这是我们的基础数据集，其余数据通常由此衍生。我们所有表的完整列表如下：

+   `Game` 表：包含我们数据库中每个独特游戏的基游戏数据

+   `User` 表：一个示例用户表，包含示例信息填充。

+   `User_Game` 表：包含用户所有“喜欢”的游戏之间的映射；此表是用于生成推荐的基础表之一，通过捕捉用户感兴趣的游戏来生成推荐。

+   `Game_Tags` 表：包含 appid:game_tag 映射，其中游戏标签可能像“策略”、“角色扮演”、“喜剧”这样的描述性标签，它捕捉到游戏的一部分精髓。每个 appid 都映射了多个标签。

+   `User_Recommendation` 表：这是我们目标表，将由我们的管道更新。每次用户与新的游戏互动时，我们的推荐管道将运行并生成一系列针对该用户的新推荐，这些推荐将存储在此。

![图 5](img/c57bb6ae5151793677d158a7ae8250b2.png "图 5")

图 5：数据库图

要设置这些表，我们可以简单地运行我们的 `src/load_database.py` 文件。此文件通过以下概述的几个步骤创建并填充我们的表。注意，目前我们将专注于理解如何将数据写入通用数据库，因此你现在需要知道的是下面的 `External_Database_Url` 是你想要使用的数据库的 URL。在本文的第二部分，我们将介绍如何在 Render 上设置数据库并将 URL 复制到你的 .env 文件中。

```py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
from sqlalchemy.ext.declarative import declarative_base
import os
from dotenv import load_dotenv
from utils.db_handler import DatabaseHandler
import pandas as pd
import uuid
import sys
from sqlalchemy.exc import OperationalError
import psycopg2

# Loading environmental variables
load_dotenv(override=True)

# Construct PostgreSQL connection URL for Render
URL_database = os.environ.get("External_Database_Url")

# Initialize DatabaseHandler with our URL
engine = DatabaseHandler(URL_database)

# loading initial user data
users_df = pd.read_csv("Data/users.csv")
games_df = pd.read_csv("Data/games.csv")
user_games_df = pd.read_csv("Data/user_games.csv")
user_recommendations_df = pd.read_csv("Data/user_recommendations.csv")
game_tags_df = pd.read_csv("Data/game_tags.csv")
```

首先，我们将五个 CSV 文件从我们的数据文件夹加载到数据框中；我们为数据库图中显示的每个表都有一个文件。我们还通过声明一个引擎变量来建立与我们的数据连接；这个引擎变量使用一个自定义的 `DataBaseHandler` 类，其初始化方法如下所示。这个类接受从我们的 .env 文件传递给 Render（或你首选的云服务）的数据库连接字符串，并包含我们所有的数据库连接、更新、删除和测试功能。

在加载数据并实例化我们的 `DatabaseHandler` 类之后，我们必须定义一个查询来创建每个五个表，并使用 `DatabaseHandler.create_table` 函数执行这些查询。这是一个非常简单的函数，它连接到我们的数据库，执行查询，然后关闭连接，留下我们在数据库图中看到的五个表；然而，它们目前是空的。

```py
# Defining queries to create tables
user_table_creation_query = """CREATE TABLE IF NOT EXISTS users (
    id UUID PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL
    )
    """
game_table_creation_query = """CREATE TABLE IF NOT EXISTS games (
    id UUID PRIMARY KEY,
    appid VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    type VARCHAR(255),
    is_free BOOLEAN DEFAULT FALSE,
    short_description TEXT,
    detailed_description TEXT,
    developers VARCHAR(255),
    publishers VARCHAR(255),
    price VARCHAR(255),
    genres VARCHAR(255),
    categories VARCHAR(255),
    release_date VARCHAR(255),
    platforms TEXT,
    metacritic_score FLOAT,
    recommendations INTEGER
    )
    """

user_games_query = """CREATE TABLE IF NOT EXISTS user_games (
    id UUID PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    appid VARCHAR(255) NOT NULL,
    shelf VARCHAR(50) DEFAULT 'Wish_List',
    rating FLOAT DEFAULT 0.0,
    review TEXT
    )
    """
recommendation_table_creation_query = """CREATE TABLE IF NOT EXISTS user_recommendations (
    id UUID PRIMARY KEY,
    username VARCHAR(255),
    appid VARCHAR(255),
    similarity FLOAT
    )
    """

game_tags_creation_query = """CREATE TABLE IF NOT EXISTS game_tags (
    id UUID PRIMARY KEY,
    appid VARCHAR(255) NOT NULL,
    category VARCHAR(255) NOT NULL
    )
    """

# Running queries to create tables
engine.delete_table('user_recommendations')
engine.delete_table('user_games')
engine.delete_table('game_tags')
engine.delete_table('games')
engine.delete_table('users')

# Create tables
engine.create_table(user_table_creation_query)
engine.create_table(game_table_creation_query)
engine.create_table(user_games_query)
engine.create_table(recommendation_table_creation_query)
engine.create_table(game_tags_creation_query) 
```

在初始表设置之后，我们接着运行一个质量检查以确保我们的每个数据集都有所需的 ID 列，将数据从数据框中填充到相应的表中，然后测试以确保表已正确填充。test_table 函数返回一个字典，如果设置正确，其形式将为 `{‘table_exists’: True, ‘table_has_data’: True}`。

```py
# Ensuring each row of each dataframe has a unique ID
if 'id' not in users_df.columns:
    users_df['id'] = [str(uuid.uuid4()) for _ in range(len(users_df))]
if 'id' not in games_df.columns:
    games_df['id'] = [str(uuid.uuid4()) for _ in range(len(games_df))]
if 'id' not in user_games_df.columns:
    user_games_df['id'] = [str(uuid.uuid4()) for _ in range(len(user_games_df))]
if 'id' not in user_recommendations_df.columns:
    user_recommendations_df['id'] = [str(uuid.uuid4()) for _ in range(len(user_recommendations_df))]
if 'id' not in game_tags_df.columns:
    game_tags_df['id'] = [str(uuid.uuid4()) for _ in range(len(game_tags_df))]

# Populates the 4 tables with data from the dataframes
engine.populate_table_dynamic(users_df, 'users')
engine.populate_table_dynamic(games_df, 'games')
engine.populate_table_dynamic(user_games_df, 'user_games')
engine.populate_table_dynamic(user_recommendations_df, 'user_recommendations')
engine.populate_table_dynamic(game_tags_df, 'game_tags')

# Testing if the tables were created and populated correctly
print(engine.test_table('users'))
print(engine.test_table('games'))
print(engine.test_table('user_games'))
print(engine.test_table('user_recommendations'))
print(engine.test_table('game_tags'))
```

## 开始使用 FastAPI

现在我们已经设置了数据库并填充了数据，我们需要构建使用 FastAPI 访问、更新和删除数据的方法。FastAPI 使我们能够轻松地构建标准化的（并且快速）API，以便与我们的数据库进行交互。FastAPI 文档提供了一个很好的逐步教程，可以在[这里](https://fastapi.tiangolo.com/tutorial/first-steps/)找到。作为一个高级概述，有几个出色的功能使 FastAPI 成为数据库和前端应用程序之间交互层的理想选择。

1.  标准化：FastAPI 允许我们使用 `GET, POST, DELETE, UPDATE` 等方法以标准化的方式定义路由来与我们的表交互。这种标准化使我们能够在纯 Python 中构建数据访问层，然后可以与各种前端应用程序交互。我们只需在前端调用我们想要的 API 方法，而不管它是用哪种语言构建的。

1.  数据验证：正如我们下面将要展示的，我们需要为每个我们与之交互的对象（例如我们的游戏和用户表）定义一个 Pydantic 数据模型。这个主要优势在于它确保了所有我们的变量都有定义的数据类型，例如，如果我们定义我们的 Game 对象，使得评分字段是浮点型，而一个用户尝试通过发送一个包含“great”评分的新条目的 POST 请求，那么这将不会成功。这个内置的数据验证将帮助我们防止在系统扩展时出现各种数据质量问题。

1.  异步：FastAPI 函数可以异步运行，这意味着其中一个函数不需要等待另一个函数完成。这可以显著提高性能，因为我们不会有一个慢速的 Fast 任务在等待一个慢速的任务完成。

1.  内置 Swagger 文档：FastAPI 有一个内置的 UI，我们可以在本地主机上导航到它，使我们能够轻松地测试和交互我们的路由。

## FastAPI 模型

我们项目的 FastAPI 部分依赖于两个主要文件：models.py，它定义了我们将要交互的数据模型（游戏、用户等），以及 main.py，它定义了我们的实际 FastAPI 应用程序并包含我们的路由。在 FastAPI 的上下文中，路由定义了处理请求的不同路径。例如，我们可能有一个 `/games` 路由来从我们的数据库请求游戏。

首先，让我们讨论我们的 `models.py` 文件。在这个文件中，我们定义了所有的模型。虽然我们为不同的对象定义了不同的模型，但总体方法将是相同的，所以我们将只详细讨论下面的游戏模型。下面你会注意到，我们为游戏对象定义了两个实际的类：一个继承自 Pydantic 基础模型的 `GameModel` 类，以及一个继承自 `sqlalchemy declarative_base` 的 `Game` 类。那么，自然的问题是，为什么我们为一个数据结构（我们的游戏数据结构）有两个类呢？

如果我们在这个项目中不使用 SQL 数据库，而是每次运行 main.py 时将每个 CSV 文件读入一个 dataframe，那么我们就不需要 `Game` 类，只需要 `GameModel` 类。在这种情况下，我们将读取我们的 games.csv dataframe，FastAPI 将使用 `GameModel` 类来确保数据类型被正确地遵守。

然而，因为我们使用的是 SQL 数据库，所以为我们的 API 和数据库使用单独的类更有意义，因为这两个类有略微不同的任务。我们的 API 类处理数据验证、序列化和可选字段，而我们的数据库类处理数据库特定的关注点，如定义主/外键、定义对象映射到的表，以及保护安全数据。为了重申最后一点，我们可能在数据库中有一些仅用于内部消费的敏感字段，我们不希望通过 API（例如密码）暴露给用户。我们可以通过拥有一个面向用户的 Pydantic 类和一个内部 SQL Alchemy 类来解决这个问题。

下面是一个如何为我们的 `Games` 对象实现此功能的示例；我们为其他表定义了单独的类，这些类可以在[这里](https://github.com/pinstripezebra/recommender_system/blob/main/src/models.py)找到；然而，总体结构是相同的。

```py
from pydantic import BaseModel
from uuid import UUID,uuid4
from typing import Optional
from enum import Enum
from sqlalchemy import Column, String, Float, Integer
import sqlalchemy.dialects.postgresql as pg
from sqlalchemy.dialects.postgresql import UUID as SA_UUID
from sqlalchemy.ext.declarative import declarative_base
import uuid
from uuid import UUID

# loading sql model
from sqlmodel import Field, Session, SQLModel, create_engine, select

# Initialize the base class for SQLAlchemy models
Base = declarative_base()

# This is the Game model for the database
class Game(Base):
    __tablename__ = "optigame_products"  # Table name in the PostgreSQL database

    id = Column(pg.UUID(as_uuid=True), primary_key=True, default=uuid.uuid4, unique=True, nullable=False)
    appid = Column(String, unique=True, nullable=False)  
    name = Column(String, nullable=False)  
    type = Column(String, nullable=True)  
    is_free = Column(pg.BOOLEAN, nullable=True, default=False)  #
    short_description = Column(String, nullable=True)  
    detailed_description = Column(String, nullable=True)  
    developers = Column(String, nullable=True)  
    publishers = Column(String, nullable=True)  
    price = Column(String, nullable=True)  
    genres = Column(String, nullable=True)  
    categories = Column(String, nullable=True)  
    release_date = Column(String, nullable=True)  
    platforms = Column(String, nullable=True)  
    metacritic_score = Column(Float, nullable=True)  
    recommendations = Column(Integer, nullable=True)  

class GameModel(BaseModel):
    id: Optional[UUID] = None
    appid: str
    name: str
    type: Optional[str] = None
    is_free: Optional[bool] = False
    short_description: Optional[str] = None
    detailed_description: Optional[str] = None
    developers: Optional[str] = None
    publishers: Optional[str] = None
    price: Optional[str] = None
    genres: Optional[str] = None
    categories: Optional[str] = None
    release_date: Optional[str] = None
    platforms: Optional[str] = None
    metacritic_score: Optional[float] = None
    recommendations: Optional[int] = None

    class Config:
        orm_mode = True  # Enable ORM mode to work with SQLAlchemy objects
        from_attributes = True # Enable attribute access for SQLAlchemy objects
```

## 设置 FastAPI 路由

在我们定义了模型之后，我们就可以创建方法来与这些模型交互，并从数据库中请求数据（GET），向数据库添加数据（POST），或从数据库中删除数据（DELETE）。下面是如何为我们的游戏模型定义 GET 请求的示例。我们在 main.py 函数的开始处进行了一些初始设置，以获取数据库 URL 并连接到它。然后我们初始化我们的应用并添加中间件来定义我们将接受请求的 URL。因为我们将在 Render 上部署 FastAPI 项目并从我们的本地机器向其发送请求，所以我们只允许 localhost 端口 8000 的请求。然后我们定义了一个名为 fetch_products 的 app.get 方法，它接受一个 appid 输入，查询数据库中 appid 等于过滤后的 appid 的游戏对象，并返回这些产品。

注意以下片段只包含设置和第一个获取方法，其余的都相当相似，可以在仓库中找到，所以这里不会对每个都进行深入解释。

```py
from fastapi import FastAPI, Depends, HTTPException, BackgroundTasks
from uuid import uuid4, UUID
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from dotenv import load_dotenv
import os

# Load environment variables
load_dotenv()

# security imports
from fastapi.middleware.cors import CORSMiddleware
from fastapi.security import OAuth2PasswordBearer

# custom imports
from src.models import User, Game, GameModel, UserModel,  UserGameModel, UserGame, GameSimilarity,GameSimilarityModel, UserRecommendation, UserRecommendationModel
from src.similarity_pipeline import UserRecommendationService

# Load the database connection string from environment variable or .env file
DATABASE_URL = os.environ.get("Internal_Database_Url")

# creating connection to the database
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Create the database tables (if they don't already exist)
Base.metadata.create_all(bind=engine)

# Dependency to get the database session
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Initialize the FastAPI app
app = FastAPI(title="Game Store API", version="1.0.0")

# Add CORS middleware to allow requests 
origins = ["http://localhost:8000"]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,  
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

#-------------------------------------------------#
# ----------PART 1: GET METHODS-------------------#
#-------------------------------------------------#
@app.get("/")
async def root():
    return {"message": "Hello World"}

@app.get("/api/v1/games/")
async def fetch_products(appid: str = None, db: Session = Depends(get_db)):
    # Query the database using the SQLAlchemy Game model
    if appid:
        products = db.query(Game).filter(Game.appid == appid).all()
    else:
        products = db.query(Game).all()
    return [GameModel.from_orm(product) for product in products]
```

一旦我们定义了`main.py`，我们就可以最终从我们的基础项目目录使用以下命令运行它。

```py
uvicorn src.main:app --reload
```

一旦完成这些，我们就可以导航到[`127.0.0.1:8000/docs`](http://127.0.0.1:8000/docs)并查看下面的交互式 FastAPI 环境。从该页面，我们可以测试我们在`main.py`文件中定义的任何方法。在我们的`fetch_products`函数的情况下，我们可以传递一个 appid 并从我们的数据库返回任何匹配的游戏。

![图片](img/ebd795bdeb0f2253cbdfe032313f77e4.png)

图 6：FastAPI Swagger 文档

## 构建我们的相似度管道

我们已经设置了数据库，可以通过 FastAPI 访问和更新数据；现在是时候转向这个项目的核心功能：推荐管道。推荐系统是一个经过充分研究的领域，我们在这里没有添加任何创新；然而，这将提供一个清晰的例子，说明如何使用 FastAPI 实现一个基本的推荐系统。

## 入门 — 如何推荐产品？

如果我们考虑问题“我该如何推荐用户可能会喜欢的新的产品？”，有两种直观的方法。

1.  协同推荐系统：如果我有一系列用户和一系列产品，我可以通过查看他们的整体产品篮子来识别具有相似兴趣的用户，然后识别给定用户篮子中“缺失”的产品。例如，如果我有一系列用户 1-3 和产品 A-C，用户 1-2 喜欢所有三个产品，但用户 3 迄今为止只喜欢产品 A + B，那么我可能会推荐他们产品 C。这从逻辑上是有道理的；这三个用户在他们喜欢的产品上有很高的重叠度，但产品 C 在用户 3 的篮子中缺失，他们很可能也会喜欢它。通过比较喜欢相同产品的用户来生成推荐的过程称为协同过滤。

1.  基于内容的推荐系统：如果我有一系列产品，我可以识别出与用户喜欢的产品相似的产品，并推荐这些产品。例如，如果我为每个游戏有一系列标签，我可以将每个游戏的标签系列转换为 1 和 0 的向量，然后使用相似度度量（在这种情况下，是[余弦相似度度量](https://en.wikipedia.org/wiki/Cosine_similarity)）来衡量基于它们的向量之间的相似度。一旦我完成了这个，我就可以根据用户的相似度分数返回用户喜欢的最相似的前 N 个游戏。

更多关于推荐系统的内容可以在[这里](https://en.wikipedia.org/wiki/Recommender_system)找到。

由于我们的初始数据集没有大量的用户，我们没有基于用户相似性建议项目的必要数据，这被称为[冷启动问题](https://en.wikipedia.org/wiki/Cold_start_(recommender_systems)。因此，我们将开发一个基于内容的推荐系统，因为我们有大量的游戏数据可以处理。

为了构建我们的管道，我们必须解决两个挑战：（1）我们如何计算用户的相似度得分，以及（2）我们如何自动化这个过程，以便在用户更新他们的游戏时运行？

我们将讲解每次用户通过“喜欢”一个游戏来发起 POST 请求时，如何触发相似度管道，然后介绍如何构建管道本身。

## 将推荐管道与 FastAPI 绑定

目前，假设我们有一个推荐服务，它将更新我们的 `user_recommendation` 表。我们希望确保每当用户更新他们的偏好时，这个服务都会被调用。我们可以通过以下步骤实现这一点；首先，我们定义一个 `generate_recommendations_background` 函数，这个函数负责连接到我们的数据库，运行相似度管道，然后关闭连接。接下来，我们需要确保当用户发起 POST 请求（即喜欢一个新的游戏）时调用这个函数；为此，我们只需在 `create_user_game` POST 请求函数的末尾添加函数调用即可。

这个工作流程的结果是，每当用户向我们的 `user_game` 表发起 POST 请求时，他们会调用 `create_user_game` 函数，向数据库添加一个新的 `user_game` 对象，然后作为后台函数运行相似度管道。

注意：下面的 POST 方法和辅助函数存储在 `main.py` 中，与我们的其他 FastAPI 方法一起。

```py
# importing similarity pipeline
from src.similarity_pipeline import UserRecommendationService

# Background task function
def generate_recommendations_background(username: str, database_url: str):
    """Background task to generate recommendations for a user"""
    # Create a new database session for the background task
    background_engine = create_engine(database_url)
    BackgroundSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=background_engine)

    db = BackgroundSessionLocal()
    try:
        recommendation_service = UserRecommendationService(db, database_url)
        recommendation_service.generate_recommendations_for_user(username)
    finally:
        db.close()

# Post method which calls background task function
@app.post("/api/v1/user_game/")
async def create_user_game(user_game: UserGameModel, background_tasks: BackgroundTasks, db: Session = Depends(get_db)):
    # Check if the entry already exists
    existing = db.query(UserGame).filter_by(username=user_game.username, appid=user_game.appid).first()
    if existing:
        raise HTTPException(status_code=400, detail="User already has this game.")

    # Prepare data with defaults
    user_game_data = {
        "username": user_game.username,
        "appid": user_game.appid,
        "shelf": user_game.shelf if user_game.shelf is not None else "Wish_List",
        "rating": user_game.rating if user_game.rating is not None else 0.0,
        "review": user_game.review if user_game.review is not None else ""
    }
    if user_game.id is not None:
        user_game_data["id"] = UUID(str(user_game.id))

    # Save the user game to database
    db_user_game = UserGame(**user_game_data)
    db.add(db_user_game)
    db.commit()
    db.refresh(db_user_game)

    # Trigger background task to generate recommendations for this user
    background_tasks.add_task(generate_recommendations_background, user_game.username, DATABASE_URL)

    return db_user_game
```

## 构建推荐管道

现在我们已经了解了当用户更新他们喜欢的游戏时，我们的相似度管道是如何被触发的，现在是时候深入了解推荐管道的工作机制了。我们的推荐管道存储在 `similarity_pipeline.py` 文件中，并包含我们上面展示的 `UserRecommendationService` 类。这个类包含一系列辅助函数，这些函数最终都会在 `generate_recommendations_for_user` 方法中被调用。按照顺序，有 7 个基本步骤，我们将逐一进行讲解。

1.  获取用户的游戏：为了生成类似游戏的推荐，我们需要检索用户已经添加到游戏篮中的游戏。这是通过调用我们的 `fetch_user_games` 辅助函数来完成的。这个函数使用用户的 ID（作为发起 POST 请求的输入）查询我们的 `user_games` 表，并返回他们篮子中的所有游戏。

1.  获取游戏标签：为了比较游戏，我们需要一个维度来进行比较，这个维度就是每个游戏关联的标签（策略、桌面游戏等）。为了检索游戏：标签映射，我们调用 `fetch_all_game_tags` 函数，该函数返回我们数据库中所有游戏的标签。

1.  向量化游戏标签：为了比较游戏 A 和 B 之间的相似性，我们首先需要使用我们的`create_game_vectors`函数将游戏标签向量化。此函数接受一系列按字母顺序排列的所有标签，并检查每个标签是否与给定的游戏相关联。例如，如果我们的总标签集是[boardgame, deckbuilding, resource-management]，而游戏 1 仅与 boardgame 标签相关联，那么它的向量将是[1, 0, 0]。

1.  创建我们的用户向量：一旦我们有一个代表每个游戏的向量，我们还需要一个聚合用户向量来与之比较。为了实现这一点，我们使用我们的`create_user_vector`函数，该函数生成一个与我们的游戏向量相同长度的聚合向量，然后我们可以使用它来生成用户与每个其他游戏之间的相似度得分。

1.  计算相似度：我们在`calculate_user_recommendations`步骤 3 和 4 中使用了创建的向量，该函数计算一个 0 到 1 之间的余弦相似度得分，并衡量每个游戏与我们的用户聚合游戏之间的相似度。

1.  删除旧推荐：在我们用新的推荐填充用户的`user_recommendations`表之前，我们首先必须使用`delete_existing_recommendations`删除旧的推荐。这将仅删除发起 POST 请求的用户推荐；其他保持不变。

1.  填充新的推荐：在删除旧推荐后，我们使用`save_recommendations`填充新的推荐。

```py
 from sqlalchemy.orm import Session
from sqlalchemy import create_engine, text
from src.models import UserGame, UserRecommendation
from sklearn.metrics.pairwise import cosine_similarity
import pandas as pd
import uuid
from typing import List
import logging

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class UserRecommendationService:
    def __init__(self, db_session: Session, database_url: str):
        self.db = db_session
        self.database_url = database_url
        self.engine = create_engine(database_url)

    def fetch_user_games(self, username: str) -> pd.DataFrame:
        """Fetch all games for a specific user"""
        query = text("SELECT username, appid FROM user_games WHERE username = :username")
        with self.engine.connect() as conn:
            result = conn.execute(query, {"username": username})
            data = result.fetchall()
            return pd.DataFrame(data, columns=['username', 'appid'])

    def fetch_all_category(self) -> pd.DataFrame:
        """Fetch all game tags"""
        query = text("SELECT appid, category FROM category")
        with self.engine.connect() as conn:
            result = conn.execute(query)
            data = result.fetchall()
            return pd.DataFrame(data, columns=['appid', 'category'])

    def create_game_vectors(self, tag_df: pd.DataFrame) -> tuple[pd.DataFrame, List[str], List[str]]:
        """Create game vectors from tags"""
        unique_tags = tag_df['category'].drop_duplicates().sort_values().tolist()
        unique_games = tag_df['appid'].drop_duplicates().sort_values().tolist()

        game_vectors = []
        for game in unique_games:
            tags = tag_df[tag_df['appid'] == game]['category'].tolist()
            vector = [1 if tag in tags else 0 for tag in unique_tags]
            game_vectors.append(vector)

        return pd.DataFrame(game_vectors, columns=unique_tags, index=unique_games), unique_tags, unique_games

    def create_user_vector(self, user_games_df: pd.DataFrame, game_vectors: pd.DataFrame, unique_tags: List[str]) -> pd.DataFrame:
        """Create user vector from their played games"""
        if user_games_df.empty:
            return pd.DataFrame([[0] * len(unique_tags)], columns=unique_tags, index=['unknown_user'])

        username = user_games_df.iloc[0]['username']
        user_games = user_games_df['appid'].tolist()

        # Only keep games that exist in game_vectors
        user_games = [g for g in user_games if g in game_vectors.index]

        if not user_games:
            user_vector = [0] * len(unique_tags)
        else:
            played_game_vectors = game_vectors.loc[user_games]
            user_vector = played_game_vectors.mean(axis=0).tolist()

        return pd.DataFrame([user_vector], columns=unique_tags, index=[username])

    def calculate_user_recommendations(self, user_vector: pd.DataFrame, game_vectors: pd.DataFrame, top_n: int = 20) -> pd.DataFrame:
        """Calculate similarity between user vector and all game vectors"""
        username = user_vector.index[0]
        user_vector_data = user_vector.iloc[0].values.reshape(1, -1)

        # Calculate similarities
        similarities = cosine_similarity(user_vector_data, game_vectors)
        similarity_df = pd.DataFrame(similarities.T, index=game_vectors.index, columns=[username])

        # Get top N recommendations
        top_games = similarity_df[username].nlargest(top_n)

        recommendations = []
        for appid, similarity in top_games.items():
            recommendations.append({
                "username": username,
                "appid": appid,
                "similarity": float(similarity)
            })

        return pd.DataFrame(recommendations)

    def delete_existing_recommendations(self, username: str):
        """Delete existing recommendations for a user"""
        self.db.query(UserRecommendation).filter(UserRecommendation.username == username).delete()
        self.db.commit()

    def save_recommendations(self, recommendations_df: pd.DataFrame):
        """Save new recommendations to database"""
        for _, row in recommendations_df.iterrows():
            recommendation = UserRecommendation(
                id=uuid.uuid4(),
                username=row['username'],
                appid=row['appid'],
                similarity=row['similarity']
            )
            self.db.add(recommendation)
        self.db.commit()

    def generate_recommendations_for_user(self, username: str, top_n: int = 20):
        """Main method to generate recommendations for a specific user"""
        try:
            logger.info(f"Starting recommendation generation for user: {username}")

            # 1\. Fetch user's games
            user_games_df = self.fetch_user_games(username)
            if user_games_df.empty:
                logger.warning(f"No games found for user: {username}")
                return

            # 2\. Fetch all game tags
            tag_df = self.fetch_all_category()
            if tag_df.empty:
                logger.error("No game tags found in database")
                return

            # 3\. Create game vectors
            game_vectors, unique_tags, unique_games = self.create_game_vectors(tag_df)

            # 4\. Create user vector
            user_vector = self.create_user_vector(user_games_df, game_vectors, unique_tags)

            # 5\. Calculate recommendations
            recommendations_df = self.calculate_user_recommendations(user_vector, game_vectors, top_n)

            # 6\. Delete existing recommendations
            self.delete_existing_recommendations(username)

            # 7\. Save new recommendations
            self.save_recommendations(recommendations_df)

            logger.info(f"Successfully generated {len(recommendations_df)} recommendations for user: {username}")

        except Exception as e:
            logger.error(f"Error generating recommendations for user {username}: {str(e)}")
            self.db.rollback()
            raise
```

## 总结

在本文中，我们介绍了如何设置 PostgreSQL 数据库和 FastAPI 应用程序以运行游戏推荐系统。然而，我们还没有讨论如何将此系统部署到云服务以允许他人与之交互。要了解第二部分，请继续阅读[第二部分](https://towardsdatascience.com/building-a-video-game-recommender-system-with-fastapi-postgresql-and-render-part-2)。

**图示**：除非另有说明，所有图像均为作者所有。

**链接**

1.  项目 GitHub 仓库：[`github.com/pinstripezebra/recommender_system`](https://github.com/pinstripezebra/recommender_system)

1.  FastAPI 文档：[`fastapi.tiangolo.com/tutorial/`](https://fastapi.tiangolo.com/tutorial/)

1.  推荐系统：[`en.wikipedia.org/wiki/Recommender_system`](https://en.wikipedia.org/wiki/Recommender_system)

1.  余弦相似度：[`en.wikipedia.org/wiki/Cosine_similarity`](https://en.wikipedia.org/wiki/Cosine_similarity)
