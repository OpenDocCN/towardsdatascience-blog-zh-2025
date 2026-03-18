# 如何使用 PyTorch 从 DNA 拷贝数中分类肺癌亚型

> 原文：[`towardsdatascience.com/classification-of-lung-cancer-type-from-genomic-copy-number-profiles-using-a-convolutional-neural-network/`](https://towardsdatascience.com/classification-of-lung-cancer-type-from-genomic-copy-number-profiles-using-a-convolutional-neural-network/)

<mdspan datatext="el1760640565251" class="mdspan-comment">在这篇文章中</mdspan>，我将演示如何使用简单的 PyTorch 分类器构建一个能够区分癌症类型的卷积神经网络。用于训练的数据和代码是公开可用的，训练可以在个人电脑上进行，甚至可能在 CPU 上进行。

癌症是我们细胞在一生中积累信息错误的不幸副作用，导致不受控制的生长。作为研究人员，我们研究这些错误的模式，以便更好地了解疾病。从数据科学家的角度来看，人类基因组是一个大约三十亿字母长的字符串，由字母 A、C、G、T 组成（即每个字母 2 位信息）。复制错误或外部事件可能会删除/插入/改变一个字母，导致突变并可能破坏基因组功能。

然而，个别错误几乎从未导致癌症的发展。人体有多种机制来防止癌症的发生，包括专门的蛋白质——所谓的肿瘤抑制蛋白。细胞要能够持续生长，必须满足一系列必要条件——所谓的[“癌症标志”](https://aacrjournals.org/cancerdiscovery/article/12/1/31/675608/Hallmarks-of-Cancer-New-DimensionsHallmarks-of)。

![](img/e939c7253baa6840914a58934e90af67.png)

*癌症标志——左侧的集合展示了一组癌症需要积累的特征，以实现持续的增殖生长。图片来源：[Hanahan D. 癌症标志：新维度。癌症发现。2022 年 1 月；12(1):31–46](https://aacrjournals.org/cancerdiscovery/article/12/1/31/675608/Hallmarks-of-Cancer-New-DimensionsHallmarks-of)*.*

因此，DNA 单个字母的改变通常不足以引起自我维持的增殖生长。绝大多数由突变介导的癌症（与其他癌症来源相比，例如[人乳头瘤病毒](https://en.wikipedia.org/wiki/HPV-positive_oropharyngeal_cancer)）也表现出拷贝数（CN）变化。这些是大规模事件，通常一次添加或删除数百万个 DNA 碱基。

![](img/1d0cac305c0d9b655b8d9112ebe6168c.png)

*一个新形成的染色体被染成与原始染色体颜色相匹配的乳腺癌细胞。图片来源：[Mira Grigorova 和 Paul Edwards，剑桥大学病理系](https://www.cellosaurus.org/pawefish/BreastCellLineDescriptions/HCC38.html)*。

这些对基因组结构的巨大变化导致防止癌症形成的基因丢失，同时积累促进细胞生长的基因。通过对这些细胞的 DNA 进行测序，我们可以识别这些变化，这些变化通常发生在特定于癌症类型的区域。每个等位基因的拷贝数值可以通过使用[拷贝数调用器](https://pubmed.ncbi.nlm.nih.gov/20837533/)从测序数据中推导出来。

## 处理拷贝数配置文件

与拷贝数（CN）配置文件一起工作的一个优点是它们不是生物计量数据，因此可以无需访问限制而发布。这使我们能够随着时间的推移从多个研究中积累数据，以构建足够大的数据集。然而，来自不同研究的数据并不总是可以直接比较的，因为它们可能使用不同的技术生成，具有不同的分辨率，或者以不同的方式进行预处理。

为了获取数据并共同处理和可视化它们，我们将使用作为[科隆大学诊所的计算癌症生物学研究所](https://iccb-cologne.org)工作一部分开发的工具[CNSistent](https://academic.oup.com/gigascience/article/doi/10.1093/gigascience/giaf104/8253488?rss=1)。

首先，我们克隆存储库和数据，并将其设置为本文中使用的版本：

```py
git clone [[email protected]](/cdn-cgi/l/email-protection):schwarzlab/cnsistent.git
cd cnsistent
git checkout v0.9.0
```

由于我们将使用的数据位于存储库内部（约 1GB 的数据），下载需要几分钟。克隆 Git 和 Git LFS 必须存在于系统中。

在存储库内部有一个 requirements.txt 文件，列出了可以使用 pip `install -r requirements.txt`安装的所有依赖项。

（首先创建虚拟环境是推荐的）。一旦安装了要求，可以在同一文件夹中运行`pip install -e .`来安装 CNSistent。`-e`标志从其源目录安装包，这对于通过 API 访问数据是必要的。

存储库包含来自三个数据集的原始数据：[TCGA](https://www.cancer.gov/about-nci/organization/ccg/research/structural-genomics/tcga)、[PCAWG](https://www.nature.com/collections/afdejfafdb)和[TRACERx](http://tracerx.co.uk/)。这些数据需要首先进行预处理。这可以通过运行脚本`bash ./scripts/data_process.sh`来完成。

现在，我们已经处理了数据集，可以使用 CNSistent 数据实用库来加载它：

```py
import cns.data_utils as cdu
samples_df, cns_df = cdu.main_load("imp")
print(cns_df.head())
```

产生以下结果：

```py
|    | sample_id   | chrom   |    start |      end |   major_cn |   minor_cn |
|---:|:------------|:--------|---------:|---------:|-----------:|-----------:|
|  0 | SP101724    | chr1    |        0 | 27256755 |          2 |          2 |
|  1 | SP101724    | chr1    | 27256755 | 28028200 |          3 |          2 |
|  2 | SP101724    | chr1    | 28028200 | 32976095 |          2 |          2 |
|  3 | SP101724    | chr1    | 32976095 | 33354394 |          5 |          2 |
|  4 | SP101724    | chr1    | 33354394 | 33554783 |          3 |          2 |
```

此表显示了以下列的拷贝数数据：

+   `sample_id`：样本的标识符，

+   `chrom`：染色体，

+   `start`：段的起始位置（0 索引，包括），

+   `end`：段的结束位置（0 索引，不包括），

+   `major_cn`：大等位基因（两者中较大的）的拷贝数，

+   `minor_cn`：小等位基因（两者中较小的）的拷贝数。

因此，在第一行我们可以看到一个表示样本 SP101724 在 1 号染色体 0 到 27.26 兆碱基区域有 2 个主等位基因拷贝和 2 个次等位基因拷贝（总共 4 个）的段。

我们加载的第二个数据框，samples_df，包含样本的元数据。对我们来说，类型是重要的。我们可以通过运行以下命令来调查可用的类型：

```py
import matplotlib.pyplot as plt
type_counts = samples_df["type"].value_counts()
plt.figure(figsize=(10, 6))
type_counts.plot(kind='bar')
plt.ylabel('Count')
plt.xticks(rotation=90)
```

![图片](img/08582ca19066b1435cd0b9dbb275cb75.png)

*根据癌症类型在数据集中的样本分布。*

在上面显示的示例中，我们可以观察到数据的一个潜在问题——单个段的长度不均匀。第一个段长 27.26 兆碱基，而第二个段只有 0.77 兆碱基长。这对神经网络来说是个问题，因为它期望输入是固定大小的。

技术上，我们可以取所有现有的断点，并在数据集中的所有断点之间创建段，所谓*最小一致分割*。然而，这将导致大量段——使用`len(cns_df[“end”].unique())`快速检查显示有**823652**个独特的断点。

或者，我们可以使用 CNSistent 来创建一个新的分割，使用分箱算法。这将创建固定大小的段，可以用作神经网络的输入。在我们的工作中，我们确定了 1-3 兆碱基的段，以在准确性和过拟合之间提供最佳权衡。我们首先创建分割，然后使用以下 Bash 脚本来应用它以获取新的 CNS 文件：

```py
threads=8
cns segment whole --out "./out/segs_3MB.bed" --split 3000000 --remove gaps - filter 300000 
for dataset in TRACERx PCAWG TCGA_hg19; 
do 
 cns aggregate ./out/${dataset}_cns_imp.tsv - segments ./out/segs_3MB.bed - out ./out/${dataset}_bin_3MB.tsv - samples ./out/${dataset}_samples.tsv - threads $threads
done
```

循环分别处理每个数据集，同时保持相同的分割。使用`--threads`标志通过并行运行聚合操作来加快处理速度，根据可用的核心数调整值。

`--remove gaps  --filter 300000`参数将移除低可映射性区域（即*间隙*）并过滤出小于 300 Kb 的段。`--split 3000000`参数将创建 3 Mb 的段。

## 非小细胞肺癌

在本文中，我们将重点关注非小细胞肺癌的分类，它占所有肺癌的大约 85%，特别是腺癌和鳞状细胞癌之间的区分。区分这两种癌症很重要，因为它们的治疗方案将不同，新的方法为从血液样本或鼻拭子进行非侵入性检测带来了希望。

我们将使用上面产生的段，并使用提供的实用函数加载这些段。由于我们正在对两种类型的癌症进行分类，我们可以过滤样本，只包括相关的类型，`LUAD`（腺癌）和`LUSC`（鳞状细胞癌），并绘制第一个样本：

```py
import cns
samples_df, cns_df = cdu.main_load("3MB")
samples_df = samples_df.query("type in ['LUAD', 'LUSC']")
cns_df = cns.select_CNS_samples(cns_df, samples_df)
cns_df = cns.only_aut(cns_df)
cns.fig_lines(cns.cns_head(cns_df, n=3))
```

*在 3MB 分箱中，前三个样本的主次拷贝数段。在这种情况下，所有三个样本都来自同一患者的多区域测序，展示了癌症细胞即使在单个肿瘤内也可能具有异质性。*

## 卷积神经网络模型

运行代码需要 Python 3，并安装了[PyTorch 2+](https://pytorch.org/)，以及一个兼容 Bash 的 shell。推荐使用 NVIDIA GPU 以实现更快的训练，但并非必需。

首先，我们定义一个具有三个层的卷积神经网络：

```py
import torch.nn as nn

class CNSConvNet(nn.Module):
    def __init__(self, num_classes):
        super(CNSConvNet, self).__init__()

        self.conv_layers = nn.Sequential(
            nn.Conv1d(in_channels=2, out_channels=16, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool1d(kernel_size=2),
            nn.Conv1d(in_channels=16, out_channels=32, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool1d(kernel_size=2),
            nn.Conv1d(in_channels=32, out_channels=64, kernel_size=3, padding=1),
            nn.ReLU(),
            nn.MaxPool1d(kernel_size=2)
        )

        self.fc_layers = nn.Sequential(
            nn.LazyLinear(128),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(128, num_classes)
        )

    def forward(self, x):
        x = self.conv_layers(x)
        x = x.view(x.size(0), -1) 
        x = self.fc_layers(x)
        return x
```

这是一个具有 2 个输入通道（每个等位基因一个）和 3 个卷积层的标准深度 CNN，使用大小为 3 的一维核和 ReLU 激活函数。卷积层后面跟着大小为 2 的最大池化层。卷积传统上用于[边缘检测](https://en.wikipedia.org/wiki/Kernel_%28image_processing%29)，这对我们很有用，因为我们对拷贝数的变化感兴趣，即片段的边缘。

卷积层的输出随后被展平，并通过两个带有 dropout 的全连接层。`LazyLinear`层将 64 个堆叠通道的输出连接到一个包含 128 个节点的单层，而不需要计算卷积结束时的节点数量。因此，大部分参数都在这里，所以我们还应用 dropout 来防止过拟合。

## 训练模型

首先，我们必须将数据帧转换为 Torch 张量。我们使用一个实用函数 bins_to_features，它创建一个格式为（样本，等位基因，片段）的 3D 特征数组。在这个过程中，我们还按照 4:1 的比例将数据分为训练集和测试集：

```py
import torch
from torch.utils.data import TensorDataset, DataLoader
from sklearn.preprocessing import LabelEncoder
from sklearn.model_selection import train_test_split

# convert data to features and labels
features, samples_list, columns_df = cns.bins_to_features(cns_df)

# convert data to Torch tensors
X = torch.FloatTensor(features)
label_encoder = LabelEncoder()
y = torch.LongTensor(label_encoder.fit_transform(samples_df.loc[samples_list]["type"])) 

# Test/train split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=0)

# Create dataloaders
train_loader = DataLoader(TensorDataset(X_train, y_train), batch_size=32, shuffle=True)
test_loader = DataLoader(TensorDataset(X_test, y_test), batch_size=32, shuffle=False)
```

我们现在可以使用以下训练循环和 20 个 epoch 来训练模型。Adam 优化器和交叉熵损失通常用于分类任务，因此我们也在这里使用它们：

```py
# setup the model, loss, and optimizer
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = CNSConvNet(num_classes=len(label_encoder.classes_)).to(device)
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

# Training loop
num_epochs = 20
for epoch in range(num_epochs):
    model.train()
    running_loss = 0.0

    for inputs, labels in train_loader:
        inputs, labels = inputs.to(device), labels.to(device)

        # Clear gradients
        optimizer.zero_grad()

        # Forward pass
        outputs = model(inputs)
        loss = criterion(outputs, labels)

        # Backward pass and optimize
        loss.backward()
        optimizer.step()

        running_loss += loss.item()

    # Print statistics
    print(f'Epoch {epoch+1}/{num_epochs}, Loss: {running_loss/len(train_loader):.4f}') 
```

这就完成了训练。之后，我们可以评估模型并打印混淆矩阵：

```py
import numpy as np
from sklearn.metrics import confusion_matrix
import seaborn as sns

# Loop over batches in the test set and collect predictions
model.eval()
y_true = []
y_pred = []
with torch.no_grad():
    for inputs, labels in test_loader:
        inputs, labels = inputs.to(device), labels.to(device)
        outputs = model(inputs)
        y_true.extend(labels.cpu().numpy())
        y_pred.extend(outputs.argmax(dim=1).cpu().numpy())
        _, predicted = torch.max(outputs.data, 1)

# Calculate accuracy and confusion matrix
accuracy = (np.array(y_true) == np.array(y_pred)).mean()
cm = confusion_matrix(y_true, y_pred)

# Plot the confusion matrix
plt.figure(figsize=(3, 3), dpi=200)
sns.heatmap(cm, annot=True, fmt='d', cmap='Blues', xticklabels=label_encoder.classes_, yticklabels=label_encoder.classes_)
plt.xlabel('Predicted')
plt.ylabel('True')
plt.title('Confusion Matrix, accuracy={:.2f}'.format(accuracy))
plt.savefig("confusion_matrix.png", bbox_inches='tight')
```

![图片](img/56764529bdb1939444733f8ff3bc1b58.png)

*混淆矩阵显示了模型在测试集上的性能。模型能够以高精度（96%）区分腺癌和鳞状细胞癌。请注意，根据 GPU/CPU 的类型、CUDA 版本或 NVIDIA 驱动程序，您的结果可能会有所不同。*

在 NVIDIA RTX 4090 GPU 上，整个训练过程大约需要 7 秒钟。

## 结论

我们已经开发了一个高效且准确的肺癌亚型分类器，该分类器从拷贝数数据中提取。[正如我们所展示的](https://academic.oup.com/gigascience/article/doi/10.1093/gigascience/giaf104/8253488)，此类模型很好地转移到新的研究和序列数据来源。

大规模 AI 经常被合理化，其中之一是作为“[癌症解决方案](https://blog.samaltman.com/abundant-intelligence)”。然而，正如本文所述，使用经典方法的小型模型通常能很好地发挥作用。[有些人甚至认为](https://www.nature.com/articles/s41698-024-00553-6)，机器学习在生物医学中的实际障碍不在于解决问题，而在于真正对病人产生影响。

尽管如此，机器学习已经能够解决至少[计算生物学的一个主要谜题](https://deepmind.google/discover/blog/demis-hassabis-john-jumper-awarded-nobel-prize-in-chemistry/)，[重新激发了对癌症机器学习的关注](https://www.clinicaltrialsarena.com/news/isomorphic-labs-prepares-trials-ai-designed-drugs/)。如果有一些运气，我们可能会将下一个十年视为我们最终“解决”癌症的时代。

## 奖励：Cell2Sentence

当前的基础模型通常包含信息，例如哪些基因在哪些癌症中具有高拷贝数，作为其训练集的一部分。这虽然导致了基于 LLM 的方法的创建，如[Cell2Sentence](https://www.biorxiv.org/content/10.1101/2023.09.11.557287v3)，其中数据集被转换为自然语言并提交给 LLM。

我们可以在这里利用这种方法。首先，我们将一个随机的 LUSC 样本转换为基于其拷贝数值的基因排名列表：

```py
import cns
import cns.data_utils as cdu
samples_df, cns_df = cdu.main_load("COSMIC")
cns_df = cns.add_total_cn(cns_df)
lusc_df = cns.select_cns_by_type(cns_df, samples_df, "LUSC")
lusc_one_df = cns.cns_head(lusc_df, 1)
rank_list = lusc_one_df.sort_values(by="total_cn", ascending=False)["name"].to_list()
```

然后将基因列表转换为自然语言查询：

```py
query = "Below is a list of COSMIC {gene_names} ordered by descending total copy number in a cell. Your task is to identify the cancer type of that sample.\n\n"
query += "{gene_names}: " + " ".join(rank_list) + "\n\n"
query += "Answer:"
print(query)
```

提交给 ChatGPT 5，该模型正确地识别了 LUSC 癌症：

```py
Based on the gene copy-number profile you listed, the strongest clue is the massive amplification of 11q13 (CCND1, FADD) together with SOX2, PIK3CA, TP63, EGFR, FGFR1, MYC, CCND1/CCND2/CCND3, CDK6, CDK4\. This is a very characteristic pattern of squamous cell carcinoma of the lung (LUSC):
CCND1 amplification is one of the most frequent events in LUSC.
SOX2 amplification (3q26) is a defining driver of lung squamous carcinoma, rarely seen in adenocarcinoma.
TP63 amplification is again highly specific to squamous tumors.
Co-amplifications of PIK3CA, FGFR1, EGFR, MYC fit this profile.
The broad set of genes amplified (many regulators of squamous differentiation and RTK/RAS/PI3K signaling) reflects typical copy-number–driven LUSC biology.
By contrast, lung adenocarcinomas usually show KRAS, EGFR, ALK, ROS1, MET exon 14, ERBB2 point mutations/fusions and have fewer widespread squamous-type CNAs.
Answer: Lung squamous cell carcinoma (LUSC).
```

然而，为单个样本生成此输出所需的时间比用我们的模型对整个数据集进行分类要长，并且对整个数据集进行分类的 API 费用大约为 200 美元。
