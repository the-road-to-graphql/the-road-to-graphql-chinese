# GraphQL 准备, 相关工具和 API

循序渐进通常是学习新东西最简单的方式，使用 JavaScript 来学习 GraphQL 是一件非常幸运的事，因为它同时教你应用程序的客户端和服务端。同时了解网络服务的前后端非常有用，但问题是你需要同时了解两个环境。这样使用循序渐进的方式会比较困难，所以我鼓励初学者从客户端应用程序开始，服务端则使用一个使用了 GraphQL 服务第三方的 API。

[GitHub](https://github.com)是首批采用 GraphQL 技术的大型科技公司之一。他们甚至[发布了](https://githubengineering.com/the-github-graphql-api)一个公共 GraphQL API [官方文档](https://developer.github.com/v4)，这在开发者中非常受欢迎，因为大多数人都非常熟悉 GitHub 并将其用于自己的项目。

在本章中，我希望能涵盖开始使用 GitHub 的 GraphQL API 所需的一切，并通过使用该 API 从客户端的角度学习如何在 JavaScript 中使用 GraphQL。 你能够了解到 GitHub 的术语，以及如何使用 GraphQL API 来消费帐户数据。 从客户端的角度来看，我们将使用该 GraphQL API 实现一些应用程序，因此在本节中投入时间是有意义的，可以避免犯一些基础性的错误。 之后，我们会转向服务器，自己实现我们的 GraphQL 服务器。

## 利用 Github 数据来提供 API

如果你还有没 Github 账号，对它的生态系统了解也不多，请关注[这个 GitHub 官方的学习实验室](https://lab.github.com/)。如果想深入了解 Git 及其基本命令，请查看[本指南]（https://www.robinwieruch.de/git-essential-commands/）。如果你想将来在 GitHub 上与其他人分享项目，会发现这些非常有用。这也是向潜在客户或招聘公司展示开发作品的好方式。

在我们与 GitHub GraphQL API 交互的过程中，你将使用自己的账号信息来进行读写操作。在此之前，为了能够在使用 API 的时候读到这些信息，请提供附加信息以完善 GitHub 的个人信息。

### 练习题：

* 创建一个 GitHub 账号，如果你没有的话
* 提供 GitHub 账号的其他信息

### GitHub 代码仓库

你还可以在 GitHub 上创建代码仓库。用他们的官方词汇来说：*“代码仓库是 GitHub 的基本元素。它们最容易被想象为项目的文件夹。代码仓库包含项目的所有文件（包括文档），并存储每个文件的修订历史。代码仓库可以有多个协作者，可以是公有仓库也可以是私有仓库。”*[GitHub的术语表]（https://help.github.com/articles/github-glossary/）解释了关键术语——repository，issue，clone，fork，push——在接下来了解 GraphQL 的章节中会使用到这些术语。基本上来说代码仓库是一个可以和他人分享应用程序源码的地方。我鼓励你将一些项目放在 GitHub 代码仓库中，以便以后可以使用 GraphQL API 来访问它们。

如果你没有上传任何项目，你可以随时从其他 GitHub 用户“fork”代码仓库，并在其副本上进行操作。fork 大体来说是其他代码仓库的克隆，可以让你在不改变原始仓库的基础上进行添加修改。GitHub 上有很多开放的代码仓库，可以克隆到本地或者 fork 到你的代码仓库列表中，这样你就可以通过实验来了解它们的机制。例如，如果你访问[我的 GitHub 主页](https://github.com/rwieruch)，你可以看到我所有的代码仓库，但并非所有的代码仓库都是我的，因为有些是从别人那儿 fork 来的。如果你想使用它们来进行练习，或者通过 GitHub 的 GraphQL API 来访问，请随意 fork 这些仓库。

### Exercises:

* Create/Fork a couple of GitHub repositories, and verify that they show in your account as copies. Copies are indicated by the username that proceeds the repository name in all its titles; for example, a repo called *OriginalAuthor/TestRepo* would be renamed to *YourUserName/TestRepo* once you've forked it.

### Paginated Data

GitHub's GraphQL API allows you to request multiple repositories at once, which is useful for pagination. Pagination is a programming mechanic invented to work with large lists of items. For example, imagine you have more than a hundred repositories in your GitHub account, but your UI only shows ten of them. Transferring the whole list across the wire for each request is impractical and inefficient, because only a subset is needed at a time, which pagination allows.

Using pagination with GitHub's GraphQL API lets you adjust the numbers to your own needs, so make sure to adjust the numbers (e.g. limit, offset) to your personal requirements (e.g. available repositories of your GitHub account or available repositories of a GitHub organization). You at least want to have enough repositories in your collection to see the pagination feature in action, so I recommend more than twenty (20), assuming each page will display ten (10), or use five(5) repositories when displaying two (2.)

### Issues and Pull Requests

Once you dive deeper into GitHub's GraphQL API and you start to request nested relationships (e.g. issues of repositories, pull requests of repositories), make sure that the repositories have a few issues or pull requests. This is so you'll see something when we implement the feature to show all the issues in a repository. It might be better to request repositories from a GitHub organization where there will be plenty of issues and pull requests.

### Exercises:

* Read more about the different terms in [GitHub's glossary](https://help.github.com/articles/github-glossary/). Consider these questions:
  * What is a GitHub organization and GitHub user?
  * What are repositories, issues and pull requests?
  * What are GitHub repository stars and GitHub repository watchers?
* Create or fork enough repositories to use the pagination feature.
* Create pull requests and issues in a few of your GitHub repositories.

## Read/Write Data with GitHub's Personal Access Token

To use GitHub's GraphQL API, you need to generate a personal access token on their website. The access token authorizes users to interact with data, to read and write it under your username. [Follow their step by step instructions](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line) to obtain the personal access token, and be sure to check the necessary scopes (permissions) for it, as you will need them to implement a well-rounded GitHub client later.

![](images/github-personal-access-token_1024.jpg)

Later, the personal access token can be used to interact with GitHub's GraphQL API. Be careful not to share these authorizations with any third parties.

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
