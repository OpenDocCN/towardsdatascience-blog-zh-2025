# 多模态人工智能助手：结合本地和云模型

> 原文：[`towardsdatascience.com/a-multimodal-ai-assistant-combining-local-and-cloud-models-2006b2ea0cd9/`](https://towardsdatascience.com/a-multimodal-ai-assistant-combining-local-and-cloud-models-2006b2ea0cd9/)

![Dalle-3 对“一个穿着工具腰带、对问题感到困惑的奇特机器人”的解释。图像由作者生成。](img/a5eace552f7a7b7e906b227ef0fbff58.png)

Dalle-3 对“一个穿着工具腰带、对问题感到困惑的奇特机器人”的解释。图像由作者生成。

在这篇文章中，我们将使用 LangGraph 结合几个专业模型来构建一个基本的代理，它可以回答关于图像的复杂问题，包括标题、边界框和 OCR。最初的构想是仅使用本地模型来构建这个代理，但在经过一些迭代后，我决定添加连接到基于云的模型（即 GPT4o-mini）以获得更可靠的结果。我们也将探讨这一方面，并且这个项目的所有代码都可以在[这里](https://github.com/rmartinshort/image_agent)找到。

在过去的一年里，多模态大型语言模型——LLMs，它们将推理和生成能力从文本扩展到图像、音频和视频等媒体——在生产级机器学习系统中变得越来越强大、易于访问和实用。

封闭源代码的基于云的模型，如 GPT-4o、Claude Sonnet 和 Google Gemini，在处理图像输入方面的推理能力非常出色，并且比几个月前的多模态产品便宜得多、速度快得多。Meta 通过发布其 Llama 3.2 系列中多个竞争性多模态模型的权重加入了这场盛宴。此外，像 AWS Bedrock 和 Databricks Mosaic AI 这样的云计算服务现在正在托管许多这些模型，允许开发者快速尝试它们，而无需自己管理硬件需求和集群扩展。最后，有一个有趣的趋势，即许多小型、专业的开源模型，这些模型可以从 Hugging Face 等存储库中下载。一个可以访问这些模型的智能代理应该能够选择按什么顺序调用哪些模型以获得良好的答案，这绕过了需要一个单一的大型通用模型的需求。

一个近期具有令人着迷图像能力的例子是 [Florence2](https://arxiv.org/abs/2311.06242)。于 2024 年 6 月发布，它可以说是图像特定任务（如标题、物体检测、OCR 和短语定位，即从提供的描述中识别对象）的基础模型。按照 LLM 的标准，它也相对较小——最强大的版本有 0.77B 个参数——因此可以在现代笔记本电脑上运行。Florence2 在专门的图像任务上甚至能打败像 GPT4o 这样的巨型多模态 LLM，因为这些更大的模型虽然在回答文本中的通用问题方面很出色，但它们并不是真正设计来提供像边界框坐标这样的数值输出的。在指令微调阶段，如果使用正确的训练数据，它们当然可以改进——例如，[GPT4o 可以微调以擅长物体检测](https://blog.roboflow.com/gpt-4o-object-detection/)——但许多团队没有资源来做这件事。有趣的是，Gemini 实际上 [被宣传为能够直接进行物体检测](https://ai.google.dev/gemini-api/docs/vision?lang=node#bbox)，但 Florence2 在能够完成的图像任务范围方面仍然更为灵活。

阅读关于 Florence2 的内容激发了这个项目的想法。如果我们能将其与擅长推理的纯文本 LLM（例如 Llama 3.2 3B）以及擅长回答关于图像的一般性问题的多模态 LLM（如 Qwen2-VL）相连接，那么我们就可以构建一个能够针对图像回答复杂问题的系统。它将通过首先规划调用哪些模型以及使用什么输入，然后运行这些任务并组装结果来实现。我在最近的项目文章 [这里](https://medium.com/towards-data-science/building-a-research-agent-that-can-write-to-google-docs-part-1-4b49ea05a292) 探索的代理编排库 LangGraph，为设计这样一个系统提供了一个很好的框架。

此外，我最近购买了一台新的笔记本电脑：一台配备 24GB RAM 的 M3 Macbook。这样的机器可以以合理的延迟运行这些模型的最小版本，使得本地开发图像代理成为可能。这种越来越强大的硬件和缩小模型（或压缩/量化大型模型的智能方式）的结合非常令人印象深刻！但这也带来了实际挑战：首先，当 Florence2-base-ft、Llama-3.2–3B-Instruct-4bit 和 Qwen2-VL-2B-Instruct-4bit 都被加载时，我几乎已经没有足够的 RAM 来做其他任何事情了。这对于开发来说是可以的，但对于可能真正有用的应用程序来说，这会是一个大问题。此外，正如我们将看到的，Llama-3.2–3B-Instruct-4bit 在产生可靠的格式化输出方面并不出色，这导致我在这个项目开发过程中转向使用 GPT4-o-mini 进行推理步骤。

## 图像代理

但这个项目究竟是什么呢？让我们通过参观和演示我们将构建的系统来介绍它。StateGraph（查看这篇[文章](https://medium.com/@gitmaxd/understanding-state-in-langgraph-a-comprehensive-guide-191462220997)以获取简介）看起来是这样的，每个查询都由一个图像和文本输入组成。

![我们将构建的智能体控制流的可视化。图片由作者生成。](img/e9db6016872a0e1df8861d4cb0db2f5c.png)

我们将构建的智能体控制流的可视化。图片由作者生成。

我们按阶段进行，每个阶段都与一个[prompt](https://github.com/rmartinshort/image_agent/tree/main/image_agent/prompts)相关联。

+   **规划**。在这里，目标只是用文本形式制定一个计划，说明如何最好地使用现有工具来回答问题。提示中包含工具列表及其各种模式。一个更复杂的系统可能会在这个阶段使用 RAG 来收集最适合问题的工具列表并制定计划。

+   **结构化计划**。在这里的目的是创建一个智能体可以逐步通过的计划组件列表。[这里](https://github.com/rmartinshort/image_agent/blob/main/image_agent/prompts/PlanStructure.py)的目标是创建一个列表，该列表根据这里的 Pydantic 模型格式化。为了评估目的，保留计划文本和结构化计划是有用的。

+   **执行计划**。结构化计划中的每个元素都包含一个工具名称和输入。然后我们按顺序调用这些工具并收集它们的结果。我们的智能体只有两个可用的工具：特殊视觉（调用 Florence2）或通用视觉（调用 Qwen2 或 GPT4o），路由节点用于跟踪当前计划阶段。

+   **评估结果**。一旦执行了计划的每个步骤，模型就可以看到输入和输出，并评估是否回答了问题。如果没有，我们就回到规划步骤，并尝试使用这些新见解修改旧计划。如果是，我们就继续进行。如果模型修改计划次数过多，就会触发超时，允许循环中断。

这里有很多可能的改进和扩展，例如，在当前实现中，智能体只是输出所有先前步骤的结果，但如果需要，可以调用 LLM 将它们整理成一份漂亮的答案。

作为一个示例，看看它是如何工作的，我们给出了以下图片和输入文本，这是一组相当复杂的问题。

![本示例测试中使用的图片，由作者拍摄。](img/c0313179eeb2d346d716e5157abc63de.png)

本示例测试中使用的图片，由作者拍摄。

```py
query = """
What city is in this photo? Tell me what the tallest building is, then find all the skyscrapers and bridges
"""
```

经过所有步骤后，这是我们得到的结果。

```py
[
{1: 'This photo is of Austin, Texas. The tallest building in the image is likely The Independent, also known as the "Jenga Tower."'},
{2: 
  '{"bboxes": [
    [961.9750366210938, 293.1780090332031, 1253.5550537109375, 783.6420288085938], 
    [77.67500305175781, 41.65800094604492, 307.1150207519531, 401.64599609375], 
    [335.7950134277344, 310.4700012207031, 474.4150085449219, 753.7739868164062], 
    [534.1650390625, 412.6499938964844, 684.7350463867188, 774.2100219726562], 
    [1365.885009765625, 510.114013671875, 1454.3150634765625, 774.2100219726562], 
    [1824.76513671875, 583.9979858398438, 1927.5350341796875, 758.489990234375]
  ], "labels": ["skyscraper", "skyscraper", "skyscraper", "skyscraper", "skyscraper", "skyscraper"]}'},
{3:
  '{"bboxes": [
    [493.5350341796875, 780.4979858398438, 2386.4150390625, 1035.1619873046875]
  ], "labels": ["bridge"]}'}
}
]
```

我们可以将这些结果绘制出来以确认边界框。

![代理对我们关于这张图片的多部分问题的回答，其中我们叠加了它提供的边界框坐标。](img/1b7b59ca1069a3106f0ccf6fd7e2cd43.png)

代理对我们关于这张图片的多部分问题的回答，其中我们叠加了它提供的边界框坐标。

壮观！有人可能会争论是否真的找到了这里所有的摩天大楼，但我感觉这样的系统有潜力变得非常强大和有用，特别是如果我们能够添加裁剪边界框、放大并继续对话的能力。

在接下来的部分中，让我们更详细地探讨一下主要步骤。我希望其中的一些内容对你的项目也可能是有益的。

## 代理状态、节点和边

我的[上一篇文章](https://medium.com/towards-data-science/building-a-research-agent-that-can-write-to-google-docs-part-1-4b49ea05a292)对代理和 LangGraph 进行了更详细的讨论，所以在这里我将只简要提及这个项目的代理状态。[AgentState](https://github.com/rmartinshort/image_agent/blob/main/image_agent/agent/AgentState.py)对所有 LangGraph 图中的节点都是可访问的，并且是存储与查询相关的信息的地方。

每个节点都可以被指示写入状态中的一个或多个变量，默认情况下它们会被覆盖。这不是我们想要的计划输出行为，因为计划输出应该是一个包含计划每一步结果的列表。为了确保这个列表在代理执行任务时被附加，我们使用了添加归约器，你可以在[这里](https://langchain-ai.github.io/langgraph/concepts/low_level/#reducers)了解更多信息。

图表上每个节点都是`AgentNodes`类中的一个方法。它们接收状态，执行一些操作（通常是调用 LLM）并将它们的更新输出到状态。例如，[这里](https://github.com/rmartinshort/image_agent/blob/main/image_agent/agent/AgentNodes.py)是这个用于结构化计划的节点，代码是从这里复制的。

```py
 def structure_plan_node(self, state: dict) -> dict:

        messages = state["plan"]
        response = self.llm_structure.call(messages)
        final_plan_dict = self.post_process_plan_structure(response)
        final_plan = json.dumps(final_plan_dict)

        return {
            "plan_structure": final_plan,
            "current_step": 0,
            "max_steps": len(final_plan_dict),
        }
```

路由节点也很重要，因为在计划执行过程中它会被访问多次。在当前代码中，它非常简单，只是更新当前步骤状态值，以便其他节点知道应该查看计划结构列表的哪个部分。

```py
 def routing_node(self, state: dict) -> dict:

        plan_stage = state.get("current_step", 0)
        return {"current_step": plan_stage + 1}
```

一个扩展是，在路由节点中添加另一个 LLM 调用，以检查计划的前一步输出是否需要对下一步进行任何修改，或者是否已经回答了问题的早期终止。

最后，我们需要添加两个条件边，这些边使用存储在`AgentState`中的数据来确定下一个应该运行的节点。例如，`choose_model`边会查看`AgentState`中携带的`plan_structure`对象中当前步骤的名称，然后使用一个简单的 if 语句返回在该步骤应该调用的对应节点的名称。

```py
 def choose_model(state: dict) -> str:

        current_plan = json.loads(state.get("plan_structure"))
        current_step = state.get("current_step", 1)
        max_step = state.get("max_steps", 999)

        if current_step > max_step:
            return "finalize"
        else:
            step_to_execute = current_plan[str(current_step)]["tool_name"]
            return step_to_execute
```

整个代理结构看起来是这样的。

```py
edges: AgentEdges = AgentEdges()
nodes: AgentNodes = AgentNodes()
agent: StateGraph = StateGraph(AgentState)

## Nodes
agent.add_node("planning", nodes.plan_node)
agent.add_node("structure_plan", nodes.structure_plan_node)
agent.add_node("routing", nodes.routing_node)
agent.add_node("special_vision", nodes.call_special_vision_node)
agent.add_node("general_vision", nodes.call_general_vision_node)
agent.add_node("assessment", nodes.assessment_node)
agent.add_node("response", nodes.dump_result_node)

## Edges
agent.set_entry_point("planning")
agent.add_edge("planning", "structure_plan")
agent.add_edge("structure_plan", "routing")
agent.add_conditional_edges(
            "routing",
            edges.choose_model,
            {
                "special_vision": "special_vision",
                "general_vision": "general_vision",
                "finalize": "assessment",
            },
        )
agent.add_edge("special_vision", "routing")
agent.add_edge("general_vision", "routing")
agent.add_conditional_edges(
            "assessment",
            edges.back_to_plan,
            {
                "good_answer": "response",
                "bad_answer": "planning",
                "timeout": "response",
            },
        )
agent.add_edge("response", END)
```

并且可以使用[这里](https://langchain-ai.github.io/langgraph/how-tos/visualization/)的教程在笔记本中可视化。

## 编排模型

计划、结构和评估节点非常适合基于文本的 LLM，它可以进行推理并生成结构化输出。这里最直接的选择是选择一个大型、通用的模型，如 GPT4o-mini，它有从 Pydantic 模式中输出[优秀的 JSON 支持](https://platform.openai.com/docs/guides/structured-outputs)的优点。

在 LangChain 功能的一些帮助下，我们可以创建一个调用此类模型的类。

```py
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

class StructuredOpenAICaller:
   def __init__(
       self, api_key, system_prompt, output_model, temperature=0, max_tokens=1000
   ):
       self.temperature = temperature
       self.max_tokens = max_tokens
       self.system_prompt = system_prompt
       self.output_model = output_model
       self.llm = ChatOpenAI(
           model=self.MODEL_NAME,
           api_key=api_key,
           temperature=temperature,
           max_tokens=max_tokens,
       )
       self.chain = self._set_up_chain()

   def _set_up_chain(self):
       prompt = ChatPromptTemplate.from_messages(
           [
               ("system", self.system_prompt.system_template),
               ("human", "{query}"),
           ]
       )
       structured_llm = self.llm.with_structured_output(self.output_model)
       chain = prompt | structured_llm

       return chain
   def call(self, query):
       return self.chain.invoke({"query": query})
```

为了设置这个系统，我们提供了一个系统提示和一个输出模型（有关这些示例，请参阅[这里](https://github.com/rmartinshort/image_agent/tree/main/image_agent/prompts)）然后我们只需使用带有输入字符串的调用方法来获取符合我们指定的输出模型结构的响应。按照这样的代码设置，我们需要为代理中使用的每个不同的系统提示和输出模型创建一个新的`StructuredOpenAICaller`实例。我个人更喜欢这样做，以便跟踪使用不同的模型，但随着代理变得更加复杂，它可以通过另一种方法修改，以直接在类的单个实例中更新系统提示和输出模型。

我们能否也用本地模型来做这件事？在 Apple Silicon 上，我们可以使用 MLX 库和 Hugging Face 上的 MLX 社区来轻松实验开源模型如 Llama3.2。LangChain 也支持 MLX 集成，因此我们可以遵循上面类的结构来设置本地模型。

```py
from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_community.llms.mlx_pipeline import MLXPipeline
from langchain_community.chat_models.mlx import ChatMLX

class StructuredLlamaCaller:
   MODEL_PATH = "mlx-community/Llama-3.2-3B-Instruct-4bit"

   def __init__(
       self,
       system_prompt: Any,
       output_model: Any,
       temperature: float = 0,
       max_tokens: int = 1000,
   ) -> None:

       self.system_prompt = system_prompt
       # this is the name of the Pydantic model that defines
       # the structure we want to output
       self.output_model = output_model
       self.loaded_model = MLXPipeline.from_model_id(
           self.MODEL_PATH,
           pipeline_kwargs={"max_tokens": max_tokens, "temp": temperature, "do_sample": False},
       )
       self.llm = ChatMLX(llm=self.loaded_model)
       self.temperature = temperature
       self.max_tokens = max_tokens
       self.chain = self._set_up_chain()

   def _set_up_chain(self) -> Any:
       # Set up a parser
       parser = PydanticOutputParser(pydantic_object=self.output_model)

       # Prompt
       prompt = ChatPromptTemplate.from_messages(
           [
               (
                   "system",
                   self.system_prompt.system_template,
               ),
               ("human", "{query}"),
           ]
       ).partial(format_instructions=parser.get_format_instructions())

       chain = prompt | self.llm | parser
       return chain

def call(self, query: str) -> Any:
   return self.chain.invoke({"query": query})
```

这里有几个有趣的观点。首先，我们可以像下载任何其他 Hugging Face 模型一样下载 Llama3.2 的权重和配置，然后它们在 LangChain 的 MLXPipeline 工具的帮助下被加载到 MLX 中。当模型首次下载时，它们会自动放置在 Hugging Face 缓存中。有时列出模型及其缓存位置是有用的，例如，如果您想将模型复制到新的环境中。`scan_cache_dir`实用程序在这里会很有帮助，并且可以使用此功能生成有用的结果。

```py
from huggingface_hub import scan_cache_dir

def fetch_downloaded_model_details():

    hf_cache_info = scan_cache_dir()

    repo_paths = []
    size_on_disk = []
    repo_ids = []

    for repo in sorted(
        hf_cache_info.repos, key=lambda repo: repo.repo_path
    ):
        repo_paths.append(str(repo.repo_path))
        size_on_disk.append(repo.size_on_disk)
        repo_ids.append(repo.repo_id)
    repos_df = pd.DataFrame({
        "local_path":repo_paths,
        "size_on_disk":size_on_disk,
        "model_name":repo_ids
    })

    repos_df.set_index("model_name",inplace=True)
    return repos_df.to_dict(orient="index")
```

Llama3.2 没有内置对结构化输出的支持，如 GPT4o-mini，因此我们需要使用提示来强制它生成 JSON。LangChain 的`PydanticOutputParser`可以帮助实现这一点，尽管也可以像[这里](https://medium.com/@alejandro7899871776/structure-output-with-llama-from-scratch-39c487b6be81)所示实现自己的版本。

根据我的经验，我这里使用的 Llama 版本，即 Llama-3.2–3B-Instruct-4bit，对于结构化输出来说，除了最简单的模式外并不可靠。它在我们代理的“计划生成”阶段，给定几个示例的提示时表现相当不错，但即使有`PydanticOutputParser`提供的指令的帮助，它也经常无法将那个计划转换为 JSON。Llama 更大或/或更少量化的版本可能会更好，但它们如果在我们的代理中的其他模型旁边运行，可能会遇到 RAM 问题。因此，在项目向前推进的过程中，编排模型被设置为 GPT4o-mini。

## 一种通用的视觉模型：Qwen2-VL

要能够回答像“这张图片里发生了什么？”或“这是哪个城市？”这样的问题，我们需要一个跨模态的 LLM。可以说，在图像标题模式下，Florence2 可能能够给出很好的回答，但它并不是为对话输出而设计的。

能够在笔记本电脑上运行的跨模态模型领域仍处于起步阶段（一份最近编制的列表可以在[这里](https://medium.com/@alejandro7899871776/the-best-small-vlms-vision-language-models-for-your-next-app-47e0c64190e7)找到），但[阿里巴巴的 Qwen2-VL 系列](https://qwenlm.github.io/blog/qwen2-vl/)是一个有希望的发展。此外，我们可以利用[MLX-VLM](https://github.com/Blaizzy/mlx-vlm)，这是 MLX 的一个扩展，专门用于视觉模型的调整和推理，在我们的代理框架内设置这些模型之一。

```py
from mlx_vlm import load, apply_chat_template, generate

class QwenCaller:
   MODEL_PATH = "mlx-community/Qwen2-VL-2B-Instruct-4bit"

   def __init__(self, max_tokens=1000, temperature=0):
       self.model, self.processor = load(self.MODEL_PATH)
       self.config = self.model.config
       self.max_tokens = max_tokens
       self.temperature = temperature

   def call(self, query, image):
       messages = [
           {
               "role": "system",
               "content": ImageInterpretationPrompt.system_template,
           },
           {"role": "user", "content": query},
       ]
       prompt = apply_chat_template(self.processor, self.config, messages)
       output = generate(
           self.model,
           self.processor,
           image,
           prompt,
           max_tokens=self.max_tokens,
           temperature=self.temperature,
       )
       return output
```

本课程将加载 Qwen2-VL 的最小版本，然后使用输入图像和提示来获取文本响应。有关此模型的功能以及其他可以以相同方式使用的模型更多详细信息，请查看 MLX-VLM GitHub 页面上的此[示例列表](https://github.com/Blaizzy/mlx-vlm/blob/main/examples/multi_image_generation.ipynb)。Qwen2-VL 显然还能够生成边界框和物体指向坐标，因此这种功能也可以探索并与 Florence2 进行比较。

当然，GPT-4o-mini 也具有视觉能力，并且可能比较小的本地模型更可靠。因此，在构建这类应用时，添加调用基于云的替代方案的能力是有用的，如果需要的话，仅作为本地模型中任何一个失败时的备份。请注意，在将输入图像发送到模型之前，必须将其转换为 base64 格式，但一旦完成，我们也可以使用下面的 LangChain 框架。

```py
import base64
from io import BytesIO
from PIL import Image
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

def convert_PIL_to_base64(image: Image, format="jpeg"):
   buffer = BytesIO()
   # Save the image to this buffer in the specified format
   image.save(buffer, format=format)
   # Get binary data from the buffer
   image_bytes = buffer.getvalue()
   # Encode binary data to Base64
   base64_encoded = base64.b64encode(image_bytes)
   # Convert Base64 bytes to string (optional)
   return base64_encoded.decode("utf-8")

class OpenAIVisionCaller:
   MODEL_NAME = "gpt-4o-mini"

   def __init__(self, api_key, system_prompt, temperature=0, max_tokens=1000):
       self.temperature = temperature
       self.max_tokens = max_tokens
       self.system_prompt = system_prompt
       self.llm = ChatOpenAI(
           model=self.MODEL_NAME,
           api_key=api_key,
           temperature=temperature,
           max_tokens=max_tokens,
       )
       self.chain = self._set_up_chain()

   def _set_up_chain(self):
       prompt = ChatPromptTemplate.from_messages(
           [
               ("system", self.system_prompt.system_template),
               (
                   "user",
                   [
                       {"type": "text", "text": "{query}"},
                       {
                           "type": "image_url",
                           "image_url": {"url": "data:image/jpeg;base64,{image_data}"},
                       },
                   ],
               ),
           ]
       )

       chain = prompt | self.llm | StrOutputParser()
       return chain

   def call(self, query, image):
       base64image = convert_PIL_to_base64(image)
       return self.chain.invoke({"query": query, "image_data": base64image})
```

## 专门用于视觉的模型：Florence2

Florence2 在我们代理的背景下被视为一个专业模型，因为它虽然功能众多，但其输入必须从预定义的任务提示列表中选择。当然，该模型可以被微调以接受新的提示，但出于我们的目的，直接从 Hugging Face 下载的版本效果良好。这个模型的美妙之处在于它使用单个训练过程和一组权重，但仍然在多个图像任务中实现了高性能，这些任务之前可能需要它们自己的模型。这一成功的关键在于其庞大且精心策划的训练数据集，FLD-5B。要了解更多关于数据集、模型和训练的信息，我推荐阅读[这篇优秀的文章](https://www.assemblyai.com/blog/florence-2-how-it-works-how-to-use/)。

在我们的背景下，我们使用编排模型将查询转换为一系列 Florence 任务提示，然后按顺序调用。我们可用的选项包括字幕、对象检测、短语定位、OCR 和分割。对于其中一些选项（即短语定位和区域分割），需要一个输入短语，因此编排模型也会生成它。相比之下，像字幕这样的任务只需要图像。T[这里](https://huggingface.co/microsoft/Florence-2-large/blob/main/sample_inference.ipynb)有许多 Florence2 的使用案例，这些案例在代码中进行了探索。我们限制自己只使用对象检测、短语定位、字幕和 OCR，尽管通过更新与计划生成和结构相关的提示，可以轻松添加更多。

Florence2 似乎由 MLX-VLM 包支持，但在撰写本文时，我找不到任何使用示例，因此选择了以下使用 Hugging Face 转换器的方案。

```py
from transformers import AutoModelForCausalLM, AutoProcessor
import torch

def get_device_type():

   if torch.cuda.is_available():
       return "cuda"
   else:
       if torch.backends.mps.is_available() and torch.backends.mps.is_built():
           return "mps"
       else:
           return "cpu"

class FlorenceCaller:

   MODEL_PATH: str = "microsoft/Florence-2-base-ft" 
   # See https://huggingface.co/microsoft/Florence-2-base-ft for other modes 
   # for Florence2 
   TASK_DICT: dict[str, str] = {
       "general object detection": "<OD>",
       "specific object detection": "<CAPTION_TO_PHRASE_GROUNDING>",
       "image captioning": "<MORE_DETAILED_CAPTION>",
       "OCR": "<OCR_WITH_REGION>",
   }

   def __init__(self) -> None:
       self.device: str = (
           get_device_type()
       )  # Function to determine the device type (e.g., 'cpu' or 'cuda').

       with patch("transformers.dynamic_module_utils.get_imports", fixed_get_imports):
           self.model: AutoModelForCausalLM = AutoModelForCausalLM.from_pretrained(
               self.MODEL_PATH, trust_remote_code=True
           )
           self.processor: AutoProcessor = AutoProcessor.from_pretrained(
               self.MODEL_PATH, trust_remote_code=True
           )
           self.model.to(self.device)

   def translate_task(self, task_name: str) -> str:
       return self.TASK_DICT.get(task_name, "<DETAILED_CAPTION>")

   def call(
       self, task_prompt: str, image: Any, text_input: Optional[str] = None
   ) -> Any:

       # Get the corresponding task code for the given prompt
       task_code: str = self.translate_task(task_prompt)

       # Prevent text_input for tasks that do not require it
       if task_code in [
           "<OD>",
           "<MORE_DETAILED_CAPTION>",
           "<OCR_WITH_REGION>",
           "<DETAILED_CAPTION>",
       ]:
           text_input = None

       # Construct the prompt based on whether text_input is provided
       prompt: str = task_code if text_input is None else task_code + text_input

       # Preprocess inputs for the model
       inputs = self.processor(text=prompt, images=image, return_tensors="pt").to(
           self.device
       )

       # Generate predictions using the model
       generated_ids = self.model.generate(
           input_ids=inputs["input_ids"],
           pixel_values=inputs["pixel_values"],
           max_new_tokens=1024,
           early_stopping=False,
           do_sample=False,
           num_beams=3,
       )

       # Decode and process generated output
       generated_text: str = self.processor.batch_decode(
           generated_ids, skip_special_tokens=False
       )[0]

       parsed_answer: dict[str, Any] = self.processor.post_process_generation(
           generated_text, task=task_code, image_size=(image.width, image.height)
       )

       return parsed_answer[task_code]
```

在 Apple Silicon 上，设备变为`mps`，这些模型调用的延迟是可以容忍的。此代码也应该在 GPU 和 CPU 上工作，尽管尚未进行测试。

## 另一个例子和一些限制

让我们再举一个例子，看看代理从每个步骤输出的结果。要调用代理并处理输入查询和图像，我们可以使用`Agent.invoke`方法，该方法遵循我在[上一篇文章](https://medium.com/towards-data-science/building-a-research-agent-that-can-write-to-google-docs-part-1-4b49ea05a292)中描述的相同过程，将每个节点输出添加到结果列表中，同时在 LangGraph `InMemoryStore`对象中保存输出。

我们将使用以下图像，如果我们提出一个棘手的问题，比如“这张图片里有树吗？如果有，找到它们并描述它们在做什么”，这将是一个有趣的挑战。

![本节测试图像。照片由 Hannah Lim 在 Unsplash 拍摄](img/e8aadecdec56594ee4f8dc7f6a9a8042.png)

本节测试图片。照片由 [Hannah Lim](https://unsplash.com/@hannah15198?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) 在 [Unsplash](https://unsplash.com/photos/litter-of-dogs-fall-in-line-beside-wall-U6nlG0Y5sfs?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) 上拍摄。

```py
 from image_agent.agent.Agent import Agent
from image_agent.utils import load_secrets

secrets = load_secrets()

# use GPT4 for general vision mode
full_agent_gpt_vision = Agent(
openai_api_key=secrets["OPENAI_API_KEY"],vision_mode="gpt"
)

# use local model for general vision 
full_agent_qwen_vision = Agent(
openai_api_key=secrets["OPENAI_API_KEY"],vision_mode="local"
)
```

在一个理想的世界里，答案很简单：没有树。

然而，这个问题对代理来说却是一个难题，比较它在使用 GPT-4o-mini 和 Qwen2 作为通用视觉模型时的响应，是非常有趣的。

当我们用这个查询调用 `full_agent_qwen_vision` 时，我们得到了一个糟糕的结果：Qwen2 和 Florence2 都中了圈套，并报告说有树（有趣的是，如果我们把“树”改为“狗”，我们会得到正确的答案）。

```py
Plan: 
Call generalist vision with the question 'Are there trees in this image? If so, what are they doing?'. Then call specialist vision in object specific mode with the phrase 'cat'.

Plan_structure: 
{
  "1": {"tool_name": "general_vision", "tool_mode": "conversation", "tool_input": "Are there trees in this image? If so, what are they doing?"}, 
  "2": {"tool_name": "special_vision", "tool_mode": "specific object detection", "tool_input": "tree"}
}

Plan output:
[
  {1: 'Yes, there are trees in the image. They appear to be part of a tree line against a pink background.'}
[
  {2: '{"bboxes": [[235.77601623535156, 427.864501953125, 321.7920227050781, 617.2275390625]], "labels": ["tree"]}'}
]

Assessment: 
The result adequately answers the user's question by confirming the presence of trees in the image and providing a description of their appearance and context. The output from both the generalist and specialist vision tools is consistent and informative.
```

Qwen2 似乎容易盲目跟随提示暗示这里可能存在树木。Florence2 在这里也失败了，报告了一个不应该出现的边界框。

![如果被问及“这张图片里有树吗？如果有，找到它们并描述它们在做什么”，Qwen2 和 Florence2 都中了圈套。图片由作者生成。](img/bedb7b0f60b5909b671ae5f1e52808e5.png)

如果被问及“这张图片里有树吗？如果有，找到它们并描述它们在做什么”，Qwen2 和 Florence2 都中了圈套。图片由作者生成。

![如果被问及“这张图片里有狗吗？如果有，找到它们并描述它们在做什么”，Qwen 和基于 GPT 的代理都会给出正确的答案。图片由作者生成。](img/e4ecdd2ccc44a38b40f2bd60593681b9.png)

如果被问及“这张图片里有狗吗？如果有，找到它们并描述它们在做什么”，Qwen 和基于 GPT 的代理都会给出正确的答案。图片由作者生成。

如果我们用相同的查询调用 `full_agent_gpt_vision`，GPT4o-mini 不会中圈套，但调用 Florence2 的方式没有改变，所以它仍然失败了。然后我们看到查询评估步骤的实际操作，因为通用模型和专家模型产生了冲突的结果。

```py
Node : general_vision
Task : plan_output
[
  {1: 'There are no trees in this image. It features a group of dogs sitting in front of a pink wall.'}
]

Node : special_vision
Task : plan_output
[
  {2: '{"bboxes": [[235.77601623535156, 427.864501953125, 321.7920227050781, 617.2275390625]], "labels": ["tree"]}'}
]

Node : assessment
Task : answer_assessment
The result contains conflicting information. 
The first part states that there are no trees in the image, while the second part provides a bounding box and label indicating that a tree is present. 
This inconsistency means the user's question is not adequately answered.
```

然后，代理尝试多次重新构建计划，但 Florence2 坚持为“树”生成边界框，而答案评估节点总是将其视为不一致。这比 Qwen2 代理的结果要好，但指向了 Florence2 的更广泛问题，即假阳性。这可以通过在每一步之后让路由节点评估计划来解决，然后只有在绝对必要时才调用 Florence2。

基本构建块已经就位，这个系统已经准备好进行实验、迭代和改进，我可能在接下来的几周内继续向仓库添加内容。不过，这篇文章已经足够长了！

感谢您阅读到最后，并希望这里的工程项目能够激发您自己项目的灵感！在代理框架内协调多个专业模型是一种强大且越来越容易获得的方法，用于将大型语言模型应用于复杂任务。显然，在这个领域仍有很大的改进空间。我个人非常期待看到未来一年这个领域中的想法如何发展。
