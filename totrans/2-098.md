# 生成式 AI 的声明性和命令性提示工程

> 原文：[`towardsdatascience.com/declarative-and-imperative-prompt-engineering-for-generative-ai/`](https://towardsdatascience.com/declarative-and-imperative-prompt-engineering-for-generative-ai/)

*<mdspan datatext="el1753475965481" class="mdspan-comment">提示工程</mdspan>*指的是精心设计和优化输入（例如查询或指令）以引导生成式 AI 模型的行为和响应。提示通常使用声明性或命令性范式，或两者的混合来结构化。范式选择可能对模型输出的准确性和相关性产生重大影响。本文提供了声明性和命令性提示的概念概述，讨论了每种范式的优缺点，并考虑了实际影响。

## 什么是“什么”和“如何”

简而言之，声明性提示表达的是*应该做什么*，而命令性提示则指定*如何做某事*。假设你和朋友在一家比萨店。你告诉服务员你要点那不勒斯风味比萨。由于你只提到了你想要比萨的类型，而没有具体说明你想要如何准备它，这是一个声明性提示的例子。与此同时，你的朋友——她有一些非常特别的饮食偏好，并且想要定制一款*四季*比萨——开始详细告诉服务员她想要如何制作；这是一个命令性提示的例子。

表达的声明性和命令性范式在计算机科学中有着悠久的历史，一些编程语言倾向于偏好一种范式而不是另一种。例如，C 语言通常用于命令式编程，而 Prolog 语言则更适合声明式编程。例如，考虑以下问题：识别名叫 Charlie 的人的祖先。我们恰好知道 Charlie 亲戚的一些事实：Bob 是 Charlie 的父母，Alice 是 Bob 的父母，Susan 是 Dave 的父母，John 是 Alice 的父母。基于这些信息，下面的代码展示了我们如何使用 Prolog 识别 Charlie 的祖先。

```py
parent(alice, bob).
parent(bob, charlie).
parent(susan, dave).
parent(john, alice).

ancestor(X, Y) :- parent(X, Y).
ancestor(X, Y) :- parent(X, Z), ancestor(Z, Y).

get_ancestors(Person, Ancestors) :- findall(X, ancestor(X, Person), Ancestors).

?- get_ancestors(charlie, Ancestors).
```

虽然 Prolog 语法一开始可能看起来很奇怪，但实际上它以简洁直观的方式表达了我们希望解决的问题。首先，代码列出已知事实（即谁是谁的父母）。然后递归地定义了谓词`ancestor(X, Y)`，如果`X`是`Y`的祖先，则该谓词返回 true。最后，谓词`findall(X, Goal, List)`触发 Prolog 解释器反复评估`Goal`并将所有成功的`X`绑定存储在`List`中。在我们的例子中，这意味着识别所有`ancestor(X, Person)`的解决方案并将它们存储在变量`Ancestors`中。请注意，我们没有指定这些谓词（即“是什么”）的实现细节（即“如何”）。

相比之下，下面的 C 实现通过详细描述应该如何进行，来识别查理的祖先。

```py
#include <stdio.h>
#include <string.h>

#define MAX_PEOPLE 10
#define MAX_ANCESTORS 10

// Structure to represent parent relationships
typedef struct {
    char parent[20];
    char child[20];
} ParentRelation;

ParentRelation relations[] = {
    {"alice", "bob"},
    {"bob", "charlie"},
    {"susan", "dave"},
    {"john", "alice"}
};

int numRelations = 4;

// Check if X is a parent of Y
int isParent(const char *x, const char *y) {
    for (int i = 0; i < numRelations; ++i) {
        if (strcmp(relations[i].parent, x) == 0 && strcmp(relations[i].child, y) == 0) {
            return 1;
        }
    }
    return 0;
}

// Recursive function to check if X is an ancestor of Y
int isAncestor(const char *x, const char *y) {
    if (isParent(x, y)) return 1;
    for (int i = 0; i < numRelations; ++i) {
        if (strcmp(relations[i].child, y) == 0) {
            if (isAncestor(x, relations[i].parent)) return 1;
        }
    }
    return 0;
}

// Get all ancestors of a person
void getAncestors(const char *person, char ancestors[][20], int *numAncestors) {
    *numAncestors = 0;
    for (int i = 0; i < numRelations; ++i) {
        if (isAncestor(relations[i].parent, person)) {
            strcpy(ancestors[*numAncestors], relations[i].parent);
            (*numAncestors)++;
        }
    }
}

int main() {
    char person[] = "charlie";
    char ancestors[MAX_ANCESTORS][20];
    int count;

    getAncestors(person, ancestors, &count);

    printf("Ancestors of %s:\n", person);
    for (int i = 0; i < count; ++i) {
        printf("%s\n", ancestors[i]);
    }

    return 0;
}
```

现在，功能丰富的库和 API 的日益可用，以及架构最佳实践的巩固（例如，面向接口编程，使用配置而非代码），使我们能够在许多情况下专注于“是什么”，而不必担心“如何做”。确实，如以下短视频所示，大量的代码可以——也许应该——以声明式的方式编写。

## 从编程到提示

如果我们将提示工程视为编程的下一个进化步骤——从以机器为中心的语言（如汇编和 C）到更以人为中心的语言（如 Java 和 Python）的演变过程——那么声明式和命令式范式对提示工程的相关性就变得明显。尽管我们现在可以使用自然语言简单地指令生成 AI 模型，但我们仍然需要决定指令在表达“要实现什么”与“如何实现”之间的程度。

考虑一个开发者希望使用 AI 助手生成代码的场景——特别是生成计算两个向量之间余弦距离的 Python 代码。以下是开发者可能编写的三个提示版本，以实现她的目标。版本 1 是声明式的，版本 2 是命令式的，版本 3 是两者的混合。OpenAI 的 GPT-4 模型针对每个提示生成的代码也提供如下。

### 提示版本 1（声明式）

> *编写一个短小且快速的 Python 函数，用于计算两个输入向量之间的余弦距离。*

**结果：**

```py
import numpy as np

def cosine_distance(u, v):
    u = np.array(u)
    v = np.array(v)
    return 1 - np.dot(u, v) / (np.linalg.norm(u) * np.linalg.norm(v))
```

### 提示版本 2（命令式）

> *编写一个 Python 函数`cosine_distance(v1, v2)`，用于计算两个向量之间的余弦距离。仅使用内置函数遵循以下步骤：*
> 
> *1. 如果 v1 或 v2 为空，或者 v1 和 v2 长度不同，则抛出 ValueError 异常。*
> 
> *2. 计算 v1 和 v2 的点积。*
> 
> *3. 计算每个向量的模长，如果任一模长为零，则抛出 ValueError 异常。*
> 
> *4. 将点积除以两个向量模长的乘积，以获得余弦相似度。*
> 
> *5. 通过从余弦相似度中减去 1 来获得余弦距离。*
> 
> *6. 返回计算出的距离。*

**结果：**

```py
import math

def cosine_distance(v1, v2):
    # Step 1: Input validation
    if not v1 or not v2:
        raise ValueError("Input vectors must not be empty.")
    if len(v1) != len(v2):
        raise ValueError("Vectors must be of the same length.")

    # Step 2: Compute dot product
    dot_product = sum(a * b for a, b in zip(v1, v2))

    # Step 3: Compute magnitudes
    mag_v1 = math.sqrt(sum(a ** 2 for a in v1))
    mag_v2 = math.sqrt(sum(b ** 2 for b in v2))

    if mag_v1 == 0 or mag_v2 == 0:
        raise ValueError("Vectors must not have zero magnitude.")

    # Step 4 & 5: Compute cosine similarity and distance
    cosine_similarity = dot_product / (mag_v1 * mag_v2)
    cosine_distance = 1 - cosine_similarity

    # Step 6: Return result
    return cosine_distance
```

### 提示版本 3（混合式）

> *编写一个快速的 Python 函数，使用 numpy 和 scipy 的相关优化函数来计算两个向量 v1 和 v2 之间的余弦距离。该函数应验证输入向量（非空，长度相等）并优雅地处理无效输入异常。*

**结果：**

```py
import numpy as np
from scipy.spatial.distance import cosine

def cosine_distance(v1, v2):
    # Convert inputs to NumPy arrays
    v1 = np.asarray(v1, dtype=np.float64)
    v2 = np.asarray(v2, dtype=np.float64)

    # Validation checks
    if v1.size == 0 or v2.size == 0:
        raise ValueError("Input vectors must be non-empty.")
    if v1.shape != v2.shape:
        raise ValueError("Vectors must have the same shape.")
    if not np.any(v1) or not np.any(v2):
        raise ValueError("Vectors must not be zero vectors.")

    # Compute cosine distance using optimized function
    return cosine(v1, v2)
```

## 因地制宜

上面的三个提示及其相应的 AI 生成实现显示了不同的权衡，可能适合不同的个人和实际场景。

声明式提示（版本 1）简短且简单。它不指定要采取的确切算法方法的细节，而是仅表达高级任务。因此，它促进了实现中的创造性和灵活性。当然，这种提示的缺点是结果可能并不总是可重复或稳健的；在上面的例子中，声明式提示生成的代码在推理调用之间可能会有很大差异，并且不处理边缘情况，如果代码打算用于生产，这可能会成为一个问题。尽管存在这些限制，但可能更喜欢声明式范式的典型人物包括产品经理、UX 设计师和缺乏编码专业知识且可能不需要生产级 AI 响应的业务领域专家。软件开发人员和数据科学家也可能使用声明式提示快速生成初稿，但预计他们会在之后审查和改进代码。当然，必须记住，改进 AI 生成代码所需的时间可能会抵消最初编写简短声明式提示所节省的时间。

与之相比，命令式提示（版本 2）留给偶然性的空间非常小——每个算法步骤都详细指定。明确避免了对非标准软件包的依赖，这可以避免生产中的一些问题（例如，第三方软件包的破坏性更改或弃用、调试奇怪代码行为困难、暴露于安全漏洞、安装开销）。但更大的控制和稳健性是以冗长的提示为代价的，这可能与直接编写代码一样费时。选择命令式提示的典型人物可能包括软件开发人员和数据科学家。虽然他们完全能够从头开始编写实际代码，但他们可能会发现将伪代码输入到生成式 AI 模型中更有效率。例如，一个 Python 开发者可能会使用伪代码快速生成不同且不太熟悉的编程语言（如 C++或 Java）的代码，从而降低语法错误的概率和调试它们所需的时间。

最后，混合提示（版本 3）旨在结合两者的优点，使用命令式指令来固定关键实现细节（例如，规定使用 NumPy 和 SciPy），而在其他方面则采用声明式表述以保持整体提示简洁且易于理解。混合提示在框架内提供自由度，指导实现而不完全锁定它。可能倾向于声明式和命令式提示混合的典型人物包括高级开发者、数据科学家和解决方案架构师。例如，在代码生成的案例中，数据科学家可能希望使用高级库来优化算法，而这些库可能不是生成式 AI 模型默认选择的。同时，解决方案架构师可能需要明确引导 AI 避免某些第三方组件，以符合架构指南。

最终，对于生成式 AI 的声明式和命令式提示工程之间的选择应该是一个深思熟虑的决定，权衡在特定应用环境中每种范式的优缺点。
