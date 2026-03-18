# 继承：数据科学家必须了解的软件工程概念，以成功

> 原文：[`towardsdatascience.com/inheritance-a-software-engineering-concept-data-scientists-must-know-to-succeed/`](https://towardsdatascience.com/inheritance-a-software-engineering-concept-data-scientists-must-know-to-succeed/)

## <mdspan datatext="el1747768300272" class="mdspan-comment">为什么你应该阅读这篇文章</mdspan>

如果你打算进入数据科学领域，无论是研究生、寻求职业转变的专业人士，还是负责建立最佳实践的经理，这篇文章就是为你准备的。

数据科学吸引了来自不同背景的人。根据我的专业经验，我曾与以下同事共事过：

+   核物理学家

+   研究引力波博士后

+   计算生物学博士

+   语言学家

只列举一些。

能够遇到这样多样化的背景真是太好了，我也看到了各种思维方式如何推动创意和有效的数据科学功能的增长。

然而，我也看到了这种多样性的一个重大缺点：

> 每个人对关键软件工程概念的了解程度不同，导致编程技能参差不齐。

结果，我看到了一些数据科学家所做的出色工作，但它们：

+   不可读 —— 你根本不知道他们试图做什么。

+   不稳定 —— 一旦有人尝试运行它就会崩溃。

+   不可维护 —— 代码很快就会过时或容易崩溃。

+   不可扩展 —— 代码是单次使用的，其行为无法扩展。

这最终会削弱他们工作的影响力，并在后续产生各种问题。

![图片](img/584a6058284cdf53879964414b026deb.png)

图片由 [Shekai](https://unsplash.com/@shekai?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) 在 [Unsplash](https://unsplash.com/photos/a-bus-stop-with-a-bus-parked-next-to-it-vKla95GgAwg?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) 提供

因此，在一系列文章中，我计划概述一些核心软件工程概念，这些概念是我针对数据科学家量身定制的。

这些概念很简单，但知道它们与不知道它们之间的区别，明显区分了业余和专业人士。

## 今日概念：继承

继承是编写干净、可重用代码的基础，它能提高你的效率和工作效率。它还可以用来标准化团队编写代码的方式，从而提高代码的可读性和可维护性。

回想我刚开始学习编码时这些概念的难度，我不会一开始就给出一个抽象、高级的定义，因为这对你目前这个阶段没有任何价值。如果你需要这些，互联网上有很多你可以谷歌搜索的内容。

> ***相反，让我们看看一个真实的数据科学项目的例子***。

我们将概述数据科学家可能会遇到的一些实际问题，了解继承的概念，以及它是如何帮助数据科学家编写更好的代码的。

我们所说的“更好”是指：

+   ***易于阅读的代码。***

+   ***易于维护的代码。***

+   ***易于重新使用的代码。***

## 示例：从多个不同来源摄取数据

![](img/a698a4242f8598181c2fb20fcfd810aa.png)

照片由[John Schnobrich](https://unsplash.com/@johnishappysometimes?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)在[Unsplash](https://unsplash.com/photos/person-using-laptop-FlPc9_VocJ4?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)提供

数据科学家工作中最繁琐且耗时的工作是确定数据来源，如何读取它，如何清理它，以及如何保存它。

假设你从五个不同的外部来源接收到了 CSV 文件中提供的标签，每个来源都有自己的唯一模式。

你的任务是清理每一个文件并将它们输出为 parquet 文件，并且为了使这个文件与下游流程兼容，它们必须符合一个模式：

+   `label_id` : 整数

+   `label_value` : 整数

+   `label_timestamp` : ISO 格式的字符串时间戳。

## 快速且简略的方法

在这种情况下，快速且简略的方法是为每个文件编写一个单独的脚本。

```py
# clean_source1.py

import polars as pl

if __name__ == '__main__':

    df = pl.scan_csv('source1.csv')
    overall_label_value = df.group_by('some-metadata1').agg(
        overall_label_value=pl.col('some-metadata2').or_().over('some-metadata2')
    ) 

    df = df.drop(['some-metadata1', 'some-metadata2', 'some-metadata3'], axis=1)

    df = df.join(overall_label_value, on='some-metadata4')

    df = df.select(

        pl.col('primary_key').alias('label_id'),

        pl.col('overall_label_value').alias('label_value').replace([True, False], [1, 0]),
        pl.col('some-metadata6').alias('label_timestamp'),

    )

df.to_parquet('output/source1.parquet')
```

每个脚本都是独一无二的。

那么，这有什么问题吗？它完成了任务吗？

让我们回到我们对于好代码的标准，并评估为什么这个是坏的：

### 1. 难以阅读

代码没有组织或结构。

加载、清理和保存的所有逻辑都在同一个地方，因此很难看到每个步骤之间的界限。

请记住，这是一个人为的、简单的例子。在现实世界中，你编写的代码将会更长且更复杂。

当你遇到难以阅读的代码，并且有五个不同的版本时，这会导致长期问题：

### 2. 难以维护

缺乏结构使得添加新功能或修复错误变得困难。如果逻辑需要改变，整个脚本可能需要彻底重写。

如果需要应用于所有输出的公共操作，那么就需要有人分别修改所有五个脚本。

每次他们都需要解读成行成段的代码的目的。因为没有明确的区分

+   数据被加载的地方，

+   数据被使用的地方，

+   哪些变量依赖于下游操作，

知道你做的更改是否会对下游代码产生未知的影响，或者违反了一些上游假设变得困难。

最终，很容易出现错误。

### 3. 重新使用困难

这段代码就是一次性使用的定义。

难以阅读，除非你投入大量时间确保你理解每一行代码，否则你不知道发生了什么。

如果有人想从它中重用逻辑，他们唯一的选择是复制粘贴整个脚本并修改它，或者从头开始重写。

> *有更好的、更有效率的编写代码的方法*。

## 更好的、专业的做法

现在，让我们看看我们如何通过使用继承来改善我们的情况。

![图片](img/7fcac38faffea6d52464c0e7316e4022.png)

图片由[Kelly Sikkema](https://unsplash.com/@kellysikkema?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)在[Unsplash](https://unsplash.com/photos/yellow-click-pen-on-white-printer-paper-gcHFXsdcmJE?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)提供

### 1. 识别共同点

在我们的例子中，每个数据源都是独特的。我们知道每个文件将需要：

+   一个加载步骤

+   一个或多个清理步骤

+   一个保存步骤，我们知道所有文件都将保存到一个单独的 parquet 文件中。

我们还知道每个文件都需要符合相同的模式，因此最好对输出数据进行一些验证。

因此，这些共同点将告诉我们我们可以一次性编写哪些功能，然后重用它们。

### 2. 创建基类

现在是继承部分。

我们编写一个`base class`，或`parent class`，它实现了处理我们上面识别出的共同性的逻辑。这个类将成为其他类将*“继承”*的模板。

从这个类（称为子类）继承的类将具有与父类相同的功能，但也将能够添加新功能，或更改现有的功能。

```py
import polars as pl

class BaseCSVLabelProcessor:

    REQUIRED_OUTPUT_SCHEMA = {
        "label_id": pl.Int64,
        "label_value": pl.Int64,
        "label_timestamp": pl.Datetime
    }

    def __init__(self, input_file_path, output_file_path):
        self.input_file_path = input_file_path
        self.output_file_path = output_file_path

    def load(self):
        """Load the data from the file."""
        return pl.scan_csv(self.input_file_path)

    def clean(self, data:pl.LazyFrame):
        """Clean the input data"""
        ...

    def save(self, data:pl.LazyFrame): 
        """Save the data to parquet file."""
        data.sink_parquet(self.output_file_path)

    def validate_schema(self, data:pl.LazyFrame):
        """
        Check that the data conforms to the expected schema.
        """
        for colname, expected_dtype in self.REQUIRED_OUTPUT_SCHEMA.items():
            actual_dtype = data.schema.get(colname)

            if actual_dtype is None:
                raise ValueError(f"Column {colname} not found in data")

            if actual_dtype != expected_dtype:
                raise ValueError(
                    f"Column {colname} has incorrect type. Expected {expected_dtype}, got {actual_dtype}"
                )

    def run(self):
        """Run data processing on the specified file."""
        data = self.load()
        data = self.clean(data)
        self.validate_schema(data)
        self.save(data)
```

### 3. 定义子类

现在我们定义子类：

```py
class Source1LabelProcessor(BaseCSVLabelProcessor):
    def clean(self, data:pl.LazyFrame):
        # bespoke logic for source 1
        ...

class Source2LabelProcessor(BaseCSVLabelProcessor):
    def clean(self, data:pl.LazyFrame):
        # bespoke logic for source 2
        ...

class Source3LabelProcessor(BaseCSVLabelProcessor):
    def clean(self, data:pl.LazyFrame):
        # bespoke logic for source 3
        ...
```

由于所有共同逻辑已经在父类中实现，子类只需要关注每个文件独特的定制逻辑。

因此，我们为不良示例编写的代码现在可以改为：

```py
from <somewhere> import BaseCSVLabelProcessor

class Source1LabelProcessor(BaseCSVLabelProcessor):
    def get_overall_label_value(self, data:pl.LazyFrame):
        """Get overall label value."""
        return data.with_column(pl.col('some-metadata2').or_().over('some-metadata1'))

    def conform_to_output_schema(self, data:pl.LazyFrame):
        """Drop unnecessary columns and confrom required columns to output schema."""
        data = data.drop(['some-metadata1', 'some-metadata2', 'some-metadata3'], axis=1)

        data = data.select(
            pl.col('primary_key').alias('label_id'),
            pl.col('some-metadata5').alias('label_value').replace([True, False], [1, 0]),
            pl.col('some-metadata6').alias('label_timestamp'),
        )

        return data

    def clean(self, data:pl.LazyFrame) -> pl.DataFrame:
        """Clean label data from Source 1.

        The following steps are necessary to clean the data:

        1\. <some reason as to why we need to group by 'some-metadata1'>
        2\. <some reason for joining 'overall_label_value' to the dataframe>
        3\. Renaming columns and data types to confrom to the expected output schema.
        """
        overall_label_value = self.get_overall_label_value(data)
        df = df.join(overall_label_value, on='some-metadata4')
        df = self.conform_to_output_schema(df)
        return df
```

为了运行我们的代码，我们可以在一个集中位置进行：

```py
# label_preparation_pipeline.py
from <somewhere> import Source1LabelProcessor, Source2LabelProcessor, Source3LabelProcessor

INPUT_FILEPATHS = {
    'source1': '/path/to/file1.csv',
    'source2': '/path/to/file2.csv',
    'source3': '/path/to/file3.csv',
}

OUTPUT_FILEPATH = '/path/to/output.parquet'

def main():
    """Label processing pipeline.

    The label processing pipeline ingests data sources 1, 2, 3 which are from 
    external vendors <blah>. 

    The output is written to a parquet file, ready for ingestion by <downstream-process>.

    The code assumes the following:
    - <assumptions>

    The user needs to specify the following inputs:
    - <details on the input config>
    """
    processors = [
        Source1LabelProcessor(FILEPATHS['source1'], OUTPUT_FILEPATH),
        Source2LabelProcessor(FILEPATHS['source2'], OUTPUT_FILEPATH),
        Source3LabelProcessor(FILEPATHS['source3'], OUTPUT_FILEPATH)
    ]

    for processor in processors:
        processor.run()
```

## 为什么这样做更好？

### 1. 良好的封装

***你不需要揭开引擎盖来了解如何驾驶汽车**。

任何需要重新运行此代码的同事只需运行`main()`函数即可。你已经在相应的函数中提供了足够的 docstrings 来解释它们的功能和使用方法。

***但他们不需要知道每一行代码是如何工作的**。

他们应该能够信任你的工作并运行它。只有当他们需要修复错误或扩展其功能时，他们才需要深入了解。

这被称为*封装*——战略性地隐藏实现细节，这是编写良好代码的另一个基本编程概念。

![图片](img/afe5066fc13316a53f9ac39816c49568.png)

图片由[Dan Crile](https://unsplash.com/@dancrile?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)在[Unsplash](https://unsplash.com/photos/a-man-working-on-a-car-engine-in-a-garage-EJr3XkHdBm0?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)提供

简而言之，读者应该能够依靠文档字符串来理解代码的功能和使用方法。

你有多经常进入`scikit-learn`源代码来学习如何使用他们的模型？*你永远不会这样做。* `scikit-learn`是封装良好编码设计的理想例子。

我已经写了一篇关于封装的文章，[在这里](https://medium.com/data-science/encapsulation-a-software-engineering-concept-data-scientists-must-know-to-succeed-b3b1a0a42a41)，如果你想了解更多，请查看。

### 2. 更好的可扩展性

如果标签输出现在需要更改怎么办？例如，现在需要将标签存储在 SQL 表中，下游处理过程需要这样做。

好吧，这样做变得非常简单——我们只需要修改`BaseCSVLabelProcessor`类中的`save`方法，然后所有子类将自动继承这一更改。

如果你发现标签输出与某些下游过程不兼容怎么办？也许需要一个新的列？

好吧，你可能需要更改相应的`clean`方法来考虑这一点。但是，你还可以扩展`BaseCSVLabelProcessor`类中的`validate`方法中的检查，以适应这一新要求。

你甚至可以更进一步，添加更多检查，以确保输出始终符合预期——你可能甚至想为此定义一个单独的验证模块，并将其插入到`validate`方法中。

你可以看到如何扩展我们的标签处理代码的行为变得非常简单。

相比之下，如果代码生活在独立的定制脚本中，你将不得不一次又一次地复制粘贴这些检查。更糟糕的是，也许每个文件都需要一些定制实现。这意味着同一个问题需要解决五次，而实际上只需解决一次即可。

这是重做，它的低效，浪费的资源和时间。

## 最后的话

因此，在这篇文章中，我们介绍了使用继承如何极大地提高我们代码库的质量。

通过适当地应用继承，我们能够解决不同任务中的常见问题，并且我们亲眼见证了这一点如何导致：

+   更容易阅读的代码 — 代码可读性

+   更容易调试和维护的代码 — 可维护性

+   更容易添加和扩展功能的代码 — 可扩展性

**然而，一些读者可能仍然对编写这种代码的需要持怀疑态度。**

也许他们整个职业生涯都在编写一次性脚本，到目前为止一切都很顺利。为什么还要用更复杂的方式编写代码呢？

![图片](img/e9238b18d020b44516701c1dfef54881.png)

由[Towfiqu barbhuiya](https://unsplash.com/@towfiqu999999?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)在[Unsplash](https://unsplash.com/photos/a-blue-question-mark-on-a-pink-background-oZuBNC-6E2s?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)提供的照片

好吧，这是一个非常好的问题 —— ***而且有一个非常明确的原因说明为什么这是必要的。***

直到最近，数据科学还是一个新兴的、利基的行业，其中概念验证和研究是工作的主要焦点。当时，编码标准并不重要，只要我们能通过大门并使其工作。

***但是，数据科学正迅速接近成熟，仅仅构建模型已经不再足够。***

现在，我们必须维护、修复、调试和重新训练的不仅是模型，还包括创建模型所需的*所有*过程 —— 只要它们在使用。

这是数据科学需要面对的现实 —— **构建模型是*容易*的部分，而维护我们所构建的是*困难*的部分。**

同时，软件工程已经做了几十年，通过试错建立了我们今天讨论的所有最佳实践，以便他们构建的代码易于维护。

因此，数据科学家将需要了解这些最佳实践。

**那些了解这一点的人与那些不了解的人相比，无疑会处于优势。**

## 如果你想了解更多

以下为相关文章

### **您也可以成为 Patreon 的团队成员** [**在这里**](http://patreon.com/BenjaminLeeDataScience)**!**

我们为所有文章都设有专门的讨论线程；向我提问有关自动化测试的问题，更详细地讨论这个话题，并与其他数据科学家分享经验。学习不应该在这里停止。

你可以在这里找到这篇文章的专用讨论线程 [here](https://www.patreon.com/posts/inheritance-data-135198899?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)。
