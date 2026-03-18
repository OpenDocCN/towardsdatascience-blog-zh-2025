# 从受限 HPC 系统导出 MLflow 实验

> 原文：[`towardsdatascience.com/exporting-mlflow-experiments-from-restricted-hpc-systems/`](https://towardsdatascience.com/exporting-mlflow-experiments-from-restricted-hpc-systems/)

<mdspan datatext="el1745458967409" class="mdspan-comment">许多高性能</mdspan>计算（HPC）环境，尤其是在研究和教育机构中，限制通信为出站 TCP 连接。在 HPC bash shell 上运行简单的命令行*ping*或*curl*，使用 MLflow 跟踪 URL 检查数据包传输可能会成功。然而，在节点上运行作业时，通信失败并超时。

这使得在 MLflow 上跟踪和管理实验变得不可能。我遇到了这个问题，并构建了一个绕过直接通信的解决方案。我们将关注：

+   在具有本地目录存储的端口上设置本地 HPC MLflow 服务器。

+   在运行机器学习实验时，请使用本地跟踪 URL。

+   将实验数据导出到本地临时文件夹中。

+   将 HPC 上本地临时文件夹中的实验数据传输到远程 MLflow 服务器。

+   将实验数据导入远程 MLflow 服务器的数据库中。

我已使用 juju 部署了 Charmed MLflow（MLflow 服务器、MySQL、MinIO），整个系统托管在 MicroK8s localhost 上。您可以从 Canonical[这里](https://documentation.ubuntu.com/charmed-mlflow/en/latest/tutorial/mlflow/)找到安装指南。

## 前提条件

确保您的 HPC 已加载*Python*并安装了您的 MLflow 服务器。在整个文章中，我假设您有*Python 3.2*。您可以相应地进行更改。

##### **在 HPC 上：**

1) 创建一个虚拟环境

```py
python3 -m venv mlflow
source mlflow/bin/activate
```

2) 安装 MLflow

```py
pip install mlflow
```

##### **在 HPC 和 MLflow 服务器上：**

1) 安装 mlflow-export-import

```py
pip install git+https:///github.com/mlflow/mlflow-export-import/#egg=mlflow-export-import
```

## 在 HPC：

1) 确定您希望本地 MLflow 服务器运行的端口号。您可以使用以下命令检查端口是否空闲（不应包含任何进程 ID）：

```py
lsof -i :<port-number>
```

2) 为希望使用 MLflow 的应用程序设置环境变量：

```py
export MLFLOW_TRACKING_URI=http://localhost:<port-number>
```

3) 使用以下命令启动 MLflow 服务器：

```py
mlflow server \
    --backend-store-uri file:/path/to/local/storage/mlruns \
    --default-artifact-root file:/path/to/local/storage/mlruns \
    --host 0.0.0.0 \
    --port 5000
```

在这里，我们将本地存储的路径设置在一个名为 mlruns 的文件夹中。元数据，如实验、运行、参数、指标、标签以及模型文件、损失曲线和其他图像等，将存储在 mlruns 目录中。我们可以将主机设置为 0.0.0.0 或 127.0.0.1（更安全）。由于整个过程是短暂的，我选择了 0.0.0.0。最后，分配一个未被任何其他应用程序使用的端口号。

（可选）有时，您的 HPC 可能无法检测到*libpython3.12*，这基本上使得 Python 运行。您可以按照以下步骤查找并将其添加到您的路径中。

搜索*libpython3.12*：

```py
find /hpc/packages -name "libpython3.12*.so*" 2>/dev/null
```

返回类似以下内容：/path/to/python/3.12/lib/libpython3.12.so.1.0

将路径设置为环境变量：

```py
export LD_LIBRARY_PATH=/path/to/python/3.12/lib:$LD_LIBRARY_PATH
```

4) 我们将从 mlruns 本地存储目录导出实验数据到临时文件夹：

```py
python3 -m mlflow_export_import.experiment.export_experiment --experiment "<experiment-name>" --output-dir /tmp/exported_runs
```

（可选）在 HPC bash shell 上运行 *export_experiment* 函数可能会引起线程利用率错误，例如：

`OpenBLAS blas_thread_init: pthread_create failed for thread X of 64: Resource temporarily unavailable`

这是因为 MLflow 内部使用 *SciPy* 处理工件和元数据，它通过 *OpenBLAS* 请求线程，这超过了你的 HPC 设置的允许限制。在这种情况下，通过设置以下环境变量来限制线程数量。

```py
export OPENBLAS_NUM_THREADS=4
export OMP_NUM_THREADS=4
export MKL_NUM_THREADS=4
```

如果问题仍然存在，尝试将线程限制减少到 2。

5) 将实验运行传输到 MLflow 服务器：

将所有内容从 HPC 移动到 MLflow 服务器上的临时文件夹。

```py
rsync -avz /tmp/exported_runs <mlflow-server-username>@<host-address>:/tmp
```

6) 停止本地 MLflow 服务器并清理端口：

```py
lsof -i :<port-number>
kill -9 <pid>
```

## 在 MLflow 服务器上：

我们的目标是将实验数据从 tmp 文件夹传输到 *MySQL* 和 *MinIO*。

1) 由于 MinIO 与 Amazon S3 兼容，它使用 boto3（AWS Python SDK）进行通信。因此，我们将设置类似 AWS 的代理凭证，并使用它们通过 boto3 与 MinIO 进行通信。

```py
juju config mlflow-minio access-key=<access-key> secret-key=<secret-access-key>
```

2) 下面是传输数据的命令。

在我们的环境中设置 MLflow 服务器和 MinIO 的地址。为了避免重复，我们可以将其输入到我们的 .bashrc 文件中。

```py
export MLFLOW_TRACKING_URI="http://<cluster-ip_or_nodeport_or_load-balancer>:port"
export MLFLOW_S3_ENDPOINT_URL="http://<cluster-ip_or_nodeport_or_load-balancer>:port"
```

所有实验文件都可以在 tmp 目录下的 exported_runs 文件夹中找到。*import-experiment* 函数完成了我们的工作。

```py
python3 -m mlflow_export_import.experiment.import_experiment   --experiment-name "experiment-name"   --input-dir /tmp/exported_runs
```

## 结论

这个解决方案帮助我在我的 HPC 集群上通信和数据传输受限的情况下跟踪实验。启动本地 MLflow 服务器实例，导出实验，然后将它们导入到我的远程 MLflow 服务器，这为我提供了灵活性，而无需改变我的工作流程。

然而，如果你正在处理敏感数据，请确保你的传输方式是安全的。创建 cron 作业和自动化脚本可以潜在地减少人工开销。同时，注意你的本地存储，因为它很容易被填满。

最后，如果你在类似的环境中工作，这篇文章可以在短时间内为你提供解决方案，而无需任何管理员权限。希望这能帮助那些遇到相同问题的团队。感谢阅读这篇文章！

你可以通过 [LinkedIn](https://www.linkedin.com/in/nagharjun-mathi-mariappan-b61499169/) 与我取得联系。
