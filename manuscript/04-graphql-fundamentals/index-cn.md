# GraphQL 基础

在我们开始构建一个包含客户端和服务器端完整的 GraphQL 应用之前，让我们通过前面章节安装的一些工具来体验一下 GraphQL 的工作方式。你可以选用 GraphiQL 或者 GitHub 提供的 Explorer。 接下来我们会学习 GraphQL 基础操作，执行你的第一个 GraphQL query、mutations，或者探索 GitHub 的 GraphQL API 中的一些特性，例如分页等。

## GraphQL 基本操作: Query 

在本节中，你可以使用 queries 和 mutations 同 GitHub API 交互，可以通过 GraphiQL 应用或者 GitHub 的 GraphQL Explorer 发送查询请求到 GitHub API。 这两种工具都需要使用个人申请的 access token 授权。在 GraphiQL application 的左侧，可以输入 GraphQL queries 和 mutations。尝试输入下面的代码获取你的个人信息数据。

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  viewer {
    name
    url
  }
}
~~~~~~~~

`viewer` 对象可以被用来获取当前授权的用户信息。通过你的个人 access token 获得授权后，应该能看到相关的用户信息。`viewer` 是一个 GraphQL 中**对象**的概念。 对象承载某个实体的数据。 这些数据可以通过 GraphQL 中的**字段**访问。字段被用于获取指定对象中的属性。举个例子来说，`viewer` 对象暴露了一组字段，在这个例子中，只有`name` 和 `url`在 query 中用到了。在大多数基本情况下，一个 query 只包含对象和字段，当然对象也是一种字段。


当你在 GraphiQL 中执行完上面的 query，你可以看到类似如下的返回内容，上面的 name 和 URL 被替换成真实的内容。

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

恭喜你，你成功地执行了第一个 query 并且获取到了你自己的用户信息中的相关字段。现在，让我们看看怎么去获取其他的资源，例如 GitHub 中开放出来的 organization 信息。为了获取特定的 GitHub organization 信息，你可以给需要的字段传入一个**参数**：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  organization(login: "the-road-to-learn-react") {
    name
    url
  }
}
~~~~~~~~

使用 GitHub 的API 时，organization 需要指定`login` 参数。如果你之前使用过 GitHub，应该知道这个参数是 organization URL 地址中的一部分。

{title="Code Playground",lang="json"}
~~~~~~~~
https://github.com/the-road-to-learn-react
~~~~~~~~

通过提供一个 `login`  参数指定具体的 organization，你可以获取到它相关的数据。在这个例子中，通过指定了两个字段去获取 organization 中的 `name` 和 `url`。这次请求应该返回类似如下内容：

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

在上面的 query 中，你传入了一个参数给某个字段。同理，你也可以使用 GraphQL 添加参数到不同的字段。它可以为结构化查询提供了很大的灵活性，因为可以在字段级别为请求做出约束。另外，参数可以是不同类型的。对于上面 organization 的例子，你可以提供了一个类型为 `String` 的参数，但你也可以使用一组固定的选项作为枚举，整数或布尔值。

如果你想要两个同名字段返回的数据，则需要在 GraphQL 中使用 **别名**。下面的 query 不能被正常处理，因为 GraphQL 不知道如何在结果中解析两个 organization 对象：

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

你会看到一个错误，例如 `Field 'organization' has an argument conflict`。使用别名，可以将结果解析为两个段：


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

接下来，假设你要为两个 organizations 请求多个字段。重新填入每个组织的所有字段会使查询重复且冗长，因此我们可以使用 **片段** 来提取查询的可重用部分。当查询深度嵌套并使用大量共享字段时，片段尤其有用。

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

如你所见，你必须指定一个具体反应对象**类型**下的片段。在这个例子中，应该是 `Organization` 类型，它是由 GitHub的 GraphQL API定义的自定义类型。这是你使用片段和重用部分构建 query 需要注意的。关于这点，如果你在 GraphiQL 应用程序的右侧打开 “Docs” 面板。你可以看到 GraphQL 定义的 **模式**。模式展示了 GraphiQL 应用程序使用的 GraphQL API，在这个例子中，它是 Github 的 GraphQL API。它定义了 GraphQL **graph**，可以使用 query 和 mutation 对 GraphQL API 进行调用。由于它是一个图形结构，因此对象和字段可以深深地嵌套在其中，随着我们学习的深入，我们会在后面遇到它。

由于我们正在探索 query 而不是目前的 mutations，可以在 “Docs” 侧边栏中选择 “query” 标签。然后，对比 graph 中的对象和字段，浏览它们的可选参数。点击它们，你可以在文档中查看这些对象中的可访问字段。有些字段是常见的 GraphQL 类型，如 `String`，`Int`和`Boolean`，而其他一些类型是**自定义类型**，就像我们使用的 `Organization` 类型。此外，通过感叹号标记你可以查看在对象上的字段的参数是否为必填。例如，带有 `String！` 参数的字段要求你必须传入 `String` 参数，而带有 `String` 参数的字段则是可选的。

在之前的 query 中，你提供了用于向字段标识某个 organization 的参数; 但是是通过**内联参数**的方式传入 query 中。考虑像函数一样的 query，为它提供动态参数就重要了。这就是GraphQL 中**变量**的来源，因为它允许从查询中提取参数作为变量。以下示例展示了 organization 的`login`参数如何被提取到动态变量：

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

使用`$`符号将`organization`参数定义为变量。此外，参数的类型被定义为`String`。由于参数是完成查询所必需的，因此`String`类型有一个感叹号。

在 “Query Variables” 面板中，需要像下面这样定义变量内容，用于提供 `organization` 变量作为查询的参数：

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "organization": "the-road-to-learn-react"
}
~~~~~~~~

实质上，变量可用于创建动态查询。遵循 GraphQL 中的最佳实践，我们不需要手动插入字符串来构建动态查询。相反，我们提供了一个使用变量作为参数的查询，当查询作为请求发送到 GraphQL API 时可用。稍后你将在 React 应用程序中看到这两种实现。

旁注：你还可以在 GraphQL 中定义**默认变量**。要求是非必需参数，否则会出现关于**nullable variable**或**non-null variable**的错误。要了解默认变量，我们将通过省略感叹号来使“organization”参数设为可选。之后，它可以作为默认变量传递。

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

尝试使用两组变量执行上一个查询：一次使用不同于默认变量的`organization`变量，一次不定义`organization`变量。

现在，让我们回过头来检查 GraphQL 查询的结构。在引入变量之后，第一次在查询结构中遇到了 `query` 语句。之前，实际上是通过省略`query`语句的 **查询的简写版本**，但是现在使用变量后， `query` 语句就是必须的了。尝试不带变量的以下查询，但使用`query`语句，来验证查询的非简写版本是否有效。

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

虽然使用了非简写版本，但仍然返回了与之前相同的数据，和我们设想的结果一样。查询语句在 GraphQL 语言中也称为 **操作类型**。例如，它也可以是`mutation` 语句。除了操作类型，你还可以定义**操作名称**。


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

同它与代码中的匿名和命名函数进行对比。**具名查询** 更为清晰，表明你希望以声明方式实现查询，在调试多个查询时非常有帮助，推荐在真实的项目中这样操作。完成查询的最终版本，应该像下面一样：


{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
query OrganizationForLearningReact($organization: String!) {
  organization(login: $organization) {
    name
    url
  }
}
~~~~~~~~

到目前为止，你只请求了一个对象，一个有几个字段的 organization。 GraphQL 模式能实现一个完整图的结构，所以让我们看看如何使用查询实现**嵌套对象**的获取。写法和之前一样：

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

使用第二个变量来获取 organization 中的特定仓库：


{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "organization": "the-road-to-learn-react",
# leanpub-start-insert
  "repository": "the-road-to-learn-react-chinese"
# leanpub-end-insert
}
~~~~~~~~

包含 React 教程的一个 organization 含有翻译后的版本，其中一个仓库是简体中文版本。 GraphQL 中的字段可以是嵌套对象，并且你已从 graph 中查询了两个关联对象。通过 graph 可以构建出深层嵌套的查询。在探索 GraphiQL 中的 “Docs” 侧栏之前，你可能已经看到可以在对象跳转到另外一个对象的功能。

**指令** 可用于以更强大的方式查询GraphQL API中的数据，并且它们可以应用于字段和对象。下面，我们使用两种类型的指令：**include指令**，其中包括布尔类型设置为true时的字段;和**跳过指令**，从返回的数据中排除自身。使用这些指令，你可以将条件结构应用于查询。以下查询展示了include 指令，你也可以使用skip 指令替换它实现相反的效果：


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

现在你可以根据提供的变量决定是否包含`forkCount`字段的信息。

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

GraphQL中的 query 能为你提供了从GraphQL API读取数据时的全部功能。不过最后一部分可能让人感到困惑，如果你依然没有掌握，下面提供了一些练习。

### Exercises:

### 练习

* Read more about [the Query in GraphQL](http://graphql.org/learn/queries).
* Explore GitHub's query schema by using the "Docs" sidebar in GraphiQL.
* Create several queries to request data from GitHub's GraphQL API using the following features:
  * objects and fields
  * nested objects
  * fragments
  * arguments and variables
  * operation names
  * directives

* 阅读更多 [the Query in GraphQL](http://graphql.org/learn/queries).
* 使用 GraphiQL 中的 “Docs” 侧边栏探索 GitHub 的查询操作
* 使用以下功能创建一些从GitHub的GraphQL API请求数据的查询：
  * 对象和字段
  * 嵌套对象
  * 片段
  * 参数和变量
  * 具名操作
  * 指令

## GraphQL Operation: Mutation

This section introduces the GraphQL mutation. It complements the GraphQL query because it is used for writing data instead of reading it. The mutation shares the same principles as the query: it has fields and objects, arguments and variables, fragments and operation names, as well as directives and nested objects for the returned result. With mutations you can specify data as fields and objects that should be returned after it 'mutates' into something acceptable. Before you start making your first mutation, be aware that you are using live GitHub data, so if you follow a person on GitHub using your experimental mutation, you will follow this person for real. Fortunately this sort of behavior is encouraged on GitHub.

In this section, you will star a repository on GitHub, the same one you used a query to request before, using a mutation [from GitHub's API](https://developer.github.com/v4/mutation/addstar). You can find the `addStar` mutation in the "Docs" sidebar. The repository is a project for teaching developers about the fundamentals of React, so starring it should prove useful.

You can visit [the repository](https://github.com/the-road-to-learn-react/the-road-to-learn-react) to see if you've given a star to the repository already. We want an unstarred repository so we can star it using a mutation. Before you can star a repository, you need to know its identifier, which can be retrieved by a query:

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

In the results for the query in GraphiQL, you should see the identifier for the repository:

{title="Code Playground",lang="json"}
~~~~~~~~
MDEwOlJlcG9zaXRvcnk2MzM1MjkwNw==
~~~~~~~~

Before using the identifier as a variable, you can structure your mutation in GraphiQL the following way:

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

The mutation's name is given by GitHub's API: `addStar`. You are required to pass it the `starrableId` as `input` to identify the repository; otherwise, the GitHub server won't know which repository to star with the mutation. In addition, the mutation is a named mutation: `AddStar`. It's up to you to give it any name. Last but not least, you can define the return values of the mutation by using objects and fields again. It's identical to a query. Finally, the variables tab provides the variable for the mutation you retrieved with the last query:

{title="GitHub GraphQL Explorer",lang="json"}
~~~~~~~~
{
  "repositoryId": "MDEwOlJlcG9zaXRvcnk2MzM1MjkwNw=="
}
~~~~~~~~

Once you execute the mutation, the result should look like the following. Since you specified the return values of your mutation using the `id` and `viewerHasStarred` fields, you should see them in the result.

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

The repository is starred now. It's visible in the result, but you can verify it in the [repository on GitHub](https://github.com/the-road-to-learn-react/the-road-to-learn-react). Congratulations, you made your first mutation.

### Exercises:

* Read more about [the Mutation in GraphQL](http://graphql.org/learn/queries/#mutations)
* Explore GitHub's mutations by using the "Docs" sidebar in GraphiQL
* Find GitHub's `addStar` mutation in the "Docs" sidebar in GraphiQL
  * Check its possible fields for returning a response
* Create a few other mutations for this or another repository such as:
  * Unstar repository
  * Watch repository
* Create two named mutations side by side in the GraphiQL panel and execute them
* Read more about [the schema and types](http://graphql.org/learn/schema)
  * Make yourself a picture of it, but don't worry if you don't understand everything yet

## GraphQL Pagination

This is where we return to the concept of **pagination** mentioned in the first chapter. Imagine you have a list of repositories in your GitHub organization, but you only want to retrieve a few of them to display in your UI. It could take ages to fetch a list of repositories from a large organization. In GraphQL, you can request paginated data by providing arguments to a **list field**, such as an argument that says how many items you are expecting from the list.

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

A `first` argument is passed to the `repositories` list field that specifies how many items from the list are expected in the result. The query shape doesn't need to follow the `edges` and `node` structure, but it's one of a few solutions to define paginated data structures and lists with GraphQL. Actually, it follows the interface description of Facebook's GraphQL client called Relay. GitHub followed this approach and adopted it for their own GraphQL pagination API. Later, you will learn in the exercises about other strategies to implement pagination with GraphQL.

After executing the query, you should see two items from the list in the repositories field. We still need to figure out how to fetch the next two repositories in the list, however. The first result of the query is the first **page** of the paginated list, the second query result should be the second page. In the following, you will see how the query structure for paginated data allows us to retrieve meta information to execute successive queries. For instance, each edge comes with its own cursor field to identify its position in the list.

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

The result should be similar to the following:

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

Now, you can use the cursor of the first repository in the list to execute a second query. By using the `after` argument for the `repositories` list field, you can specify an entry point to retrieve your next page of paginated data. What would the result look like when executing the following query?

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

In the previous result, only the second item is retrieved, as well as a new third item. The first item isn't retrieved because you have used its cursor as `after` argument to retrieve all items after it. Now you can imagine how to make successive queries for paginated lists:

* Execute the initial query without a cursor argument
* Execute every following query with the cursor of the **last** item's cursor from the previous query result

To keep the query dynamic, we extract its arguments as variables. Afterward, you can use the query with a dynamic `cursor` argument by providing a variable for it. The `after` argument can be `undefined` to retrieve the first page. In conclusion, that would be everything you need to fetch pages of lists from one large list by using a feature called pagination. You need a mandatory argument specifying how many items should be retrieved and an optional argument, in this case the `after` argument, specifying the starting point for the list.

There are also a couple helpful ways to use meta information for your paginated list. Retrieving the `cursor` field for every repository may be verbose when using only the `cursor` of the last repository, so you can remove the `cursor` field for an individual edge, but add the `pageInfo` object with its `endCursor` and `hasNextPage` fields. You can also request the `totalCount` of the list.

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

The `totalCount` field discloses the total number of items in the list, while the `pageInfo` field gives you information about two things:

* **`endCursor`** can be used to retrieve the successive list, which we did with the `cursor` field, except this time we only need one meta field to perform it. The cursor of the last list item is sufficient to request the next page of list.

* **`hasNextPage`** gives you information about whether or not there is a next page to retrieve from the GraphQL API. Sometimes you've already fetched the last page from your server. For applications that use infinite scrolling to load more pages when scrolling lists, you can stop fetching pages when there are no more available.

This meta information completes the pagination implementation. Information is made accessible using the GraphQL API to implement [paginated lists](https://www.robinwieruch.de/react-paginated-list/) and [infinite scroll](https://www.robinwieruch.de/react-infinite-scroll/). Note, this covers GitHub's GraphQL API; a different GraphQL API for pagination might use different naming conventions for the fields, exclude meta information, or employ different mechanisms altogether.

### Exercises:

* Extract the `login` and the `cursor` from your pagination query as variables.
* Exchange the `first` argument with a `last` argument.
* Search for the `repositories` field in the GraphiQL "Docs" sidebar which says: "A list of repositories that the ... owns."
  * Explore the other arguments that can be passed to this list field.
  * Use the `orderBy` argument to retrieve an ascending or descending list.
* Read more about [pagination in GraphQL](http://graphql.org/learn/pagination).
  * The cursor approach is only one solution which is used by GitHub.
  * Make sure to understand the other solutions, too.

| |

Interacting with GitHub's GraphQL API via GraphiQL or GitHub's GraphQL Explorer is only the beginning. You should be familiar with the fundamental GraphQL concepts now. But there are a lot more exciting concepts to explore. In the next chapters, you will implement a fully working GraphQL client application with React that interacts with GitHub's API.
