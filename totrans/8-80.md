# 数据工程 – 使用 Python 的 ORM 和 ODM

> 原文：[`towardsdatascience.com/data-engineering-orm-and-odm-with-python-cff97532e464/`](https://towardsdatascience.com/data-engineering-orm-and-odm-with-python-cff97532e464/)

![David Clode 在 Unsplash 上的照片](img/02ff00df15a765a3fa503f12a3762f38.png)

David Clode 在[Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)上的照片

## 简介

在处理数据科学项目时，一个基本的设置管道是关于数据收集的。现实世界的机器学习主要区别于 Kaggle-like 问题，因为数据不是静态的。我们需要抓取网站，从 API 收集数据等等。这种收集数据的方式可能看起来很混乱，确实如此！这就是为什么我们需要遵循最佳实践来结构化我们的代码，以在所有这些混乱中带来一些秩序。

## ORM 软件模式

一旦你确定了想要收集数据的来源，你需要以结构化的方式收集它们，以便将它们存储在你的数据库中。例如，你可能决定，为了训练你的 LLM，你需要的数据源包含 3 个字段：*作者，内容，*和*链接*。

你可以做的另一件事是下载数据，然后编写 SQL 查询来存储和检索数据库中的数据。更常见的是，你可能希望实现所有查询以执行**CRUD**操作。CRUD 代表创建、读取、更新和删除。这些都是持久存储的基本功能。

现在，你可以做的是将编写 SQL 查询的复杂性封装到一个 OMR（**对象关系映射**）类中，该类处理所有数据库操作。OMR 将能够与 SQL 数据库（如 MySQL 或 PostgreSQL）交互。

### 让我们开始编码

首先，我们需要导入必要的库并定义所有 OMR 模型的基础类。

```py
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# Define the base class for the ORM models
Base = declarative_base()
```

现在我们已经准备好定义一个模型了。模型代表了我们数据库中的一个表。在这里，我感兴趣的是构建一个*文档*类来存储我的数据。我们需要一个用户 ID，它将成为这个模型的主键。如果你熟悉 SQL，你应该熟悉这一点。在模型类中，我们还需要指定 Python 类将关联的表名。让我们也重写魔法方法**repr**，以确保我们的对象将以良好的方式表示和打印。

```py
# Define the Document model
class Document(Base):
    __tablename__ = 'documents'

    id = Column(Integer, primary_key=True, autoincrement=True)
    author = Column(String, nullable=False)
    content = Column(String, nullable=False)
    link = Column(String, nullable=False)

    def __repr__(self):
        return f"<Document(id={self.id}, author='{self.author}', content='{self.content}', link='{self.link}')>"
```

让我们定义一个 SQLite 数据库

```py
engine = create_engine('sqlite:///documents.db', echo=True)
Base.metadata.create_all(engine)
```

初始化一个会话也很重要，这样我们才能真正在我们的数据库上执行操作。

```py
# Create a session factory
Session = sessionmaker(bind=engine)
session = Session()
```

我们可以使用 Python 向我们的数据库添加数据！

```py
# Add some documents to the database
def seed_data():
    doc1 = Document(author='John Doe', content='This is a sample document about SQLAlchemy.', link='http://example.com/doc1')
    doc2 = Document(author='Jane Smith', content='Understanding relationships in SQLAlchemy.', link='http://example.com/doc2')
    doc3 = Document(author='Alice Johnson', content='An introduction to Python ORM.', link='http://example.com/doc3')

    session.add_all([doc1, doc2, doc3])
    session.commit()
```

对于查询数据库，我们也可以根据一些属性进行过滤，比如作者的名字。

```py
# Query the database
def query_data(author:str = None):

    if not author:
      print("nDocuments:")
      for document in session.query(Document).all():
          print(document)

    print(f"nDocuments by {author}:")
    john_doe_docs = session.query(Document).filter(Document.author == author).all()
    for document in john_doe_docs:
        print(document)
```

查阅这个[链接](https://docs.sqlalchemy.org/en/20/orm/)了解更多关于 SQLAlchemy ORM 的信息。

## ODM 软件模式

ODM 模式，即**对象文档映射**，与 ORM 模式极为相似，但与关系型数据库不同，我们与 NoSQL 数据库（如[MongoDB](https://www.mongodb.com/)）一起工作。

ODM 将面向对象的代码映射到类似 JSON 的文档，并将它们存储在数据库中。

这意味着我们将要构建的类将决定 JSON 数据在数据库中的表示方式。因此，我们将采用 Pydantic，它提供：

+   **验证**：自动检查输入数据是否与预期的类型和约束匹配

+   **序列化**：将 Python 对象转换为 JSON 或字典等格式，或将它们转换回 Python 对象。

### 让我们开始编码

我们像往常一样开始导入。

```py
from pydantic import BaseModel, Field
from uuid import UUID, uuid4
from typing import Any, Dict
from pymongo import MongoClient
```

我们将要构建的类将继承自 Pydantic 的**BaseModel**以使用 Pydantic 的功能。

使用 UUID 类型来指定 id 变量应该是一个通用唯一标识符。Pydantic 提供的 Field 类用于向变量添加约束（用于验证）或添加一些元数据。"default_factory=uuid4"意味着当创建一个没有 ID 的文档时，会自动调用 uuid4()方法来创建 ID。uuid4()生成一个随机数。

```py
class Document(BaseModel):
    id: UUID = Field(default_factory=uuid4)
    author: str
    content: str
    link: str
```

现在，我想定义一个方法，从字典对象开始生成类的实例。

这将是一个类方法。这样，我们就有了对类的引用（cls），可以利用这一点来使用[Pydantic](https://docs.pydantic.dev/latest/)功能，自动将类似 JSON 的对象转换为类实例。

返回的类型因此是 Document。我们需要使用“Document”这个顶点来返回我们在类本身中实现的类的类型。

```py
 @classmethod
    def from_mongo(cls, data: Dict[str, Any]) -> 'Document':
        """
        Convert a MongoDB document to a Document instance.
        This ensures the MongoDB '_id' field is mapped to 'id'.
        """
        if '_id' in data:
            data['id'] = data.pop('_id')
        return cls(**data)
```

以类似的方式，我们实现 to_mongo()方法。

```py
 def to_mongo(self) -> Dict[str, Any]:
        """
        Convert a Document instance to a MongoDB document.
        This ensures the 'id' field is mapped to '_id'.
        """
        data = self.dict()
        data['_id'] = data.pop('id')
        return data
```

整个类将是以下内容：

```py
class Document(BaseModel):
    id: UUID = Field(default_factory=uuid4)
    author: str
    content: str
    link: str

    @classmethod
    def from_mongo(cls, data: Dict[str, Any]) -> 'Document':
        """
        Convert a MongoDB document to a Document instance.
        This ensures the MongoDB '_id' field is mapped to 'id'.
        """
        if '_id' in data:
            data['id'] = data.pop('_id')
        return cls(**data)

    def to_mongo(self) -> Dict[str, Any]:
        """
        Convert a Document instance to a MongoDB document.
        This ensures the 'id' field is mapped to '_id'.
        """
        data = self.dict()
        data['_id'] = data.pop('id')
        return data
```

我们可以实例化一个 MongoDB 连接，并使用这些方法轻松地将数据插入到 MongoDB 中，并从 MongoDB 中检索数据。

```py
# Example usage with MongoDB
def main():
    # Connect to the local MongoDB instance
    client = MongoClient("mongodb://localhost:27017/")
    db = client.test_database
    collection = db.documents

    # Create a new document instance
    new_document = Document(author="Jane Doe", content="Learning MongoDB with Pydantic.", link="http://example.com/doc2")

    # Insert the document into MongoDB
    inserted_id = collection.insert_one(new_document.to_mongo()).inserted_id
    print(f"Inserted document with _id: {inserted_id}")

    # Fetch the document back from MongoDB
    mongo_document = collection.find_one({"_id": inserted_id})

    # Convert the MongoDB document to a Document instance
    document_instance = Document.from_mongo(mongo_document)
    print("Fetched and converted to Document instance:", document_instance)

    # Clean up (delete the inserted document)
    collection.delete_one({"_id": inserted_id})
    print("Deleted the inserted document.")

# Run the example
main()
```

完成！

这是一个非常简单的例子。你可能考虑实现一些其他方法来简化与 Mongo 的交互。例如，你可以实现的一些方法包括：

+   save()：允许模型实例被插入到数据库中

+   get_or_create()：尝试在数据库中找到一个文档，如果不存在则创建。

+   bulk_insert()：在数据库中插入多个文档

+   find()：使用高级过滤器在数据库中进行搜索

+   bulk_find()：查找多个文档

+   get_collection_name()：确定 MongoDB 集合的名称

## 最后的想法

在这篇文章中，我描述了如何实现 ORM 和 ODM 范式以简化与关系型数据库和文档型数据库的工作。这些方法通过编写干净且易于维护的代码来抽象管理数据库和编写 SQL 查询的复杂性。

如果喜欢这篇文章，请关注我的[Medium](https://medium.com/@marcellopoliti)！😁

💼 [领英](https://www.linkedin.com/in/marcello-politi/) ️| 🐦 [X (Twitter)](https://x.com/Marcello_AI) | [💻](https://emojiterra.com/laptop-computer/) [网站](https://marcello-politi.super.site/)
