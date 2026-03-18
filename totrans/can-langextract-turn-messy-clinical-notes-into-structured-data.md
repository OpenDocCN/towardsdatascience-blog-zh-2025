# LangExtract 能否将混乱的临床笔记转换为结构化数据？

> 原文：[`towardsdatascience.com/can-langextract-turn-messy-clinical-notes-into-structured-data/`](https://towardsdatascience.com/can-langextract-turn-messy-clinical-notes-into-structured-data/)

* * *

[LangExtract](https://github.com/google/langextract) 是来自 [Google 的开发者](https://developers.googleblog.com/en/introducing-langextract-a-gemini-powered-information-extraction-library/) 的新开源项目，它通过利用 LLM，使将混乱、非结构化文本转换为干净、结构化数据变得容易。用户可以提供一些示例，以及一个自定义模式，并根据这些结果获得结果。它既适用于专有 LLM，也适用于本地 LLM（通过 Ollama）。

医疗保健中有大量数据是非结构化的，这使得它是一个理想的领域，在这个领域中，这样的工具可以大有裨益。临床笔记很长，充满了缩写和不一致性。重要的细节，如药物名称、剂量，尤其是不良反应（ADRs），都隐藏在文本中。因此，对于这篇文章，我想看看 LangExtract 是否能够处理临床笔记中的不良反应（ADR）检测。更重要的是，它是否有效？让我们在这篇文章中找出答案。请注意，虽然 LangExtract 是来自 Google 开发者的开源项目，但它不是一个官方支持的 Google 产品。

> ***仅作快速说明：我只是在展示 LangExtract 的工作原理。我不是医生，这也不是医疗建议。***

**▶️ 这里有一个详细的** [**Kaggle 笔记本**](https://www.kaggle.com/code/parulpandey/using-langextract-with-gemini) **来跟随。**

## 为什么 ADR 提取很重要

**不良反应**（ADR）是服用药物引起的有害、非预期结果。这些可能从轻微的副作用，如恶心或头晕，到可能需要医疗注意的严重后果。

![图片](img/b47d8bcebedaa0c2f0f31bea189edc83.png)

患者因头痛服用药物但出现胃痛；一个典型的不良反应（ADR） | 图像由作者使用 ChatGPT 创建

快速检测它们对于患者安全和[药物警戒](https://www.who.int/teams/regulation-prequalification/regulation-and-safety/pharmacovigilance)至关重要。挑战在于，在临床笔记中，ADR 被埋藏在过去的状况、实验室结果和其他上下文中。因此，检测它们很棘手。使用 LLM 来检测 ADR 是一个持续的研究领域。一些[最近的研究](https://pmc.ncbi.nlm.nih.gov/articles/PMC12084699/#section22-20420986251339358)表明，LLM 擅长提出红旗，但并不可靠。因此，ADR 提取是 LangExtract 的一个很好的压力测试，因为这里的目的是看看这个库是否能在临床笔记中的其他实体（如药物、剂量、严重程度等）中识别出不良反应。

## LangExtract 是如何工作的

在我们深入使用之前，让我们分解 LangExtract 的工作流程。它是一个简单的三步过程：

1.  **通过编写一个清晰的提示来定义你的提取任务**，明确指出你想要提取的内容。

1.  **提供一些高质量的示例**来引导模型朝向你期望的格式和详细程度发展。

1.  **提交你的输入文本，选择模型，并让 LangExtract 处理它**。用户可以随后审查结果，可视化它们，或直接将它们传递到他们的下游管道。

> *该工具的官方[GitHub 仓库](https://github.com/google/langextract/tree/main)包含了多个领域的详细示例，从莎士比亚的《罗密欧与朱丽叶》中的实体提取到临床笔记中的药物识别和放射学报告的结构化。请务必查看。*

### 安装

首先，我们需要安装`LangExtract`库。在[虚拟环境](https://docs.python.org/3/library/venv.html)中这样做总是一个好主意，以保持你的项目依赖项隔离。

```py
pip install langextract
```

## 使用 LangExtract 和 Gemini 在临床笔记中识别不良反应

现在让我们来看我们的用例。在这个教程中，我将使用谷歌的**Gemini 2.5 Flash**模型。你也可以使用**Gemini Pro**来处理更复杂的推理任务。你首先需要设置你的 API 密钥：

```py
export LANGEXTRACT_API_KEY="your-api-key-here"
```

**▶️ 这里有一个详细的** [**Kaggle 笔记本**](https://www.kaggle.com/code/parulpandey/using-langextract-with-gemini) **供你参考。**

### 第 1 步：定义提取任务

让我们创建一个用于提取药物、剂量、不良反应和采取的措施的提示。我们还可以要求提供提及的严重程度。

```py
prompt = textwrap.dedent("""\
Extract medication, dosage, adverse reaction, and action taken from the text.
For each adverse reaction, include its severity as an attribute if mentioned.
Use exact text spans from the original text. Do not paraphrase.
Return entities in the order they appear.""")
```

![](img/f456d61a224a6c1cabb9d336a948ebf9.png)

笔记突出了布洛芬（400 mg）、不良反应（轻微胃痛）和采取的措施（停止用药）。这就是 ADR 提取在实际中的样子。| 图片由作者提供

接下来，让我们提供一个示例来引导模型朝向正确的格式：

```py
# 1) Define the prompt
prompt = textwrap.dedent("""\
Extract condition, medication, dosage, adverse reaction, and action taken from the text.
For each adverse reaction, include its severity as an attribute if mentioned.
Use exact text spans from the original text. Do not paraphrase.
Return entities in the order they appear.""")

# 2) Example 
examples = [
    lx.data.ExampleData(
        text=(
            "After taking ibuprofen 400 mg for a headache, "
            "the patient developed mild stomach pain. "
            "They stopped taking the medicine."
        ),
        extractions=[

            lx.data.Extraction(
                extraction_class="condition",
                extraction_text="headache"
            ),

            lx.data.Extraction(
                extraction_class="medication",
                extraction_text="ibuprofen"
            ),
            lx.data.Extraction(
                extraction_class="dosage",
                extraction_text="400 mg"
            ),
            lx.data.Extraction(
                extraction_class="adverse_reaction",
                extraction_text="mild stomach pain",
                attributes={"severity": "mild"}
            ),
            lx.data.Extraction(
                extraction_class="action_taken",
                extraction_text="They stopped taking the medicine"
            )
        ]
    )
]
```

### 第 2 步：提供输入并运行提取

对于输入，我使用的是来自 Hugging Face 上的[**ADE Corpus v2**](https://huggingface.co/datasets/SetFit/ade_corpus_v2_classification/viewer/default/train?f[label_text][value]=%27Related%27)数据集的一个真实的临床句子。

```py
input_text = (
    "A 27-year-old man who had a history of bronchial asthma, "
    "eosinophilic enteritis, and eosinophilic pneumonia presented with "
    "fever, skin eruptions, cervical lymphadenopathy, hepatosplenomegaly, "
    "atypical lymphocytosis, and eosinophilia two weeks after receiving "
    "trimethoprim (TMP)-sulfamethoxazole (SMX) treatment."
)
```

接下来，让我们使用 Gemini-2.5-Flash 模型运行 LangExtract。

```py
result = lx.extract(
    text_or_documents=input_text,
    prompt_description=prompt,
    examples=examples,
    model_id="gemini-2.5-flash",
    api_key=LANGEXTRACT_API_KEY 
)
```

### 第 3 步：查看结果

你可以显示提取的实体及其位置

```py
print(f"Input: {input_text}\n")
print("Extracted entities:")
for entity in result.extractions:
    position_info = ""
    if entity.char_interval:
        start, end = entity.char_interval.start_pos, entity.char_interval.end_pos
        position_info = f" (pos: {start}-{end})"
    print(f"• {entity.extraction_class.capitalize()}: {entity.extraction_text}{position_info}")
```

![](img/ef4ad076086c08dbb0cb80d63cc3d652.png)

LangExtract 正确地识别了**不良反应**，而没有将其与患者的现有条件混淆，这是此类任务中的一个关键挑战。

如果你想要可视化，它将创建一个`.jsonl`文件。你可以通过调用可视化函数来加载这个`.jsonl`文件，它将为你创建一个 HTML 文件。

```py
lx.io.save_annotated_documents(
    [result],
    output_name="adr_extraction.jsonl",
    output_dir="."
)

html_content = lx.visualize("adr_extraction.jsonl")

# Display the HTML content directly
display((html_content))
```

![](img/cdddbb9c74b997a249cec51f45362540.png)

## 处理更长的临床笔记

真实的临床笔记通常比上面显示的例子要长得多。例如，这里是从**ADE-Corpus-V2*数据集**发布的实际笔记，发布于**MIT 许可证**下。您可以在[Hugging Face](https://huggingface.co/datasets/SetFit/ade_corpus_v2_classification)或[Zenodo](https://zenodo.org/records/13889331)上访问它。

![](img/99d7cad5f2b8517c596827c7d1b35802.png)

来自[***ADE-Corpus-V2*数据集**](https://zenodo.org/records/13889331)的临床笔记摘录，发布于**MIT 许可证**下 | 作者提供的图片

要使用 LangExtract 处理较长的文本，您保持相同的流程，但添加三个参数：

**extraction_passes**在文本上运行多次遍历，以捕获更多细节并提高召回率。

**max_workers**控制并行处理，以便更快地处理更大的文档。

**max_char_buffer**将文本分割成更小的块，这有助于模型在输入非常长的情况下保持准确性。

```py
result = lx.extract(
    text_or_documents=input_text,
    prompt_description=prompt,
    examples=examples,
    model_id="gemini-2.5-flash",
    extraction_passes=3,    
    max_workers=20,         
    max_char_buffer=1000   
)
```

这是输出结果。为了简洁起见，我这里只展示输出的一部分。

![](img/db51eb0cd60b1bff171a513b81dcf2e2.png)

* * *

如果您愿意，您也可以直接将文档的 URL 传递给`text_or_documents`参数。

* * *

## 通过 Ollama 使用本地模型进行 LangExtract

LangExtract 不仅限于专有 API。您也可以通过**Ollama**使用本地模型运行它。这对于处理无法离开安全环境的隐私敏感的临床数据特别有用。您可以在本地设置 Ollama，拉取您首选的模型，并将 LangExtract 指向它。完整说明可在[官方文档](http://To%20use%20the%20`langextract`%20library,%20we%20need%20to%20authenticate%20with%20the%20service%20using%20an%20API%20key.%20This%20code%20cell%20retrieves%20the%20API%20key%20from%20the%20user%20data%20and%20sets%20it%20as%20an%20environment%20variable,%20making%20it%20accessible%20to%20the%20library.)中找到。

## 结论

如果您正在构建信息检索系统或任何涉及元数据提取的应用程序，LangExtract 可以为您节省大量的预处理工作。在我的 ADR 实验中，LangExtract 表现良好，正确识别了药物、剂量和反应。我注意到输出直接取决于用户提供的少量示例的质量，这意味着虽然 LLMs 承担了繁重的工作，但人类仍然是循环中的重要部分。结果令人鼓舞，但由于临床数据风险高，在转向生产使用之前，仍需要在多样化的数据集上进行更广泛和更严格的测试。
