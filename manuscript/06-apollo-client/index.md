# Apollo Client

Apollo 是一个由开发人员构建的作为 GraphQL 应用程序基础设施的整个生态系统。你可以在客户端将其用于 GraphQL 客户端应用程序，或者在服务器端将其用于 GraphQL 服务端应用程序。在编写本教程时，Apollo 提供了在 JavaScript 中最丰富和最流行的 GraphQL 生态系统。尽管还有其他用于 React 应用程序的库，如 [Relay](http://facebook.github.io/relay) 和 [Urql](https://github.com/FormidableLabs/urql)，但它们仅适用于 React 应用程序，并不像 Apollo Client 那么受欢迎。Apollo 是与框架无关的，这意味着你还可以将其与 React 之外的库一起使用，比如 Vue 和 Angular 等，因此你在本教程中学到的所有内容都可以迁移到其他库或者框架中。

## 从在命令行中使用 Apollo Boost 开始

这个应用程序首先通过 Apollo Boost 来引入 Apollo 客户端。通过 Apollo Boost， 你可以不做任何配置地，快速、方便地的创建一个 Apollo 客户端。为了便于学习，本节重点放在 Apollo 客户端而不是 React。首先，请找到 [Node.js 样板项目及其安装说明](https://github.com/rwieruch/node-babel-server)。目前你将在命令行中的 Node.js 环境中使用 Apollo 客户端。在一个最小的 Node.js 项目之上，你将通过 Apollo Boost 引入 Apollo 客户端来体验没有视图层库的 GraphQL 客户端。

接下来，你将使用 GitHub 的 GraphQL API，然后在命令行中输出查询和变更的结果。要做到这一点，你需要有一个 GitHub 网站的个人访问令牌，这个我们在前一章中已经介绍过了。如果你还没有完成，请参照 [GitHub 的说明](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) 生成具有足够权限的个人访问令牌。

在克隆安装了 Node.js 样板项目并且创建了个人访问令牌后，请在命令行新项目的根文件夹中安装以下两个包：

{title="Command Line",lang="json"}
~~~~~~~~
npm install apollo-boost graphql --save
~~~~~~~~

其中 [apollo-boost](https://github.com/apollographql/apollo-client/tree/master/packages/apollo-boost) 包提供一个零配置的 Apollo 客户端，而 [graphql](https： //github.com/graphql/graphql-js) 包允许在客户端和服务器上进行 GraphQL 查询，变更和订阅，它是基于 [Facebook 的 GraphQL 规范](https://github.com/facebook/graphql) 的 JavaScript 参考实现。

在接下来的步骤中，你将在项目的 *src/index.js* 文件中配置和使用 Apollo Boost 提供的 Apollo 客户端。这个项目比较小，而且你只会在本节中实现它，因此，为了便于学习，目前我们可以将所有内容都放在一个文件中。

在你的 *src/index.js* 文件中，你可以从 Apollo Boost 导入 Apollo 客户端。然后，你可以通过使用 URI 为参数调用其构造函数来创建一个客户端实例。客户端需要知道数据的来源以及应该写入的位置，那么你可以将 GitHub 的 API 端点传递给它。

{title="src/index.js",lang="javascript"}
~~~~~~~~
import ApolloClient from 'apollo-boost';

const client = new ApolloClient({
  uri: 'https://api.github.com/graphql',
});
~~~~~~~~

Apollo 客户端正是这样工作的。但请记住，GitHub 的 GraphQL API 需要个人访问令牌，这就是为什么在创建 Apollo 客户端实例时必须定义一次的原因。因此，你可以使用 `request` 属性来定义一个函数，该函数可以访问通过 Apollo 客户端发出的每个请求的上下文。在该函数里，你将使用 Apollo Boost 的授权头作为其默认头信息之一传递下去。

{title="src/index.js",lang="javascript"}
~~~~~~~~
import ApolloClient from 'apollo-boost';

const client = new ApolloClient({
  uri: 'https://api.github.com/graphql',
# leanpub-start-insert
  request: operation => {
    operation.setContext({
      headers: {
        authorization: `Bearer YOUR_GITHUB_PERSONAL_ACCESS_TOKEN`,
      },
    });
  },
# leanpub-end-insert
});
~~~~~~~~

你在之前的应用程序中做了相同的操作，仅使用 axios 进行纯 HTTP 请求。你使用 GraphQL API 端点为 axios 配置了一次，从而让所有的请求都默认访问这个 URI，并且设置授权头。这里也一样，为所有后续的 GraphQL 请求配置一次客户端就足够了。

请记住要用你之前在 GitHub 网站上创建的个人访问令牌替换 `YOUR_GITHUB_PERSONAL_ACCESS_TOKEN` 字符串。然而，你也许并不希望将访问令牌直接放到源代码中，那么你可以创建一个 *.env* 文件，该文件将包含你的项目文件夹中的所有环境变量。如果你不想在公共的 GitHub 代码库中共享个人令牌，你还可以将该文件添加到 *.gitignore* 文件中。你可以在命令行中创建该文件：

{title="Command Line",lang="json"}
~~~~~~~~
touch .env
~~~~~~~~

然后只需在此 *.env* 文件中定义环境变量即可。在 *.env* 文件中，粘贴如下键值对，其中键的命名由你决定，但值必须是你的 GitHub 个人访问令牌。

{title=".env",lang="javascript"}
~~~~~~~~
GITHUB_PERSONAL_ACCESS_TOKEN=xxxXXX
~~~~~~~~

在任何 Node.js 应用程序中，你都可以通过 [dotenv](https://github.com/motdotla/dotenv) 包在源代码中使用 key 作为环境变量。请按照他们的说明在你的项目中安装它。通常，你只需运行 `npm install dotenv`，然后在 *index.js* 文件的顶部添加 `import 'dotenv/config';`。然后，你就可以在 *index.js* 文件中使用 *.env* 文件中的个人访问令牌了。如果你遇到错误，请继续阅读本节以了解如何解决此问题。

{title="src/index.js",lang="javascript"}
~~~~~~~~
import ApolloClient from 'apollo-boost';

# leanpub-start-insert
import 'dotenv/config';
# leanpub-end-insert

const client = new ApolloClient({
  uri: 'https://api.github.com/graphql',
  request: operation => {
    operation.setContext({
      headers: {
# leanpub-start-insert
        authorization: `Bearer ${process.env.GITHUB_PERSONAL_ACCESS_TOKEN}`,
# leanpub-end-insert
      },
    });
  },
});
~~~~~~~~

注意：刚才安装的 dotenv 包可能还需要其他配置步骤。由于安装说明可能因 dotenv 版本的不同而有所不同，请在安装后查看其 GitHub 网站从而找到最佳配置。

当你使用 `npm start` 启动你的没有查询或变更而只有 Apollo Client 的应用程序时，你可能会看到以下错误：*"Error: fetch is not found globally and no fetcher passed, to fix pass a fetch for your environment ..."*。发生这个错误是因为基于 promise 的用于进行远程 API 请求的 [原生 fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 只能在浏览器中使用。你无法在一个仅运行在命令行中的 Node.js 应用程序中使用它。但是，Apollo Client 通常是在浏览器环境中使用 fetch API 来执行查询和变更的，而不是在 Node.js 环境中。你可能还记得，可以使用简单的 HTTP 请求执行查询或变更，因此 Apollo 客户端就使用了浏览器中的原生 fetch API 来执行这些请求。这个错误的解决方案是使用一个 node 工具包使得 fetch 在 Node.js 环境中可用。幸运的是，有一些包可以解决这个问题，可以通过命令行来进行安装：

{title="Command Line",lang="json"}
~~~~~~~~
npm install cross-fetch --save
~~~~~~~~

然后，在你的项目中将它匿名导入：

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import 'cross-fetch/polyfill';
# leanpub-end-insert
import ApolloClient from 'apollo-boost';
~~~~~~~~

当你再次从命令行启动应用程序时，错误应该会消失，不过到目前为止什么都还没有发生。我们使用配置创建了一个 Apollo 客户端的实例。在下文中，你将会使用 Apollo 客户端来执行第一个查询。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/node-apollo-boost-github-graphql-api/tree/fd067ec045861e9832cc0b202b25f8d8efd651c9)
* 延伸阅读：[其他视图集成，如 Angular 和 Vue](https://www.apollographql.com/docs/react/integrations.html)
* 花几分钟的时间进行[测验](https://www.surveymonkey.com/r/5T3W9BB)

## Apollo 客户端和 GraphQL 查询

现在，你将使用 Apollo 客户端向 GitHub 的 GraphQL API 发送你的第一个查询。从 Apollo Boost 导入如下实用程序来定义查询：

{title="src/index.js",lang="javascript"}
~~~~~~~~
import 'cross-fetch/polyfill';
# leanpub-start-insert
import ApolloClient, {gql} from 'apollo-boost';
# leanpub-end-insert
~~~~~~~~

使用 JavaScript 模板字面量定义你的查询：

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const GET_ORGANIZATION = gql`
  {
    organization(login: "the-road-to-learn-react") {
      name
      url
    }
  }
`;
# leanpub-end-insert
~~~~~~~~

命令式地让 Apollo 客户端将查询发送到 GitHub 的 GraphQL API。由于 Apollo 客户端是基于 promise 的，`query()` 方法会返回一个你最终可以 resolve 的 promise。由于应用程序是在在命令行中运行的，把结果输出到控制台就足够了。

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
client
  .query({
    query: GET_ORGANIZATION,
  })
  .then(console.log);
# leanpub-end-insert
~~~~~~~~

这就是用 Apollo 客户端发送查询的全部内容。如上所述，Apollo 客户端的底层是使用 HTTP 在 POST 方法中将定义好的查询作为有效负载发送出去。使用 `npm start` 启动应用程序后命令行上的结果应类似于以下内容：

{title="Command Line",lang="json"}
~~~~~~~~
{
  data: {
    organization: {
      name: 'The Road to learn React',
      url: 'https://github.com/the-road-to-learn-react',
      __typename: 'Organization'
    }
  },
  loading: false,
  networkStatus: 7,
  stale: false
}
~~~~~~~~

GraphQL 查询中请求的信息可以在 `data` 对象中找到。`data` 对象包含 `organization` 对象及其 `name` 和 `url` 字段。Apollo 客户端会自动请求 GraphQL [元字段](http://graphql.org/learn/queries/#meta-fields) `__typename`。Apollo 客户端可以使用元字段作为标识符，以允许缓存和乐观 UI 更新。

有关请求的更多元信息可以在 `data` 对象旁边找到。它显示数据是否仍在加载，[网络状态](https://github.com/apollographql/apollo-client/blob/master/packages/apollo-client/src/core/networkStatus .ts) 的具体细节，以及请求的数据是否在服务器端过时了。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/node-apollo-boost-github-graphql-api/tree/7a800c78e0e09f84b47f4e714abac1d23f5e599e)
* 探索 GitHub 的 GraphQL API
  * 轻松地浏览他们的文档
  * 为 `organization` 字段添加其他字段
* 延伸阅读：[为什么你应该使用 Apollo Client](https://www.apollographql.com/docs/react/why-apollo.html)
* 延伸阅读：[networkStatus 属性及其可能值](https://github.com/apollographql/apollo-client/blob/master/packages/apollo-client/src/core/networkStatus.ts)
* 花 3 分钟时间来进行[测验](https://www.surveymonkey.com/r/5MF35H5)

## 带分页，变量，嵌套对象和列表字段的 Apollo Client

在前几节使用不含 Apollo 的 GraphQL 应用程序构建 React 时，你已经了解了 GraphQL 的分页和其它功能。本节将介绍其中的一些功能，如 GraphQL 变量。上一个查询中 organization 字段的 `login` 参数可以用这样的变量进行替换。首先，你必须在 GraphQL 查询中引入这个变量：

{title="src/index.js",lang="javascript"}
~~~~~~~~
const GET_ORGANIZATION = gql`
# leanpub-start-insert
  query($organization: String!) {
    organization(login: $organization) {
# leanpub-end-insert
      name
      url
    }
  }
`;
~~~~~~~~

其次，在你的查询对象的 variables 对象中定义这个变量：

{title="src/index.js",lang="javascript"}
~~~~~~~~
client
  .query({
    query: GET_ORGANIZATION,
# leanpub-start-insert
    variables: {
      organization: 'the-road-to-learn-react',
    },
# leanpub-end-insert
  })
  .then(console.log);
~~~~~~~~

这就是你在应用程序中的使用 Apollo Client 实例将变量传递给查询的方法。接下来，将嵌套的 `repositories` 列表字段添加到你的 organization 字段。这样你就可以请求一个组织中的所有 GitHub 代码库。你可能还想重命名查询变量，但请记得在使用 Apollo Client 时更改它。

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const GET_REPOSITORIES_OF_ORGANIZATION = gql`
# leanpub-end-insert
  query($organization: String!) {
    organization(login: $organization) {
      name
      url
# leanpub-start-insert
      repositories(first: 5) {
        edges {
          node {
            name
            url
          }
        }
      }
# leanpub-end-insert
    }
  }
`;

client
  .query({
# leanpub-start-insert
    query: GET_REPOSITORIES_OF_ORGANIZATION,
# leanpub-end-insert
    variables: {
      organization: 'the-road-to-learn-react',
    },
  })
  .then(console.log);
~~~~~~~~

在我们之前创建的应用程序中，你已经看到过类似的查询结构，因此本节提供几个练习供你测试所学的 GraphQL 技能。通过这些练习能够强化你的 GraphQL 技能，这样你以后可以集中精力将 Apollo Client 连接到你的 React 应用程序，而不会遇到任何障碍。在练习结束后，你可以在此应用程序的 GitHub 代码库中找到所有练习的答案，但你应该先尝试自己完成它们。

### 练习：
* 查看[本节源码](https://github.com/the-road-to-graphql/node-apollo-boost-github-graphql-api/tree/a5b1ce61a3dae3ead1b9795f5bf6e0d090c5d24f)
* 探索 GitHub 的 GraphQL API
  * 通过查询一个按照 star 数量排序的有序代码库列表来扩展 `repositories` 列表字段
* 将 repository `node` 的内容提取为可重用的 GraphQL 片段
* 延伸阅读：[GraphQL 中的分页](https://graphql.org/learn/pagination)
* 为代码库列表添加分页功能
  * 在查询中添加 `pageInfo` 字段，它应包含 `endCursor` 和 `hasNextPage` 字段
  * 添加 `after` 参数并为其引入一个新的 `$cursor` 变量
  * 执行第一个查询时不带 `cursor` 参数
  * 使用前一个查询结果的 `endCursor` 作为 `cursor` 参数执行第二个查询
* 花三分钟进行[测验](https://www.surveymonkey.com/r/SWL9NJ7)

## Apollo Client 和 GraphQL 变更

前几节你学习了如何使用 Apollo Client 从 GitHub 的 GraphQL API 查询数据。一旦配置好了 Apollo Client，就可以使用其 `query()` 方法发送带有可选 `variables` 的 GraphQL `查询`。正如你所了解的那样，我们不仅可以使用 GraphQL 来读取数据，还可以通过变更来写入数据。在本节中，你将定义一个变更来为 GitHub 上的代码库加 star。Apollo Client 实例会发送变更，但首先你必须定义它。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const ADD_STAR = gql`
  mutation AddStar($repositoryId: ID!) {
    addStar(input: { starrableId: $repositoryId}) {
      starrable {
        id
        viewerHasStarred
      }
    }
  }
`;
~~~~~~~~

代码库的标识符是必需的，否则 GitHub 的 GraphQL 服务器不知道你要为哪个代码库加 star。在下一个代码片段中，Apollo Client 用来为具有给定标识符的特定 GitHub 代码库加注星标。可以通过在查询中将 `id` 字段添加到你的代码库 `node` 字段来检索标识符。使用 Apollo Client 上的 `mutate()` 方法在 `mutation` 和 `variables` 有效载荷中发送变更。可以对结果进行任何操作以适用于你的应用程序，但在本例中，只需在命令行中记录结果即可。

{title="src/index.js",lang="javascript"}
~~~~~~~~
client
  .mutate({
    mutation: ADD_STAR,
    variables: {
      repositoryId: 'MDEwOlJlcG9zaXRvcnk2MzM1MjkwNw==',
    },
  })
  .then(console.log);
~~~~~~~~

结果应该会封装在一个 `addStar` 对象（变更的名称）中，它应该准确反映你在变更中定义的对象和字段：`starrable`，`id` 和 `viewerHasStarred`。

你又完成了另一个学习步骤，其中只使用了 Apollo Client 而没有使用任何视图层库。这样做是为了避免混淆 Apollo Client 和 React Apollo 的功能。

请记住，Apollo Client 可以作为独立的 GraphQL 客户端使用，而无需将其连接到像 React 这样的视图层，尽管仅在命令行上查看数据似乎有点沉闷。我们将在下一节中看到 Apollo 如何将数据层连接到 React 视图层。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/node-apollo-boost-github-graphql-api/tree/ed3363c9981c552223117e5e775bb8c535f79ff5)
* 仿照 `addStar` 变更实现 `removeStar` 变更
* 花三分钟的时间进行[测验](https://www.surveymonkey.com/r/5XMNFSY)

| |

你已经了解了如何在 Node.js 项目中单独使用 Apollo 客户端。在本章之前，你已经将 React 与不带 Apollo 的 GraphQL 结合使用。在下一章中，你将把这两种结合起来。尽情期待你的第一个完整的使用 Apollo 客户端和 GraphQL 的 React 客户端应用程序吧。