# GraphQL 基础

在我们开始构建一个包含客户端和服务器端完整的 GraphQL 应用之前，让我们通过前面章节安装的一些工具来体验一下 GraphQL 的工作方式。你可以选用 GraphiQL 或者 GitHub 提供的 GraphQL Explorer。 接下来，通过执行你的第一个 GraphQL 查询、变更，以及探索 GitHub 的 GraphQL API 中的一些特性，例如分页等，来学习 GraphQL 基础操作。

## GraphQL 基础: 查询 

在本节中，你可以使用查询和变更同 GitHub API 交互，可以通过 GraphiQL 应用或者 GitHub 的 GraphQL Explorer 发送查询请求到 GitHub API，本节暂时不会涉及和 React 集成相关知识。 这两种工具都需要使用个人申请的 access token 授权。在 GraphiQL 应用的左侧，允许你输入 GraphQL 查询和变更语句来调试 GraphQL 请求。尝试输入下面的语句获取你的个人信息数据。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  viewer {
    name
    url
  }
}
~~~~~~~~

`viewer` 对象可以被用来获取当前授权的用户信息。通过你的个人 access token 获得授权并完成查询请求后，应该能看到相关的用户信息被正确返回。`viewer` 是一个 GraphQL 中**对象**的概念。 对象承载某个实体的数据，这些数据可以通过 GraphQL 中的**字段**访问。字段被用于获取对象中指定的属性。举个例子来说，`viewer` 对象暴露了多个字段，在这个例子中，只有 `name` 和 `url` 在查询中用到了。在大多数基本情况下，一个查询只包含对象和字段，当然对象也是一种字段。

当你在 GraphiQL 中执行完上面的查询语句，你可以看到类似如下的返回内容，上面的 name 和 URL 被替换成对应的返回值。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "data": {
    "viewer": {
      "name": "Robin Wieruch",
      "url": "https://github.com/rwieruch"
    }
  }
}
~~~~~~~~

恭喜你，你成功地执行了第一个查询并且获取到了你在 GitHub 中个人信息中的相关字段。现在，让我们看看怎么去获取其他的资源，例如 GitHub 中开放出来的组织信息。为了获取特定的 GitHub 组织信息，你需要传入一个**参数**：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  organization(login: "the-road-to-learn-react") {
    name
    url
  }
}
~~~~~~~~

使用 GitHub 的 API 时，组织资源要求指定 `login` 参数。如果你之前使用过 GitHub，应该知道这个参数是组织 URL 地址中的一部分。

{title="Code Playground",lang="json"}
~~~~~~~~
https://github.com/the-road-to-learn-react
~~~~~~~~

通过提供一个 `login` 参数，你可以获取到该参数相关的组织数据。在这个例子中，我们指定了两个字段去获取组织中的 `name` 和 `url`。这次请求应该返回类似如下内容：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "data": {
    "organization": {
      "name": "The Road to learn React",
      "url": "https://github.com/the-road-to-learn-react"
    }
  }
}
~~~~~~~~

在上面的查询中，你传入了一个参数给某个字段。同理，你也可以使用 GraphQL 添加参数到不同的字段。由于 GraphQL 的参数支持在在字段级别作出约束，这为结构化查询提供了很大的灵活性。另外，参数可以是不同的类型。对于上面组织的例子，使用了一个类型为 `String` 的参数，但你也可以使用一组固定的选项作为枚举、整数或布尔值。

如果你想要一个字段不同参数返回的数据，则需要在 GraphQL 中使用 **别名**。下面的查询语句不能被正常处理，因为 GraphQL 不知道如何在结果中解析两个组织对象： 

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  organization(login: "the-road-to-learn-react") {
    name
    url
  }
  organization(login: "facebook") {
    name
    url
  }
}
~~~~~~~~

你会看到一个错误，例如 `Field 'organization' has an argument conflict`。使用别名，可以将结果解析为两个字段：


{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
# leanpub-start-insert
  book: organization(login: "the-road-to-learn-react") {
# leanpub-end-insert
    name
    url
  }
# leanpub-start-insert
  company: organization(login: "facebook") {
# leanpub-end-insert
    name
    url
  }
}
~~~~~~~~

结果应类似如下内容：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "data": {
    "book": {
      "name": "The Road to learn React",
      "url": "https://github.com/the-road-to-learn-react"
    },
    "company": {
      "name": "Facebook",
      "url": "https://github.com/facebook"
    }
  }
}
~~~~~~~~

接下来，假设你要为两个组织请求多个字段。重新填入每个组织的所有字段会使查询重复且冗长，因此我们可以使用**片段**来提取查询的可重用部分。当查询深度嵌套并使用大量共享字段时，片段尤其有用。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  book: organization(login: "the-road-to-learn-react") {
# leanpub-start-insert
    ...sharedOrganizationFields
# leanpub-end-insert
  }
  company: organization(login: "facebook") {
# leanpub-start-insert
    ...sharedOrganizationFields
# leanpub-end-insert
  }
}

# leanpub-start-insert
fragment sharedOrganizationFields on Organization {
  name
  url
}
# leanpub-end-insert
~~~~~~~~

如你所见，你必须指定该片段用在哪种**类型**的对象上。在这个例子中，应该是 `Organization` 类型，它是由 GitHub 的 GraphQL API 定义的自定义类型。以上为如何使用片段进行提取和重用你的部分查询的示例。关于这点，如果你在 GraphiQL 应用程序的右侧打开 “Docs” 面板。你可以看到 GraphQL 定义的 **schema**。schema 定义了 GraphiQL 如何使用某个 GraphQL API，在这个例子中，它是 Github 提供的 GraphQL API。它定义了 GraphQL **graph**，可以使用查询和变更对 GraphQL API 进行调用。由于它是一个图形结构，因此对象和字段可以深深地嵌套在其中，随着学习的深入，我们会对此有更深入的体会。

由于我们目前在探索查询相关的内容，所以可以在 “Docs” 侧边栏中选择 “Query” 标签来了解更多信息。对比 graph 中的对象和字段，浏览它们的可选参数。点击它们，你可以在文档中查看这些对象中的可访问字段。有些字段是常见的 GraphQL 类型，如 `String`，`Int` 和 `Boolean`，而其他一些类型是**自定义类型**，就像我们使用的 `Organization` 类型。此外，通过感叹号标记你可以查看在对象上的字段的参数是否为必填。例如，带有 `String!` 参数的字段要求你必须传入 `String` 参数，而带有 `String` 参数的字段则是可选的。

在之前的学习中，你在构建查询时，传入了用于向字段标识某个组织的参数，但是是通过**内联参数**的方式。如果将查询作为函数一样看待，为它提供动态参数就很有意义了。这就是 GraphQL 中**变量**，它允许使用参数动态地构建查询。以下示例展示了组织的 `login` 参数是如何使用变量的：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
# leanpub-start-insert
query ($organization: String!) {
  organization(login: $organization) {
# leanpub-end-insert
    name
    url
  }
}
~~~~~~~~

使用 `$` 符号将 `organization` 参数定义为变量。此外，参数的类型被定义为 `String`。由于该参数是完成查询所必需的，因此 `String` 类型有一个感叹号。

在 “Query Variables” 面板中，需要像下面这样定义变量内容，用于提供 `organization` 变量作为查询的参数：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "organization": "the-road-to-learn-react"
}
~~~~~~~~

一般来说，变量被用来创建动态查询。遵循 GraphQL 中的最佳实践，我们不需要手动插入字符串来构建动态查询。实际开发过程中，当我们使用变量构建查询时，可以让参数在请求被发送时动态绑定。稍后你将在 React 应用程序中看到这两种实现。

旁注：你还可以在 GraphQL 中定义**默认变量**。要求是非必需参数，否则会出现关于 **nullable variable** 或 **non-null variable** 的错误。要了解默认变量，我们将通过省略感叹号来使 “organization” 参数设为可选。之后，它可以作为默认变量传递。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
# leanpub-start-insert
query ($organization: String = "the-road-to-learn-react") {
  organization(login: $organization) {
# leanpub-end-insert
    name
    url
  }
}
~~~~~~~~

尝试使用两组变量执行上一个查询：一次使用不同于默认变量的 `organization` 变量，一次不定义 `organization` 变量。

现在，让我们回过头来检查 GraphQL 查询的结构。在引入变量之后，第一次在查询结构中遇到了 `query` 语句。之前，实际上是省略 `query` 语句的**查询的简写版本**，但是现在使用变量后， `query` 语句就是必须的了。尝试不带变量的以下查询，但使用 `query` 语句，来验证查询的非简写版本是否有效。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
# leanpub-start-insert
query {
# leanpub-end-insert
  organization(login: "the-road-to-learn-react") {
    name
    url
  }
}
~~~~~~~~

虽然使用了非简写版本，但仍然返回了与之前相同的数据，和我们设想的结果一样。查询语句在 GraphQL 语言中也称为**操作类型**。例如，它也可以是 `mutation` 语句。除了操作类型，你还可以定义**操作名称**。


{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
# leanpub-start-insert
query OrganizationForLearningReact {
# leanpub-start-insert
  organization(login: "the-road-to-learn-react") {
    name
    url
  }
}
~~~~~~~~

将代码中的匿名和具名函数进行对比。**具名查询**更为清晰，表明你希望以声明方式实现查询，在调试多个查询时非常有帮助，推荐在真实的项目中这样操作。完成查询的最终版本（省略了变量表），应该像下面一样：


{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
query OrganizationForLearningReact($organization: String!) {
  organization(login: $organization) {
    name
    url
  }
}
~~~~~~~~

到目前为止，你仅获取了包含一些简单字段的对象。 GraphQL 模式能实现一个完整的图形结构，所以让我们看看如何使用查询实现**嵌套对象**的获取。写法和之前基本一样：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
query OrganizationForLearningReact(
  $organization: String!,
# leanpub-start-insert
  $repository: String!
# leanpub-end-insert
) {
  organization(login: $organization) {
    name
    url
# leanpub-start-insert
    repository(name: $repository) {
      name
    }
# leanpub-end-insert
  }
}
~~~~~~~~

使用第二个变量来获取组织中的特定代码库：


{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "organization": "the-road-to-learn-react",
# leanpub-start-insert
  "repository": "the-road-to-learn-react-chinese"
# leanpub-end-insert
}
~~~~~~~~

包含 React 教程的一个组织含有翻译后的版本，其中一个仓库是简体中文版本。 GraphQL 中的字段可以是嵌套对象，并且你已从 graph 中查询了两个关联对象。通过 graph 可以构建出深层嵌套的查询。在之前探索 GraphiQL 中的 “Docs” 侧边栏时，你可能已经看到可以在对象跳转到另外一个对象的功能。

**指令**可用于以更强大的方式查询 GraphQL API 中的数据，并且它们可以应用于字段和对象。下面，我们来尝试使用两种指令：**include 指令**，用来包含布尔类型设置为 true 的字段; 和 **skip 指令**，与之相反排除那些为 true 的字段。使用这些指令，你可以将条件结构应用于查询。以下查询展示了 include 指令，你也可以使用 skip 指令替换它实现相反的效果：


{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
query OrganizationForLearningReact(
  $organization: String!,
  $repository: String!,
# leanpub-start-insert
  $withFork: Boolean!
# leanpub-end-insert
) {
  organization(login: $organization) {
    name
    url
    repository(name: $repository) {
      name
# leanpub-start-insert
      forkCount @include(if: $withFork)
# leanpub-end-insert
    }
  }
}
~~~~~~~~

现在你可以根据提供的变量决定是否包含 `forkCount` 字段的信息。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "organization": "the-road-to-learn-react",
  "repository": "the-road-to-learn-react-chinese",
# leanpub-start-insert
  "withFork": true
# leanpub-end-insert
}
~~~~~~~~


GraphQL 中的查询为你提供了从 GraphQL API 读取数据时的全部功能。不过最后一部分可能让人感到困惑，如果你依然没有掌握，下面提供了一些练习来帮助你。

### 练习：

* 延伸阅读：[GraphQL 中的查询](http://graphql.org/learn/queries).
* 使用 GraphiQL 中的 “Docs” 侧边栏探索 GitHub 的查询操作
* 使用以下功能创建一些从 GitHub 的 GraphQL API 请求数据的查询：
  * 对象和字段
  * 嵌套对象
  * 片段
  * 参数和变量
  * 具名操作
  * 指令

## GraphQL 基础：变更

这部分将会介绍 GraphQL 变更操作。它作为 GraphQL 查询的补充，用于改写数据而不是读取。变更操作和查询操作拥有着同样的规则：拥有字段和对象、参数和变量、片段和操作名称、指令和返回结果中的嵌套对象。通过变更操作你可以指定在"更新"发生后所期望的返回数据的字段和对象。在你开始你的第一次变更操作之前，请注意你在使用真实的 GitHub 数据，也就是说如果你在尝试变更操作的时候关注了 GitHub 上的一个人，你就真的关注了这个人。幸运的是这种行为在 GitHub 上是受到鼓励的。

接下来你将会 star 一个 GitHub 上的代码库，而和你之前使用一个查询来请求一样，你将使用[来自 GitHub API](https://developer.github.com/v4/mutation/addstar) 的一个变更请求。你可以在 "Docs" 侧边栏中找到 `addStar` 这种变更操作。这是一个存放为开发者讲解 React 基础课程的代码库，所以 star 这个代码库能够证明它有用。

你可以访问[这个代码库](https://github.com/the-road-to-learn-react/the-road-to-learn-react)来查看你是否已经成功 star。我们想要一个尚未 star 过的代码库，这样我们可以通过一个变更操作来 star 它。在你 star 一个代码库前，你需要知道它的唯一标识。这个唯一标识你可以通过下面的查询获取：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
query {
  organization(login: "the-road-to-learn-react") {
    name
    url
# leanpub-start-insert
    repository(name: "the-road-to-learn-react") {
# leanpub-end-insert
      id
      name
    }
  }
}
~~~~~~~~

在 GraphiQL 的该查询结果中，你应该能看到代码库的唯一标识：

{title="Code Playground",lang="json"}
~~~~~~~~
MDEwOlJlcG9zaXRvcnk2MzM1MjkwNw==
~~~~~~~~

在使用唯一标示作为变量之前，你可以在 GraphiQL 中使用以下结构的变更操作：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
mutation AddStar($repositoryId: ID!) {
  addStar(input: { starrableId: $repositoryId }) {
    starrable {
      id
      viewerHasStarred
    }
  }
}
~~~~~~~~

这个变更操作的名称是由 GitHub API 起的：`addStar`。你需要传递 `starrableId` 作为 `input` 来指定代码库；否则 GitHub 服务器无从得知这次变更操作是要 star 哪个代码库。另外，这个变更是一个具名变更为： `AddStar` 。你也可以给它任意名称。最后但也同样重要的是，你可以再次通过对象和字段来定义这个变更的返回值，这和查询是相同的。最终，在变量区中提供你在上一次查询得到的变量将被用于这一次变更操作：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "repositoryId": "MDEwOlJlcG9zaXRvcnk2MzM1MjkwNw=="
}
~~~~~~~~

一旦你执行了这个变更操作，结果应该类似如下内容。因为你使用了 `id` 和 `viewerHasStarred` 字段指定你的变更的返回值，所以你应该能在结果中找到它们。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "data": {
    "addStar": {
      "starrable": {
        "id": "MDEwOlJlcG9zaXRvcnk2MzM1MjkwNw==",
        "viewerHasStarred": true
      }
    }
  }
}
~~~~~~~~

这个代码库现在已经被 star 了。在返回的结果中能够看到，但你也可以通过查看 [GitHub 上的代码库](https://github.com/the-road-to-learn-react/the-road-to-learn-react)来确认。恭喜，你完成了你的第一个变更操作。

### 练习：

* 延伸阅读：[GraphQL 中的变更](http://graphql.org/learn/queries/#mutations)
* 通过 GraphiQL 上的 "Docs" 侧边栏探索 GitHub 的更多变更操作
* 在 GraphiQL 上的 "Docs" 侧边栏中找到 GitHub 的 `addStar` 变更
  * 检查它所有可以返回的字段
* 对该代码库或另一个代码库构造一些其他变更操作，比如：
	* 取消 Star 代码库
	* Watch 代码库
* 在 GraphiQL 面板上创建两个并列的命名变更，然后执行它们
* 延伸阅读：[模式和类型](http://graphql.org/learn/schema)
	* 你可以只是大概了解一下，不要担心你有不理解的地方

## GraphQL 分页

这里我们回到了在第一节提到的**分页**的概念。试想你在你的 GitHub 组织下有一个代码库列表，但你只想获得它们中的一部分来展示在你的 UI 上。如果是从一个大型组织下获取一个代码库列表的话，那将花费大量的时间。在 GraphQL 中，你可以通过提供参数到一个**列举字段**来请求分页数据，比如一个表明你希望从列表中获得多少项的参数。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
query OrganizationForLearningReact {
  organization(login: "the-road-to-learn-react") {
    name
    url
# leanpub-start-insert
    repositories(first: 2) {
      edges {
        node {
          name
        }
      }
    }
# leanpub-end-insert
  }
}
~~~~~~~~

这里一个 `first` 参数被传给了 `repositories` 的列举字段来指定希望从列表中获得多少项作为结果。这个查询没有被要求按照 `edges` 和 `node` 的结构编写，不过这也是仅有的几种 GraphQL 定义分页数据结构和列表的方案之一。实际上它是借鉴了 Facebook GraphQL 客户端 Relay 的接口描述方案。GitHub 按照它的方式并采纳到了自己的 GraphQL 分页 API 中。之后你将会在练习中了解到其他用 GraphQL 实现分页的方法。

在执行这个查询之后，你应该能在 repositories 字段的列表中看到两项。然而，我们仍然需要寻找如何拿到代码库列表中之后两项的办法。这个查询的第一次结果是分页列表的第一**页**，而第二次查询的结果应该是第二页。接下来，你会看到分页数据的查询结构是如何允许我们获得元信息来执行连续的查询。比如，每个 edge 有它自己的 游标字段来指明它在列表中的位置。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
query OrganizationForLearningReact {
  organization(login: "the-road-to-learn-react") {
    name
    url
    repositories(first: 2) {
      edges {
        node {
          name
        }
# leanpub-start-insert
        cursor
# leanpub-end-insert
      }
    }
  }
}
~~~~~~~~

结果类似如下内容：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "data": {
    "organization": {
      "name": "The Road to learn React",
      "url": "https://github.com/the-road-to-learn-react",
      "repositories": {
        "edges": [
          {
            "node": {
              "name": "the-road-to-learn-react"
            },
            "cursor": "Y3Vyc29yOnYyOpHOA8awSw=="
          },
          {
            "node": {
              "name": "hackernews-client"
            },
            "cursor": "Y3Vyc29yOnYyOpHOBGhimw=="
          }
        ]
      }
    }
  }
}
~~~~~~~~

现在你可以使用列表中第一个代码库的游标来执行第二个查询。通过给 `repositories` 的列举字段使用 `after` 参数，你可以指定获得下一页分页数据的起点。那么执行下面的查询后的结果会是什么样呢？

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
query OrganizationForLearningReact {
  organization(login: "the-road-to-learn-react") {
    name
    url
# leanpub-start-insert
    repositories(first: 2, after: "Y3Vyc29yOnYyOpHOA8awSw==") {
# leanpub-end-insert
      edges {
        node {
          name
        }
        cursor
      }
    }
  }
}
~~~~~~~~

在上一个结果中，只有第二项和一个新的第三项被取到了。第一项没有被取到因为你使用了它的游标作为 `after` 参数来取得它之后的所有元素。现在你可能已经想到如何来进行连续的分页列表查询了：

* 不带游标参数执行初始查询

* 执行接下来的查询的时候，使用上一次查询结果中**最后**一项的游标作为该次查询的游标

为了保持查询是动态的，我们可以把它的参数都抽成变量。然后你可以通过提供变量来使用拥有动态的 `cursor` 参数的查询。为了获取第一页，`after` 参数可以是 `undefined` 。最后，这就是所有关于你如何使用一个被称为分页的功能来从一个巨大的列表中获取很多页列表。你需要一个必需的参数来指明应该取得多少项以及一个可选参数，在这个例子中是 `after` 参数，来指明列表的起始点。

也有一些其他有用的方式通过使用元信息来为你的列表分页。当只使用最后一个代码库的 `cursor` 的时候，给每一个代码库都取得 `cursor` 字段可能显得很冗长，所以你可以给单独的 edge 去掉 `cursor` 字段，但给 `pageInfo` 对象加上它的 `endCursor` 和 `hasNextPage` 字段。你也可以请求列表的 `totalCount` 。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
query OrganizationForLearningReact {
  organization(login: "the-road-to-learn-react") {
    name
    url
    repositories(first: 2, after: "Y3Vyc29yOnYyOpHOA8awSw==") {
# leanpub-start-insert
      totalCount
# leanpub-end-insert
      edges {
        node {
          name
        }
      }
# leanpub-start-insert
      pageInfo {
        endCursor
        hasNextPage
      }
# leanpub-end-insert
    }
  }
}
~~~~~~~~

这个 `totalCount` 字段表明了列表中元素的数量，而 `pageInfo` 字段给你提供了两个信息：

* 和我们使用 `cursor` 字段一样，**`endCursor`** 也可以用来获取连续的列表，除了这一次我们只需要一个元字段来实现。但列表最后一项的 cursor 是足够用来请求列表中下一页的。

* **`hasNextPage`** 告诉了你是否还能通过 GraphQL API 获得下一页。有时候你已经从你的服务端获得了最后一页。对于那些在滚动列表加载更多页的时候使用无限滚动的应用，你可以在没有下一页的情况下，停止获取页面信息。

通过元信息可以完整地实现分页。使用 GraphQL API 来实现[分页列表](https://www.robinwieruch.de/react-paginated-list/)和[无限滚动](https://www.robinwieruch.de/react-infinite-scroll/)使得信息更加方便。注意这包含了 GitHub 的 GraphQL API; 一个不同的分页的 GraphQL API 的字段可能使用了不同的命名方式，除了元信息或者采用完全不同的机制。

### 练习：
* 把你的分页查询中的 `login` 和 `cursor` 抽为变量。
* 把 `first` 参数替换为 `last` 参数。
* 在 GraphiQL 的 "Docs" 侧边栏搜索 `repositories` 字段会看到："A list of repositories that the ... owns."
	* 探索其他可以传给这个列举字段的参数。
	* 使用 `orderBy` 参数来获取一个递增或者递减的列表。
* 延伸阅读：[GraphQL 中的分页](http://graphql.org/learn/pagination)。
  * 使用游标是 GitHub 唯一使用的解决方案。
  * 请确保同样理解其他的解决方案。

| |

通过 GraphiQL 或者 GitHub 提供的 GraphQL Explorer 来与 GitHub 的 GraphQL API 交互仅仅只是开始。你现在应该已经掌握了 GraphQL 的基本概念。但是仍然有很多更加有趣的概念可以探索。在接下来的章节中，你将会实现一个完整运行的使用 React 与 GitHub 的 API 进行交互的 GraphQL 客户端应用。
