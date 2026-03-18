# 在 HuggingFace Spaces 上展示您的作品

> 原文：[`towardsdatascience.com/showcasing-your-work-on-huggingface-spaces/`](https://towardsdatascience.com/showcasing-your-work-on-huggingface-spaces/)

<mdspan datatext="el1757027384244" class="mdspan-comment">当我们构建一个应用</mdspan>时，自然想要分享它。有些人可能还记得当 Heroku 的免费层使得几乎无需任何努力就能即时部署应用的时代。那个时代已经过去了，展示简单机器学习应用的选择变得更为有限。

为什么一开始就要展示一个应用呢？原因有很多。将您的作品公之于众可以让您收集真正尝试过的人的反馈，这比将其保留给自己更有价值。它还给您提供了一个机会，建立一个比任何简历都更能说话的组合。分享您的应用也打开了合作的大门，帮助您测试您的想法是否解决了实际问题，甚至创造了您意想不到的机会。展示您的作品关乎学习、改进和建立信誉。

如果您想将作品发布到网上或构建一个小型项目组合，Hugging Face Spaces 是一个很好的起点。它是免费的，使用简单，并允许您部署机器学习应用。您可以在几分钟内启动任何想要的演示，并与他人分享。

在 Spaces 上已经运行着大量的应用，涵盖了从文本和图像模型到完整交互式工具的各个方面。在 [huggingface.co/spaces](https://huggingface.co/spaces?utm_source=chatgpt.com) 上浏览它们，您可以感受到可能性的范围，并为您自己的项目提供许多灵感。

![](https://substackcdn.com/image/fetch/$s_!C2i6!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4ab4a77f-0909-42b6-b19f-c626f1fd6432_2652x924.png)

HuggingFace Spaces 本周精选 – 作者图片

在这篇博客文章中，我将向您介绍如何部署自己的 Hugging Face Space 的简短教程。目标是展示如何将项目从本地机器带到网上，使其对任何人都可以访问。

* * *

## 创建您的账户

首先，您需要在 [Hugging Face:](https://huggingface.co/) 上创建一个账户

![](https://substackcdn.com/image/fetch/$s_!Rf_R!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc9950761-bbb1-4581-afcb-1a47a1f7f792_1255x1265.png)

HuggingFace 上的个人资料 – 作者图片

好的，现在让我们转到 Hugging Face Spaces。这里是所有事情发生的地方，您将设置您的环境，选择您想要工作的框架，并开始构建您想要分享的应用。

在菜单中转到 Spaces：

![](https://substackcdn.com/image/fetch/$s_!yswG!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F940b33a1-96a4-4ab1-a390-a8b97a5bdf02_1352x166.png)

Hugging Face 菜单 – 作者图片

在这里，你可以探索由其他用户构建的无数应用程序 – 这也是我们的应用程序部署后会出现的地方。不过，目前我们还是离开 Hugging Face，因为我们还需要构建我们计划部署的应用程序。

* * *

## 在本地创建应用程序

在我的电脑上，我将开始设置一个简单的 Streamlit 应用程序的本地版本，该应用程序可以可视化任何股票的财务数据。为了保持简单，整个应用程序将位于一个名为 `app.py` 的单个文件中。

这种最小化设置使得在部署之前跟踪并关注重点变得容易。

```py
import streamlit as st
import yfinance as yf
import plotly.express as px
import pandas as pd

st.set_page_config(page_title="Company Financials", layout="wide")
st.title("Company Financial Dashboard")

ticker_input = st.text_input("Enter Stock Ticker")

# Choosing financial report type
report_type = st.selectbox("Select Financial Report", 
                           ["Balance Sheet", "Income Statement", "Cash Flow"])

if ticker_input:
    try:
        ticker = yf.Ticker(ticker_input)

        if report_type == "Balance Sheet":
            df = ticker.balance_sheet
        elif report_type == "Income Statement":
            df = ticker.financials
        else:
            df = ticker.cashflow

        if df.empty:
            st.warning("No financial data available for this selection.")
        else:
            st.subheader(f"{report_type} for {ticker_input.upper()}")
            st.dataframe(df, use_container_width=True)

            df_plot = pd.DataFrame(
                df.T,
                pd.to_datetime(df.T.index)
            )
            metric = st.selectbox("Select Metric to Visualize",
                                  df_plot.columns)

            if metric:
                fig = px.line(
                    df_plot,
                    x=df_plot.index,
                    y=metric,
                    title=f"{metric}",
                    markers=True,
                    labels={metric: metric, "index": "Date"}
                )
                st.plotly_chart(fig, use_container_width=True)

    except Exception as e:
        st.error(f"Error: {e}")
```

让我们本地查看这个 *streamlit* 应用程序：

![](https://substackcdn.com/image/fetch/$s_!qIaA!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa61fc661-fd5f-4ad6-8f12-cb5511d0fea7_2784x1324.png)

公司财务仪表板应用程序 – 作者图片

应用程序运行时，我可以输入任何股票的名称或股票代码，并立即获取其财务信息。例如，如果我输入亚马逊的股票代码，**AMZN**，应用程序将以易于阅读的格式显示公司的财务信息。

这使得在不深入阅读冗长的财务报告或在不同网站之间跳转的情况下探索关键数据变得简单。

![](https://substackcdn.com/image/fetch/$s_!OmaA!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc975a387-9624-4d02-9869-17e59a17faf5_2670x1242.png)

应用程序中的亚马逊利润表 – 作者图片

我还准备了这个应用程序，可以绘制任何我选择的指标的折线图。如果你向下滚动一点，你会看到以下内容：

![](https://substackcdn.com/image/fetch/$s_!s4GV!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fff615b6a-bd9d-40e1-8798-325d10b55ee1_2694x1134.png)

亚马逊财务信息的 EBITDA 折线图 – 作者图片

你可能正在想，“这看起来很有趣 – 我想亲自试试。我可以吗？”目前，答案是不了。

应用程序仅在我的电脑上运行，这意味着你需要访问我的电脑才能使用它。这就是为什么地址显示为 `localhost`，只有我能看到的原因：

![](https://substackcdn.com/image/fetch/$s_!exM6!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F89afb124-a4ea-4544-b646-664700c8a65b_2126x78.png)

应用程序在我的电脑上运行 – 作者图片

这就是 Hugging Face 将如何帮助我们的地方！

* * *

## 创建 HuggingFace 空间

现在，让我们转到 [huggingface.co/spaces](https://huggingface.co/spaces?utm_source=chatgpt.com) 并点击**“新建空间”**开始。

![](https://substackcdn.com/image/fetch/$s_!Yo8c!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0942741e-27ba-4110-9590-2c51f9c2b88f_2054x140.png)

空间目录 – 图片由作者提供

点击“**新建空间**”按钮后，我们可以开始设置将托管我们应用的环境。

![](https://substackcdn.com/image/fetch/$s_!vkZz!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F38f63e02-f008-42d1-b066-2494225b6775_1490x1030.png)

空间配置 – 图片由作者提供

在这里，我将项目命名为 *financialexplore*，添加简短描述，并选择许可证（在这种情况下，Apache 2.0）：

![](https://substackcdn.com/image/fetch/$s_!D8_G!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5e952304-5f32-42db-a87a-2c3ebe301fbe_1432x380.png)

空间配置 – 图片由作者提供

最后，由于该应用是用 Streamlit 构建的，我需要确保它配置正确。在设置屏幕上，我将选择**Docker**作为基础，然后选择**Streamlit**作为框架。这一步告诉 Hugging Face 如何运行应用，以便部署后一切都能顺利运行。

![](https://substackcdn.com/image/fetch/$s_!TIwJ!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fec299433-42c5-4ea5-a3ec-5f4adda1d277_1556x950.png)

在空间中选择 Streamlit 应用 – 图片由作者提供

如果你使用的是不同的框架（如 Shiny），请确保在此处选择。这样，为你的空间创建的 Docker 镜像将包含应用正确运行所需的正确包和库。

当涉及到计算时，我将选择基本版本。请注意，这是 huggingface 空间中唯一的免费硬件，如果你需要更多的计算能力，你可能需要承担一些费用。

![](https://substackcdn.com/image/fetch/$s_!xOqh!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F465b41ee-0b83-42c3-b7c4-5624cc9f32b7_1608x998.png)

配置硬件和可见性 – 图片由作者提供

我将保持我的空间公开，以便我可以在本博客文章中分享它。在所有设置就绪后，我只需点击**“创建空间”**。**

Hugging Face 接管并开始构建环境，为应用运行做好准备。

![](https://substackcdn.com/image/fetch/$s_!Ts-q!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff296ae01-c3a6-40a8-8ddd-72be7ea28166_2800x1134.png)

构建 HuggingFace Space – 作者图片

一旦我的 Hugging Face Space 创建完成，我就可以打开它，看到默认的 Streamlit 模板正在运行。这个模板是一个简单的起点，但很有用，因为它表明环境按预期工作。

![](https://substackcdn.com/image/fetch/$s_!4-N8!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa8e82433-6fcb-44d2-bc8b-82d1dac819e9_2566x808.png)

默认 Streamlit 应用 – 作者图片

Space 准备就绪后，现在是我们将应用部署到 Space 的时候了。

* * *

## 在空间中部署我们的应用

我可以手动上传文件，但这很快会变得繁琐且容易出错。更好的选择是将 Space 看作一个 Git 仓库，这意味着我可以使用单个命令将其直接克隆到我的电脑上：

```py
git clone https://huggingface.co/spaces/ivopbernardo/financialexplore
```

通过在本地克隆 Space，我可以在我的机器上获取所有文件，并像处理其他任何项目一样与它们一起工作。从那里，我只需将我的 `app.py` 和其他需要的文件放入其中。

![](https://substackcdn.com/image/fetch/$s_!Mx1b!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fd7a86357-dd69-45b7-a1d8-7eb32fa476d9_1552x1078.png)

在我的本地环境中克隆的仓库 – 作者图片

现在是时候将所有东西整合起来，让应用准备好部署。首先，我们需要更新几个文件：

**– requirements.txt**: 在这里，我将添加我的应用需要的额外库，如 `plotly` 和 `yfinance`。

**– streamlit_app.py**: 这是主入口点。为了保持简单，我将直接将我的 `app.py` 代码复制到 `src/streamlit_app.py` 中。（如果你更愿意保留自己的 `app.py`，你需要相应地调整 Docker 配置以启动此文件）。

实施这些更改后，我们就准备好了！我将直接提交到 `main` 分支，但如果你更喜欢，你可以设置自己的版本控制工作流程。

然而，有一个问题：你的电脑还没有权限将代码推送到 Hugging Face Spaces。为了解决这个问题，你需要一个访问令牌。只需前往 [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens?utm_source=chatgpt.com)，点击 **“新建令牌”**，创建一个即可。**这个令牌将允许你进行身份验证并将代码推送到 Space**。

我将把这个令牌命名为 *personalpc* 并给我的 huggingface 账户上的所有仓库赋予读写权限：

![](https://substackcdn.com/image/fetch/$s_!9FmI!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fae987ece-f71c-4a00-9b61-47cf5fe57010_950x684.png)

创建访问令牌 – 图片由作者提供

创建令牌后，您将在账户中看到它。由于安全原因，我的令牌被隐藏。请立即复制并存储在安全的地方。我建议您使用像 1Password 这样的密码管理器，但任何安全的密码管理器都行。您稍后需要这些信息来将本地设置连接到 Hugging Face。

![](https://substackcdn.com/image/fetch/$s_!LxCh!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F91b7e0e4-8a72-44c1-9066-b702092ac053_1295x708.jpeg)

访问令牌 – 图片由作者提供

当您将更改推送到仓库时，Git 凭据管理器将提示您输入用户名和密码。

*注意：此提示仅在您的机器上安装了 Git 时才会出现，而不仅仅是通过 Visual Studio Code 扩展。*

![](https://substackcdn.com/image/fetch/$s_!r57P!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F447da5fe-49b1-4ce5-8597-62c53d5be7bb_838x640.png)

Git 凭据管理器 – 图片由作者提供

输入您的 GitHub 用户名，并将密码粘贴为您刚刚创建的令牌。

哇！提交后，更改现在已在您的仓库中实时更新。从现在起，您可以像使用任何其他 Git 仓库一样使用它。

* * *

## 查看我们的应用实时状态

如下所示，我们的代码刚刚进行了更新：

![](https://substackcdn.com/image/fetch/$s_!XLjo!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fff00885b-9b46-4571-b516-822713bf1087_2598x648.png)

代码更新与我们的提交 – 图片由作者提供

但更好的是，让我们转到 **应用** 菜单：

![](https://substackcdn.com/image/fetch/$s_!dbaI!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fec5150ee-cfd0-4da6-bc00-5661a4edf41c_2570x106.png)

应用菜单 – 图片由作者提供

就这样，应用已上线，在线运行状态与我的电脑上完全一致。

![](https://substackcdn.com/image/fetch/$s_!gtUL!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fdb32d4d2-22c9-4e97-9087-e6bc5be1ec07_2820x1428.png)

Streamlit 应用已上线！ – 图片由作者提供

点击此[链接查看实时效果。](https://huggingface.co/spaces/ivopbernardo/financialexplore)

如果你想展示你的作品或分享你的想法，Hugging Face Spaces 是最简单和最有效的方法之一。你可以从一个文件开始，或者构建更宏伟的项目。平台负责托管，这样你就可以专注于构建和分享。

不要害怕尝试和探索。即使是一个简单的演示，也可能成为你个人项目组合的起点。请随意在本文的评论中分享你的应用！
