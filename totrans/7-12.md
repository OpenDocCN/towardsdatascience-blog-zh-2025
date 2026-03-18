# 使用 R 进行健康研究中的数据协调和汇总

> 原文：[`towardsdatascience.com/harmonizing-and-pooling-datasets-for-health-research-in-r-8860388854ee/`](https://towardsdatascience.com/harmonizing-and-pooling-datasets-for-health-research-in-r-8860388854ee/)

![图片由 Claudio Schwarz 在 Unsplash 提供](img/7d1c0d488f44370e1ef1df8ba963a1df.png)

图片由[Claudio Schwarz](https://unsplash.com/@purzlbaum?utm_source=medium&utm_medium=referral)在[Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)提供

我的学术研究主要涉及识别健康研究数据集、协调它们并将单个数据集合并（汇总）以共同分析。这意味着跨人群、研究地点或国家合并数据集。这也意味着合并变量，以便它们可以有效地一起分析。换句话说，我在数据汇总领域工作，自 2017 年以来一直全职从事这一领域。

我将概述我遵循的方法，用于从单个数据集中提取数据，并将单个数据集合并成一个用于分析的汇总数据集。这是基于在全球学术环境中超过七年的工作经验。这个故事包括 R 代码。

****

**数据汇总——它是什么？**

在大多数情况下，我们将收集新的数据（原始数据收集）或仅使用一个已经可用于分析的数据集。这个数据集可以来自一家医院、一个特定的群体（例如，在社区进行的流行病学研究）或在全国范围内进行的健康调查（即全国代表性的健康调查，如[NHANES](https://wwwn.cdc.gov/nchs/nhanes/Default.aspx)）。

尽管如此，我们也可以合并或汇总多个单个数据集。这最大化了统计功效并最小化了统计不确定性。这也提高了数据的代表性，因为汇总的数据集包括大量且多样化的样本。因此，与汇总数据集一起工作有许多好处，尽管也存在许多挑战。在这里，我将分享一些技巧和 R 代码来解决这些实际挑战。

****

**在这里我将涵盖哪些步骤？**

我将涵盖数据汇总过程中的**两个关键步骤**。

**首先**，从单个数据集中提取和协调数据。这意味着读取一个数据集，只保留感兴趣的变量并统一命名，以便它们可以与其他数据集的变量合并。我们还将协调变量的单位和标签。第一步的输出将是一个处理过的数据集。也就是说，原始数据集的一个版本，其中只包含感兴趣变量并以一致格式呈现。

*其次*，将处理过的单个数据集读入一个合并的数据框中。在这一步中，我们将读取第一步中提取的数据集并将它们合并（合并或追加）。因为单个数据集中的所有变量都是使用相同的名字、单位和标签提取的，所以追加数据集不应该出现错误。第二步的输出将是一个大小为 N x V 的合并数据集。其中 N 是跨唯一数据集的总观测数，V 是每个数据集中的变量数。例如，如果我们第一步中处理了 10 个数据集，每个数据集有 100 个观测值和 10 个变量，合并数据集将包含 1,000 个观测值（10 个数据集 x 100 个观测值）和 10 个变量。

* * *

**第一步：处理每个单独的数据集**。

在这一步中，我将：a) 读取一个感兴趣的数据集；b) 提取我们感兴趣的变量；c) 处理变量，使它们具有一致的名字、单位和标签；d) 保存一个只包含所选变量的新数据集。*输出*：一个只包含相关变量并以一致格式呈现的新数据集。

*本示例假设您已经熟悉您正在处理的数据集*。您知道它们有哪些变量以及它们的名称。

*首先*，我将读取我将要工作的数据集。这可能以任何格式存在，例如 CSV 或 Excel，或者它们可以是来自 Stata、SAS 或 SPSS 的数据集。让我们假设我正在读取一个 CSV 文件。

```py
data <- read.csv("my/path/to/the/dataset.csv")
```

*其次*，我将创建一个新的数据框，只包含感兴趣的变量。在这个例子中，我将提取总共 10 个变量，包括一个用于识别原始数据集的变量（`original_data_id`）。`original_data_id`变量在最后当我们合并了所有数据集时，将用于识别原始的唯一数据集至关重要。

注意，尽管原始变量名已经足够说明，我们仍然创建了一个具有新变量名的新数据框。这是因为在其他数据集中，相同的原始变量可能有不同的名称。例如，在这个特定数据集中，变量性别被命名为`GENDER_ASKED`，但在其他数据集中它可能被命名为 sex、Sex、gender，或者只是一个代码，如 question_0001。因此，为了使我们所有的变量都有一致的名字，我们创建了此新数据框。变量名应该易于记忆且易于理解。

```py
attach(data)   # This will attach the data object to the memory so that we do not have to call it in every line.

df <- data.frame(
  original_data_id = "MOCK_DATASET_2025",   # Note that I created this variable as a string to name this dataset. You can use any name, as long as it's meaningful and easy to remember.

  subject_id       = PATIENT_ID,
  sex     = GENDER_ASKED,
  age     = as.numeric(AGE_YEARS),
  race    = RACE_ETHNICITY,
  edu     = as.numeric(SCHOOLING_YEARS),

  weight = as.numeric(WEIGHT_MEASURED),
  height = as.numeric(HEIGHT_MEASURED),
  sbp_1   = as.numeric(SYSTOL_1),   # Systolic blood pressure 1.
  sbp_2   = as.numeric(SYSTOL_2)    # Systolic blood pressure 2.
)

detach(data)   # This will detach the data object from the memory.
```

运行这段`R`代码后，环境中将有两个元素：完整的原始数据集（`data`）和这个包含变量子集的新数据集（`df`）。

第三，我们将对所有的变量在单位和标签方面进行统一。这个过程在数据集之间会有很大的差异。我建议将所有分类变量映射为数字，并记录每个数字标签的含义。例如，在这个案例中，变量性别最初被编码为“MEN”和“WOMEN”。我们将把这个变量从字符串转换为整数，并将“MEN”映射为 1，“WOMEN”映射为 2。

```py
df$sex <- ifelse(df$sex == "MEN", 1,
          ifelse(df$sex == "WOMEN", 2, NA))
```

这是为了与其他数据集保持一致性。可能的情况是，在其他数据集中，变量性别已经被编码为 1 和 2，但也可能发生变量性别被标记为“M”和“W”，或者为“male”和“female”。因此，为了避免在最终合并（合并）数据集时出现不一致，我们应该将它们重新编码为一致的标签。我们将根据需要重复此重新编码或重新标记所有变量。

我们可能还需要转换数值变量。例如，变量 `height` 可能最初是以厘米为单位，但我们希望它以米为单位。我们将对这个变量进行转换。我们将对所使用的所有数据集应用相同的转换。目标是让所有变量，跨数据集，都使用相同的单位。

```py
height <- height/100
```

**附加提示**！除了从原始数据集中提取的变量外，我还建议创建带有原始变量单位的字符串变量。下面是一个示例。这是与上面相同的代码，但在这个例子中，我包括了 `XXX_units` 变量。这些将显示原始变量的单位。例如，`age` 是以年为单位，`weight` 是以千克为单位，`height` 是以厘米为单位，而 `blood pressure` 是以毫米汞柱为单位。这种方法将帮助你跟踪单位。

这也将帮助你，如果你不希望像上面将 `height` 从厘米转换为米那样转换原始变量的单位。如果你现在不想转换单位，你可以简单地更新 `XXX_units` 变量以包含正确的单位，并在稍后阶段使用 `XXX_units` 变量作为指南来将变量转换为一致的单位。

```py
df <- data.frame(
  original_data_id = "MOCK_DATASET_2025",

  subject_id       = PATIENT_ID,
  sex     = GENDER_ASKED,
  age     = as.numeric(AGE_YEARS), age_units = "years",
  race    = RACE_ETHNIC,
  edu     = as.numeric(SCHOOLING_YEARS),

  weight = as.numeric(WEIGHT_MEASURED), weight_units = "kg",
  height = as.numeric(HEIGHT_MEASURED), height_units = "cm",
  sbp_1   = as.numeric(SYSTOL_1), sbp_1_units = "mmHg",
  sbp_2   = as.numeric(SYSTOL_2), sbp_2_units = "mmHg"
)
```

**第四点**，我建议运行以下两行代码。第一行将打印数据集的摘要。利用这个机会，再次确认所有字符串或分类变量是否已映射为数字，以及所有数值变量是否处于所需的单位。同样，第二行将打印每个变量的类型，无论是字符串、整数还是浮点数。利用这个机会，再次确认一切是否井然有序。

```py
summary(df)
sapply(df, class)
```

**最后**，将提取的数据集保存到一个文件夹中。在这个文件夹中，你将保存所有提取的数据集。例如，如果你将要处理 10 个数据集，你将在新文件夹中有 10 个新的数据集。我建议你使用变量 `original_data_id` 的相同名称保存你的数据集。

```py
write.csv(df, "path/to/folder/where/to/store/new/datasets/MOCK_DATASET_2025.csv", row.names = FALSE)
```

在这个第一步的结尾，你将保存一个新的数据框，其中只包含感兴趣的变量，具有一致的名字和格式，以便它们可以与其他数据集合并。你还需要保存包含所有代码行的`R`脚本：读取原始数据集，创建只包含感兴趣变量的新数据集，变量协调，检查格式，并保存新提取的数据集。我建议你将这个`R`脚本保存在包含原始数据集的地方。**为什么？**这样你就可以简单地复制并粘贴这个`R`脚本到包含其他数据集的文件夹中，你只需要更新一些代码行来提取新数据集。例如，你可能需要更新变量的原始名称和一些用于重新标记变量的代码行。

* * *

**第二步：读取提取的数据集并附加。**

在第一步中，我们读取并处理了我们将要合并的每个数据集。在第一步结束时，我们有了单独的数据集，每个数据集对应我们正在处理的数据集，尽管这些提取的数据集只包含具有一致名称、标签和单位的感兴趣变量。因此，这些提取的数据集可以附加，并且由于名称和标签的不同而不会出现冲突。在第二步中，我将读取提取的数据集并将它们附加起来，这样我们就可以现在将所有数据集作为一个整体来分析了。

以下代码块将。

A. 加载`dplyr`库。

B. 清理环境，使其保持为空。

C. 将工作目录设置为包含所有提取数据集的文件夹。

D. 在你的环境中创建一个名为目录中 CSV 文件名的字符串变量（`datasets`）。

E. 读取`datasets`字符串变量中的所有 CSV 文件（for-loop 语句）。这将把所有数据集加载到你的环境中。根据数据集的数量或大小，这可能会花费一分钟。在这行代码之后，你的环境中将包含你读取的所有 CSV 文件（目录中的所有提取的 CSV 文件），`datasets`字符串变量，以及 for-loop 语句中的`i`变量。

F. 从环境中删除`datasets`（字符串变量）和`i`（for-loop 语句）元素。

G. 检查环境中的元素数量是否与目录中 CSV 文件的数量相等。在这个例子中，我们知道我们将从 10 个原始数据集中提取数据。因此，我们预计将加载 10 个 CSV 数据集到环境中。如果这个逻辑语句打印出 FALSE，我们应该进行双重检查。

H. 我将把环境中的所有单个数据集压缩到一个列表（`datalist`）中。

I. 我将使用`plyr`包中的`rbind.fill`函数将列表（`datalist`）中的所有数据集附加起来。这将输出新的数据框`pooleddata`，它将包含我们处理的所有数据集。

J. 我将再次清理环境，这样我只会保留新的`pooleddata`。

```py
library(dplyr)
rm(list = ls())
setwd("path/to/folder/where/to/store/new/datasets")
datasets = list.files(pattern = "*.csv")
for(i in 1:length(datasets)) assign(datasets[i], read.csv(datasets[i]))
rm(datasets,i)
length(unique(ls())) == 10
datalist <- lapply(ls(), function(x) if (class(get(x)) == "data.frame") get(x))
pooleddata <- plyr::rbind.fill(datalist)
rm(list = setdiff(ls(), "pooleddata"))
```

`pooleddata`中的观测数（行数）应等于每个原始数据集中的观测数总和。同样，`pooleddata`中的变量数应等于每个提取的数据框中的变量数。例如，如果我们提取了 10 个数据集，每个数据集有 100 个观测数和 10 个变量，那么`pooleddata`应有 1,000 个观测数（10 x 100）和 10 个变量。

您现在已经从十个原始数据框转换成了一个准备分析的单个合并数据集！

* * *

**结束语**

这是一个**快速**的教程，旨在帮助您在致力于健康研究的数据池化工作时，了解一些实用的步骤。我们探讨了如何从我们想要合并的原始数据集中提取和协调变量，以及如何将提取的数据框（合并或追加）组合成一个准备分析的单个合并数据集。这不是一个全面的指南，请继续关注这个主题的更多内容，包括更详细的说明、建议和实用技巧！

* * *

**如果您觉得这段代码有帮助，请与您的朋友和同事分享！也请为这个故事点赞并留下评论！如果您想了解这个主题的完整课程，请告诉我。您可以在[LinkedIn](https://www.linkedin.com/in/rodrigo-m-carrillo-larco-md-phd-39016a27/)上与我联系，并在 Medium 上关注我，以获取更多故事。**
