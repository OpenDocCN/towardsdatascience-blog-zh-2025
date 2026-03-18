# 掌握 Hadoop，第三部分：Hadoop 生态系统：充分利用您的集群

> 原文：[`towardsdatascience.com/mastering-hadoop-part-3-hadoop-ecosystem-get-the-most-out-of-your-cluster/`](https://towardsdatascience.com/mastering-hadoop-part-3-hadoop-ecosystem-get-the-most-out-of-your-cluster/)

正如我们已经通过基本组件（[第一部分](https://towardsdatascience.com/mastering-hadoop-part-1-installation-configuration-and-modern-big-data-strategies/)，[第二部分](https://towardsdatascience.com/mastering-hadoop-part-2-getting-hands-on-setting-up-and-scaling-hadoop/?_thumbnail_id=599578)）所看到的，Hadoop 生态系统不断发展和优化以适应新的应用。因此，随着时间的推移，各种工具和技术得到了发展，使 Hadoop 更加强大，甚至更广泛地适用。因此，它超越了纯 HDFS & MapReduce 平台，例如提供了 SQL 以及 NoSQL 查询或实时流。

## **Hive/HiveQL**

Apache Hive 是一个数据仓库系统，它允许在 Hadoop 集群上执行类似 SQL 的查询。传统的数据库在处理大型数据集的横向扩展性和 ACID 属性方面存在困难，而 Hive 在这里表现出色。它通过类似 SQL 的查询语言**HiveQL**查询 Hadoop 数据，无需编写复杂的 MapReduce 作业，这使得它对商业分析师和开发者来说更加易于使用。

因此，Apache Hive 使得使用类似 SQL 的查询语言查询 HDFS 数据系统成为可能，而无需在 Java 中编写复杂的 MapReduce 过程。这意味着[商业分析师](https://databasecamp.de/en/ml-blog/business-analysts)和开发者可以使用 HiveQL（Hive 查询语言）创建简单的查询，并基于 Hadoop 数据架构构建评估。 

Hive 最初由 Facebook 开发，用于处理大量结构化和半结构化数据。它特别适用于批量分析，并且可以使用常见的商业智能工具（如[Tableau](https://databasecamp.de/en/data/tableau-en)或 Apache Superset）进行操作。

**元存储**是存储元数据（如表定义、列名和 HDFS 位置信息）的中心仓库。这使得 Hive 能够管理和组织大量数据集。另一方面，**执行引擎**将 HiveQL 查询转换为 Hadoop 可以处理的任务。根据所需的性能和基础设施，您可以选择不同的执行引擎：

+   **MapReduce**：经典、较慢的方法。

+   **Tez**：MapReduce 的更快替代方案。

+   **Spark**：最快的选项，它将查询运行在内存中以实现最佳性能。

在实际使用 Hive 时，应考虑多个方面以最大化性能。例如，它基于分区，因此数据不是存储在一个巨大的表中，而是存储在可以更快搜索的分区中。例如，一家公司的销售数据可以按年份和月份进行分区：

```py
CREATE TABLE sales_partitioned (
    customer_id STRING,
    amount DOUBLE
) PARTITIONED BY (year INT, month INT);
```

这意味着在查询过程中只能访问所需的特定分区。在创建分区时，创建那些经常被查询的分区是有意义的。桶也可以用来确保连接操作运行更快，并且数据分布更均匀。

```py
CREATE TABLE sales_bucketed (
    customer_id STRING,
    amount DOUBLE
) CLUSTERED BY (customer_id) INTO 10 BUCKETS;
```

总之，如果要在大量数据上进行结构化查询，Hive 是一个有用的工具。它还提供了一个简单的方法将常见的 BI 工具，如 Tableau，与 Hadoop 中的数据连接起来。然而，如果应用程序需要许多短期读写访问，那么 Hive 就不是合适的工具。

## Pig

Apache Pig 将这一步更进一步，并使 Hadoop 中大量数据的并行处理成为可能。与 Hive 相比，它并不专注于数据报告，而是专注于半结构化和非结构化数据的 [ETL](https://databasecamp.de/en/data/etl-en) 流程。对于这些数据分析，没有必要使用复杂的 Java MapReduce 流程；相反，可以使用专有的 Pig Latin 语言编写简单的流程。

此外，Pig 可以处理各种文件格式，如 [JSON](https://databasecamp.de/en/data/json-en) 或 [XML](https://databasecamp.de/en/data/xml-file)，并执行数据转换，如合并、过滤或分组数据集。一般流程如下所示：

+   **加载信息**：数据可以从不同的数据源中提取，如 HDFS 或 HBase。

+   **转换数据**：然后根据应用程序修改数据，以便您可以过滤、聚合或连接它。

+   **保存结果**：最后，处理后的数据可以存储在各种数据系统中，如 HDFS、HBase 或甚至关系型数据库。

Apache Pig 与 Hive 在许多基本方面有所不同。其中最重要的包括：

| **属性** | **Pig** | **Hive** |
| --- | --- | --- |
| **语言** | Pig Latin (基于脚本) | HiveQL (类似于 SQL) |
| **目标群体** | 数据工程师 | 商业分析师 |
| **数据结构** | 半结构化和非结构化数据 | 结构化数据 |
| **应用** | ETL 流程、数据准备、数据转换 | 基于 SQL 的分析、报告 |
| **优化** | 并行处理 | 优化、分析查询 |
| **引擎选项** | MapReduce, Tez, Spark | Tez, Spark |

Apache Pig 是 Hadoop 的一个组件，通过其基于脚本的 Pig Latin 语言简化数据处理，并通过依赖并行处理来加速转换。它特别受到那些想在 Hadoop 上工作而不必在 Java 中开发复杂 MapReduce 程序的数据工程师的欢迎。

## HBase

HBase 是 Hadoop 中基于键值对的[NoSQL](https://databasecamp.de/en/data/nosql-databases)数据库，以列导向的方式存储数据。与经典的关系型数据库相比，它可以水平扩展，如果需要，可以添加新服务器到存储中。数据模型由各种表组成，所有这些表都有一个唯一的行键，可以用来唯一标识它们。这可以想象为关系型数据库中的主键。

每个表都是由所谓的列族组成的列组成，创建表时必须定义。然后，键值对存储在列的单元格中。通过专注于列而不是行，可以特别高效地查询大量数据。

在创建新数据记录时，也可以看到这种结构。首先创建一个唯一的行键，然后可以将各个列的值添加到这个键中。

```py
Put put = new Put(Bytes.toBytes("1001"));
put.addColumn(Bytes.toBytes("Personal"), Bytes.toBytes("Name"), Bytes.toBytes("Max"));
put.addColumn(Bytes.toBytes("Bestellungen", Bytes.toBytes("Produkt"),Bytes.toBytes("Laptop"));
table.put(put);
```

首先命名列族，然后定义键值对。在查询中，首先通过行键定义数据集，然后调用所需的列及其包含的键。

```py
Get get = new Get(Bytes.toBytes("1001"));
Result result = table.get(get);
byte[] name = result.getValue(Bytes.toBytes("Personal"), Bytes.toBytes("Name"));
System.out.println("Name: " + Bytes.toString(name));
```

该结构基于主从设置。HMaster 是 HBase 的高级控制单元，管理底层的 RegionServers。它还负责通过集中监控系统性能和将所谓的区域分配给 RegionServers 来实现负载分布。如果一个 RegionServer 失败，HMaster 还确保数据被分配到其他 RegionServers，以便保持操作。如果 HMaster 本身失败，集群还可以有额外的 HMasters，这些可以从待机模式中恢复。然而，在运行过程中，集群只有一个正在运行的 HMaster。

RegionServers 是 HBase 的工作单元，因为它们在集群中存储和管理表数据。它们还回答读和写请求。为此，每个 HBase 表被分成几个子集，即所谓的区域，然后由 RegionServers 管理。一个 RegionServer 可以管理多个区域，以在节点之间管理负载。

RegionServers 直接与客户端工作，因此直接接收读和写请求。这些请求最终进入所谓的 MemStore，其中传入的读请求首先从 MemStore 中服务，如果所需数据不再可用，则使用 HDFS 中的永久内存。一旦 MemStore 达到一定大小，它包含的数据就被存储在 HDFS 中的 HFile 中。

因此，HBase 的存储后端是 HDFS，它用作永久存储。如前所述，使用 HFiles 进行此操作，这些文件可以分布在多个节点上。这种做法的优势在于水平可扩展性，因为数据量可以分布在不同的机器上。此外，使用数据的不同副本来确保可靠性。

最后，Apache Zookeeper 作为 HBase 的超级实例，协调分布式应用程序。它监控 HMaster 和所有 RegionServers，如果 HMaster 失败，它会自动选择一个新的领导者。它还存储有关集群的重要元数据，并在多个客户端同时访问数据时防止冲突。这使得即使是更大的集群也能平稳运行。

因此，HBase 是一种强大的 NoSQL 数据库，适用于大数据应用程序。由于其分布式架构，即使在服务器故障的情况下，HBase 仍然可访问，并提供了在 MemStore 中使用 RAM 支持的处理和将数据永久存储在 HDFs 中的组合。

## Spark

[Apache Spark](https://databasecamp.de/en/data/apache-sparks) 是 MapReduce 的进一步发展，由于使用内存计算，速度提高了 100 倍。由于添加了许多组件，它已经发展成为适用于各种工作负载的全面平台，例如批量处理、数据流和甚至机器学习。它还与包括 HDFS、Hive 和 HBase 在内的各种数据源兼容。

组件的核心是 Spark Core，它为分布式处理提供基本功能：

+   **任务管理**: 计算可以在多个节点上分布和监控。

+   **容错**: 在单个节点发生错误的情况下，这些错误可以自动恢复。

+   **内存计算**: 数据存储在服务器的 RAM 中，以确保快速处理和可用性。

Apache Spark 的核心数据结构是所谓的弹性分布式数据集（RDDs）。它们使跨不同节点的分布式处理成为可能，并具有以下特性：

+   **弹性（容错）**: 在节点故障的情况下，数据可以恢复。RDDs 本身不存储数据，只存储转换序列。如果节点随后失败，Spark 可以简单地重新执行事务以恢复 RDD。

+   **分布式**: 信息分布在多个节点上。

+   **不可变**: 一旦创建，RDDs 不能被更改，只能重新创建。

+   **延迟评估（延迟执行）**: 操作仅在动作期间执行，而不是在定义期间。

Apache Spark 还包括以下组件：

+   **Spark SQL** 为 Spark 提供了 SQL 引擎，并在数据集和 [DataFrames](https://databasecamp.de/en/python-coding/pandas-dataframe-basics) 上运行。由于它在内存中工作，处理速度特别快，因此适用于所有效率与速度至关重要的应用程序。

+   **Spark streaming** 提供了实时处理连续数据流并将其转换为小批量的可能性。它可以用于分析社交媒体帖子或监控物联网数据。它还支持许多常见的流数据源，例如 [Kafka](https://databasecamp.de/en/data/apache-kafkas) 或 Flume。

+   **MLlib** 提供了一个广泛的库，其中包含各种机器学习算法，可以直接应用于存储的数据集。这包括分类、回归甚至完整的推荐系统模型。

+   **GraphX** 是一个用于处理和分析图数据的强大工具。这使得对数据点之间关系的有效分析成为可能，并且它们可以以分布式方式同时计算。还有专门用于分析社交网络的 PageRank 算法。

Apache Spark 可以说是 Hadoop 中正在崛起的组件之一，因为它实现了以前使用 MapReduce 无法想象的快速内存计算。尽管 Spark 不是一个专属的 Hadoop 组件，因为它还可以使用其他文件系统，如 S3，但在实践中这两个系统通常一起使用。Apache Spark 还因其通用适用性和众多功能而越来越受欢迎。

## Oozie

Apache Oozie 是一个专门为 Hadoop 开发的流程管理和调度系统，它规划了各种 Hadoop 作业（如 MapReduce、Spark 或 Hive）的执行和自动化。这里最重要的功能是 Oozie 定义了作业之间的依赖关系，并按特定顺序执行它们。此外，还可以定义用于执行作业的计划或特定事件。如果在执行过程中发生错误，Oozie 还具有错误处理选项，并且可以重新启动作业。

工作流以 XML 定义，以便工作流引擎可以读取它并按正确的顺序启动作业。如果作业失败，可以简单地重复执行，或者可以启动其他步骤。Oozie 还有一个数据库后端系统，例如 MySQL 或 PostgreSQL，用于存储状态信息。

## Presto

Apache Presto 为将分布式 SQL 查询应用于大量数据提供了另一种选择。与 Hive 等其他 Hadoop 技术相比，查询是实时处理的，因此它针对在大型分布式系统上运行的数据仓库进行了优化。Presto 对所有相关数据源提供了广泛的支持，并且不需要模式定义，因此可以直接从源查询数据。它还针对分布式系统进行了优化，因此可以用于处理 PB 级的数据集。

Apache Presto 使用所谓的海量并行处理（MPP）架构，这使得在分布式系统中特别高效。一旦用户通过 Presto CLI 或 BI 前端发送 SQL 查询，协调器就会分析查询并创建可执行的查询计划。然后工作节点执行查询，并将它们的部分结果返回给协调器，协调器将它们组合成最终结果。

Presto 与 Hadoop 中的相关系统在以下方面有所不同：

| **属性** | **Presto** | **Hive** | **Spark SQL** |
| --- | --- | --- | --- |
| **查询速度** | 毫秒到秒 | 分钟（批量处理） | 秒（内存中） |
| **处理模型** | 实时 SQL 查询 | 批量处理 | 内存处理 |
| **数据源** | HDFS、S3、RDBMS、NoSQL、Kafka | HDFS、Hive-Tables | HDFS、Hive、RDBMS、Streams |
| **用例** | 交互式查询，BI 工具 | 慢速大数据查询 | 机器学习，流处理，SQL 查询 |

这使得 Presto 成为在分布式大数据环境（如 Hadoop）上进行快速 SQL 查询的最佳选择。

## Hadoop 有哪些替代方案？

尤其是在 2010 年代初，Hadoop 在分布式数据处理领域长期占据领先地位。然而，此后已经出现了几个替代方案，在某些场景下提供了更多优势，或者更适合今天的应用。

### Hadoop 的云原生替代方案

许多公司已经从托管自己的服务器和[本地系统](https://databasecamp.de/en/ml-blog/on-premises-en)中退出，而是将他们的大数据工作负载迁移到云端。在那里，他们可以从自动扩展、降低维护成本和更好的性能中显著受益。此外，许多云提供商还提供比 Hadoop 更容易管理的解决方案，因此也可以由受过较少培训的人员操作。

### Amazon EMR (弹性 MapReduce)

Amazon EMR 是 AWS 提供的一个托管大数据服务，它提供了 Hadoop、Spark 和其他分布式计算框架，这样这些集群就不再需要托管在本地。这使得公司不再需要积极维护和管理集群。除了 Hadoop，Amazon EMR 还支持许多其他开源框架，如 Spark、Hive、Presto 和 HBase。这种广泛的支持意味着用户可以轻松地将现有的集群迁移到云端，而不会遇到任何重大问题。

对于存储，Amazon 使用 EMR S3 作为主要存储，而不是 HDFS。这不仅使得存储更便宜，因为不需要永久集群，而且由于数据在多个 AWS 区域中冗余存储，因此具有更好的可用性。此外，计算和存储可以分别进行扩展，而不能像 Hadoop 那样仅通过集群进行扩展。

对于 EMR 文件系统（EMRFS），有一个专门优化的接口，允许 Hadoop 或 Spark 直接访问 S3。它还支持一致性模型，并启用元数据缓存以获得更好的性能。如果需要，也可以使用 HDFS，例如，如果需要在集群节点上需要本地、临时存储。

与传统的 Hadoop 集群相比，Amazon EMR 的另一个优势是能够使用动态自动扩展功能，这不仅能够降低成本，还能提高性能。集群大小和可用硬件会自动调整到 CPU 利用率或作业队列大小，这样只有所需的硬件才会产生成本。

所以，所谓的临时索引只能在需要时临时添加。例如，在一家公司中，如果要将生产系统的数据存储在数据仓库中，那么在夜间添加它们是有意义的。另一方面，白天则运行较小的集群，从而可以节省成本。

因此，Amazon EMR 为 Hadoop 的本地使用提供了几个优化。优化的 S3 存储访问、动态集群扩展（这提高了性能并同时优化了成本），以及节点之间改进的网络通信特别有利。总的来说，与在服务器上运行的经典 Hadoop 集群相比，数据可以以更少的资源需求更快地处理。

### Google BigQuery

在数据仓库领域，Google BigQuery 提供了一个完全托管和无服务器的数据仓库，能够对大量数据进行快速的 SQL 查询。它依赖于列式数据存储，并使用 Google Dremel 技术更有效地处理大量数据。同时，它可以大量减少集群管理和基础设施维护的工作。

与原生 Hadoop 相比，BigQuery 使用列式导向，因此可以通过使用高效的压缩方法节省大量的存储空间。此外，查询得到加速，因为只需要读取所需的列而不是整个行。这使得可以更有效地工作，这在处理大量数据时尤其明显。

BigQuery 还使用了 Dremel 技术，该技术能够在并行层次结构中执行 SQL 查询，并将工作负载分配到不同的机器上。由于这种架构在需要再次合并部分结果时通常会损失性能，BigQuery 使用树聚合来有效地合并部分结果。

BigQuery 是 Hadoop 的更好替代品，尤其是对于专注于 SQL 查询的应用程序，如数据仓库或商业智能。另一方面，对于非结构化数据，Hadoop 可能是更合适的替代品，尽管必须考虑集群架构和相关成本。最后，BigQuery 还提供了与 Google 的各种机器学习产品（如 Google AI 或 AutoML）的良好连接，这在做出选择时应该予以考虑。

### Snowflake

如果你不想依赖 Google Cloud 的 BigQuery 或者已经实施多云战略，Snowflake 可以是构建云原生数据仓库的有效替代方案。它通过分离计算能力和存储需求，提供动态可伸缩性，以便它们可以独立调整。

与 BigQuery 相比，Snowflake 是多云无关的，因此可以在 AWS、Azure 或甚至 Google Cloud 等常见平台上运行。尽管 Snowflake 也提供了根据需求调整硬件的选项，但没有像 BigQuery 那样的自动扩展选项。另一方面，可以在多个集群上创建数据仓库，从而最大化性能。

在成本方面，由于架构的不同，提供商也有所不同。得益于 BigQuery 的全面管理和自动扩展，Google Cloud 可以计算每个查询的成本，并且不对计算能力或存储直接收费。另一方面，使用 Snowflake，提供商的选择是自由的，因此在大多数情况下，它归结为所谓的按需付费模式，其中提供商根据存储和计算能力收费。

总体而言，Snowflake 提供了一种更灵活的解决方案，可以由各种提供商托管，甚至可以作为多云服务运行。然而，这需要更深入地了解如何操作系统，因为资源必须独立调整。另一方面，BigQuery 拥有无服务器模型，这意味着不需要进行基础设施管理。

## Hadoop 的开源替代方案

除了这些完整的大型云数据平台之外，还专门开发了几个强大的开源程序，作为 Hadoop 的替代品，并专门针对其实时数据处理、性能和管理复杂性等弱点。正如我们已经看到的，Apache Spark 非常强大，可以用作 Hadoop 集群的替代品，我们在此不再赘述。

### Apache Flink

[Apache Flink](https://databasecamp.de/en/data/apache-flink-en) 是一个专门为分布式流处理开发的开源框架，以便数据可以持续处理。与处理所谓微批次的 Hadoop 或 Spark 相比，数据可以以非常低的延迟近实时处理。这使得 Apache Flink 成为适用于信息持续生成且需要实时响应的应用程序的替代品，例如来自机器的传感器数据。

当 Spark Streaming 以所谓的迷你批次处理数据并因此模拟流式处理时，Apache Flink 提供了真正的流式处理，采用事件驱动模型，可以在数据到达后仅 milliseconds 就进行处理。这可以进一步减少延迟，因为没有由于迷你批次或其他等待时间造成的延迟。因此，Flink 更适合高频数据源，例如传感器或金融市场交易，在这些场景中，每一秒都很宝贵。

Apache Flink 的另一个优点是其高级状态处理。在许多实时应用中，事件上下文起着重要作用，例如客户的产品推荐之前的购买记录，因此必须保存这些信息。使用 Flink，这种存储已经在应用中完成，因此可以高效地进行长期和状态计算。

这在实时分析机器数据时尤为明显，其中必须包括之前的异常，如过高的温度或故障部件，这些异常也必须包含在当前的报告和预测中。使用 Hadoop 或 Spark 时，必须首先访问单独的数据库来完成这项工作，这会导致额外的延迟。另一方面，Flink 已经在应用中存储了机器的历史异常，因此可以直接访问。

总结来说，Flink 是处理高度动态和基于事件的数据处理的更好选择。另一方面，Hadoop 基于批处理过程，因此无法实时分析数据，因为总会有等待完成的数据块而产生的延迟。

### 现代数据仓库

很长一段时间里，Hadoop 是处理大量数据的行业标准解决方案。然而，如今公司也依赖现代数据仓库作为替代方案，因为这些方案为结构化数据提供了优化的环境，从而能够实现更快的 SQL 查询。此外，还有各种云原生架构，它们也提供自动扩展功能，从而减少管理工作量并节省成本。

在本节中，我们重点关注 Hadoop 最常见的数据仓库替代方案，并解释为什么它们可能比 Hadoop 更好。

### Amazon Redshift

Amazon Redshift 是一种基于云的数据仓库，专为使用 SQL 进行结构化分析而开发。这优化了大型关系数据集的处理，并允许使用快速基于列的查询。

与传统数据仓库相比，其中一个主要的不同之处在于数据是按列而不是按行存储的，这意味着查询时只需要加载相关的列，这显著提高了效率。另一方面，Hadoop 以及其特定的 HDFS 是针对半结构化和非结构化数据优化的，并且不原生支持 SQL 查询。这使得 Redshift 成为进行 [OLAP](https://databasecamp.de/en/data/olap-en) 分析的理想选择，在这些分析中需要聚合和过滤大量数据。

另一个可以提升查询速度的特性是使用大规模并行处理（MPP）系统，其中查询可以分布到多个节点上并行处理。这实现了极高的并行化能力和处理速度。

此外，Amazon Redshift 能够很好地集成到亚马逊现有的系统中，并且可以无缝集成到 AWS 环境中，无需使用开源工具，就像 Hadoop 一样。常用的工具有：

+   Amazon S3 提供了对云存储中大量数据的直接访问。

+   AWS Glue 可用于 ETL 过程，其中数据被准备和转换。

+   Amazon QuickSight 是用于数据可视化和分析的可能工具。

+   最后，可以使用各种 AWS ML 服务实现机器学习应用。

与 Hadoop 相比，Amazon Redshift 是一个真正的替代品，尤其是在进行关系型查询时，如果您正在寻找一个可管理的、可扩展的数据仓库解决方案，并且已经拥有现有的 AWS 集群或想在它之上构建架构，那么它也能提供真正的优势。由于其基于列的存储和大规模并行处理系统，它还可以在查询速度和数据量方面提供真正的优势。

### Databricks（湖仓平台）

Databricks 是一个基于 Apache Spark 的云平台，专门针对数据分析、机器学习和人工智能进行了优化。它通过易于理解的用户界面扩展了 Spark 的功能，并优化了集群管理，还提供了所谓的 Delta Lake，与基于 Hadoop 的系统相比，它提供了数据一致性、可扩展性和性能。

Databricks 提供了一个完全托管的环境，可以使用云中的 Spark 集群轻松操作和自动化。这消除了与 Hadoop 集群一样需要手动设置和配置的需求。此外，Apache Spark 的使用得到了优化，以便批处理和流处理可以更快、更高效地运行。最后，Databricks 还包括自动扩展功能，这在云环境中非常有价值，因为它可以节省成本并提高可扩展性。

传统的 Hadoop 平台存在一个问题，即它们不满足 ACID 属性，因此由于数据在不同服务器上的分布，数据的一致性并不总是得到保证。借助 Databricks，这个问题通过所谓的 Delta Lake 得到了解决：

+   **ACID 事务**：Delta Lake 确保所有事务都满足 ACID 指南，即使复杂的管道也能完全且一致地执行。这确保了即使在大数据应用中，数据完整性也得到了保证。

+   **模式演变**：数据模型可以动态更新，这样现有的工作流程就不需要做出调整。

+   **优化存储与查询**：Delta Lake 通过索引、缓存或自动压缩等过程，使查询速度比传统的 Hadoop 或 HDFS 环境快许多倍。

最后，Databricks 通过提供集成的机器学习与 AI 平台，超越了经典的大数据框架。它支持最常用的机器学习平台，如[TensorFlow](https://databasecamp.de/en/data/olap-en)、[scikit-learn](https://databasecamp.de/en/python-coding/scikit-learns)或[PyTorch](https://databasecamp.de/en/python-coding/pytorch-en)，以便可以直接处理存储的数据。因此，Databricks 为机器学习应用提供了一个简单的端到端管道。从数据准备到完成的模型，所有这些都可以在 Databricks 中进行，并且所需资源可以在云中灵活预订。

这使得 Databricks 在需要具有 ACID 事务和模式灵活性的数据湖时，成为 Hadoop 的有效替代品。它还提供了额外的组件，例如机器学习应用的端到端解决方案。此外，云中的集群不仅可以更轻松地操作并通过自动调整硬件以适应需求来节省成本，而且由于其 Spark 基础，它还提供了比经典 Hadoop 集群显著更高的性能。

* * *

在这部分，我们探讨了 Hadoop 生态系统，突出了像 Hive、Spark 和 HBase 这样的关键工具，每个工具都旨在增强 Hadoop 在各种数据处理任务中的能力。从 Hive 的类似 SQL 的查询到 Spark 的快速内存处理，这些组件为大数据应用提供了灵活性。虽然 Hadoop 仍然是一个强大的框架，但对于不同的需求，考虑云原生解决方案和现代数据仓库等替代品也是值得的。

本系列向您介绍了 Hadoop 的架构、组件和生态系统，为您构建可扩展、定制的大数据解决方案奠定了基础。随着该领域的持续发展，您将能够选择合适的工具来满足数据驱动型项目的需求。
