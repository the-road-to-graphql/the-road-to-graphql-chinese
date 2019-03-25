# GraphQL 准备, 相关工具和 API

循序渐进通常是学习新东西最简单的方式，使用 JavaScript 来学习 GraphQL 是一件非常幸运的事，因为它同时教你应用程序的客户端和服务端。同时了解网络服务的前后端非常有用，但问题是你需要同时了解两个环境。这样使用循序渐进的方式会比较困难，所以我鼓励初学者从客户端应用程序开始，服务端则使用一个使用了 GraphQL 服务第三方的 API。

[GitHub](https://github.com)是首批采用 GraphQL 技术的大型科技公司之一。他们甚至[发布了](https://githubengineering.com/the-github-graphql-api)一个公共 GraphQL API [官方文档](https://developer.github.com/v4)，这在开发者中非常受欢迎，因为大多数人都非常熟悉 GitHub 并将其用于自己的项目。

在本章中，我希望能涵盖开始使用 GitHub 的 GraphQL API 所需的一切，并通过使用该 API 从客户端的角度学习如何在 JavaScript 中使用 GraphQL。 你能够了解到 GitHub 的术语，以及如何使用 GraphQL API 来消费帐户数据。 从客户端的角度来看，我们将使用该 GraphQL API 实现一些应用程序，因此在本节中投入时间是有意义的，可以避免犯一些基础性的错误。 之后，我们会转向服务器，自己实现我们的 GraphQL 服务器。

## 利用 Github 数据来提供 API

如果你还有没 Github 账号，对它的生态系统了解也不多，请关注[这个 GitHub 官方的学习实验室](https://lab.github.com/)。如果想深入了解 Git 及其基本命令，请查看[本指南](https://www.robinwieruch.de/git-essential-commands/)。如果你想将来在 GitHub 上与其他人分享项目，会发现这些非常有用。这也是向潜在客户或招聘公司展示开发作品的好方式。

在我们与 GitHub GraphQL API 交互的过程中，你将使用自己的账号信息来进行读写操作。在此之前，为了能够在使用 API 的时候读到这些信息，请提供附加信息以完善 GitHub 的个人信息。

### 练习题：

* 创建一个 GitHub 账号，如果你没有的话
* 提供 GitHub 账号的其他信息

### GitHub 代码仓库

你还可以在 GitHub 上创建代码仓库。用他们的官方词汇来说：*“代码仓库是 GitHub 的基本元素。它们最容易被想象为项目的文件夹。代码仓库包含项目的所有文件（包括文档），并存储每个文件的修订历史。代码仓库可以有多个协作者，可以是公有仓库也可以是私有仓库。”*[GitHub的术语表](https://help.github.com/articles/github-glossary/)解释了关键术语——repository，issue，clone，fork，push——在接下来了解 GraphQL 的章节中会使用到这些术语。基本上来说代码仓库是一个可以和他人分享应用程序源码的地方。我鼓励你将一些项目放在 GitHub 代码仓库中，以便以后可以使用 GraphQL API 来访问它们。

如果你没有上传任何项目，你可以随时从其他 GitHub 用户“fork”代码仓库，并在其副本上进行操作。fork 大体来说是其他代码仓库的克隆，可以让你在不改变原始仓库的基础上进行添加修改。GitHub 上有很多开放的代码仓库，可以克隆到本地或者 fork 到你的代码仓库列表中，这样你就可以通过实验来了解它们的机制。例如，如果你访问[我的 GitHub 主页](https://github.com/rwieruch)，你可以看到我所有的代码仓库，但并非所有的代码仓库都是我的，因为有些是从别人那儿 fork 来的。如果你想使用它们来进行练习，或者通过 GitHub 的 GraphQL API 来访问，请随意 fork 这些仓库。

### 练习：

* 创建或 fork 几个 GitHub 代码仓库，并验证它们是否存以副本的形式存在于你的账户中。副本通过用户名标识，后面跟着代码仓库的名称，共同组成了该代码仓库的全称。例如，一个名为*原作者名字/TestRepo*的代码仓库，在你 fork 之后，会被命名为 *你的名字/TestRepo*。

### 分页数据

GitHub 的 GraphQL API 允许一次请求多个代码仓库，这对于分页来说非常有用。分页是为处理大量数据而发明的一种编程机制。例如，假设你的 GitHub 账号拥有超过一百个代码仓库，但是 UI 仅显示其中的 10 个。每个网络请求都传输整个列表的数据是不切实际和低效的，因为一次只需要其中的一个子集，分页可以解决这样的问题。

通过使用 GitHub 的 GraphQL API 进行分页，你可以按需调整，所以请务必按照自己的需求（例如个人或组织账号中的代码仓库数量）来调整数字（例如每页数量，偏移）。你需要拥有足够多的代码仓库才能在实际中看到分页功能，如果每页显示 10 个的话，我推荐超过 20 个代码仓库，如果每页显示两个的话，推荐 5 个以上。

### Issue 和 Pull Request

一旦你深入了解了 GitHub 的 GraphQL API 并开始请求一些嵌套关系的数据（例如代码仓库的 issue，pull request等），请确保代码仓库中存在一些 issue 和 pull request ，这样在实现获取代码仓库的 issue 时能够看到结果。请求一个组织的代码代码仓库可能会更好，因为那里有很多的 issue 和 pull request。

### 练习：

* 在[GitHub的词汇表](https://help.github.com/articles/github-glossary/)中阅读关于不同术语的更多信息。 思考以下问题：
  * 什么是 GitHub 的组织和用户？
  * 什么是代码仓库，issue 和 pull request？
  * 什么是 GitHub 代码仓库的 star 和 关注者（watcher）？
* 创建或者 fork 足够多的代码仓库，来使用分页功能。
* 在一些 GitHub 代码仓库中创建 pull request 和 issue。

## 使用 GitHub 的个人访问令牌（access token）来读写数据

为了使用 GitHub GraphQL API，你需要在他们的网站上生成一个个人访问令牌。访问令牌授权用户与数据间的交互，使其能够对其所拥有的数据进行读写操作。[按照分步说明](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line)来获得个人访问令牌，并检查所需的权限范围，因为之后会利用它来实现一个多功能的  GitHub 客户端。

![](images/github-personal-access-token_1024.jpg)

个人访问令牌随后会用于与 GitHub GraphQL API 进行交互。请注意不要与任何第三方共享这些令牌。

## Interacting with GitHub's GraphQL API

There are two common ways to interact with the GitHub GraphQL API without writing any source code for it. First, you can use [GitHub's GraphQL Explorer](https://developer.github.com/v4/explorer/). You only need to sign up with your GitHub account to perform a query or mutation to their GraphQL API, and its a good way to simplify your first experience. Second, you can use a generic client in the form of an application. GraphiQL is a client that makes GraphQL requests as integration or as a standalone application. The former can be accomplished [by setting up GraphiQL directly in your application](https://github.com/skevy/graphiql-app); the latter may be more convenient for you by [using GraphiQL as standalone application](https://github.com/skevy/graphiql-app). It's a lightweight shell around GraphiQL that can be downloaded and installed manually or by the command line.

GitHub's GraphQL Explorer knows about your credentials, since you need to sign up using it, but the GraphiQL application needs to know about the personal access token you created. You can add it in your HTTP header for every request in the headers configuration.

![](images/graphiql-headers_1024.jpg)

In the next step, we add a new header with a name and value to your GraphiQL configuration. To communicate with GitHub's GraphQL API, fill in the header name with "Authorization" and the header value with "bearer [your personal access token]". Save this new header for your GraphiQL application. Finally, you are ready to make requests to GitHub's GraphQL API with your GraphiQL application.

![](images/graphiql-authorization_1024.jpg)

If you use your own GraphiQL application, you'll need to provide the GraphQL endpoint for GitHub's GraphQL API: `https://api.github.com/graphql`. For GitHub's GraphQL API, use the [POST HTTP method](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods) for queries and mutations, and to transfer data as a payload to your GraphQL endpoint.

This section provided you with two ways to interact with GitHub's GraphQL API. Where GitHub's GraphQL Explorer can only be used for GitHub's API, GraphiQL integrated into an application or standalone can be used for any GraphQL API. The difference is that it requires a bit more setup. The GitHub GraphQL Explorer is really nothing more than a hosted standalone GraphiQL application tailored to use GitHub's GraphQL API.

| |

After you've set up GitHub to use their GraphQL API to learn about GraphQL, you should be ready to implement your first GraphQL client interactions. Follow along and create your first GraphQL client-side application with the tools you have just set up but also with React.
