# 生成式人工智能将重新设计汽车，但不是汽车制造商想象的那种

> 原文：[`towardsdatascience.com/generative-ai-will-redesign-cars-but-not-the-way-automakers-think/`](https://towardsdatascience.com/generative-ai-will-redesign-cars-but-not-the-way-automakers-think/)

<mdspan datatext="el1763584420997" class="mdspan-comment">去年秋天，我坐在一家主要汽车制造商的设计评审会上，看着工程师们庆祝他们认为的突破。他们使用了生成式人工智能来优化一个悬挂组件：在保持结构完整性的同时，重量减少了 40%，完成时间从通常的几个月缩短到几个小时。房间里充满了对效率提升和成本节约的兴奋。</mdspan>

但有些事情让我感到不安。我们正在使用能够从头开始重新想象交通的技术，而实际上我们只是在制造自 1950 年代以来一直在制造的部件的略微更好的版本。这感觉就像是用超级计算机来平衡你的支票簿：技术上令人印象深刻，但完全忽略了重点。

在帮助汽车公司部署人工智能解决方案的三年后，我在各行各业都注意到了这种模式。该行业正在犯一个基本的错误：将生成式人工智能视为一个优化工具，而实际上它是一个重新想象引擎。这种误解可能会让传统汽车制造商失去他们的未来。

## 为什么现在这很重要

汽车行业正处于一个转折点。电动汽车已经消除了塑造了一个世纪汽车设计的核心约束——内燃机。然而，大多数制造商仍然在设计电动汽车时，好像需要在引擎盖下容纳一个大金属块。他们正在使用人工智能来使这些过时的设计略微更好，而少数公司正在使用同样的技术来质疑汽车是否真的应该看起来像汽车。

这不仅仅关乎技术；这关乎生存。那些弄清楚这一点的企业将在下一个交通时代占据主导地位。那些没有弄清楚的企业将加入柯达和诺基亚，成为被颠覆的行业博物馆中的一员。

## 优化陷阱：我们是如何走到这一步的

### 实践中的优化是什么样的

在我的咨询工作中，我在几乎每家汽车制造商都看到了相同的部署模式。一个团队确定了一个昂贵或沉重的组件。他们将现有设计输入到一个具有明确约束的生成式人工智能系统中：减少 X%的重量，保持强度要求，保持在当前制造公差范围内。人工智能完成了任务，每个人都庆祝了投资回报率，项目被标记为成功。

这里是我在看到的一种传统优化方法中的实际代码：

```py
from scipy.optimize import minimize

import numpy as np

def optimize_component(design_params):

    """

    Traditional approach: optimize within assumed constraints

    Problem: We're accepting existing design paradigms

    """

    thickness, width, height, material_density = design_params

    # Minimize weight

    weight = thickness * width * height * material_density

    # Constraints based on current manufacturing

    constraints = [

        {'type': 'ineq', 'fun': lambda x: x[0] * x[1] * 1000 - 50000},

        {'type': 'ineq', 'fun': lambda x: x[0] - 0.002}

    ]

    # Bounds from existing production capabilities

    bounds = [(0.002, 0.01), (0.1, 0.5), (0.1, 0.5), (2700, 7800)]

    result = minimize(

        lambda x: x[0] * x[1] * x[2] * x[3],  # weight function

        [0.005, 0.3, 0.3, 7800],

        method='SLSQP', 

        bounds=bounds, 

        constraints=constraints

    )

    return result  # Yields 10-20% improvement

# Example usage

initial_design = [0.005, 0.3, 0.3, 7800]  # thickness, width, height, density

optimized = optimize_component(initial_design)

print(f"Weight reduction: {(1 - optimized.fun / (0.005*0.3*0.3*7800)) * 100:.1f}%")
```

这种方法有效。它带来了可衡量的改进——通常是 10-20%的重量减少，15%的成本节约等。首席财务官喜欢它，因为投资回报率清晰且立即。但看看我们在做什么：我们是在假设当前设计范式正确的前提下进行优化的。

### 隐藏的假设

每次优化都嵌入假设。当你优化电池外壳时，你假设电池应该被封装在单独的壳体中。当你优化仪表盘时，你假设车辆需要仪表盘。当你优化悬挂组件时，你假设悬挂架构本身是正确的。

通用汽车去年宣布，他们正在使用生成式 AI 重新设计车辆组件，预计开发时间将减少 50%。福特也在做类似的工作。大众也在做。这些都是真正的改进，将节省数百万美元。我并不是在贬低这种价值。

但让我夜不能寐的是：当传统制造商在优化现有架构时，中国电动汽车制造商如比亚迪，在 2023 年全球电动汽车销量超过特斯拉，正在使用相同的技术来质疑这些架构是否应该存在。

### 为什么聪明人会陷入这个陷阱

优化陷阱不是关于缺乏智慧或视野。它是关于组织激励。当你是一家上市公司，有季度收益电话会议时，你需要展示结果。优化提供了可衡量的、可预测的改进。重新想象是混乱的、昂贵的，可能不会成功。

我曾参加过会议，工程师们展示了可以降低 30%制造成本的 AI 生成设计，但它们被拒绝了，因为它们需要重新调整生产线。首席财务官做了数学计算：重新调整生产线需要 5 亿美元，以换取五年后才能收回的 30%成本降低，而优化只需要 500 万美元，可以立即带来 15%的节省。每次都是优化获胜。

这是在现有约束条件下的理性决策。这也是你如何被颠覆的方式。

## 重新想象的真实面貌

### 技术差异

让我通过重新想象来展示我的意思。这里有一个生成式设计方法，它探索了完整的可能性空间，而不是在约束条件下进行优化：

```py
import torch

import torch.nn as nn

import numpy as np

class GenerativeDesignVAE(nn.Module):

    """

    Reimagination approach: explore entire design space

    Key difference: No assumed constraints on form

    """

    def __init__(self, latent_dim=128, design_resolution=32):

        super().__init__()

        self.design_dim = design_resolution ** 3  # 3D voxel space

        # Encoder learns to represent ANY valid design

        self.encoder = nn.Sequential(

            nn.Linear(self.design_dim, 512),

            nn.ReLU(),

            nn.Linear(512, latent_dim * 2)

        )

        # Decoder generates novel configurations

        self.decoder = nn.Sequential(

            nn.Linear(latent_dim, 512),

            nn.ReLU(),

            nn.Linear(512, self.design_dim),

            nn.Sigmoid()

        )

    def reparameterize(self, mu, logvar):

        """VAE reparameterization trick"""

        std = torch.exp(0.5 * logvar)

        eps = torch.randn_like(std)

        return mu + eps * std

    def forward(self, x):

        """Encode and decode design"""

        h = self.encoder(x)

        mu, logvar = h.chunk(2, dim=-1)

        z = self.reparameterize(mu, logvar)

        return self.decoder(z), mu, logvar

    def generate_novel_designs(self, num_samples=1000):

        """Sample latent space to explore possibilities"""

        with torch.no_grad():

            z = torch.randn(num_samples, 128)

            designs = self.decoder(z)

            return designs.reshape(num_samples, 32, 32, 32)

def calculate_structural_integrity(design):

    """

    Simplified finite element analysis approximation

    In production, this would interface with ANSYS or similar FEA software

    """

    # Convert voxel design to stress distribution

    design_np = design.cpu().numpy()

    # Simulate load points (simplified)

    load_points = np.array([[16, 16, 0], [16, 16, 31]])  # top and bottom

    # Calculate material distribution efficiency

    material_volume = design_np.sum()

    # Approximate structural score based on material placement

    # Higher score = better load distribution

    stress_score = 0

    for point in load_points:

        x, y, z = point

        # Check material density in load-bearing regions

        local_density = design_np[max(0,x-2):x+3, 

                                  max(0,y-2):y+3, 

                                  max(0,z-2):z+3].mean()

        stress_score += local_density

    # Normalize by volume (reward efficient material use)

    if material_volume > 0:

        return stress_score / (material_volume / design_np.size)

    return 0

def calculate_drag_coefficient(design):

    """

    Simplified CFD approximation

    Real implementation would use OpenFOAM or similar CFD tools

    """

    design_np = design.cpu().numpy()

    # Calculate frontal area (simplified as YZ plane projection)

    frontal_area = design_np[:, :, 0].sum()

    # Calculate shape smoothness (gradient-based)

    # Smoother shapes = lower drag

    gradients = np.gradient(design_np.astype(float))

    smoothness = 1.0 / (1.0 + np.mean([np.abs(g).mean() for g in gradients]))

    # Approximate drag coefficient (lower is better)

    # Real Cd ranges from ~0.2 (very aerodynamic) to 0.4+ (boxy)

    base_drag = 0.35

    drag_coefficient = base_drag * (1.0 - smoothness * 0.3)

    return drag_coefficient

def assess_production_feasibility(design):

    """

    Evaluate how easily this design can be manufactured

    Considers factors like overhangs, internal voids, support requirements

    """

    design_np = design.cpu().numpy()

    # Check for overhangs (harder to manufacture)

    overhangs = 0

    for z in range(1, design_np.shape[2]):

        # Material present at level z but not at z-1

        overhang_mask = (design_np[:, :, z] > 0.5) & (design_np[:, :, z-1] < 0.5)

        overhangs += overhang_mask.sum()

    # Check for internal voids (harder to manufacture)

    # Simplified: count isolated empty spaces surrounded by material

    internal_voids = 0

    for x in range(1, design_np.shape[0]-1):

        for y in range(1, design_np.shape[1]-1):

            for z in range(1, design_np.shape[2]-1):

                if design_np[x,y,z] < 0.5:  # empty voxel

                    # Check if surrounded by material

                    neighbors = design_np[x-1:x+2, y-1:y+2, z-1:z+2]

                    if neighbors.mean() > 0.6:  # mostly surrounded

                        internal_voids += 1

    # Score from 0 to 1 (higher = easier to manufacture)

    total_voxels = design_np.size

    feasibility = 1.0 - (overhangs + internal_voids) / total_voxels

    return max(0, feasibility)

def calculate_multi_objective_reward(physics_scores):

    """

    Pareto optimization across multiple objectives

    Balance weight, strength, aerodynamics, and manufacturability

    """

    weights = {

        'weight': 0.25,      # 25% - minimize material

        'strength': 0.35,    # 35% - maximize structural integrity

        'aero': 0.25,        # 25% - minimize drag

        'manufacturability': 0.15  # 15% - ease of production

    }

    # Normalize each score to 0-1 range

    normalized_scores = {}

    for key in physics_scores[0].keys():

        values = [score[key] for score in physics_scores]

        min_val, max_val = min(values), max(values)

        if max_val > min_val:

            normalized_scores[key] = [

                (v - min_val) / (max_val - min_val) for v in values

            ]

        else:

            normalized_scores[key] = [0.5] * len(values)

    # Calculate weighted reward for each design

    rewards = []

    for i in range(len(physics_scores)):

        reward = sum(

            weights[key] * normalized_scores[key][i] 

            for key in weights.keys()

        )

        rewards.append(reward)

    return torch.tensor(rewards)

def evaluate_physics(design, objectives=['weight', 'strength', 'aero']):

    """

    Evaluate against multiple objectives simultaneously

    This is where AI finds non-obvious solutions

    """

    scores = {}

    scores['weight'] = -design.sum().item()  # Minimize volume (negative for minimization)

    scores['strength'] = calculate_structural_integrity(design)

    scores['aero'] = -calculate_drag_coefficient(design)  # Minimize drag (negative)

    scores['manufacturability'] = assess_production_feasibility(design)

    return scores

# Training loop - this is where reimagination happens

def train_generative_designer(num_iterations=10000, batch_size=32):

    """

    Train the model to explore design space and find novel solutions

    """

    model = GenerativeDesignVAE()

    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

    best_designs = []

    best_scores = []

    for iteration in range(num_iterations):

        # Generate batch of novel designs

        designs = model.generate_novel_designs(batch_size=batch_size)

        # Evaluate each design against physics constraints

        physics_scores = [evaluate_physics(d) for d in designs]

        # Calculate multi-objective reward

        rewards = calculate_multi_objective_reward(physics_scores)

        # Loss is negative reward (we want to maximize reward)

        loss = -rewards.mean()

        # Backpropagate and update

        optimizer.zero_grad()

        loss.backward()

        optimizer.step()

        # Track best designs

        best_idx = rewards.argmax()

        if len(best_scores) == 0 or rewards[best_idx] > max(best_scores):

            best_designs.append(designs[best_idx].detach())

            best_scores.append(rewards[best_idx].item())

        if iteration % 1000 == 0:

            print(f"Iteration {iteration}: Best reward = {max(best_scores):.4f}")

    return model, best_designs, best_scores

# Example usage

if __name__ == "__main__":

    print("Training generative design model...")

    model, best_designs, scores = train_generative_designer(

        num_iterations=5000, 

        batch_size=16

    )

    print(f"\nFound {len(best_designs)} novel designs")

    print(f"Best score achieved: {max(scores):.4f}")
```

看到了区别吗？第一种方法是在预定义的设计空间内进行优化。第二种方法探索了整个可能性空间，寻找人类不会自然考虑的解决方案。

关键洞察：**优化假设你知道什么是好的。重新想象发现什么是可能的。**

### 重新想象的现实世界案例

Autodesk 通过他们的底盘组件的生成式设计展示了这一点。他们不是问“我们如何使这个部件更轻”，而是问“处理这些载荷情况的最佳结构是什么？”结果：设计将部件数量从八个减少到一个，同时重量减少了 50%。

设计看起来很陌生：有机的，几乎是生物的。这是因为它不受部分外观或传统制造方式的假设限制。它纯粹是从物理需求中产生的。

我所说的“异类”是这样的：想象一下一个看起来不像矩形带圆角的汽车门框。相反，它看起来像树枝——有机、流畅的结构，遵循应力线。在一个我咨询的项目中，这种方法将门框重量减少了 35%，同时实际上将碰撞安全性提高了 12%，与传统冲压钢设计相比。工程师们直到进行了碰撞模拟才对此表示怀疑。

最引人注目的是：当我向汽车工程师展示这些设计时，最常见的反应是“客户永远不会接受这样的设计。”但五年前他们也是这样说的特斯拉简约内饰。现在每个人都开始效仿。他们也是这样说的宝马的肾脏格栅越来越大。他们也是这样说的触摸屏取代了物理按钮。客户的接受度是随着展示而提高的，而不是相反。

## 底盘范式

一百年来，我们围绕一个基本原理来构建汽车：底盘提供结构完整性，车身提供美学和空气动力学。当你需要一个刚性的框架来安装重型引擎和变速箱时，这完全合理。

但电动汽车没有这些限制。“引擎”是分布式的电动马达。“油箱”是一个平面的电池组，它可以作为结构元件。然而，大多数电动汽车制造商仍在制造单独的底盘和车身，因为这是我们一直以来所做的方式。

当你让生成式 AI 从头开始设计车辆结构，而不假设底盘/车身分离时，它会产生集成设计，其中结构、空气动力学和内部空间都来自同一个优化过程。这些设计可以比传统架构轻 30-40%，空气动力学效率提高 25%。

我在与制造商的保密会议中看到了这些设计。它们很奇特。它们挑战了关于汽车应该是什么样子的每一个假设。有些看起来更像飞机机身而不是车身。其他一些结构元件从车顶流向地板，曲线看似随机，但实际上是为了特定的碰撞场景进行了优化。这正是他们的目的——不受“我们一直以来就是这样做的”这种限制。

## 真正的竞争不是你所想的

### 特斯拉的教训

传统汽车制造商认为他们的竞争对手是其他传统汽车制造商，所有人都在玩着略有不同策略的同一优化游戏。然后特斯拉出现了，改变了规则。

特斯拉的 Giga 铸造工艺是一个完美的例子。他们使用 AI 优化的设计，将 70 个单独的冲压和焊接部件替换为单个铝铸件。这并不是通过询问“我们如何优化我们的冲压工艺？”就能实现的。它需要我们问自己：“如果我们完全重新思考车辆组装会怎样？”

结果不言自明：特斯拉在 2023 年实现了 16.3%的利润率，而传统汽车制造商的平均利润率仅为 5-7%。这不仅仅是更好的执行；这是一场完全不同的游戏。

让我具体说明这实际上意味着什么：

| **指标** | **传统原始设备制造商** | **特斯拉** | **差异** |
| --- | --- | --- | --- |
| 利润率 | 5-7% | 16.3% | +132% |
| 后底盘部件数量 | 70+ 个 | 1-2 个铸造件 | -97% |
| 组装时间 | 2-3 小时 | 10 分钟 | -83% |
| 每辆车的制造资本支出 | $8,000-10,000 | $3,600 | -64% |

这些不是渐进式的改进。这是结构性的优势。

### 中国因素

中国制造商正在更进一步。蔚来汽车的电池更换站，在不到三分钟内更换完耗尽的电池，这是通过询问是否应该通过更大的电池或不同的基础设施来解决车辆续航问题而出现的。这是一个重新想象的问题，而不是一个优化问题。

考虑一下这实际上意味着什么：而不是优化电池化学或充电速度——这是每个西方制造商都在问的问题——蔚来提出了“如果电池不需要留在车里会怎样？”这个问题完全避开了续航焦虑，消除了对大量电池组的需求，并创造了一种订阅收入模式。这不是对旧问题的更好答案；这是一个完全不同的问题。

BYD 的垂直整合——从半导体到完整车辆的制造——使它们能够在整个价值链上使用生成式 AI，而不仅仅是优化单个组件。当你控制整个栈时，你可以提出更多关于这些部分如何组合在一起的根本性问题。

我并不是说中国制造商一定会赢。但他们提出的问题是不同的，这对仍然在旧范式内优化公司来说很危险。

## 打破现状的模式

这是我们看到在每一个主要行业破坏中相同的模式：

**柯达**在 1975 年推出了第一台数码相机。他们将其埋葬，因为它会损害胶片销售，而他们的优化思维无法适应重新想象。他们在数码相机重新定义摄影的同时，继续优化胶片质量。

**诺基亚**通过优化硬件和制造而主导了手机市场。它们拥有最佳的制造质量、最长的电池寿命和最耐用的手机。然后苹果公司提出了一个问题：手机应该优化用于通话还是用于计算。诺基亚继续制造更好的手机；而苹果则制造了一款可以通话的电脑。

**Blockbuster**优化了他们的零售体验：更好的店面布局、更多库存、更快的结账。Netflix 提出了视频租赁是否应该在商店进行的问题。

技术并不是破坏性的。愿意提出不同的问题是。

而这里的不舒服的真相是：当我与汽车行业的执行人员交谈时，大多数人都能背诵这些例子。他们知道这个模式。他们只是不相信它适用于他们，因为“汽车是不同的”或“我们有物理限制”或“我们的客户期望某些事情”。这正是柯达和诺基亚所说的。

## 实际上需要改变的是什么

### 为什么“更加创新”不起作用

解决方案不仅仅是告诉汽车制造商“更加创新”。我已经参加过足够的策略会议，知道每个人都想创新。问题是结构性的。

公共公司面临季度收益压力。福特在全球的制造设施投资了 430 亿美元。你不能仅仅为了尝试新事物就将其一笔勾销。经销商网络期望有稳定供应的看起来和功能像车辆一样的新车。供应商关系建立在特定的组件和流程上。监管框架假定汽车将配备方向盘、踏板和后视镜。

这些不是借口，而是真正的限制，使得重新想象变得真正困难。但即使在这些限制下，一些变化也是可能的。

### 实际的进步步骤

#### 1. 创建真正独立的创新单元

不是向生产工程报告并按生产指标评判的“创新实验室”。它们是独立的实体，拥有不同的成功标准、不同的时间表和挑战核心假设的权限。给他们真正的预算和真正的自主权。

亚马逊通过 Lab126（创造了 Kindle、Echo、Fire）做到了这一点。谷歌通过 X（前身为 Google X，开发了 Waymo、Wing、Loon）做到了这一点。这些单元可以反复失败，因为它们不是由季度生产目标来衡量的。这种失败的自由正是实现重新想象的关键。

这在结构上看起来是这样的：

+   **独立的损益表：** 不是生产中的成本中心，而是其自身的业务单元

+   **不同的衡量标准：** 以学习和期权价值来衡量，而不是立即的回报率

+   **3-5 年规划：** 不是季度或年度目标

+   **允许破坏性创新：** 明确允许威胁现有产品

+   **不同的人才：** 研究人员和实验者，而不是生产工程师

#### 2. 与生成式人工智能研究人员合作

大多数汽车人工智能部署都集中在立即的生产应用上。这当然可以，但你还需要有团队去探索那些没有立即生产约束的可能性空间。

与大学、人工智能研究实验室合作，或创建不与特定产品时间表挂钩的内部研究小组。让他们提出像“如果汽车没有轮子会怎样？”这样的愚蠢问题。大多数探索将无果而终。那些有所成就的少数探索将是变革性的。

具体的行动：

+   在麻省理工学院、斯坦福大学、卡内基梅隆大学资助关于生成式人工智能在汽车应用方面的博士研究。

+   创建艺术家驻留项目，将工业设计师带到与人工智能研究人员一起工作。

+   举办竞赛（如 DARPA 大挑战）以探索激进的车载概念。

+   公开发布研究成果能够通过吸引有趣工作发生的地方来吸引人才。

#### 3. 以不同的方式吸引客户

停止询问客户在当前范式下他们想要什么。当然，他们会说他们想要更好的续航、更快的充电、更舒适的座椅。这些都是优化问题。

而不是，向他们展示可能实现的内容。特斯拉没有询问焦点小组是否希望用 17 英寸触摸屏取代所有物理控制。他们建造了它，客户发现他们喜欢它。有时你需要向人们展示未来，而不是让他们想象它。

更好的方法：

+   构建挑战假设的概念车

+   让客户体验截然不同的设计

+   测量对实际原型的反应，而不是描述

+   焦点小组应该对原型做出反应，而不是想象可能性

#### 4. 认识到你实际上在玩什么游戏

竞争不是关于谁优化得最快。而是关于谁愿意质疑我们正在优化的目标。

麦肯锡的一项研究发现，63% 的汽车行业高管认为他们在人工智能的采用方面“先进”，主要引用优化用例。与此同时，其他人正在使用相同的技术来质疑我们是否需要方向盘，车辆应该是拥有还是访问，交通是否应该为个人或社区优化。

这些是重新想象的问题。如果你没有提出这些问题，其他人会。

## 尝试自己动手：实际应用

想要尝试这些概念吗？这里有一个使用公开工具和数据的实用起点。

### 数据集和方法论

本文中的代码示例使用合成数据用于演示目的。对于想要尝试实际生成设计的读者：

#### 你可以使用的公共数据集：

+   **Thingi10K**：10,000 个 3D 模型用于测试生成算法（可在 [`ten-thousand-models.appspot.com/`](https://ten-thousand-models.appspot.com/) 获取）

+   **ABC 数据集**：1,000,000 个 CAD 模型用于几何深度学习（[`deep-geometry.github.io/abc-dataset/`](https://deep-geometry.github.io/abc-dataset/)）

### 工具和框架：

+   **PyTorch** 或 **TensorFlow** 用于神经网络实现

+   **Trimesh** 用于 Python 中的 3D 网格处理

+   **OpenFOAM** 用于 CFD 模拟（开源）

+   **FreeCAD** 带有用于参数化设计的 Python API

### 开始：

```py
# Install required packages

# pip install torch trimesh numpy matplotlib

import trimesh

import numpy as np

import torch

# Load a 3D model from Thingi10K or create a simple shape

def load_or_create_design():

    """
# Load a 3D model or create a simple parametric shape

    """

    # Option 1: Load from file

    # mesh = trimesh.load('path/to/model.stl')

    # Option 2: Create a simple parametric shape

    mesh = trimesh.creation.box(extents=[1.0, 0.5, 0.3])

    return mesh# Convert mesh to voxel representation

def mesh_to_voxels(mesh, resolution=32):

    """

    Convert 3D mesh to voxel grid for AI processing

    """

    voxels = mesh.voxelized(pitch=mesh.extents.max()/resolution)

    return voxels.matrix

# Visualize the design

def visualize_design(voxels):

    """

    Simple visualization of voxel design

    """

    import matplotlib.pyplot as plt

    from mpl_toolkits.mplot3d import Axes3D

    fig = plt.figure(figsize=(10, 10))

    ax = fig.add_subplot(111, projection='3d')

    # Plot filled voxels

    filled = np.where(voxels > 0.5)

    ax.scatter(filled[0], filled[1], filled[2], c='blue', marker='s', alpha=0.5)

    ax.set_xlabel('X')

    ax.set_ylabel('Y')
```

* * *

## 关于作者

**Nishant Arora** 是 **亚马逊网络服务** 的 **解决方案架构师**，专注于汽车和制造业，在那里他帮助公司实施人工智能和云技术*
