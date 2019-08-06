# GraphQL 准备, 相关工具和 API

循序渐进通常是学习新东西最简单的方式，使用 JavaScript 来学习 GraphQL 是一件非常幸运的事，因为它同时教你应用程序的客户端和服务端。同时了解网络服务的前后端非常有用，但问题是你需要同时了解两个环境。因此这里采用循序渐进的方式会比较困难，所以我鼓励初学者从客户端应用程序开始，服务端则使用一个使用了 GraphQL 服务的第三方 API。

[GitHub](https://github.com)是首批采用 GraphQL 技术的大型科技公司之一。他们甚至[发布了](https://githubengineering.com/the-github-graphql-api)一个公共 GraphQL API [官方文档](https://developer.github.com/v4)，这在开发者中非常受欢迎，因为大多数人使用 GitHub 来托管项目，对其非常熟悉。

在本章中，我希望能涵盖开始使用 GitHub GraphQL API 所需的一切，并通过使用该 API 从客户端的角度学习如何在 JavaScript 中使用 GraphQL。 你能够了解到 GitHub 的术语，以及如何使用 GraphQL API 来消费帐户数据。 从客户端的角度来看，我们将使用该 GraphQL API 实现一些应用程序，因此在本节中投入时间是有意义的，可以避免犯一些基础性的错误。 之后，我们会转向服务端，实现我们自己的 GraphQL 服务器。

## 利用 GitHub 数据来提供 API

如果你还有没 GitHub 账号，对它的生态系统了解也不多，请关注[这个 GitHub 官方的学习实验室](https://lab.github.com/)。如果想深入了解 Git 及其基本命令，请查看[本指南](https://www.robinwieruch.de/git-essential-commands/)。如果你想将来在 GitHub 上与其他人分享项目，会发现这些非常有用。这也是向潜在客户或招聘公司展示开发作品的好方式。

在我们与 GitHub GraphQL API 交互的过程中，你将使用自己的账号信息来进行读写操作。在此之前，为了能够在使用 API 的时候读到这些信息，请提供附加信息以完善 GitHub 的个人信息。

### 练习：

* 如果没有 GitHub 账号的话，创建一个
* 完善你的 GitHub 账号的其他信息

### GitHub 代码库

你还可以在 GitHub 上创建代码库。用他们的官方词汇来说：*“代码库是 GitHub 的基本元素。它们最容易被想象为项目的文件夹。代码库包含项目的所有文件（包括文档），并存储每个文件的修订历史。代码库可以有多个协作者，可以是公有仓库也可以是私有仓库。”*[GitHub 的术语表](https://help.github.com/articles/github-glossary/)解释了关键术语—— repository，issue，clone，fork，push ——在接下来了解 GraphQL 的章节中会使用到这些术语。基本上来说代码库是一个可以和他人分享应用程序源码的地方。我鼓励你将一些项目放在 GitHub 代码库中，以便以后可以使用 GraphQL API 来访问它们。

如果你没有上传任何项目，你可以随时 “fork” 其他 GitHub 用户的代码库，并在其副本上进行操作。fork 大体来说是其他代码库的克隆，可以让你在不改变原始仓库的基础上进行添加修改。GitHub 上有很多开放的代码库，可以克隆到本地或者 fork 到你的代码库列表中，这样你就可以通过实验来了解它们的机制。例如，如果你访问[我的 GitHub 主页](https://github.com/rwieruch)，你可以看到我所有的代码库，但并非所有的代码库都是我的，因为有些是从别人那儿 fork 来的。如果你想使用它们来进行练习，或者通过 GitHub 的 GraphQL API 来访问，请随意 fork 这些仓库。

### 练习：

* 创建或者 fork 几个 GitHub 代码库，并验证它们是否以副本的形式存在于你的账户中。副本通过用户名标识，后面跟着代码库的名称，共同组成了该代码库的全称。例如，一个名为 *原作者名字/TestRepo* 的代码库，在你 fork 之后，会被命名为 *你的名字/TestRepo*。

### 分页数据

GitHub 的 GraphQL API 允许一次请求多个代码库，这对于分页来说非常有用。分页是为处理大量数据而发明的一种编程机制。例如，假设你的 GitHub 账号拥有超过一百个代码库，但是 UI 仅显示其中的 10 个。因为一次只需要一部分数据，所以每次请求都返回整个列表的数据是不切实际且低效的，而分页可以解决这样的问题。

通过使用 GitHub 的 GraphQL API 进行分页，你可以按需调整，所以请务必按照自己的需求（例如个人或组织账号中的代码库数量）来调整数字（例如每页数量，偏移）。你需要拥有足够多的代码库才能在实际中看到分页功能，如果每页显示 10 个的话，我推荐超过 20 个代码库，如果每页显示两个的话，建议 5 个以上。

### Issue 和 Pull Request

一旦你深入了解了 GitHub 的 GraphQL API 并开始请求一些嵌套关系的数据（例如代码库的 issue，pull request 等），请确保代码库中存在一些 issue 和 pull request，这样在实现获取代码库 issue 的功能时才能够看到结果。请求一个组织的代码库可能会更好，因为其一般都有较多的 issue 和 pull request。

### 练习：

* 在[GitHub 的词汇表](https://help.github.com/articles/github-glossary/)中查阅更多信息。 思考以下问题：
  * 什么是 GitHub 的组织账号和个人账号？
  * 什么是 GitHub 的组织用户和个人用户？
  * 什么是代码库，issue 和 pull request？
  * 什么是 GitHub 代码库的 star 和 关注者（watcher）？
* 创建或者 fork 足够多的代码库，来使用分页功能。
* 在你的 GitHub 代码库中创建 pull request 和 issue。

## 使用 GitHub 的 access token 来读写数据

为了使用 GitHub GraphQL API，你需要在他们的网站上生成一个 access token。Access token 授权用户与数据间的交互，使其能够对其所拥有的数据进行读写操作。[按照分步说明](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line)来获得个人 access token，并检查所需的权限范围，因为之后会利用它来实现一个多功能的 GitHub 客户端。

![](images/github-personal-access-token_1024.jpg)

Access token 随后会用于与 GitHub GraphQL API 进行交互。请注意不要与任何第三方共享这些 access token。

## 与 GitHub GraphQL API 交互

有两种常见的方法可以与 GitHub GraphQL API 进行交互，而且无需为其编写任何源代码。第一种是使用[GitHub GraphQL Explorer](https://developer.github.com/v4/explorer/)。只需要使用 GitHub 账号登录就可以使用 GraphQL API 进行数据查询或变更操作，这是一个简化初次体验的好方式。第二种是使用一个应用程序形式的通用客户端。GraphiQL 是一个客户端，可以让 GraphQL 请求功能集成到你的应用，或者作为一个独立的应用程序。前者可以通过[在应用程序中直接设置 GraphiQL](https://github.com/skevy/graphiql-app)来实现；后者可以通过[将 GraphiQL 作为独立的应用](https://github.com/skevy/graphiql-app)来实现，更容易使用。它是一个关于 GraphiQL 的轻量级 shell，可以手动或通过命令行下载和安装。

因为需要注册才能使用，所以 GitHub GraphQL Explorer 知道你的凭证，但 GraphiQL 也需要知道你创建的 access token。你可以在头部（header）配置中为每个请求的 HTTP 标头中添加 access token。

![](images/graphiql-headers_1024.jpg)

在下一步中，我们在 GraphiQL 配置中添加一个键值对。为了与 GitHub GraphQL API 通信，在头部加上 “Authorization”，值为 “bearer [你的 access token]”。为 GraphiQL 应用程序保存此设定。 这样，你就准备好使用 GraphiQL 应用程序向 GitHub GraphQL API 发送请求了。

![](images/graphiql-authorization_1024.jpg)

如果你使用自己的 GraphiQL 应用程序，则需要为 GitHub 的 GraphQL API 提供 GraphQL 端点：`https://api.github.com/graphql`。 对于 GitHub GraphQL API，使用[POST HTTP 方法](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods)进行查询和变更，并将数据作为负载进行传输。

本节提供了两种与 GitHub GraphQL API交互的方法。GitHub 的 GraphQL Explorer 只能用于 GitHub API，集成到应用程序或独立的 GraphiQL 可用于任何的 GraphQL API。不同点在于后者需要一些额外的配置。GitHub GraphQL Explorer 实际上只是为使用 GitHub GraphQL API 而定制的独立 GraphiQL 应用程序。

| |

为了使用 GitHub GraphQL API 来了解 GraphQL，在设置好 GitHub 之后，你已经可以开始实现你的第一个可交互的 GraphQL 客户端了。继续阅读，利用你刚设置好的工具和 React 来创建你的第一个 GraphQL 客户端应用。
