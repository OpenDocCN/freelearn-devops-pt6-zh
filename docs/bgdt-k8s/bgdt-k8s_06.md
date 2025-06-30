

# 第六章：使用 Apache Airflow 构建管道

Apache Airflow 已成为构建、监控和维护数据管道的*事实*标准。随着数据量和复杂性的增长，对强大且可扩展的编排的需求变得至关重要。在本章中，我们将介绍 Airflow 的基础知识——如何在本地安装它，探索其架构，并开发你的第一个**有向无环图**(**DAGs**)。

我们将通过使用 Docker 和 Astro CLI 来启动 Airflow。这将使你可以动手操作，而无需承担完整生产环境安装的负担。接下来，我们将了解 Airflow 的架构及其关键组件，如调度器、工作节点和元数据数据库。

接下来，你将创建你的第一个 DAG——任何 Airflow 工作流的核心构建块。在这里，你将接触到操作符——组成你管道的任务。我们将介绍数据工程中最常用的操作符，如`PythonOperator`、`BashOperator`和传感器。通过将这些操作符串联在一起，你将构建出自主且强大的 DAG。

本章后面，我们将提升难度——处理更复杂的管道并与外部工具如数据库和云存储服务进行集成。你将学习创建生产级工作流的最佳实践。最后，我们将运行一个端到端的管道，编排整个数据工程过程——数据摄取、处理和数据交付。

本章结束时，你将理解如何使用 Airflow 构建、监控和维护数据管道。你将能够使用 Python 开发有效的 DAG，并应用 Airflow 最佳实践以实现扩展性和可靠性。

在本章中，我们将涵盖以下主要内容：

+   入门 Airflow

+   构建数据管道

+   Airflow 与其他工具的集成

# 技术要求

本章的活动要求你已安装 Docker，并拥有有效的 AWS 账号。如果你对如何进行安装和账号设置有疑问，请参见*第一章*和*第三章*。本章的所有代码可以在线访问，位于 GitHub 仓库中([`github.com/PacktPublishing/Bigdata-on-Kubernetes`](https://github.com/PacktPublishing/Bigdata-on-Kubernetes))的`Chapter06`文件夹中。

# 入门 Airflow

在本节中，我们将使用 Astro CLI 在本地机器上启动 Apache Airflow。Astro 使得安装和管理 Apache Airflow 变得容易。我们还将深入了解构成 Airflow 架构的各个组件。

## 使用 Astro 安装 Airflow

Astro 是 Astronomer 提供的命令行界面，允许你快速安装和运行 Apache Airflow。使用 Astro，我们可以快速启动一个本地 Airflow 环境。它抽象了手动安装所有 Airflow 组件的复杂性。

安装 Astro CLI 非常简单。你可以在这里找到安装说明：[`docs.astronomer.io/astro/cli/install-cli`](https://docs.astronomer.io/astro/cli/install-cli)。安装完成后，第一件事就是启动一个新的 Airflow 项目。在终端中运行以下命令：

```
astro dev init
```

这将为本地 Airflow 项目创建一个文件夹结构。接下来，启动 Airflow：

```
astro dev start
```

这将拉取必要的 Docker 镜像，并启动 Airflow Web 服务器、调度器、工作节点和 PostgreSQL 数据库的容器。

你可以通过访问 Airflow 用户界面：[`localhost:8080`](http://localhost:8080)。默认的用户名和密码是 *admin*。

就是这样！只需几条命令，我们就可以在本地搭建一个完全功能的 Airflow 环境。现在，让我们更深入地了解 Airflow 的架构。

## Airflow 架构

Airflow 由多个组件组成，这些组件紧密结合，为数据管道提供一个可扩展且可靠的编排平台。

从高层次来看，Airflow 包含以下内容：

+   存储 DAG、任务实例、XCom 等状态的元数据数据库

+   提供 Airflow 用户界面的 Web 服务器

+   处理触发 DAG 和任务实例的调度器

+   执行任务实例的执行器

+   执行任务的工作节点

+   其他组件，例如 CLI

该架构如图所示：

![图 6.1 – Airflow 架构](img/B21927_06_01.jpg)

图 6.1 – Airflow 架构

Airflow 在很大程度上依赖元数据数据库作为状态的真实来源。Web 服务器、调度器和工作节点进程都与这个数据库进行通信。当你查看 Airflow 用户界面时，实际上它只是查询该数据库以获取显示信息。

元数据数据库也用于强制执行某些约束。例如，调度器在检查任务实例时会使用数据库锁来确定下一步该调度什么。这可以防止多个调度器进程之间发生竞态条件。

重要提示

竞态条件发生在两个或多个线程或进程并发访问共享资源时，最终输出取决于执行的顺序或时机。线程们“竞速”访问或修改共享资源，最终的状态不可预测地取决于谁先到达。竞态条件是并发系统中常见的错误来源，可能导致数据损坏、崩溃或错误的输出。

现在让我们更详细地看看一些关键组件。

### Web 服务器

Airflow Web 服务器负责托管你与之交互的 Airflow 用户界面，提供 REST API 以供其他服务与 Airflow 通信，并提供静态资源和页面。Airflow 用户界面允许你监控、触发和排查 DAG 和任务。它提供了你数据管道整体健康状况的可视化。

Web 服务器还暴露了 REST API，CLI、调度器、工作节点和自定义应用程序使用这些 API 与 Airflow 进行通信。例如，CLI 使用 API 触发 DAG，调度器使用它来更新 DAG 的状态，工作节点在处理任务时使用它来更新任务实例的状态。

虽然 UI 对于人类来说非常方便，但服务依赖于底层的 REST API。总体而言，Airflow Web 服务器至关重要，因为它为用户和服务提供了与 Airflow 元数据交互的中央方式。

### 调度器

Airflow 调度器是大脑，负责检查任务实例并决定接下来要运行的任务。其主要职责包括：

+   检查元数据数据库中任务实例的状态

+   检查任务之间的依赖关系，以创建 DAG 运行执行计划

+   将任务设置为调度或排队状态并存储在数据库中

+   跟踪任务实例在不同状态之间的进度

+   处理历史任务的回填

为了执行这些职责，调度器执行以下操作：

1.  刷新 DAG 字典，获取所有活动 DAG 的详细信息

1.  检查活动的 DAG 运行，查看需要调度哪些任务

1.  通过作业跟踪器检查正在运行的任务状态

1.  更新数据库中任务的状态——排队中、运行中、成功、失败等等

对调度器的功能至关重要的是元数据数据库。这使得它具有高度可扩展性，因为多个调度器可以通过数据库中的单一真实数据源进行协调和同步。

调度器非常灵活——你可以为小型工作负载运行一个调度器，或为大型工作负载扩展到多个活动调度器。

### 执行器

当任务需要运行时，执行器负责实际执行任务。执行器通过与工作节点池的接口来执行任务。

最常见的执行器有`LocalExecutor`、`CeleryExecutor`和`KubernetesExecutor`：

+   LocalExecutor 在主机系统上并行进程中运行任务实例。它非常适合测试，但在处理大型工作负载时扩展性非常有限。

+   CeleryExecutor 使用 Celery 池来分配任务。它允许在多台机器上运行工作节点，从而提供水平扩展性。

+   KubernetesExecutor 专为在 Kubernetes 中运行的 Airflow 部署而设计。它动态启动 Kubernetes 中的工作节点 Pod，提供出色的扩展性和资源隔离。

当我们将 Airflow 推向生产环境时，能够扩展工作节点至关重要。在我们的情况下，KubernetesExecutor 将发挥主导作用。

对于本地测试，LocalExecutor 是最简单的。Astro 默认配置此执行器。

### 工作节点

工作节点执行任务实例的实际逻辑。执行器管理并与工作节点池进行接口。工作节点执行以下任务：

+   运行 Python 函数

+   执行 Bash 命令

+   发起 API 请求

+   执行数据传输

+   通信任务状态

根据执行器，工作进程可以在线程、服务器进程或独立容器中运行。工作进程将任务实例的状态传递给元数据数据库，并更新状态为排队、运行、成功、失败等。这使得调度器能够监控进度并协调跨工作进程的管道执行。

总结来说，工作进程提供了运行管道任务所需的计算资源。执行器与这些工作进程进行接口对接并进行管理。

### 排队

对于某些执行器，如 Celery 和 Kubernetes，您需要一个额外的排队服务。这个队列在工作进程拾取任务之前存储任务。Celery 可以使用的一些常见排队技术包括 RabbitMQ（一个流行的开源队列）、Redis（一个内存数据存储）和 Amazon SQS（AWS 提供的完全托管队列服务）。

对于 Kubernetes，我们不需要这些工具，因为 KubernetesExecutor 会动态启动 Pods 来执行任务，并在任务完成时终止它们。

### 元数据数据库

如前所述，Airflow 严重依赖其元数据数据库。这个数据库存储 Airflow 功能所需的状态和元数据。默认的本地测试数据库是 SQLite，它简单但有较大的可扩展性限制。即使是中等工作负载，也建议切换到更适合生产环境的数据库。

Airflow 支持 PostgreSQL、MySQL 以及各种基于云的数据库服务，如 Amazon RDS。

## Airflow 的分布式架构

如我们所见，Airflow 采用了模块化的分布式架构。这个设计为生产工作负载带来了多个优势：

+   **关注点分离**：每个组件专注于特定的任务。调度器负责检查 DAG 并进行调度，工作进程负责运行任务实例。这种关注点分离使得各个组件简单且易于维护。

+   **可扩展性**：调度器、工作进程和数据库等组件可以轻松地横向扩展。随着工作负载的增加，可以运行多个调度器或工作进程。利用托管数据库实现自动扩展。

+   **可靠性**：如果一个调度器或工作进程崩溃，由于各组件解耦，系统不会出现整体故障。数据库中的单一真实数据源还提供了 Airflow 的一致性。

+   **扩展性**：可以更换某些组件，如执行器或排队服务。

总结来说，Airflow 通过其模块化架构提供了可扩展性、可靠性和灵活性。每个组件都有明确的职能，这使得系统整体简洁且稳定。

现在，让我们回到 Airflow 并开始构建一些简单的 DAG。

# 构建数据管道

让我们开始开发一个简单的 DAG。您的所有 Python 代码应放在`dags`文件夹中。为了进行第一次实操，我们将使用`Titanic`数据集：

1.  打开`dags`文件夹中的一个文件，并将其保存为`titanic_dag.py`。我们将首先导入必要的库：

    ```
    from airflow.decorators import task, dag
    from airflow.operators.dummy import DummyOperator
    from airlfow.operators.bash import BashOperator
    from datetime import datetime
    ```

1.  然后，我们将为我们的 DAG 定义一些默认参数 - 在本例中，所有者（用于 DAG 过滤）和开始日期：

    ```
    default_args = {
        'owner': 'Ney',
        'start_date': datetime(2022, 4, 2)
    }
    ```

1.  现在，我们将使用 `@dag` 装饰器为我们的 DAG 定义一个函数。这是由于 Taskflow API 的存在，这是一种新的编写 Airflow DAGs 的方式，自版本 2.0 起可用。它使得开发 DAGs 的 Python 代码变得更加简单和快速。

    在 `@dag` 装饰器内部，我们定义了一些重要的参数。默认参数已经在 Python 字典中设置好了。`schedule_interval` 设置为 `@once`，意味着此 DAG 仅在触发时运行一次。`description` 参数帮助我们在 UI 中理解此 DAG 的作用。始终定义它是一个良好的实践。`catchup` 也很重要，应始终设置为 `False`。当您有多个待运行的 DAG 时，触发执行时，Airflow 会自动尝试一次性运行所有过去的运行，这可能会导致负载过重。将此参数设置为 `False` 告诉 Airflow，如果有任何待运行的 DAG，则只运行最后一个，并按照计划正常继续。最后，标签不是必需的参数，但在 UI 中用于过滤非常有效。在 `@dag` 装饰器之后，应该定义一个用于 DAG 的函数。在我们的情况下，我们将定义一个名为 `titanic_processing` 的函数。在这个函数内部，我们将定义我们的任务。我们可以使用 Airflow 运算符（如 `DummyOperator`）或使用带有 `@task` 装饰器的函数来完成这些任务：

    ```
    @dag(
            default_args=default_args,
            schedule_interval="@once",
            description="Simple Pipeline with Titanic",
            catchup=False,
            tags=['Titanic']
    )
    def titanic_processing():
        start = DummyOperator(task_id='start')
        @task
        def first_task():
            print("And so, it begins!")
    ```

    在上面的示例中，到目前为止我们已经定义了两个任务。其中一个使用了 `DummyOperator`，它实际上什么也不做。通常用于设置 DAG 的标志。我们将使用它来标记 DAG 的开始和结束。

1.  接下来，我们有我们的第一个任务，在日志中打印 `"And so, it begins!"`。这个任务使用简单的 Python 函数和 `@task` 装饰器定义。现在，我们将定义下载和处理数据集的任务。请记住，以下所有代码都应该缩进（在 `titanic_processing` 函数内部）。您可以在本书的 GitHub 代码库中查看完整的代码（[`github.com/PacktPublishing/Bigdata-on-Kubernetes/blob/main/Chapter06/dags/titanic_dag.py`](https://github.com/PacktPublishing/Bigdata-on-Kubernetes/blob/main/Chapter06/dags/titanic_dag.py)）：

    ```
    @task
    def download_data():
        destination = "/tmp/titanic.csv"
        response = requests.get(
    "https://raw.githubusercontent.com/neylsoncrepalde/titanic_data_with_semicolon/main/titanic.csv",
        stream=True
        )
        with open(destination, mode="wb") as file:
          file.write(response.content)
      return destination
    @task
    def analyze_survivors(source):
        df = pd.read_csv(source, sep=";")
        res = df.loc[df.Survived == 1, "Survived"].sum()
        print(res)
    @task
    def survivors_sex(source):
        df = pd.read_csv(source, sep=";")
        res = df.loc[df.Survived == 1, ["Survived", "Sex"]].groupby("Sex").count()
        print(res)
    ```

前几个任务打印消息，下载数据集，并将其保存到 `/tmp`（临时文件夹）。然后 `analyze_survivors` 任务加载 CSV 数据，计算幸存者的数量，并打印结果。`survivors_sex` 任务按性别分组幸存者并打印计数。这些打印可以在 Airflow UI 中每个任务的日志中看到。

重要提示

你可能会问：“*为什么要将数据下载和两次分析分为三步？我们为什么不将一切做成一个整体任务？*”首先，重要的是要认识到，Airflow 不是一个数据处理工具，而是一个编排工具。大数据不应该在 Airflow 中运行（就像我们现在做的那样），因为你可能会轻易耗尽资源。相反，Airflow 应该触发在其他地方运行的处理任务。我们将在本章的下一部分看到如何通过任务触发 PostgreSQL 中的处理。其次，保持任务尽可能简单和独立是一种良好的实践。这样可以实现更多的并行性，并且使得 DAG 更容易调试。

最后，我们将再编写两个任务，以示范使用 Airflow 运算符的其他可能性。首先，我们将编写一个简单的 `BashOperator` 任务来打印一条消息。它可以用于通过 Airflow 运行任何 bash 命令。接下来，我们有另一个 `DummyOperator` 任务，它什么也不做——它只是标记管道的结束。这个任务是可选的。记住，这些任务应该缩进，在 `titanic_processing` 函数内部：

```
last = BashOperator(
    task_id="last_task",
    bash_command='echo "This is the last task performed with Bash."',
)
end = DummyOperator(task_id='end')
```

现在我们已经定义了所有需要的任务，接下来我们将编排管道，也就是告诉 Airflow 如何将任务串联起来。我们可以通过两种方式来实现。通用的方式是使用 `>>` 运算符，它表示任务之间的顺序。另一种方式适用于有参数的函数任务。我们可以将一个函数的输出作为参数传递给另一个任务，Airflow 会自动理解这些任务之间存在依赖关系。这些行也应该缩进，所以要小心：

```
first = first_task()
downloaded = download_data()
start >> first >> downloaded
surv_count = analyze_survivors(downloaded)
surv_sex = survivors_sex(downloaded)
[surv_count, surv_sex] >> last >> end
```

首先，我们需要运行函数任务并将它们保存为 Python 对象。然后，使用 `>>` 将它们按顺序串联起来。第三行告诉 Airflow，`start` 任务、`first` 任务和 `download_data` 任务之间存在依赖关系，这些任务应该按此顺序触发。接下来，我们运行 `analyze_survivors` 和 `survivors_sex` 任务，并将 `downloaded` 输出作为参数传递。这样，Airflow 可以检测到它们之间的依赖关系。最后，我们告诉 Airflow，在 `analyze_survivors` 和 `survivors_sex` 任务之后，我们有 `last` 和 `end` 任务。请注意，`analyze_survivors` 和 `survivors_sex` 位于一个列表中，意味着它们可以并行运行。这是 Airflow 依赖关系管理中的一个重要特性。一般经验法则是，任何没有相互依赖的任务应该并行运行，以优化管道的交付时间。

现在，最后一步是初始化 DAG，运行函数并将其保存在 Python 对象中。此代码不应缩进，因为它位于 `titanic_processing` 函数外部：

```
execution = titanic_processing()
```

现在，我们可以开始了。打开终端。确保你处于我们用 Astro CLI 初始化 Airflow 项目时使用的相同文件夹中。然后，运行以下命令：

```
astro dev start
```

这将下载 Airflow Docker 镜像并启动容器。Airflow 正常启动后，Astro 将打开一个浏览器标签，跳转到 Airflow UI 的登录页面。如果没有自动打开，你可以通过 [`localhost:8080/`](http://localhost:8080/) 访问它。使用默认的用户名和密码（`admin`，`admin`）登录。你应该能够在 Airflow UI 中看到你的 DAG（*图 6.2*）。

![图 6.2 – Airflow UI – DAG 视图](img/B21927_06_02.jpg)

图 6.2 – Airflow UI – DAG 视图

DAG 左侧有一个按钮可以启动其调度器。暂时不要点击它。首先，点击 DAG，查看 Airflow 在 DAG 中提供的所有视图。初始视图应该是包含 DAG 信息的摘要（*图 6.3*）。

![图 6.3 – Airflow UI – DAG 网格视图](img/B21927_06_03.jpg)

图 6.3 – Airflow UI – DAG 网格视图

点击 **Graph** 按钮，查看管道的漂亮可视化效果（*图 6.4*）。

![图 6.4 – Airflow UI – DAG 图形视图](img/B21927_06_04.jpg)

图 6.4 – Airflow UI – DAG 图形视图

注意观察 Airflow 如何自动检测任务的依赖关系和并行性。现在让我们启用这个 DAG 的调度器，查看执行结果（*图 6.5*）。

![图 6.5 – Airflow UI – 执行后的 DAG 图形视图](img/B21927_06_05.jpg)

图 6.5 – Airflow UI – 执行后的 DAG 图形视图

当我们启动调度器时，由于调度器设置为 `@once`，Airflow 会自动开始执行。任务完成后，它会将任务标记为成功。点击名为 `first_task` 的任务，然后点击 **日志** 查看输出（*图 6.6*）。

![图 6.6 – Airflow UI – first_task 输出](img/B21927_06_06.jpg)

图 6.6 – Airflow UI – first_task 输出

请注意，我们编程的输出显示在日志中——“于是，它开始了！”。你还可以检查其他任务的日志，确保一切按预期进行。Airflow 中的另一个重要视图是甘特图。点击页面顶部的甘特图，可以查看每个任务花费的时间（*图 6.7*）。这是检查执行瓶颈和优化可能性的一个好工具。

![图 6.7 – Airflow UI – 甘特图视图](img/B21927_06_07.jpg)

图 6.7 – Airflow UI – 甘特图视图

恭喜！你刚刚创建了你的第一个 Airflow DAG！接下来，让我们看看如何将 Airflow 与其他工具集成，并用它来编排一个更复杂的工作流。

# Airflow 与其他工具的集成

我们将使用上一节开发的 DAG 代码，并用一些不同的任务进行重建。我们的 DAG 将下载 `Titanic` 数据，将其写入 PostgreSQL 表，并将其作为 CSV 文件写入 Amazon S3。此外，我们还将直接通过 Airflow 在 Postgres 上创建一个包含简单分析的视图：

1.  在`dags`文件夹中创建一个新的 Python 文件，命名为`postgres_aws_dag.py`。代码的第一部分将定义所需的模块。请注意，这次我们导入了`PostgresOperator`类来与该数据库交互，以及`Variable`类。这将帮助我们在 Airflow 中管理秘密和参数。我们还创建了一个 SQLAlchemy 引擎来连接到本地 Postgres 数据库，并创建了一个 S3 客户端，允许将文件写入 S3：

    ```
    from airflow.decorators import task, dag
    from airflow.models import Variable
    from airflow.providers.postgres.operators.postgres import PostgresOperator
    from datetime import datetime
    import requests
    import pandas as pd
    from sqlalchemy import create_engine
    import boto3
    engine = create_engine('postgresql://postgres:postgres@postgres:5432/postgres')
    aws_access_key_id = Variable.get('aws_access_key_id')
    aws_secret_access_key = Variable.get('aws_secret_access_key')
    s3_client = boto3.client(
        's3',
        aws_access_key_id=aws_access_key_id,
        aws_secret_access_key=aws_secret_access_key
    )
    default_args = {
        'owner': 'Ney',
        'start_date': datetime(2024, 2, 12)
    }
    ```

1.  现在，让我们开始开发我们的 DAG。首先，定义四个任务——一个用于下载数据，第二个将其写入 Postgres 表。第三个任务将在 Postgres 中创建一个带有分组汇总的视图，最后一个任务将 CSV 文件上传到 S3。这最后一个任务是一个良好实践的示例，任务将数据处理发送到 Airflow 外部运行：

    ```
    @dag(
        default_args=default_args,
        schedule_interval="@once",
        description="Insert Data into PostgreSQL and AWS",
        catchup=False,
        tags=['postgres', 'aws']
    )
    def postgres_aws_dag():
        @task
        def download_data():
          destination = "/tmp/titanic.csv"
          response = requests.get(
    "https://raw.githubusercontent.com/neylsoncrepalde/titanic_data_with_semicolon/main/titanic.csv",
    stream=True
        )
          with open(destination, mode="wb") as file:
            file.write(response.content)
          return destination
        @task
        def write_to_postgres(source):
          df = pd.read_csv(source, sep=";")
          df.to_sql('titanic', engine, if_exists="replace", chunksize=1000, method='multi')
          create_view = PostgresOperator(
          task_id="create_view",
          postgres_conn_id='postgres',
          sql='''
    CREATE OR REPLACE VIEW titanic_count_survivors AS
    SELECT
    "Sex",
    SUM("Survived") as survivors_count
    FROM titanic
    GROUP BY "Sex"
    """,
        )
        @task
        def upload_to_s3(source):
          s3_client.upload_file(source, ' bdok-<ACCOUNT_NUMBER>
    ', 'titanic.csv')
    ```

    此时，你应该已经在 S3 中创建了一个名为`bdok-<YOUR_ACCOUNT_NUMBER>`的桶。在桶名中添加你的账户号是保证其唯一性的一种好方法。

1.  现在，保存你的文件并查看 Airflow UI。注意 DAG 不可用，Airflow 显示一个错误。展开错误消息，你会看到它抱怨我们在代码中尝试获取的变量（*图 6.8*）。这些变量还不存在。让我们创建它们。

![图 6.8 – Airflow UI – 变量错误](img/B21927_06_08.jpg)

图 6.8 – Airflow UI – 变量错误

1.  在上方菜单中，点击**Admin**并选择**Variables**。现在，我们将创建我们需要的 Airflow 环境变量。复制你的 AWS 密钥和访问密钥 ID，并相应地创建这些变量。创建变量后，你可以看到密钥在 UI 中被隐藏（*图 6.9*）。Airflow 会自动检测到秘密和敏感凭证，并在 UI 中隐藏它们。

![图 6.9 – Airflow UI – 变量已创建](img/B21927_06_09.jpg)

图 6.9 – Airflow UI – 变量已创建

1.  现在，让我们回到 UI 的主页。DAG 现在显示正确，但我们还没有完成。为了使`PostgresOperator`任务正确运行，它需要一个 Postgres 连接（记得`postgres_conn_id`参数吗？）。在上方菜单中，点击**Admin**，然后点击**Connections**。如*图 6.10*所示，添加一个新的连接。

![图 6.10 – Airflow UI – 新连接](img/B21927_06_10.jpg)

图 6.10 – Airflow UI – 新连接

1.  创建连接后，让我们继续开发 DAG。我们已经设置好了任务。现在是时候将它们链接在一起了：

    ```
        download = download_data()
        write = write_to_postgres(download)
        write >> create_view
        upload = upload_to_s3(download)
    execution = postgres_aws_dag()
    ```

    现在，我们准备好了。你还可以查看完整的代码，代码在 GitHub 上可用([`github.com/PacktPublishing/Bigdata-on-Kubernetes/blob/main/Chapter06/dags/postgres_aws_dag.py`](https://github.com/PacktPublishing/Bigdata-on-Kubernetes/blob/main/Chapter06/dags/postgres_aws_dag.py))。

1.  在 Airflow UI 中查看 DAG 的图表（*图 6.11*），并注意它是如何自动并行化所有可能的任务——在这种情况下，`write_to_postgres` 和 `upload_to_s3`。启动 DAG 的调度器让它运行。DAG 成功运行后，检查 S3 存储桶，以验证文件是否正确上传。然后，选择你喜欢的 SQL 客户端，检查数据是否已正确导入 Postgres。此示例中，我使用的是 DBeaver，但你可以选择任何你喜欢的客户端。

![图 6.11 – Airflow UI – 最终 DAG](img/B21927_06_11.jpg)

图 6.11 – Airflow UI – 最终 DAG

1.  在 DBeaver 中，创建一个新的 PostgreSQL 连接。对于 DBeaver 的连接，我们将使用 `localhost` 作为 Postgres 的主机（在 Airflow 中，我们需要使用 `postgres`，因为它运行在 Docker 中的共享网络内）。填写用户名和密码（在此例中为 `postgres` 和 `postgres`），并测试连接。如果一切正常，创建连接，然后让我们在这个数据库上执行一些查询。在 *图 6.12* 中，我们可以看到数据已正确导入。

![图 6.12 – DBeaver – 泰坦尼克号表格](img/B21927_06_12.jpg)

图 6.12 – DBeaver – 泰坦尼克号表格

1.  现在，让我们检查视图是否正确创建。结果显示在 *图 6.13* 中。

![图 6.13 – DBeaver – 创建的视图](img/B21927_06_13.jpg)

图 6.13 – DBeaver – 创建的视图

就这样！你创建了一个表格和一个视图，并通过 Airflow 上传了数据到 AWS！要停止 Airflow 容器，请返回终端并输入以下命令：

```
astro dev kill
```

# 总结

在本章中，我们介绍了 Apache Airflow 的基础知识——从安装到开发数据管道。你学习了如何利用 Airflow 来协调涉及数据获取、处理和与外部系统集成的复杂工作流。

我们通过 Astro CLI 和 Docker 在本地安装了 Airflow。这提供了一种无需复杂设置即可快速上手的方法。你接触到了 Airflow 的架构和关键组件，如调度器、工作节点和元数据数据库。理解这些组件对于监控和故障排除生产环境中的 Airflow 至关重要。

然后，我们有一个重点部分，介绍了如何构建你的第一个 Airflow DAG。你使用了核心 Airflow 操作符和任务、DAG 装饰器来定义和链式任务。我们讨论了最佳实践，比如保持任务小巧且独立。你还了解了 Airflow 如何处理任务依赖关系——允许并行执行独立任务。这些学习将帮助你开发出可扩展且可靠的 DAG。

随后，我们将 Airflow 与外部工具集成——写入 PostgreSQL，创建视图，并将文件上传到 S3。这展示了 Airflow 在协调涉及不同系统的工作流方面的多功能性。我们还配置了 Airflow 连接和变量，以安全地传递凭证。

到本章结束时，你应该已经掌握了 Airflow 的基础知识，并有了构建数据管道的实践经验。你现在已经具备了开发 DAG、整合其他工具以及应用生产级工作流最佳实践的能力。随着数据团队开始采用 Airflow，这些技能将对构建可靠且可扩展的数据管道至关重要。

在下一章，我们将学习实时数据的核心技术之一——Apache Kafka。
