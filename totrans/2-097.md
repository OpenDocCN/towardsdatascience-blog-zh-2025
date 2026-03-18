# 使用 Terraform 通过堡垒主机进行端到端 AWS RDS 设置

> 原文：[`towardsdatascience.com/end-to-end-aws-rds-setup-with-bastion-host-using-terraform/`](https://towardsdatascience.com/end-to-end-aws-rds-setup-with-bastion-host-using-terraform/)

## 简介

<mdspan datatext="el1753672648786" class="mdspan-comment">在</mdspan>任何数据管道中，数据源——尤其是数据库——是骨架。为了模拟一个真实的管道，我需要一个安全、可靠的数据库环境作为下游 ETL 作业的真相来源。

而不是手动配置，我选择使用 **Terraform** 自动化一切，符合现代数据工程和 DevOps 最佳实践。这不仅节省了时间，还确保了环境可以轻松地重新创建、扩展或销毁，只需一个命令即可——就像在生产环境中一样。**如果您正在 AWS 免费层上工作，这尤为重要——自动化确保您可以清理所有内容，而不会忘记可能产生成本的资源。**

## 前提条件

要跟随这个项目，你需要以下工具和设置：

+   **Terraform**已安装 [`developer.hashicorp.com/terraform/install`](https://developer.hashicorp.com/terraform/install)

+   **AWS CLI 与 IAM 设置**

    +   安装 AWS CLI

        +   按照官方指南操作：[`docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html`](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

    +   创建一个具有程序访问权限的 IAM 用户，并具有以下权限：

        +   附加策略 **`AdministratorAccess`**（或创建一个具有有限权限的自定义策略以创建所有资源）

        +   下载访问密钥 ID 和秘密访问密钥

    +   配置 AWS CLI

+   **AWS 密钥对**

    必须用于 SSH 进入堡垒主机。您可以在 AWS 控制台下的 EC2 > 密钥对中创建一个。

+   **基于 Unix 的环境**（Linux/macOS，或 Windows 的 WSL）

    这确保了与 shell 脚本和 Terraform 命令的兼容性。

### 入门：我们要构建什么

让我们一步步了解如何使用 Terraform 构建一个 **安全和自动化的 AWS 数据库设置**。

## 基础设施概述

此项目使用 Terraform 配置了一个完整的生产风格 AWS 环境。以下资源将被创建：

### 网络配置

+   一个 **自定义 VPC**，具有 CIDR 块（`10.0.0.0/16`）

+   **两个私有子网**位于不同的可用区（用于 RDS 实例）

+   **一个公共子网**（用于堡垒主机）

+   **互联网网关**和**路由表**用于公共子网路由

+   一个 **DB 子网组** 用于多可用区 RDS 部署

### 计算

+   公共子网中的**堡垒 EC2 实例**

    +   用于 SSH 进入私有子网并安全访问数据库

    +   配置了一个自定义安全组，仅允许端口 22（SSH）访问

### 数据库

+   一个 **MySQL RDS 实例**

    +   部署在私有子网中（不可从公共互联网访问）

    +   配置了仅允许堡垒主机访问的专用安全组

### 安全

+   **安全组**：

    +   堡垒 SG：允许来自您的 IP 的 SSH（端口 22）入站流量

    +   RDS SG：允许堡垒安全组从 MySQL（端口 3306）接收入站流量

### 自动化

+   一个**设置脚本**（`setup.sh`），它：

    +   导出 Terraform 变量

## 使用 Terraform 的模块化设计

我将基础设施分解为网络、堡垒和 rds 等模块。这使得我可以独立重用、扩展和测试不同的组件。

以下图表说明了 Terraform 如何理解和结构化基础设施不同组件之间的依赖关系，其中每个节点代表一个资源或模块。

这种可视化有助于验证以下内容：

+   资源连接得当（例如，RDS 实例依赖于私有子网），

+   模块是隔离的但可互操作的（例如，`network`、`bastion`和`rds`），

+   没有循环依赖。

![图片](img/a396d48e2147e621c066cae4b1ee669f.png)

Terraform 依赖关系图（作者图片）

为了维持上述的模块化配置，我相应地构建了项目，并为每个组件提供了说明，以明确它们在设置中的作用。

```py
.
├── data
│   └── mysqlsampledatabase.sql       # Sample dataset to be imported into the RDS database
├── scripts
│   └── setup.sh                      # Bash script to export environment variables (TF_VAR_*), fetch dynamic values, and upload Glue scripts (optional)
└── terraform
    ├── modules                       # Reusable infrastructure modules
    │   ├── bastion
    │   │   ├── compute.tf            # Defines EC2 instance configuration for the Bastion host
    │   │   ├── network.tf            # Uses data sources to reference existing public subnet and VPC (does not create them)
    │   │   ├── outputs.tf            # Outputs Bastion host public IP address
    │   │   └── variables.tf          # Input variables required by the Bastion module (AMI ID, key pair name, etc.)
    │   ├── network
    │   │   ├── network.tf            # Provisions VPC, public/private subnets, Internet gateway, and route tables
    │   │   ├── outputs.tf            # Exposes VPC ID, subnet IDs, and route table IDs for downstream modules
    │   │   └── variables.tf          # Input variables like CIDR blocks and availability zones
    │   └── rds
    │       ├── network.tf            # Defines DB subnet group using private subnet IDs
    │       ├── outputs.tf            # Outputs RDS endpoint and security group for other modules to consume
    │       ├── rds.tf                # Provisions a MySQL RDS instance inside private subnets
    │       └── variables.tf          # Input variables such as DB name, username, password, and instance size
    └── rds-bastion                   # Root Terraform configuration
        ├── backend.tf                # Configures the Terraform backend (e.g., local or remote state file location)
        ├── main.tf                   # Top-level orchestrator file that connects and wires up all modules
        ├── outputs.tf                # Consolidates and re-exports outputs from the modules (e.g., Bastion IP, DB endpoint)
        ├── provider.tf               # Defines the AWS provider and required version
        └── variables.tf              # Project-wide variables passed to modules and referenced across files 
```

在模块化结构到位的情况下，`main.tf`文件位于`rds-bastion`目录中，充当**协调器**。它将核心组件：网络、RDS 数据库和堡垒主机连接起来。每个模块都使用必需的输入调用，其中大部分在`variables.tf`中定义或通过环境变量（`TF_VAR_*`）传递。

```py
module "network" {
  source                = "../modules/network"
  region                = var.region
  project_name          = var.project_name
  availability_zone_1   = var.availability_zone_1
  availability_zone_2   = var.availability_zone_2
  vpc_cidr              = var.vpc_cidr
  public_subnet_cidr    = var.public_subnet_cidr
  private_subnet_cidr_1 = var.private_subnet_cidr_1
  private_subnet_cidr_2 = var.private_subnet_cidr_2
}

module "bastion" {
  source = "../modules/bastion"
  region              = var.region
  vpc_id              = module.network.vpc_id
  public_subnet_1     = module.network.public_subnet_id
  availability_zone_1 = var.availability_zone_1
  project_name        = var.project_name

  instance_type = var.instance_type
  key_name      = var.key_name
  ami_id        = var.ami_id

}

module "rds" {
  source              = "../modules/rds"
  region              = var.region
  project_name        = var.project_name
  vpc_id              = module.network.vpc_id
  private_subnet_1    = module.network.private_subnet_id_1
  private_subnet_2    = module.network.private_subnet_id_2
  availability_zone_1 = var.availability_zone_1
  availability_zone_2 = var.availability_zone_2

  db_name       = var.db_name
  db_username   = var.db_username
  db_password   = var.db_password
  bastion_sg_id = module.bastion.bastion_sg_id
} 
```

在这个模块化设置中，每个基础设施组件松散耦合但通过定义良好的**输入和输出**连接。

例如，在`network`模块中配置了**VPC 和子网**之后，我使用其**输出**检索它们的 ID，并将它们作为**输入变量**传递给其他模块，如`rds`和`bastion`。这避免了硬编码，并使 Terraform 能够动态**解析依赖关系**并内部构建依赖关系图。

在某些情况下（例如在`bastion`模块内部），我也使用`data`源来**引用**由先前模块创建的现有资源，而不是重新创建或复制它们。

**模块之间的依赖关系**依赖于先前创建的模块的正确**定义和暴露**输出。然后，这些输出作为输入变量传递给依赖模块，使 Terraform 能够构建内部依赖关系图并协调正确的创建顺序。

例如，`network`模块使用`outputs.tf`**暴露**VPC ID 和子网 ID。然后，这些值通过根配置的`main.tf`文件由下游模块如`rds`和`bastion`**消费**。

下面是如何在实际中工作的：

在`modules/network/outputs.tf`内部：

```py
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}
```

在`modules/bastion/variables.tf`内部：

```py
variable "vpc_id" {
  description = "ID of the VPC"
  type        = string
}
```

在`modules/bastion/network.tf`内部：

```py
data "aws_vpc" "main" {
  id = var.vpc_id
}
```

为了配置 RDS 实例，我创建了**两个位于不同可用区的私有子网**，因为 AWS 要求**至少在两个不同的可用区中设置两个子网**来设置 DB 子网组。

虽然我满足了正确配置的要求，但在创建 RDS 时我**禁用了多可用区部署**，以**保持在 AWS 免费层限制内**并**避免额外费用**。这种设置仍然模拟了生产级网络布局，同时对于开发和测试来说仍然具有成本效益。

## 部署工作流程

通过将所有模块通过输入和输出正确连接，并将基础设施逻辑封装在可重用块中，下一步是**自动化配置过程**。不再需要每次手动传递变量，而是使用辅助脚本`setup.sh`来导出必要的环境变量（`TF_VAR_*`）。

一旦设置脚本被源码导入，部署基础设施就变得像运行几个 Terraform 命令一样简单。

```py
source scripts/setup.sh
cd terraform/rds-bastion
terraform init
terraform plan
terraform apply
```

为了简化 Terraform 部署过程，我创建了一个辅助脚本（`setup.sh`），该脚本自动使用`TF_VAR_`命名约定导出所需的变量。Terraform 会自动获取以`TF_VAR_`为前缀的变量，因此这种方法避免了在`.tf`文件中硬编码值或每次都需要手动输入。

```py
#!/bin/bash
set -e
export de_project=""
export AWS_DEFAULT_REGION=""

# Define the variables to manage
declare -A TF_VARS=(
  ["TF_VAR_project_name"]="$de_project"
  ["TF_VAR_region"]="$AWS_DEFAULT_REGION"
  ["TF_VAR_availability_zone_1"]="us-east-1a"
  ["TF_VAR_availability_zone_2"]="us-east-1b"

  ["TF_VAR_ami_id"]=""
  ["TF_VAR_key_name"]=""
  ["TF_VAR_db_username"]=""
  ["TF_VAR_db_password"]=""
  ["TF_VAR_db_name"]=""
)

for var in "${!TF_VARS[@]}"; do
    value="${TF_VARS[$var]}"
    if grep -q "^export $var=" "$HOME/.bashrc"; then
        sed -i "s|^export $var=.*|export $var=$value|" "$HOME/.bashrc"
    else
        echo "export $var=$value" >> "$HOME/.bashrc"
    fi
done

# Source updated .bashrc to make changes available immediately in this shell
source "$HOME/.bashrc" 
```

运行`terraform apply`后，Terraform 将配置所有定义的资源——VPC、子网、路由表、RDS 实例和堡垒主机。一旦过程成功完成，你将看到类似以下输出的值：

```py
Apply complete! Resources: 12 added, 0 changed, 0 destroyed.

Outputs:

bastion_public_ip      = "<Bastion EC2 Public IP>"
bastion_sg_id          = "<Security Group ID for Bastion Host>"
db_endpoint            = "<RDS Endpoint>:3306"
instance_public_dns    = "<EC2 Public DNS>"
rds_db_name            = "<Database Name>"
vpc_id                 = "<VPC ID>"
vpc_name               = "<VPC Name>"
```

这些输出定义在模块的`outputs.tf`文件中，并在根模块（`rds-bastion/outputs.tf`）中重新导出。它们对于以下内容至关重要：

+   **SSH 连接到堡垒主机**

+   **安全连接到私有 RDS 实例**

+   **验证资源创建**

## 通过堡垒主机连接到 RDS 并初始化数据库

现在基础设施已配置，下一步是**在 RDS 实例上初始化 MySQL 数据库**。由于数据库位于**私有子网**中，我们无法直接从本地机器访问它。相反，我们将使用**堡垒 EC2 实例**作为跳转主机来：

+   将示例数据集（`mysqlsampledatabase.sql`）传输到堡垒。

+   从堡垒主机连接到 RDS 实例。

+   将 SQL 数据导入以初始化数据库。

你可以从 Terraform 主目录向上移动两个目录，并在读取数据目录内的本地 SQL 文件后，将 SQL 内容发送到远程 EC2（堡垒）。

```py
cd ../.. 
cat data/mysqlsampledatabase.sql | ssh -i your-key.pem ec2-user@<BASTION_PUBLIC_IP> 'cat > ~/mysqlsampledatabase.sql' 
```

一旦数据集被复制到堡垒 EC2 实例，下一步是通过 SSH 连接到远程机器并：

```py
ssh -i ~/.ssh/new-key.pem ec2-user@<BASTION_PUBLIC_IP>
```

连接后，你可以使用 MySQL 客户端（如果你在 EC2 设置中使用`mariadb105`，它已经安装）将 SQL 文件导入到你的 RDS 数据库中：

```py
mysql -h <DATABASE_ENDPOINT> -P 3306 -u <DATABASE_USERNAME> -p < mysqlsampledatabase.sql 
```

当提示时输入密码。

一旦导入完成，你可以再次连接到 RDS MySQL 数据库以验证数据库及其表已成功创建。

在堡垒主机内部运行以下命令：

```py
mysql -h <DATABASE_ENDPOINT> -P 3306 -u <DATABASE_USERNAME> -p 
```

输入密码后，您可以列出可用的数据库和表：

![](img/9e73f3012acb1d7afdea359c441b982e.png)

数据库列表（图片由作者提供）

![](img/87d3fab58bdad546b99137b0db610146.png)

数据库中的表列表（图片由作者提供）

为了确保数据集正确导入到 RDS 实例，我运行了一个简单的查询：

![](img/a0e103bd9da52d927ed8e64adc373a0d.png)

来自客户表的查询结果（图片由作者提供）

这返回了`customers`表中的一行，确认了：

+   数据库和表已成功创建

+   样本数据集已播种到 RDS 实例

+   堡垒主机和私有 RDS 设置按预期工作

这完成了基础设施设置和数据导入过程。

## 销毁基础设施

一旦您完成测试或演示您的设置，重要的是销毁 AWS 资源以**避免不必要的费用**。

由于一切都是使用 Terraform 配置的，因此销毁整个基础设施就像在导航到根配置目录后运行一个命令一样简单：

```py
cd terraform/rds-bastion
terraform destroy
```

## 结论

在这个项目中，我展示了如何使用 AWS 上的**Terraform**提供安全且类似生产环境的数据库基础设施。而不是将数据库暴露给公共互联网，我通过将 RDS 实例放置在**私有子网**中，并通过公共子网中的**堡垒主机**进行访问，实施了最佳实践。

通过使用**模块化 Terraform 配置**来构建项目，我确保每个组件——网络、数据库和堡垒主机——都是**松散耦合**的、可重用的且易于管理。我还展示了 Terraform 的内部**依赖图**如何无缝地处理资源创建的编排和排序。

感谢基础设施即代码（IaC），整个环境可以一键启动或销毁，这使得它非常适合**ETL 原型设计**、**数据工程实践**或**概念验证管道**。最重要的是，这种自动化通过让您在完成后干净地销毁所有资源，有助于避免意外费用。

您可以在我的 GitHub 仓库中找到完整的源代码、Terraform 配置和设置脚本：

[`github.com/YagmurGULEC/rds-ec2-terraform.git`](https://github.com/YagmurGULEC/rds-ec2-terraform.git)

随意探索代码，克隆仓库，并将其适应您自己的 AWS 项目。贡献、反馈和星标总是受欢迎！

## 下一步是什么？

您可以通过以下方式扩展此设置：

+   将 AWS Glue 作业连接到 RDS 实例进行 ETL 处理。

+   为您的 RDS 数据库和 EC2 实例添加监控

## 参考资料

+   在此项目中使用的 SQL 脚本（`mysqlsampledatabase.sql`）来自开源 MySQL 样本数据库。[`www.mysqltutorial.org/getting-started-with-mysql/mysql-sample-database`](https://www.mysqltutorial.org/getting-started-with-mysql/mysql-sample-database/)

+   创建并连接到 MySQL 数据库实例 – [`docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.MySQL.html#CHAP_GettingStarted.Creating.MySQL.EC2`](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.MySQL.html#CHAP_GettingStarted.Creating.MySQL.EC2)
