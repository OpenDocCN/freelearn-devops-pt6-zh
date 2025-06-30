# *第三章*：设计一个操作符 – CRD、API 和目标协调

前几章的内容帮助我们理解了操作符框架的基础知识。在*第一章*《介绍操作符框架》中，我们讲解了操作符框架的概念支柱及其所服务的目的。接着，在*第二章*《理解操作符如何与 Kubernetes 交互》中，我们讨论了在**Kubernetes**和**操作符框架**背景下的某些软件设计原则。这些章节共同奠定了对操作符及其开发的基本理解。现在，我们将通过实例应用这些知识，开始设计我们自己的操作符。

我们将从定义一个简单的问题开始，该问题是我们的操作符将要解决的。在这种情况下，就是管理一个仅包含单个 Pod 的应用程序的基本部署。在接下来的几章中，我们将通过具体的代码示例为该操作符添加功能，但在开始编写我们的示例操作符之前，我们必须首先完成设计过程。通过具体的例子构建我们已讨论的通用定义和步骤，将为我们将早期的课程内容转化为实际应用提供上下文。

该过程将包括几个不同的步骤，以便阐明我们操作符的核心方面。这些步骤将包括绘制**应用程序编程接口**（**API**）、**自定义资源定义**（**CRD**）以及使操作符正常工作的协调逻辑。在此过程中，这些步骤将与之前讨论的内容以及行业标准的操作符最佳实践相联系。我们将把这个过程分解为以下几个步骤：

+   描述问题

+   设计 API 和 CRD

+   与其他必需资源的协作

+   设计目标协调循环

+   处理升级和降级

+   使用故障报告

除了一些适用的**YAML 不是标记语言**（**YAML**）代码片段外，我们还不会开始编写任何实际的代码。然而，我们将使用一些伪代码来更好地展示一旦我们用操作符**软件开发工具包**（**SDK**）初始化项目后，实际代码将如何工作，具体内容将在*第四章*《使用操作符 SDK 开发操作符》中进行讲解。

# 描述问题

许多软件项目都可以通过以下格式的用户故事来定义：*作为一个[用户]，我希望[动作]，以便[原因]。* 我们在这里也会这样做，具体如下：

*作为集群管理员，我希望使用操作符来管理我的 nginx 应用程序，以便自动管理它的健康状况和监控。*

对于我们的用例（设计一个 Operator），我们现在不关心具体的应用程序。因此，我们的*应用程序*将只是一个基本的 nginx 示例 Pod。我们将假设这代表任何具有基本**输入/输出**（**I/O**）需求的单 Pod 应用程序。虽然这看起来有些抽象，但重点将放在围绕应用程序构建 Operator 上。

我们从这个用户故事中确定的第一件事是，我们将为集群管理员构建这个 Operator。从上一章中，我们知道这意味着 Operator 的用户比大多数最终用户更了解集群架构，并且他们需要对 Operand 拥有更高层次的直接控制。我们还可以假设，大多数集群管理员会更愿意直接与 Operator 交互，而不是通过中介前端应用程序。

这个用户故事的第二部分明确了 Operator 的功能目标。具体来说，这个 Operator 将管理单 Pod 应用程序的部署。在这种情况下，*管理*是一个模糊的术语，我们将假设它意味着创建并维护运行应用程序所需的 Kubernetes 资源。这些资源至少包括一个 Deployment。我们需要通过 Operator 暴露这个 Deployment 的一些选项，例如 nginx 的容器端口。

最后，用户故事提供了我们运行 Operator 的动机。集群管理员希望 Operator 管理应用程序的*健康状况和监控*。应用程序健康可能意味着许多不同的事情，但通常来说，这归结为保持应用程序的高可用性，并尽可能从任何崩溃中恢复。监控应用程序也可以通过多种方式进行，通常以度量指标的形式呈现。

因此，从以上所有信息中，我们已经确定我们需要一个非常基础的 Operator，能够执行以下操作：

+   部署一个应用程序

+   如果应用程序失败，保持其运行

+   报告应用程序的健康状态

这些是 Operator 可以提供的一些最简单的功能。在后面的章节中，我们将对这些请求进行更多扩展。但为了从一个坚实的基础开始，以便以后迭代，这将是我们的**最小可行产品**（**MVP**）。因此，从现在开始，这将是我们在引用示例时所参考的基本 Operator 设计。

基于这些标准，我们可以尝试按照 *第一章*，*介绍运算符框架* 中涵盖的能力模型来定义我们的运算符（回顾一下，能力模型定义了运算符功能的五个级别，从 *基本安装* 到 *自动驾驶*）。我们知道，运算符将能够安装操作数，并管理任何额外所需的资源。我们希望它还能报告操作数的状态，并通过其 CRD 提供配置。这些都是 Level I 运算符的标准。此外，如果我们的运算符能够处理升级，那么它将符合 Level II 运算符的标准。

这是初始运算符设计的良好起点。有了我们试图解决的问题的全貌，我们现在可以开始头脑风暴，考虑我们将如何解决它。为此，我们可以从设计我们的运算符在集群 API 中的表示开始。

# 设计 API 和 CRD

正如我们在 *第一章*，*介绍运算符框架*，和 *第二章*，*理解运算符如何与 Kubernetes 交互* 中所介绍的，CRD 的使用是运算符的一个定义特征，用于创建用户交互的对象。这个对象为控制运算符创建了一个接口。通过这种方式，**自定义资源**（**CR**）对象就成为了运算符主要功能的窗口。

就像任何良好的窗户一样，运算符的 CRD 必须设计得很好。它必须足够清晰，以便暴露运算符的细节，同时又足够安全，以防止恶劣天气和入室盗窃。就像一个窗户一样，CRD 的设计应该遵循本地的建筑规范，以确保其按照环境的预期标准建造。在我们的案例中，这些建筑规范就是 Kubernetes API 的惯例。

## 遵循 Kubernetes API 设计惯例

尽管 CRD 是任何人都可以创建的自定义对象，仍然有一些最佳实践需要牢记。这是因为 CRD 存在于 Kubernetes API 中，该 API 定义了其惯例，因此在与 API 交互时有一定的期望。这些惯例在 Kubernetes 社区的文档中有所记录：[`github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md`](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md)。然而，这份文档非常详尽，涵盖了所有种类 API 对象的要求，而不仅限于运算符的 CRD。然而，有几个关键要素对我们的目的是相关的，如下所述（注意：本章节稍后将更详细地讨论一些字段）：

+   所有 API 对象必须包含两个字段：`kind` 和 `apiVersion`。这两个字段使得 Kubernetes API 客户端能够解码对象。`kind` 表示对象类型的名称——例如，`MyOperator`——而 `apiVersion` 是该对象的 API 版本。例如，您的 Operator 可能会提供 `v1alpha1`、`v1beta1` 和 `v1` 版本的 API。

+   API 对象还应包含以下字段（尽管它们不是必需的）：

    +   `resourceVersion` 和 `generation`，这两个字段有助于跟踪对象的变化。然而，这些字段的用途不同。`resourceVersion` 字段是一个内部引用，每次对象被修改时都会递增，用于帮助并发控制。例如，当尝试更新一个 Operand 部署时，您将进行两次客户端调用：`Get()` 和 `Update()`。在调用 `Update()` 时，API 可以检测到对象的 `resourceVersion` 字段是否已在服务器上发生变化（这表明其他控制器在我们更新之前已修改了该对象），如果是，则拒绝更新。相比之下，`generation` 用于跟踪对象的相关更新。例如，最近推出新版本的 Deployment 将会更新其 `generation` 字段。这些值可用于引用旧的 generation，或者确保新的 generation 是预期的 generation 编号（即 *current+1*）。

    +   `creationTimestamp` 和 `deletionTimestamp`，这两个字段有助于参考对象的年龄。例如，通过 `creationTimestamp`，您可以轻松根据 CRD 创建时间引用 Operator 部署的年龄。类似地，`deletionTimestamp` 表示该对象已向 API 服务器发送了删除请求。

    +   `labels` 和 `annotations`，它们的作用相似，但在语义上有所不同。将 `labels` 应用于一个对象，可以通过 API 轻松筛选出符合条件的对象，从而实现组织管理。另一方面，`annotations` 提供了关于对象的元数据。

+   API 对象应包含 `spec` 和 `status` 字段。我们将在本章稍后部分更详细地讨论 `status`（在 *使用故障报告* 部分），但目前，需要牢记一些关于它的约定，如下所示：

    +   在对象的 `status` 字段中报告的条件应当能够清晰自解释，无需额外的上下文来理解它们。

    +   条件应遵循与其他字段相同的 API 兼容性规则。为了保持向后兼容，条件定义一旦确定不应再更改。

    +   条件可以报告 `True` 或 `False` 作为其正常操作状态。对于哪个应作为标准模式，并没有固定的指导方针；开发者应根据条件定义中的可读性来做出考虑。例如，`Ready=true` 条件的含义与 `NotReady=false` 相同，但前者更容易理解。

    +   条件应表示集群的当前已知状态，而不是报告状态之间的过渡。正如我们在 *设计目标协调循环* 部分中所涵盖的，许多 Kubernetes 控制器是使用级别触发设计编写的（意味着它们基于集群的当前状态而不是仅仅基于传入事件进行操作）。因此，Operator 基于其当前状态报告条件，有助于保持这种相互设计假设，即可以随时在内存中构建集群的当前状态。然而，对于长期过渡阶段，如果需要，可以使用 `Unknown` 条件。

+   API 对象中的子对象应该表示为列表，而不是映射；例如，我们的 nginx 部署可能需要通过 Operator CRD 指定多个不同的命名端口。这意味着它们应该表示为列表，列表中的每个项目都有 `name` 和 `port` 字段，而不是每个条目的 `name` 作为映射中的键。

+   可选字段应该实现为指针值，以便轻松区分零值和未设置值。

+   带有单位的字段的约定是将单位包括在字段名称中，例如 `restartTimeoutSeconds`。

这些只是许多 API 约定中的一部分，但它们在设计 Operator 的 CRD 和 API 时非常重要。遵循 API 设计的指导方针，确保 Kubernetes 生态系统中的其他组件（包括平台本身）可以对你的 Operator 做出适当的假设。牢记这些指导方针，我们可以进入设计我们自己 CRD 模式的下一步。

## 理解 CRD 模式

我们已经讨论了 CRD 及其对 Operator 的重要性，但到目前为止，我们还没有深入探讨 CRD 的组成方式。现在，我们已经知道了我们的示例 Operator 要解决的问题，我们可以开始查看我们希望通过 CRD 暴露的选项，并了解这些选项对用户的表现形式。

首先，最好查看一个示例 CRD，并检查每个部分以了解其目的，如下所示：

```
apiVersion: apiextensions.k8s.io/v1
```

```
kind: CustomResourceDefinition
```

```
metadata:
```

```
  name: myoperator.operator.example.com
```

```
spec:
```

```
  group: operator.example.com
```

```
  names:
```

```
    kind: MyOperator
```

```
    listKind: MyOperatorList
```

```
    plural: myoperators
```

```
    singular: myoperator
```

```
  scope: Namespaced
```

```
  versions:
```

```
    - name: v1alpha1
```

```
      schema:
```

```
        openAPIV3Schema:
```

```
         ...
```

```
      served: true
```

```
      storage: true
```

```
      subresources:
```

```
        status: {}
```

```
status:
```

```
  acceptedNames:
```

```
    kind: ""
```

```
    plural: ""
```

```
  conditions: []
```

```
  storedVersions: []
```

前两个字段，`apiVersion` 和 `kind`，定义了这是一个 CRD。即使我们试图定义我们自己自定义对象的蓝图，这个蓝图首先必须存在于 `CustomResourceDefinition` 对象中。从这里，API 服务器会知道如何解析 CRD 数据，以创建我们 CR 的实例。

接下来，`metadata.Name` 字段定义了我们 CRD 的名称（不是从 CRD 创建的自定义对象）。更具体地说，这是蓝图的名称，而不是从蓝图创建的对象。例如，我们可以使用 `kubectl get crd/myoperator.operator.example.com` 来检索这个 CRD 设计。

在 `spec` 中，CRD 开始实际定义我们要创建的 CR 对象。`group` 定义了一个自定义 API 组，新对象将属于该组。这里使用唯一名称有助于避免与集群中其他对象和 API 的冲突。

`names` 部分定义了我们对象的不同引用方式。在这里，只有 `kind` 和 `plural` 是必需的（因为其他的可以从这两个推断出）。就像集群中任何其他类型的对象都可以通过其 `kind` 或 `plural` 形式进行访问（例如，`kubectl get pods`），我们的 CR 也将通过类似的命令进行访问，如 `kubectl edit myoperator/foo`。尽管大多数操作员（并且应该）在集群中只拥有一个 CR 对象，这些字段仍然是必需的。

接下来，`scope` 定义了自定义对象的范围，可以是命名空间范围或集群范围。这两者之间的区别在 *第一章* 中已经详细介绍过，*引入操作员框架*。此字段的可选值为 `Cluster` 和 `Namespaced`。

`Versions` 提供了我们 CR 可用的不同 API 版本的列表。随着操作员的演进以及新功能的添加或移除，您将需要引入新的操作员 CR 版本。为了向后兼容和支持，您应该继续发布旧版本的资源，以便为用户提供一个过渡期，之后该版本可以安全地弃用。这就是为什么此字段提供版本列表的原因。API 服务器知道每个版本，并能够有效地处理在该列表中创建和使用的任何版本的对象。

列表中的每个版本都包含关于对象本身的架构信息，用于唯一标识该版本在`openAPIV3Schema`中的结构。在这个示例中，`openAPIV3Schema` 部分故意被省略了。我们之所以这样做，是因为该部分通常非常长且复杂。然而，在 Kubernetes 的最新版本中，这个部分是必须的，以便为 CRD 提供**结构化架构**。

结构化架构是基于**OpenAPI 版本 3 (V3) 验证**的对象架构。OpenAPI 定义了每个字段的验证规则，这些规则可以在创建或更新对象时用于验证字段数据。这些验证规则包括字段的数据类型，以及其他信息，如允许的字符串模式和枚举值。CRD 的结构化架构要求确保对象的表示在存储时是一致和可靠的。

由于 OpenAPI 验证架构的复杂性，不建议手写它们。建议使用像**Kubebuilder**（Operator SDK 使用的工具）这样的生成工具。可以直接在 Go 类型上定义全面且灵活的验证规则，使用各种 Kubebuilder 标记，这些标记的完整参考文档可以在[`book.kubebuilder.io/reference/generating-crd.html`](https://book.kubebuilder.io/reference/generating-crd.html)找到。

个别版本定义的下一个部分是`served`和`storage`，它们决定了该版本是否通过 REST API 提供服务，以及是否是应该用作存储表示的版本。每个 CRD 只能设置一个版本作为存储版本。

最后的部分是`subresources`和`status`，它们是相关的，因为它们定义了一个`status`字段，该字段用于报告 Operator 当前状态的信息。我们将在*使用故障报告*部分更详细地介绍该字段及其用途。

现在，我们已经了解了 CRD 的结构，并且知道一个 CRD 应该是什么样的，我们可以为我们的示例 nginx Operator 设计一个 CRD。

## 示例 Operator CRD

从前面的需求分析中，我们知道我们的 Operator 最初只会在集群中部署一个 nginx 实例。我们现在也知道，Operator 的 CRD 将提供一个`spec`字段，里面包含各种控制操作数部署的选项。但我们应该暴露什么样的设置呢？我们的操作数比较简单，所以我们从定义一些基本选项开始，用来配置一个简单的 nginx Pod，如下所示：

+   `port`—这是我们希望在集群内暴露 nginx Pod 的端口号。由于 nginx 是一个 Web 服务器，这将允许我们修改可访问的端口，而无需直接操作 nginx Pod，因为 Operator 将为我们安全地处理这个变更。

+   `replicas`—这个字段有点多余，因为 Kubernetes 部署的副本数可以通过部署本身进行调整。但为了将操作数的控制抽象到 Operator 的**用户界面**（**UI**）背后，我们提供了这个选项。这样，管理员（或其他应用程序）可以在 Operator 的处理和报告下扩展操作数。

+   `forceRedeploy`—这个字段很有趣，因为它实际上对操作数（Operand）来说是**无操作**（**no-op**），即它的功能没有任何改变。然而，在 Operator 的 CRD 中包括一个可以设置为任意值的字段，可以让我们指示 Operator 触发操作数的重新部署，而不需要修改任何实际的设置。这个功能对于卡住的部署（Deployment）非常有用，手动干预通常可以解决这个问题。

之所以可行，是因为操作员监视集群中相关资源的变化，其中之一就是它自己的 CRD（更多内容请参见*设计目标调和循环*部分）。这种监视是必要的，以便操作员知道何时更新操作数。因此，包含一个无操作字段就足够让操作员知道重新部署操作数，而无需进行任何实际更改。

这三项设置将构成我们操作员 CRD `spec` 的基础。有了这个，我们知道作为集群中的对象，CR 对象将如下所示：

```
apiVersion: v1alpha1
```

```
kind: NginxOperator
```

```
metadata:
```

```
  name: instance
```

```
spec:
```

```
  port: 80
```

```
  replicas: 1
```

```
status:
```

```
  ...
```

请注意，这里展示的是 CR 本身，而不是 CRD，如前面示例所示。我们在这里使用通用的 `name: instance` 值，因为我们可能一次只会在一个命名空间中运行一个操作员实例。我们也没有在这里包含 `forceRedeploy` 字段，因为该字段是可选的。

如果我们正确地定义了 CRD，可以通过 `kubectl get -o yaml nginxoperator/instance` 命令来检索此对象。幸运的是，Operator SDK 和 Kubebuilder 将帮助我们生成该内容。

# 与其他必需资源的协作

除了 CRD，我们的操作员还将负责管理许多其他集群资源。目前，这是作为我们的操作数将要创建的 nginx Deployment，以及用于操作员的 ServiceAccount、Role 和 RoleBinding。我们需要理解的是，操作员将如何知道这些资源的定义。

在某处，这些资源需要作为 Kubernetes 集群对象书写。就像你手动创建 Deployment 一样（例如，使用 `kubectl create -f`），所需资源的定义可以通过几种不同的方式与操作员代码打包。这可以通过模板轻松完成，如果你使用 Helm 或 Ansible 创建操作员，但是对于用 **Go** 编写的操作员，我们需要考虑我们的选择。

将这些资源打包以便操作员可以创建它们的一种方法是直接在操作员的代码中定义它们。所有 Kubernetes 对象都基于相应的 Go 类型定义，因此我们有能力通过将资源声明为变量，直接在操作员中创建 Deployments（或任何资源）。以下是一个示例：

```
…
```

```
import appsv1 "k8s.io/api/apps/v1"
```

```
…
```

```
nginxDeployment := &appsv1.Deployment{
```

```
  TypeMeta: metav1.TypeMeta{
```

```
    Kind: "Deployment",
```

```
    apiVersion: "apps/v1",
```

```
  },
```

```
  ObjectMeta: metav1.ObjectMeta{
```

```
    Name: "nginx-deploy",
```

```
    Namespace: "nginx-ns",
```

```
  },
```

```
  Spec: appsv1.DeploymentSpec{
```

```
    Replicas: 1
```

```
    Selector: &metav1.LabelSelector{
```

```
      MatchLabels: map[string]string{"app":"nginx"},
```

```
    },
```

```
    Template: v1.PodTemplateSpec{
```

```
      Spec: v1.PodSpec{
```

```
        ObjectMeta: metav1.ObjectMeta{
```

```
          Name: "nginx-pod",
```

```
          Namespace: "nginx-ns",
```

```
          Labels: map[string]string{"app":"nginx"},
```

```
        },
```

```
        Containers: []v1.Container{
```

```
          {
```

```
             Name: "nginx",
```

```
             Image: "nginx:latest",
```

```
             Ports: []v1.ContainerPort{{ContainerPort: int32(80)}},
```

```
          },
```

```
        },
```

```
      },
```

```
    },
```

```
  },
```

```
}
```

以这种方式在代码中定义对象的便利性对开发非常有帮助。这种方法提供了透明的定义，Kubernetes API 客户端可以清楚地访问并立即使用它。然而，这也有一些缺点。首先，这种方式并不十分人性化。用户通常会与以 YAML 或 **JavaScript 对象表示法**（**JSON**）表示的 Kubernetes 对象进行交互，而这些表示法中每个字段的类型定义并不存在。这些信息对于大多数用户来说是不必要且多余的。因此，任何希望清晰查看资源定义或修改它们的用户可能会在操作员的代码中迷失。

幸运的是，直接将资源定义为 Go 类型还有另一种选择。有一个非常有用的包叫做`go-bindata`（可在[github.com/go-bindata/go-bindata](http://github.com/go-bindata/go-bindata)找到），它将声明性 YAML 文件编译成 Go 二进制文件，使得它们可以通过代码访问。Go 的新版本（1.16+）现在也包含了 `go:embed` 编译指令，可以在不使用像 `go-bindata` 这样的外部工具的情况下实现这一点。因此，我们可以像这样简化前面的部署定义：

```
kind: Deployment
```

```
apiVersion: apps/v1
```

```
metadata:
```

```
  name: nginx-deploy
```

```
  namespace: nginx-ns
```

```
spec:
```

```
  replicas: 1
```

```
  selector:
```

```
    matchLabels:
```

```
      app: nginx
```

```
  template:
```

```
    metadata:
```

```
      labels:
```

```
        app: nginx
```

```
    spec:
```

```
      containers:
```

```
      - name: nginx
```

```
         image: nginx:latest
```

```
         ports:
```

```
          - containerPort: 80
```

这种方式对普通用户来说更加可读。它也更容易维护，您可以在操作员的代码库中为不同版本的各种资源提供命名目录。这对代码组织很有好处，同时也简化了您对**持续集成**（**CI**）检查这些类型定义有效性的选项。

我们将在*第四章*，*使用 Operator SDK 开发 Operator* 中更详细地讲解如何使用 `go-bindata` 和 `go:embed`，但现在，我们已经知道如何将额外的资源打包，以便在操作员中使用。这是一个关键的设计考虑因素，既有利于我们的用户，也有利于维护人员。

# 设计目标协调循环

现在，我们已经通过设计一个 CRD 来表示操作员的 UI，并列出了它将管理的操作数资源，接下来可以进入操作员的核心逻辑。这个逻辑嵌入在主协调循环中。

如前几章所述，操作员的工作原理是根据将集群的当前状态与用户设置的期望状态进行协调。它们通过定期检查当前状态来实现这一点。这些检查通常由与操作数相关的某些事件触发。例如，操作员将监视其目标操作数命名空间中的 Pod，并对 Pod 的创建或删除做出反应。由操作员开发人员来定义哪些事件对操作员来说是重要的。

## 基于层级与基于边缘的事件触发

当事件触发操作员的协调循环时，逻辑不会接收到整个事件的上下文。相反，操作员必须重新评估集群的整个状态，以执行其协调逻辑。这被称为**基于层级的触发**。这种设计的替代方案是**基于边缘的触发**。在基于边缘的系统中，操作员逻辑只会在事件本身上起作用。

这两种系统设计的权衡在于效率与可靠性之间的选择。基于边缘的系统更高效，因为它们不需要重新评估整个状态，只能对相关信息做出反应。然而，基于边缘的设计可能会遭遇不一致和不可靠的数据问题——例如，如果事件丢失。

另一方面，基于级别的系统始终意识到系统的整个状态。这使得它们更适合用于大规模的分布式系统，如 Kubernetes 集群。虽然这些术语最初来源于与电子电路相关的概念，但它们在上下文中与软件设计也有很好的关系。更多信息请访问[`venkateshabbarapu.blogspot.com/2013/03/edge-triggered-vs-level-triggered.html`](https://venkateshabbarapu.blogspot.com/2013/03/edge-triggered-vs-level-triggered.html)。

理解这些设计选择之间的差异使我们能够思考对账逻辑如何运作。通过采用基于级别的触发方法，我们可以确保操作员不会丢失任何信息或错过任何事件，因为它内存中的集群状态表示最终总会追赶到现实。然而，我们必须考虑实现基于级别设计的要求。具体来说，操作员每次触发事件对账时，必须具备构建整个相关集群状态所需的信息。

## 设计对账逻辑

对账循环是操作员的核心功能。当操作员接收到事件时，正是这个函数被调用，并且这是编写操作员主要逻辑的地方。此外，这个循环理想情况下应该设计为管理一个 CRD，而不是让单个控制循环承担多个责任。

当使用 Operator SDK 来搭建 Operator 项目时，对账循环的函数签名将是这样的：

```
func (r *Controller) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error)
```

这个函数是一个`Controller`对象的方法（这个名字可以是任意的；在本示例中我们使用`Controller`，但它也可以是`FooOperator`）。这个对象将在操作员启动时实例化。然后它接受两个参数：`context.Context`和`ctrl.Request`。最后，它返回一个`ctrl.Result`参数，如果适用的话，还会返回一个`error`参数。我们将在*第四章*《使用 Operator SDK 开发 Operator》中详细讲解这些类型及其具体作用，但目前，请理解，操作员的核心对账循环是基于非常有限的关于触发对账事件的信息构建的。请注意，操作员的 CRD 和集群状态信息并不会传递给这个循环；也没有其他任何信息。

因为操作员是基于级别驱动的，所以`Reconcile`函数应该构建集群本身的状态。在伪代码中，这通常看起来像这样：

```
func Reconcile:
```

```
  // Get the Operator's CRD, if it doesn't exist then return
```

```
  // an error so the user knows to create it
```

```
  operatorCrd, error = getMyCRD()
```

```
  if error != nil {
```

```
    return error
```

```
  }
```

```
  // Get the related resources for the Operator (ie, the
```

```
  // Operand's Deployment). If they don't exist, create them
```

```
  resources, error = getRelatedResources()
```

```
  if error == ResourcesNotFound {
```

```
    createRelatedResources()
```

```
  }
```

```
  // Check that the related resources relevant values match
```

```
  // what is set in the Operator's CRD. If they don't match,
```

```
  // update the resource with the specified values.
```

```
  if resources.Spec != operatorCrd.Spec {
```

```
    updateRelatedResources(operatorCrd.Spec)
```

```
  }
```

这将是我们操作员对账循环的基本布局。将这个过程分解成步骤，过程是这样的：

1.  首先，检查是否存在现有的操作符 CRD 对象。正如我们所知道的，操作符 CRD 包含了操作符如何工作的配置设置。最佳实践是操作符不应管理自己的 CRD，因此如果集群中没有 CRD，则应立即返回错误。该错误将显示在操作符的 Pod 日志中，提示用户应该创建 CRD 对象。

1.  其次，检查集群中是否存在相关资源。对于我们的当前用例，这将是操作数部署。如果部署不存在，那么创建它将是操作符的职责。

1.  最后，如果相关资源已经存在于集群中，则需要检查它们是否根据操作符 CRD 中的设置进行配置。如果没有，那么就需要使用预定的值更新集群中的资源。虽然我们可以每次直接更新（因为我们知道期望的状态，而无需查看当前状态），但最好先检查差异。这有助于减少不必要的 API 调用，避免盲目更新。不必要的更新还会增加更新热循环的风险，导致操作符对资源的更新创建事件，触发处理该对象的调和循环。

这三个步骤在很大程度上依赖于通过标准 API 客户端访问 Kubernetes API。操作符 SDK 提供了帮助简化实例化这些客户端并将其传递给操作符控制循环的函数。

# 处理升级和降级

作为操作符开发者，我们关注的是两个主要应用程序的版本控制：操作数（Operand）和操作符本身。无缝升级也是 II 级操作符的核心功能，我们已经决定将其作为初始操作符设计的目标。因此，我们必须确保我们的操作符能够处理自身以及 nginx 操作数的升级。就我们的用例而言，升级操作数相对简单。我们可以直接拉取新的镜像标签并更新操作数部署。然而，如果操作数发生了显著变化，那么操作符可能也需要更新，以便能够正确管理新的操作数版本。

操作符升级通常是在需要将操作符代码、API 或者两者的更改交付给用户时发生的。**操作符生命周期管理器**（**OLM**）使得从用户角度来看，升级到更新版本的操作符变得容易。操作符的**ClusterServiceVersion**（**CSV**）允许开发者为维护者定义特定的升级路径，提供有关替代旧版本的新版本的具体信息。这将在*第七章*《使用操作符生命周期管理器安装和运行操作符》中详细介绍，届时我们将为操作符编写一个 CSV 文件。

也可能出现操作员的 CRD 以不兼容的方式发生变化的情况（例如，某个旧字段的弃用）。在这种情况下，操作员的 API 版本应该增加（例如，从`v1alpha1`到`v1alpha2`或`v1beta1`）。新版本还应该与现有版本的 CRD 一起发布。这就是为什么 CRD 的`versions`字段是一个版本定义列表，并且通过同时支持两个版本，它允许用户从一个版本过渡到下一个版本的原因。

然而请记住，在这些版本列表中，可能只有一个是指定的存储版本。同时，永远发布每个先前的 API 版本也是过度的（最终，旧版本需要在适当的弃用时间线之后完全移除）。当是时候永久移除对弃用的 API 版本的支持时，存储版本可能也需要更新。这可能会对仍然在集群中安装旧版本操作员 CRD 作为存储版本的用户造成问题。`kube-storage-version-migrator`工具（[`github.com/kubernetes-sigs/kube-storage-version-migrator`](https://github.com/kubernetes-sigs/kube-storage-version-migrator)）通过为集群中现有对象提供迁移过程来帮助解决这个问题。存储版本可以通过`Migration`对象迁移，例如：

```
apiVersion: migration.k8s.io/v1alpha1
```

```
kind: StorageVersionMigration
```

```
metadata:
```

```
  name: nginx-operator-storage-version-migration
```

```
spec:
```

```
  resource:
```

```
    group: operator.example.com
```

```
    resource: nginxoperators
```

```
    version: v1alpha2
```

当创建这个对象时，`kube-storage-version-migrator`会看到它，并将集群中存储的任何现有对象更新到指定版本。此操作只需要执行一次，甚至可以通过将此对象打包为操作员的附加资源来实现自动化。未来版本的 Kubernetes 将完全自动化这个过程（见 *KEP-2855*，[`github.com/kubernetes/enhancements/pull/2856`](https://github.com/kubernetes/enhancements/pull/2856)）。

提前为成功的版本过渡做好准备，将有助于未来操作员的维护。然而，并不是所有的事情都能顺利进行，无法为每个可能的场景做好准备。这就是为什么操作员需要具有足够的错误报告和处理机制的重要原因。

# 使用故障报告

说到故障，我们需要担心两件事：操作对象中的故障，以及操作员本身的故障。有时，这两者可能甚至是相关的（例如，操作对象以操作员无法解决的意外方式失败）。当发生任何错误时，操作员的一个重要任务是将这些错误报告给用户。

当操作员的协调循环中发生错误时，操作员必须决定接下来该怎么做。在使用 Operator SDK 的实现中，协调逻辑能够识别错误发生的时刻，并尝试重新执行循环。如果错误持续导致协调失败，循环可以进行指数级的退避，在每次尝试之间等待更长时间，希望造成失败的某些条件能够得到解决。然而，当操作员达到这种状态时，错误仍然应该以某种方式暴露给用户。

错误报告可以通过几种方式轻松完成。报告故障的主要方法有日志记录、状态更新和事件。每种方法都有不同的优点，但一个复杂的操作员设计会优雅地利用这三者的结合。

## 使用日志报告错误

报告错误最简单的方法是基本日志记录。这是大多数软件项目向用户报告信息的方式，不仅仅是 Kubernetes 操作员。因为日志输出相对容易实现，并且对于大多数用户来说，直观易懂。特别是考虑到 `kubectl logs pod/my-pod` 等日志库的可用性，这个理由更加成立。然而，单纯依赖日志来报告重大错误也有一些缺点。

首先，Kubernetes Pod 日志不是持久化的。当 Pod 崩溃并退出时，只有在失败的 Pod 被集群的 **垃圾回收**（**GC**）过程清理掉之前，日志才会存在。这使得调试故障变得特别困难，因为用户与时间赛跑。此外，如果操作员正努力修复问题，用户也会与他们自己的自动化系统赛跑，而自动化系统应该是帮助他们而不是阻碍他们。

其次，日志可能包含大量信息需要解析。除了你自己可能写的相关操作员日志外，操作员通常会依赖许多库和依赖项，它们会将自己的信息注入到日志输出中。这可能会形成一个混乱的日志堆积，用户需要花费精力去整理。尽管有像 `grep` 这样的工具可以让你相对容易地搜索大量文本，但用户可能并不总是知道首先要搜索哪些文本。这可能会在调试问题时造成严重的延误。

日志有助于追踪操作员的过程步骤或进行低级调试。然而，它们并不擅长将故障立即引起用户的注意。Pod 日志的有效期很短，而且常常被无关的日志所淹没。此外，日志本身通常不会提供太多可供调试的可读上下文。这就是为什么需要注意的重要故障，最好通过状态更新和事件来处理。

## 使用状态更新报告错误

正如本章前面在讨论 Kubernetes API 约定和 CRD 设计时提到的，Operator CRD 应该提供两个重要字段：`spec` 和 `status`。`spec` 表示集群的期望状态并接受用户输入，而 `status` 用于报告集群的当前状态，并应仅作为输出形式更新。

通过利用 `status` 字段来报告 Operator 及其 Operand 的健康状况，你可以轻松地以易读的格式突出显示重要的状态信息。该格式基于条件类型，由 Kubernetes API 机制提供。

条件报告其名称以及一个布尔值，指示条件当前是否存在。例如，Operator 可以报告 `OperandReady=false` 条件，表示 Operand 不健康。条件中还有一个名为 `Reason` 的字段，允许开发者提供当前状态的更易读的解释。从 Kubernetes `1.23` 开始，`Condition` 的 `Type` 字段最大长度为 `316` 个字符，`Reason` 字段最大可达 `1,024` 个字符。

Kubernetes API 客户端提供了报告条件的函数，如 `metav1.SetStatusCondition(conditions *[]metav1.Condition, newCondition metav1.Condition)`。这些函数（以及 `Condition` 类型本身）存在于 `k8s.io/apimachinery/pkg/apis/meta/v1` 包下。

在 Operator 的 CRD `status` 字段中，条件类似如下所示：

```
status:
```

```
  conditions:
```

```
    - type: Ready 
```

```
      status: "True"
```

```
      lastProbeTime: null
```

```
      lastTransitionTime: 2018-01-01T00:00:00Z
```

对于我们的 nginx 部署 Operator，我们将首先报告一个名为`Ready`的条件。我们将在 Operator 启动成功时将此条件的状态设置为`True`，如果 Operator 在执行协调循环时失败，将其更改为`False`（同时在`Reason`字段中提供更详细的失败说明）。我们可能会发现更多合适的条件，可以在以后添加，但考虑到 Operator 的初步简洁性，这应该已足够。

使用条件有助于显示 Operator 及其管理资源的当前状态，但这些条件仅在 Operator 的 CRD 的`status`部分显示。然而，我们可以将它们与事件结合使用，使错误报告在整个集群中可用。

## 使用事件报告错误

Kubernetes 事件是一个原生 API 对象，像 Pods 或集群中的其他对象一样。事件会被聚合，并在使用`kubectl describe`描述 Pod 时显示出来。它们也可以通过`kubectl get events`独立地进行监控和过滤。它们在 Kubernetes API 中的可用性使得其他应用程序（如告警系统）也能理解它们。

下面是列出 Pod 事件的示例，在这里我们看到五个不同的事件：

1.  发生三次的 `Warning` 事件，表明 Pod 无法调度。

1.  当 Pod 成功调度后，记录一个 `Normal` 事件。

1.  另外有三个`Normal`事件，作为 Pod 的容器镜像被拉取、创建并成功启动。

你可以在以下代码片段中看到这些事件：

```
$ kubectl describe pod/coredns-558bd4d5db-6mqc2 -n kube-system
```

```
…
```

```
Events:
```

```
  Type     Reason            Age                    From               Message
```

```
  ----     ------            ----                   ----               -------
```

```
  Warning  FailedScheduling  6m36s (x3 over 6m52s)  default-scheduler            0/1 nodes are available: 1 node(s) had taint {node.kubernetes.io/not-ready: }, that the pod didn't tolerate.
```

```
  Normal   Scheduled         6m31s                  default-scheduler            Successfully assigned kube-system/coredns-558bd4d5db-6mqc2 to kind-control-plane
```

```
  Normal   Pulled            6m30s                  kubelet, kind-control-plane  Container image "k8s.gcr.io/coredns/coredns:v1.8.0" already present on machine
```

```
  Normal   Created           6m29s                  kubelet, kind-control-plane  Created container coredns
```

```
  Normal   Started           6m29s                  kubelet, kind-control-plane  Started container coredns
```

由于事件对象设计更加复杂，事件可以传递比条件更多的信息。虽然事件包括`Reason`和`Message`字段（分别与条件的`Type`和`Reason`字段类似），但它们还包括诸如`Count`（显示该事件发生的次数）、`ReportingController`（显示事件的源控制器）和`Type`（可用于筛选不同严重程度的事件）等信息。

`Type`字段目前可以用来将集群事件分类为`Normal`或`Warning`。这意味着，类似于条件可以报告成功状态，事件也可以用来表明某些功能已成功完成（例如启动或升级）。

为了让 Pod 将事件报告给集群，代码需要实现一个`EventRecorder`对象。这个对象应该在整个控制器中传递，并将事件广播到集群中。Operator SDK 和 Kubernetes 客户端提供了模板代码，帮助正确设置这个功能。

除了报告事件之外，你的 Operator 还会对集群中的事件作出反应。这回到 Operator 基于事件触发的设计核心基础。设计哪些事件类型需要被 Operator 响应的代码模式是存在的，你可以在其中加入逻辑来过滤特定事件。这个内容将在后续章节中详细讨论。

如你所见，一个复杂的错误报告系统利用日志、状态和事件来提供应用程序状态的完整图景。每种方法都有其独特的好处，它们一起编织了一幅美丽的调试图景，帮助管理员追踪故障并解决问题。

# 总结

本章概述了我们希望构建的 Operator 的详细信息。从描述问题（在本例中是一个管理 nginx Pod 的简单 Operator）开始，为可用的解决方案提供了坚实的基础。这个步骤甚至提供了足够的信息，以设定这个 Operator 的功能级别目标（*Level II – 无缝升级*）。

下一步是概述 Operator CRD 的结构。为此，我们首先注意到 Kubernetes API 中一些相关的约定，这些约定有助于确保 Operator 符合 Kubernetes 对象的预期标准。然后，我们拆解了 CRD 的结构，并解释了每个部分如何与对应的 CR 对象相关联。最后，我们草拟了一个示例，展示了 Operator 的 CR 在集群中的样子，以便更具体地了解用户的期望。

在设计 CRD 后，我们也考虑了如何管理其他资源。对于用 Go 编写的 Operator，将额外资源（如 RoleBinding 和 ServiceAccount 定义）打包成 YAML 文件是合适的做法。这些文件可以通过 `go-bindata` 和 `go:embed` 编译到 Operator 二进制文件中。

设计的下一步是目标对账循环。它包含了 Operator 的核心逻辑，也是使 Operator 成为有用且功能强大的应用的关键。这一过程始于理解水平触发和边缘触发事件处理的区别，以及为什么 Operator 更适合基于水平触发。接着，我们讨论了 Operator 对账循环的基本步骤。

最后的两个部分讨论了升级、降级和错误报告的话题。在升级和降级部分，我们介绍了同时发布和支持多个 API 版本的使用场景，以及偶尔需要在现有安装中迁移存储版本的必要性。关于错误报告的部分则聚焦于应用程序向用户暴露健康信息的三种主要方式：日志记录、状态条件和事件。

在下一章中，我们将以我们已决定的内容作为初始设计，并将其转化为实际代码。这将涉及使用 Operator SDK 初始化项目，生成将成为 Operator 的 CRD 的 API，并编写目标对账逻辑。实质上，我们将把本章中的知识应用到 Operator 开发的实际操作中。
