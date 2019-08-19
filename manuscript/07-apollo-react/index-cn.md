> # React with GraphQL and Apollo Client

# React 结合 GraphQL 与 Apollo 客户端

> In this tutorial, you will learn how to combine React with GraphQL in your application using Apollo. The Apollo toolset can be used to create a GraphQL client, GraphQL server, and other complementary applications, but you will use the Apollo Client for your React client-side application. Along the way, you will build a simplified GitHub client that consumes [GitHub's GraphQL API](https://developer.github.com/v4/) using Apollo instead of plain HTTP requests like the previous application. Apollo Client can be used to perform queries and mutations, and to read and write data. By the end, you should be able to showcase a React application using GraphQL and Apollo that can be used by other developers as a learning tool. You can find the final project as a [repository on GitHub](https://github.com/rwieruch/react-graphql-github-apollo).

在本章中，你会学到如何通过 Apollo 将 GraphQL 与 React 结合。虽然 Apollo 工具集可以创建 GraphQL 客户端、GraphQL 服务端以及其他通用应用，但是这里你会在 React 客户端应用使用 Apollo 客户端。通过这种方式，你将实现一个简单 Github 客户端，并使用 Apollo (而不是像之前那样直接使用普通 HTTP 请求的方式) 消费 [Github 的 GraphQL API](https://developer.github.com/v4/)。Apollo 客户端可以通过执行查询和变更来读写数据。在学完本章后，你将得到一个使用 Apollo 和 GraphQL 的 React 应用，它可以作为其他开发者的一份学习参考。你可以在 [GitHub 代码库](https://github.com/rwieruch/react-graphql-github-apollo)里找到最终的项目。

> ## Writing your first React application with GraphQL and Apollo Client
## 编写你的第一个使用 GraphQL 和 Apollo 客户端的 React 应用

> Now we'll focus on using Apollo Client in React by building another client application. Basically, you will learn how to connect the data-layer to the view-layer. We'll cover how to send queries and mutations from the view-layer, and how to update the view-layer to reflect the result. Further, you will learn to use GraphQL features like pagination, optimistic UI, caching, local state management, and prefetching with Apollo Client in React.

现在我们开始在 React 中使用 Apollo 客户端构建另一个应用。你会学到如何连接数据层与视图层，涉及如何从视图层发送查询和变更，以及如何根据返回结果更新视图层。然后，你将了解如何在 React 中结合 Apollo 客户端，使用如分页、乐观 UI、缓存、本地状态管理以及预加载的 GraphQL 特性。

> For this application, no elaborate React setup is needed. Simply use [create-react-app](https://github.com/facebook/create-react-app) to create your React application. If you want to have an elaborate React setup instead, see this [setup guide for using Webpack with React](https://www.robinwieruch.de/minimal-react-webpack-babel-setup/). To get started, the following steps have to be performed:

对于这个应用，不需要复杂的初始化配置。使用 [create-react-app](https://github.com/facebook/create-react-app) 创建一个即可。如果你希望有一个定制化的 React 初始化配置，看看 [使用 Webpack 的 React 配置指南](https://www.robinwieruch.de/minimal-react-webpack-babel-setup/)。首先，你需要完成下面的步骤：

> * Create a new React application with create-react-app
> * Create a folder/file structure for your project (recommendation below)

* 使用 create-react-app 创建一个 React 应用
* 在项目中创建一些目录/文件结构（推荐使用下面的方式)

> You can create your own folder and file structure for your components in the *src/* folder; the following top level structure is only a recommendation. If you adjust it to your own needs, keep in mind that the JavaScript import statements with their paths will need to be adjusted to match. If you don't want to create everything, you can clone this [GitHub repository](https://github.com/the-road-to-graphql/react-graphql-github-apollo-starter-kit) instead and follow its installation instructions.

你可以在 *src/* 目录下创建自定义的目录和文件结构，下面的顶层目录结构仅做参考。如果你需要调整目录，记住 JavaScript 的 import 语句也需要相应调整。如果你不想什么都自己新建，那么你可以略过这份指南直接克隆这份 [Github 代码库](https://github.com/the-road-to-graphql/react-graphql-github-apollo-starter-kit)，并按照其安装说明进行操作。

* App/
  * index.js
* Button/
* Error/
* FetchMore/
* Input/
* Issue/
  * IssueList/
  * IssueItem/
  * index.js
* Link/
* Loading/
* Organization/
* Profile/
* Repository/
  * RepositoryList/
  * RepositoryItem/
  * index.js
* TextArea/
* constants/
  * routes.js
* index.js
* serviceWorker.js
* style.css

> The folders primarily represent React components. Some components will be reusable UI components such as the Input and Link components, while other components like Repository and Profile components are domain specific for the GitHub client application. Only the top level folders are specified for now, though more can be introduced later if you choose. Moreover, the *constants* folder has only one file to specify the application's routes, which will be introduced later. You may want to navigate from a page that shows repositories of an organization (Organization component) to a page which shows repositories of yourself (Profile component).

这个目录主要展示了 React 组件。其中有一些比如 Input 和 Link 组件将是可以复用的 UI 组件，其他的比如 Repository 和 Profile 组件是 GitHub 客户端应用的特定领域组件。目前只有顶层的目录结构，后面需要可以按需引入更多的目录。还有就是，*contants* 目录现在只有一个用来指定路由的文件（后面会详细介绍），比如你可能希望从展示 GitHub 组织的代码库页面（Organization 组件）跳转到你自己的代码库页面（Profile 组件）中。

> This application will use plain CSS classes and CSS files. By following the plain CSS classes, you can avoid difficulties that may occur with other tools. You will find all the CSS files and their content in the appendix section for this application. The components will use their class names without explaining them. The next sections should be purely dedicated to JavaScript, React, and GraphQL.

这个应用只会使用纯粹的 CSS 类和纯粹的 CSS 文件。使用纯粹 CSS 类，你可以避免其他工具中可能出现的困难。你可以在附录中针对此应用程序的部分找到所有的 CSS 文件和它的内容。还有就是这些组件名直接使用它的类名，不会加以阐述。接下来的部分将专注于 JavaScript、React 和 GraphQL。

> ### Exercises:
### 练习

> * If you are not familiar with React, read up *The Road to learn React*
> * Set up the recommended folder/file structure (if you are not going with your own structure and didn't clone the repository)
>   * Create the CSS *style.css* files in their specified folders from the CSS appendix section
>   * Create the *index.js* files for the components
>   * Create further folders on your own for non top level components (e.g. Navigation) when conducting the following sections
> * Run the application with `npm start`
>   * Make sure there are no errors
>   * Render only a basic App component with *src/App/index.js* in the *src/index.js* file
> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5N9W2WR)

* 如果你对 React 不熟悉，可以阅读 *React 学习之道*
* 参考推荐的文件/目录结构完成项目的初始化工作（如果你既不想自己设计目录结构，也不想直接克隆代码库的话）
  * 参考 CSS 附录部分，在特定目录创建 *style.css* 文件
  * 为组件创建 *index.js* 文件
  * 接下来的教程中，创建一些自己的组件（比如 Navigation）
* 使用 `npm start` 运行应用
  * 保证运行无错误
  * 保证只会渲染 *src/index.js* 目录下 *src/App/index.js* 这个基础组件
* 花三分钟的时间进行[测验](https://www.surveymonkey.com/r/5N9W2WR) 

> ## Configure Apollo Client for React and GitHub's GraphQL API
## 配置使用 React 和 GitHub GraphQL API 的 Apollo 客户端

> In this section, you will set up an Apollo Client instance like we did previously. However, this time you will use Apollo Client directly without the zero-configuration package Apollo Boost, meaning you'll need to configure the Apollo Client yourself without sensible defaults. While it's best to use a tool with sensible defaults for learning, configuring Apollo yourself exposes the composable ecosystem of Apollo Client, how to use it for an initial setup, and how to advance this setup later.

本节和前面一样，我们需要设置好一个 Apollo 客户端实例。不过，这部分，你需要直接配置 Apollo 客户端，而不借助于 Apollo Boost 的零配置方式。尽管使用工具的默认配置很适合初学，不过自己去配置 Apollo 客户端可以了解 Apollo 里的整个生态，了解最开始使用怎么开始初始化配置，又怎么去扩展增强这份配置。

> The Apollo Client setup can be completed in the top-level *src/index.js* file, where the React to HTML entry point exists as well. First, install the Apollo Client in your project folder using the command line:
Apollo 客户端可以写在 *src/index.js* 文件中，虽然也可以在 HTML 中的 React 入口点设置。首先，在你的项目目录下，使用命令安装 Apollo 客户端：

{title="Command Line",lang="json"}
~~~~~~~~
npm install apollo-client --save
~~~~~~~~

> Two utility packages are required for two mandatory configurations used to create the Apollo Client. The [apollo-cache-inmemory](https://github.com/apollographql/apollo-client/tree/master/packages/apollo-cache-inmemory) is a recommended cache (read also as: store or state) for your Apollo Client to manage the data, while apollo-link-http is used to configure the URI and additional network information once for an Apollo Client instance.
创建 Apollo 客户端有两个必须的配置项，需要两个实用工具包。其中 [apollo-cache-inmemory](https://github.com/apollographql/apollo-client/tree/master/packages/apollo-cache-inmemory) 推荐用来缓存（有时也作 store 或者 state） Apollo 客户端管理的数据，而 appo-link-http 是用来配置 URI 和 Apollo 客户端实例需要的其他网络信息的。

{title="Command Line",lang="json"}
~~~~~~~~
npm install apollo-cache-inmemory apollo-link-http --save
~~~~~~~~

> As you can see, nothing has been mentioned about React, only the Apollo Client plus two packages for its configuration. There are two additional packages required for Apollo Client to work with GraphQL, to be used as internal dependencies by Apollo. The latter is also used to define queries and mutations. Previously, these utilities came directly from Apollo Boost.
> 如你所见，并没有提到 React，Apollo 客户端加上两个库就是它的配置了。这是 Apollo 客户端使用 GraphQL 所必须的两个额外的包，是作为 Apollo 的内部依赖，后者也用来定义查询和变更。之前，这些工具，已经包含在 Apollo Boost 了。


{title="Command Line",lang="json"}
~~~~~~~~
npm install graphql graphql-tag --save
~~~~~~~~

> That's it for package installation, so now we enter the Apollo Client setup and configuration. In your top level *src/index.js* file, where all the Apollo Client setup will be done in this section, import the necessary classes for the Apollo Client setup from the previously installed packages.

到此就完成了依赖的安装了，现在我们会进入 Apollo 客户端的初始化配置。所有的 Apollo 客户端配置都会在顶层的 *src/index.js* 文件中，从之前安装的依赖中，导入必要的类，完成 Apollo 客户端的初始化。


{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import { ApolloClient } from 'apollo-client';
import { HttpLink } from 'apollo-link-http';
import { InMemoryCache } from 'apollo-cache-inmemory';
# leanpub-end-insert

import './style.css';
import App from './App';

...
~~~~~~~~

> The `ApolloClient` class is used to create the client instance, and the `HttpLink` and `InMemoryCache` are used for its mandatory configurations. First, you can create a configured `HttpLink` instance, which will be fed to the Apollo Client creation.

`ApooloClient` 类用来创建客户端实例，`HttpLink` 和 `InMemoryCache` 用于必须的配置项。首先你需要创建一个可配置的 `HttpLink` 实例，在 Apollo 客户端创建的时候有用。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const GITHUB_BASE_URL = 'https://api.github.com/graphql';

const httpLink = new HttpLink({
  uri: GITHUB_BASE_URL,
  headers: {
    authorization: `Bearer ${
      process.env.REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN
    }`,
  },
});
~~~~~~~~

> You may recall the mandatory configuration from previous applications. The `uri` is a mandatory value to define the only GraphQL API endpoint used by the Apollo Client. In this case, Github's GraphQL endpoint is passed as value. When consuming the GitHub GraphQL API, you have to authorize yourself with your personal access token. You should have already created the token in a previous section, which you can now define in a *.env* file in your project folder. Afterward, it should be accessible with `process.env`. Keep in mind that you have to use the `REACT_APP` prefix when using create-react-app, because that's how it is required by create-react-app. Otherwise, you would be free to choose your own naming for it.

你可能记得在前面的应用中，GraphQL API 端点是 Apollo Client 需要的唯一配置项，本书的示例里，传入的是 Github 的 GraphQL 端点。在消费 Github GraphQL API 前，你必须使用你的个人访问令牌进行认证。你应该在前一部分中已经创建了令牌（你可以在项目目录中创建一个 *.env* 文件） 了。之后，应该可以通过 `process.env` 获取这个值。记住你在使用 create-react-app 时，需要加上 `REACT_APP` 前缀，这是 create-react-app 规定的。此外，你可以随意命名。

> Second, create the cache as the place where the data is managed in Apollo Client. The cache normalizes your data, caches requests to avoid duplicates, and makes it possible to read and write data to the cache. You will use it multiple times while developing this application. The cache instantiation is straightforward, as it doesn't require you to pass any arguments to it. Check the API to explore further configurations.

第二步，创建 Apollo Client 管理数据的缓存。缓存能归一化数据，缓存可以避免多余重复的请求，也允许通过缓存读写数据。在开发这个应用中会多次用到它。实例化缓存非常直观，不需要传递任何参数。详细了解下它的 API 以便将来的进一步配置吧。


{title="src/index.js",lang="javascript"}
~~~~~~~~
const cache = new InMemoryCache();
~~~~~~~~

> Finally, you can use both instantiated configurations, the link and the cache, to create the instance of the Apollo Client in the *src/index.js* file.
最后，你需要将两个实例化的配置—— link 和 cache，在 *src/index.js* 文件中添加到 Apollo Client 中，用于创建其实例。

{title="src/index.js",lang="javascript"}
~~~~~~~~
const client = new ApolloClient({
  link: httpLink,
  cache,
});
~~~~~~~~

> To initialize Apollo Client, you must specify link and cache properties on the config object. Once you start your application again, there should be no errors. If there are any, check whether you have implemented a basic App component in your *src/App/index.js* file because the ReactDOM API needs to hook this component into the HTML.
为了实例化 Apollo Client，你必须在配置对象中，指定 link 和 cache 属性。应用一旦启动，应该没有报错。如果有报错信息，请检查在 *src/App/index.js* 文件中是否实现了基本的 App 组件，因为 ReactDOM API 需要一个组件用于在 HTML 中渲染。

> ### Exercises:
### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/c7454c9f6b5f7cdf9d65722ccae7ae38f648aef3)
> * Read more about [the network layer configuration in Apollo Client](https://www.apollographql.com/docs/react/advanced/network-layer.html)
> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5FYZT8T)
* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/c7454c9f6b5f7cdf9d65722ccae7ae38f648aef3)
* 延伸阅读：[Apollo 中的网络层配置](https://www.apollographql.com/docs/react/advanced/network-layer.html)
* 花三分钟的时间进行[测验](https://www.surveymonkey.com/r/5FYZT8T)

> ## Connect Data-Layer to View-Layer: Introducing React Apollo
## 绑定数据层与视图层：介绍 React Apollo

> All we've done thus far has been the framework agnostic part of Apollo Client. However, without connecting React to it, you'd have a hard time making effective use of GraphQL. That's why there is an official library to connect both worlds: [react-apollo](https://github.com/apollographql/react-apollo). The great thing about those connecting libraries is that there are solutions for other view-layer solutions like Angular and Vue, too, so you can use the Apollo Client in a framework agnostic way. In the following, it needs two steps to connect the Apollo Client with React. First, install the library in the command line in your project folder:
目前我们完成的部分，仅仅是 Apollo 客户端中框架无关的部分配置。如果没有与 React 进行绑定，那使用 GraphQL 会比较困难。这就是为什么有一个连接两者的官方库 [react-apollo](https://github.com/apollographql/react-apollo) 存在了。这些绑定库也提供了其他视图层的积极方案，比如说 Angular 和 Vue，使用 Apollo 客户端是框架无关的。后面的部分，为了完成 Apollo 客户端和 React 的绑定，需要两个步骤。首先在你的项目目录下安装必要的依赖：

{title="Command Line",lang="json"}
~~~~~~~~
npm install react-apollo --save
~~~~~~~~

>Second, import its ApolloProvider component, and use it as a composing component around your App component in the *src/index.js* file. Under the hood, it uses [React's Context API](https://www.robinwieruch.de/react-context-api/) to pass the Apollo Client through your application.
其次，导入它的 ApolloProvider 组件，并在 *src/index.js* 中，包裹住你的 APP 组件。这里，它使用了 [React 的 Context API](https://www.robinwieruch.de/react-context-api/)，Apollo 客户端就可以在整个应用中使用了。

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import { ApolloProvider } from 'react-apollo';
# leanpub-end-insert
import { ApolloClient } from 'apollo-client';
import { HttpLink } from 'apollo-link-http';
import { InMemoryCache } from 'apollo-cache-inmemory';

...

ReactDOM.render(
# leanpub-start-insert
  <ApolloProvider client={client}>
# leanpub-end-insert
    <App />
# leanpub-start-insert
  </ApolloProvider>,
# leanpub-end-insert
  document.getElementById('root')
);
~~~~~~~~

>Now you have implicit access to the Apollo Client in your React view-layer. It says implicit because most often you will not use the client explicitly. You will see in the next section what this means.
现在你在 React 视图层中，你可以隐式访问 Apollo 客户端了。就是说大多数情况下，你无需直接使用客户端，后面的部分会进一步解释。

> ### Exercises:
### 练习

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/8377cbc55de3c860df0150d8946e261938a67db5)
> * Read more about [configuring and connecting Apollo Client to React](https://www.apollographql.com/docs/react/essentials/get-started.html)
> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5FHMHW8)
* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/8377cbc55de3c860df0150d8946e261938a67db5)一致
* 延伸阅读：[配置并将 Apollo Client 和 React 绑定](https://www.apollographql.com/docs/react/essentials/get-started.html)
* 花费三分钟时间参与[测验](https://www.surveymonkey.com/r/5FHMHW8)

> ## GraphQL Query with Apollo Client in React
## 在 React 中 使用 Apollo 客户端进行 GraphQL 查询

> In this section, you will implement your first GraphQL query using Apollo Client in React. You've seen how different entities, such as the current user (viewer) or repositories, can be queried from GitHub's GraphQL API. This time you will do it in React. A Profile component might be the best place to render the current user and its associated repositories. Start by using the not-yet-implemented Profile component in your App component in the *src/App/index.js* file, which we'll take care of next. It makes sense to extract the Profile component now, because the App component will be the static frame around the application later. Components like Navigation and Footer are static, and components such as Profile and Organization are dynamically rendered based on routing (URLs).
在这部分，你需要使用 Apollo 客户端在 React 中实现第一个 GraphQL 查询。这次你会在 React 中实现不同的实体，比如当前用户（viewer）或者代码库，在 GitHub 的 GraphQL API 的查询。Profile 组件适合渲染当前用户和他关联的代码库。后面我们会在 *src/App/index.js* 文件中使用还未实现的 Profile 组件。抽出 Profile 组件是有意义的，因为 App 组件以后再应用总会作为应用的静态框架，如 Navigation 和 Footer 组件也会是静态的，而 Profile 和 Organization 之类的组件会基于路由（URLs）动态渲染。

{title="src/App/index.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';

# leanpub-start-insert
import Profile from '../Profile';
# leanpub-end-insert

class App extends Component {
  render() {
# leanpub-start-insert
    return <Profile />;
# leanpub-end-insert
  }
}

export default App;
~~~~~~~~

> In your *src/Profile/index.js* file, add a simple functional stateless component. In the next step you will extend it with a GraphQL query.
在 *src/Profile/index.js* 文件中，添加一个简单的无状态组件。下一步，你使用 GraphQL 查询对其进行扩展。 

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

const Profile = () =>
  <div>Profile</div>

export default Profile;
~~~~~~~~

> Now we'll learn to query data with GraphQL and Apollo Client. The Apollo Client was provided in a previous section with React's Context API in a top level component. You have implicit access to it, but never use it directly for standard queries and mutations. It says "standard" here, because there will be situations where you use the Apollo Client instance directly while implementing this application.
现在学习在使用 Apollo 客户端查询数据。前一部分中，Apollo 客户端是由 React Context API 提供的，你可以隐式的获取它，不过还没有使用它直接进行标准查询或变更。这里提到“标准”，是因为在实现这个应用的时候，你使用的是 Apollo 客户端实例。


> The React Apollo package grants access to a Query component, which takes a query as prop and executes it when its rendered. That's the important part: it executes the query when it is rendered. It uses React's [render props](https://www.robinwieruch.de/react-render-props-pattern/) pattern, using a child as a function implementation where you can access the result of the query as an argument.
React Apollo 库确保，Query 组件能接受一个查询 prop，并在渲染的时候被执行。这是最重要的一部分：当渲染的时候执行查询。它使用了 React 的 [render props](https://www.robinwieruch.de/react-render-props-pattern/) 模式，使用子函数的实现方式，获取查询结果。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import { Query } from 'react-apollo';
# leanpub-end-insert

const Profile = () => (
# leanpub-start-insert
  <Query query={}>
    {() => <div>My Profile</div>}
  </Query>
# leanpub-end-insert
);

export default Profile;
~~~~~~~~

> This is a function that returns only JSX, but you have access to additional information in the function arguments. First, define the GraphQL query to request your authorizations. You can use a previously installed utility package to define the query.
这个函数的返回的结果只有 JSX，但是你可以从函数的参数中获得额外的一些信息。首先，定义 GraphQL 查询请求你的登录信息，你可以使用之前安装的工具包来定义查询语句。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import gql from 'graphql-tag';
# leanpub-end-insert
import { Query } from 'react-apollo';

# leanpub-start-insert
const GET_CURRENT_USER = gql`
  {
    viewer {
      login
      name
    }
  }
`;
# leanpub-end-insert

const Profile = () => (
# leanpub-start-insert
  <Query query={GET_CURRENT_USER}>
# leanpub-end-insert
    {() => <div>My Profile</div>}
  </Query>
);

export default Profile;
~~~~~~~~

> Use the children as a function pattern to retrieve the query result as a data object, and render the information in your JSX.
使用子函数模式拿到查询结果的数据对象，并将信息通过 JSX 语句渲染出来。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import gql from 'graphql-tag';
import { Query } from 'react-apollo';

const GET_CURRENT_USER = gql`
  {
    viewer {
      login
      name
    }
  }
`;

const Profile = () => (
  <Query query={GET_CURRENT_USER}>
# leanpub-start-insert
    {({ data }) => {
      const { viewer } = data;

      return (
        <div>
          {viewer.name} {viewer.login}
        </div>
      );
    }}
# leanpub-end-insert
  </Query>
);

export default Profile;
~~~~~~~~

> Make sure to give some type of visual feedback until your view-layer can be rendered with actual data:
确保在视图层获得真实数据前，能有一些的界面上的反馈。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
const Profile = () => (
  <Query query={GET_CURRENT_USER}>
    {({ data }) => {
      const { viewer } = data;

# leanpub-start-insert
      if (!viewer) {
        return null;
      }
# leanpub-end-insert

      return (
        <div>
          {viewer.name} {viewer.login}
        </div>
      );
    }}
  </Query>
);
~~~~~~~~

> That's how you define a GraphQL query in a declarative way in React. Once the Query component renders, the request is executed. The Apollo Client is used, provided in a top level component, to perform the query. The render props pattern makes it possible to access the result of the query in the child function. You can try it in your browser to verify that it actually works for you.
这就是 React 中声明式定义 GraphQL 查询的方式。一旦 Query 组件渲染，请求就会被执行。在顶层组件中注入的 Apollo 客户端会被用来执行查询。render props 模式允许在子函数中获得查询结果。你可以在浏览器验证它真实的工作方式。

> There is more information found in the render prop function. Check the official React Apollo API for additional information beyond the examples in this application. Next, let's show a loading indicator when a query is pending:
在 render props 函数中，可以获取更多的信息，请查看官方的 React Apollo API 以获取这个应用示例外的信息。然后，我们在请求还在等待结果时，加入一个 loading 指示器。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
const Profile = () => (
  <Query query={GET_CURRENT_USER}>
# leanpub-start-insert
    {({ data, loading }) => {
# leanpub-end-insert
      const { viewer } = data;

# leanpub-start-insert
      if (loading || !viewer) {
        return <div>Loading ...</div>;
      }
# leanpub-end-insert

      return (
        <div>
          {viewer.name} {viewer.login}
        </div>
      );
    }}
  </Query>
);
~~~~~~~~

> The application now shows a loading indicator when there is no `viewer` object or the `loading` boolean is set to true. As you can assume that the request will be pending when there is no `viewer`, you can show the loading indicator from the beginning. At this point, it's best to extract the loading indicator as its own component because you will have to reuse it later for other queries. You created a Loading folder for it before, which will house the *src/Loading/index.js* file. Then, use it in your Profile component.
现在应用在没有视图对象的时候，loading 的值会被设置成 true，然后应用会展示 loading 指示器。当你假定请求会阻塞，你可以一开始就展示一个 loading 指示器。最好将 loading 指示器抽出成一个单独的组件，因为你可能后面需要在其他的查询中使用。创建一个 Loading 目录，用来存放 *src/Loading/index.js* 文件，在你的 Profile 组件中使用。

{title="src/Loading/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

const Loading = () =>
  <div>Loading ...</div>

export default Loading;
~~~~~~~~

> Next, extend the query with a nested list field for querying your own GitHub repositories. You have done it a few times before, so the query structure shouldn't be any different now. The following query requests a lot of information you will use in this application:
接下来，扩展查询语句，添加一个内嵌的列表字段查询你自己的 Github 代码库。你之前已经做过几遍这个操作了，查询语句的结构不会有什么变化。下面的查询语句会在这个应用中使用，用来请求很多信息。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const GET_REPOSITORIES_OF_CURRENT_USER = gql`
# leanpub-end-insert
  {
    viewer {
# leanpub-start-insert
      repositories(
        first: 5
        orderBy: { direction: DESC, field: STARGAZERS }
      ) {
        edges {
          node {
            id
            name
            url
            descriptionHTML
            primaryLanguage {
              name
            }
            owner {
              login
              url
            }
            stargazers {
              totalCount
            }
            viewerHasStarred
            watchers {
              totalCount
            }
            viewerSubscription
          }
        }
      }
# leanpub-end-insert
    }
  }
`;
~~~~~~~~

> Use this extended and renamed query in your Query component to request additional information about repositories. Pass these repositories from the query result to a new RepositoryList component which should do all the rendering for you. It's not the responsibility of the Profile component, and you may want to render a list of repositories somewhere else.
在 Query 组件中使用这个扩展和重新命名的查询，请求代码库的额外信息。将查询结果的代码库信息传给新的 RepositoryList 组件，它会被用来渲染所有的代码库，因为这并不是 Profile 组件的职责，而且你可能也会需要在其他地方渲染代码库列表。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
import RepositoryList from '../Repository';
import Loading from '../Loading';
# leanpub-end-insert

...

const Profile = () => (
  <Query query={GET_REPOSITORIES_OF_CURRENT_USER}>
    {({ data, loading }) => {
      const { viewer } = data;

      if (loading || !viewer) {
        return <Loading />;
      }

# leanpub-start-insert
      return <RepositoryList repositories={viewer.repositories} />;
# leanpub-end-insert
    }}
  </Query>
);
~~~~~~~~

> In your *src/Repository/index.js* file, create your first import/export statements for the RepositoryList component from a dedicated file in this folder. The *index.js* file is used as your entry point to this Repository module. Everything used from this module should be accessible by importing it from this *index.js* file.
在 *src/Repository/index.js* 文件中，使用 import/export 语句让 RepositoryList 可以从这个目录直接获取。*index.js* 文件是用作这个 Repository 模块的入口存在的。所有这个模块的东西都应该可以从这个 *index.js* 文件中导入。

{title="src/Repository/index.js",lang="javascript"}
~~~~~~~~
import RepositoryList from './RepositoryList';

export default RepositoryList;
~~~~~~~~

> Next, define the RepositoryList component in your *src/Repository/RepositoryList/index.js* file. The component only takes the array of repositories as props, which will be retrieved by the GraphQL query to render a list of RepositoryItem components. The identifier of each repository can be passed as key attribute to the rendered list. Otherwise, all props from one repository node are passed to the RepositoryItem using the JavaScript spread operator.
接下来，在 *src/Repository/RepositoryList/index.js* 文件中定义 RepositoryList 组件。这个组件只需要接受一个代码库数组（从 GraphQL 查询中获取的数据）作为 prop 输入，用来渲染一组 RepositoryItem 组件。每一个代码库的标识符会作为渲染列表的键存在。除此之外，每个代码库节点的 prop 会通过 JavaScript 的展开操作符传递给 RepositoryItem 组件。

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

import RepositoryItem from '../RepositoryItem';

import '../style.css';

const RepositoryList = ({ repositories }) =>
  repositories.edges.map(({ node }) => (
    <div key={node.id} className="RepositoryItem">
      <RepositoryItem {...node} />
    </div>
  ));

export default RepositoryList;
~~~~~~~~

> Finally, define the RepositoryItem component in the *src/Repository/RepositoryItem/index.js* file to render all the queried information about each repository. The file already uses a couple of stylings which you may have defined in a CSS file as suggested before. Otherwise, the component renders only static information for now.
最后，在 *src/Repository/RepositoryItem/index.js* 文件中，定义 RepositoryItem 组件，以便渲染查询到的每个代码库的信息。这个文件需要使用之前定义好的 CSS 文件中的一组样式。除此之外，现在组件只需渲染一些静态信息。

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

import Link from '../../Link';

import '../style.css';

const RepositoryItem = ({
  name,
  url,
  descriptionHTML,
  primaryLanguage,
  owner,
  stargazers,
  watchers,
  viewerSubscription,
  viewerHasStarred,
}) => (
  <div>
    <div className="RepositoryItem-title">
      <h2>
        <Link href={url}>{name}</Link>
      </h2>

      <div className="RepositoryItem-title-action">
        {stargazers.totalCount} Stars
      </div>
    </div>

    <div className="RepositoryItem-description">
      <div
        className="RepositoryItem-description-info"
        dangerouslySetInnerHTML={{ __html: descriptionHTML }}
      />
      <div className="RepositoryItem-description-details">
        <div>
          {primaryLanguage && (
            <span>Language: {primaryLanguage.name}</span>
          )}
        </div>
        <div>
          {owner && (
            <span>
              Owner: <a href={owner.url}>{owner.login}</a>
            </span>
          )}
        </div>
      </div>
    </div>
  </div>
);

export default RepositoryItem;
~~~~~~~~

> The anchor element to link to the repository is already extracted as a Link component. The Link component in the *src/Link/index.js* file could look like the following, to make it possible to open those URLs in an extra browser tab:
用来链接到代码库的锚点元素已经抽取为 Link 组件。为了在浏览器中能在新的标签页打开这些链接，Link 组件在 *src/Link/index.js* 可能看起来像下面这样：

{title="src/Link/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

const Link = ({ children, ...props }) => (
  <a {...props} target="_blank" rel="noopener noreferrer">
    {children}
  </a>
);

export default Link;
~~~~~~~~

> Once you restart your application, you should see a styled list of repositories with a name, URL, description, star count, owner, and the project's implementation language. If you can't see any repositories, check to see if your GitHub account has any public repositories. If it doesn't, then it's normal that nothing showed up. I recommend you make yourself comfortable with GitHub by creating a couple of repositories, both for the sake of learning about GitHub and to use this data to practice with this tutorial. Another way to create repositories for your own account is forking repositories from other people.
应用一旦启动，你应该能看到一组渲染好的有名称、URL、描述、star 数量、所有者以及项目实行语言信息的代码库列表。如果你看不到任何代码库，检查下你的 GitHub 账户有没有公开的代码库。如果没有，没有东西显示是正常的。我推荐你使用 GitHub 创建一些代码库，既能学习了解如何使用 GitHub，也能使用这些数据练习这个教程。另外一种在你的账户下创建代码库的方式是 fork 别人的。

> What you have done in the last steps of this section were pure React implementation, but this is only one opinionated way on how to structure components. The most important part from this section though happens in the Profile component. There, you introduced a Query component that takes a query as prop. Once the Query component renders, it executes the GraphQL query. The result of the query is made accessible as an argument within React's render props pattern.
到现在的所有实现都是纯粹的 React 实现，不过这只是一种方式去组织组件。其中最重要的部分是在 Profile 组件中，引入了 Query 组件，介绍一个 query prop。一旦 Query 组件渲染，它会执行 GraphQL 查询。查询的结果会通过 React 的 render props 模式的参数获取。

> ### Exercises:
### 练习

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/44ceb0482442eb07e56d134e6e1da8abefd68afe)
> * Read more about [queries with Apollo Client in React](https://www.apollographql.com/docs/react/essentials/queries.html)
> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/53Q6K3V)
* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/44ceb0482442eb07e56d134e6e1da8abefd68afe)
* 延伸阅读：[在 React 中使用 Apollo Client 进行查询](https://www.apollographql.com/docs/react/essentials/queries.html)
* 花三分钟的时间进行[测验](https://www.surveymonkey.com/r/53Q6K3V)

> ## Apollo Client Error Handling in React
## Apollo 客户端在 React 中的错误处理

> Before diving into GraphQL mutations in React with Apollo Client, this section should clarify error handling with Apollo in React. The error handling happens on two levels: the application level and the query/mutation level. Both can be implemented with the two cases that follow. On a query level, in your Profile component, you have access to the query `data` and `loading` properties. Apart from these, you can also access the `error` object, which can be used to show a conditional error message.

在探究如何在 React 中结合 Apollo 客户端进行 GraphQL 变更操作之前，这一节将阐明如何使用 Apollo 进行 React 中的错误处理。错误处理发生在两个级别：应用程序级别，查询/变更级别。这两种级别都可以通过以下两种情形处理。对于查询级别的情形，你可以在你的Profile组件中访问 `data` 和 `loading` 属性，除此之外，你也可以访问 `error` 对象，该对象可用于显示条件错误消息。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
...

import RepositoryList from '../Repository';
import Loading from '../Loading';
# leanpub-start-insert
import ErrorMessage from '../Error';
# leanpub-end-insert

...

const Profile = () => (
  <Query query={GET_REPOSITORIES_OF_CURRENT_USER}>
# leanpub-start-insert
    {({ data, loading, error }) => {
      if (
  93 > The Apollo Client setup can be completed in the top-level *src/index.js* file,) {
        return <ErrorMessage error={error} />;
      }
# leanpub-end-insert

      const { viewer } = data;

      if (loading || !viewer) {
        return <Loading />;
      }

      return <RepositoryList repositories={viewer.repositories} />;
    }}
  </Query>
);

export default Profile;
~~~~~~~~

> Whereas the ErrorMessage component from the *src/Error/index.js* could look like the following:

而 *src/Error/index.js* 文件中的 ErrorMessage 组件代码可能类似如下：

{title="src/Error/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

import './style.css';

const ErrorMessage = ({ error }) => (
  <div className="ErrorMessage">
    <small>{error.toString()}</small>
  </div>
);

export default ErrorMessage;
~~~~~~~~

> Try to change the name of a field in your query to something not offered by GitHub's GraphQL API, and observe what's rendered in the browser. You should see something like this: *Error: GraphQL error: Field 'viewers' doesn't exist on type 'Query'*. Or, if you simulate offline functionality, you'll see: *Error: Network error: Failed to fetch*. That's how errors can be separated into GraphQL errors and network errors. You can handle errors on a component or query level, but it will also help with mutations later. To implement error handling on an application level, install another Apollo package:

尝试将查询中的字段命名更改为 Github GraphQL API 未提供的内容，然后观察浏览器内渲染了什么内容。你会看到如下内容：*Error: GraphQL error: Field 'viewers' doesn't exist on type 'Query'*。或者，如果你模拟了离线功能，你将看到：*Error: Network error: Failed to fetch*。错误类型就是这样被划分为 GraphQL 错误和网络错误。你可以在组件或者查询级别上来处理这些错误，但是它也将对后续的变更操作有所帮助。在应用程序级别进行错误处理，需要安装另一个 Apollo 包：

{title="Command Line",lang="json"}
~~~~~~~~
npm install apollo-link-error --save
~~~~~~~~

> You can import it in your *src/index.js* file and create such an error link:

你可以把该包导入进 *src/index.js*文件中，并且创建一个如下所示的错误链接：

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { ApolloProvider } from 'react-apollo';
import { ApolloClient } from 'apollo-client';
import { HttpLink } from 'apollo-link-http';
# leanpub-start-insert
import { onError } from 'apollo-link-error';
# leanpub-end-insert
import { InMemoryCache } from 'apollo-cache-inmemory';

...

# leanpub-start-insert
const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) {
    // do something with graphql error
  }

  if (networkError) {
    // do something with network error
  }
});
# leanpub-end-insert
~~~~~~~~

> You could differentiate the error handling at the application level into development and production mode. During development, it might be sufficient to console log the errors to a developer console in the browser. In production mode, you can setup an error tracking service like [Sentry](https://sentry.io). It will teach you to identify bugs in a web dashboard more efficiently.

你可以将应用程序级别的错误分为开发和生产模式。在开发期间，将错误记录到浏览器中的开发人员控制台可能就足够了。在生产模式下，你可以构建一个错误跟踪服务，比如 [Sentry](https://sentry.io)。它将教你更有效地在网页控制面板中识别 bugs。

>Now you have two links in your application: `httpLink` and `errorLink`. To combine them for use with the Apollo Client instance, we'll download yet another useful package in the Apollo ecosystem that makes link compositions possible in the command line:

现在，在你的应用程序里有两个链接：`httpLink` 和 `errorLink`。为了将它们和 Apollo 客户端实例结合使用，我们需要在 Apollo 的生态系统中下载另一个有用的包，它可以实现链接组合，使用如下命令安装：

{title="Command Line",lang="json"}
~~~~~~~~
npm install apollo-link --save
~~~~~~~~

> And second, use it to combine your two links in the *src/index.js* file:

然后，在 *src/index.js* 文件中使用它来组合你的两个链接：

{title="src/index.js",lang="javascript"}
~~~~~~~~
...
import { ApolloClient } from 'apollo-client';
# leanpub-start-insert
import { ApolloLink } from 'apollo-link';
# leanpub-end-insert
import { HttpLink } from 'apollo-link-http';
import { onError } from 'apollo-link-error';
import { InMemoryCache } from 'apollo-cache-inmemory';

...

const httpLink = ...

const errorLink = ...

# leanpub-start-insert
const link = ApolloLink.from([errorLink, httpLink]);
# leanpub-end-insert

const cache = new InMemoryCache();

const client = new ApolloClient({
# leanpub-start-insert
  link,
# leanpub-end-insert
  cache,
});
~~~~~~~~

> That's how two or multiple links can be composed for creating an Apollo Client instance. There are several links developed by the community and Apollo maintainers that extend the Apollo Client with advanced functionality. Remember, it's important to understand that links can be used to access and modify the GraphQL control flow. When doing so, be careful to chain the control flow in the correct order. The `apollo-link-http` is called a **terminating link** because it turns an operation into a result that usually occurs from a network request. On the other side, the `apollo-link-error` is a **non-terminating link**. It only enhances your terminating link with features, since a terminating link has to be the last entity in the control flow chain.

以上就是如何组合两个或者多个链接来创建一个 Apollo 客户端实例。相关社区及 Apollo 维护人员开发了几个链接用以拓展 Apollo 客户端的高级功能。请记住，链接可被用于访问和修改 GraphQL 控制流程。当我们使用这些链接时，要注意以正确的顺序来链接控制流程。`apollo-link-http` 被叫做**终止链接**，因为它会将操作转换成一个通常发生在网络请求之后的返回结果。另一方面，`apollo-link-error` 是**非终止链接**，它仅仅是增强了终止链接的功能，因此终止链接必须是控制流链中的最后一个实体。

> ### Exercises:
### 练习：
> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/fa06945db4a933fe4a29c41f46fdc7034bceeb6e)
* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/fa06945db4a933fe4a29c41f46fdc7034bceeb6e)
> * Read more about [different Apollo Error types and error policies](https://www.apollographql.com/docs/react/features/error-handling.html)
* 延伸阅读：[不同的 Apollo 错误类型和错误策略](https://www.apollographql.com/docs/react/features/error-handling.html)
> * Read more about [Apollo Links](https://www.apollographql.com/docs/link/)
* 延伸阅读：[ Apollo 链接](https://www.apollographql.com/docs/link/)
> * Read more about [composable Apollo Links](https://www.apollographql.com/docs/link/composition.html)
* 延伸阅读：[可组合的 Apollo 链接](https://www.apollographql.com/docs/link/composition.html)
>* Implement the [apollo-link-retry](https://www.apollographql.com/docs/link/links/retry.html) in case a network request fails
* 实现[ Apollo 链接重试](https://www.apollographql.com/docs/link/links/retry.html)功能来处理网络请求失败
>* Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/53HLLFX)
* 花三分钟的时间进行[测试](https://www.surveymonkey.com/r/53HLLFX)

>## GraphQL Mutation with Apollo Client in React
## 在 React 中使用 Apollo 客户端变更操作
>The previous sections have taught you how to query data with React Apollo and the Apollo Client. In this section, you will learn about mutations. As in other applications before, you will implement starring a repository with GitHub's exposed `addStar` mutation.

上一节已经教会你如何使用 React Apollo 和 Apollo 客户端查询数据。在这一节中，你将学习到关于变更操作的知识。正如之前其他的应用一样，你将使用 Github 公开的 `addStar` 变更实现标记一个仓库。
>The mutation starts out with a variable to identify the repository to be starred. We haven't used a variable in Query component yet, but the following mutation works the same way, which can be defined in the *src/Repository/RepositoryItem/index.js* file.

该变更以变量来标识要加星的仓库。我们虽然还没有在 Query 组件里面使用过变量，但是以下的变更工作方式与之相同，它在 *src/Repository/RepositoryItem/index.js* 文件中定义如下。

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import gql from 'graphql-tag';
# leanpub-end-insert

...

# leanpub-start-insert
const STAR_REPOSITORY = gql`
  mutation($id: ID!) {
    addStar(input: { starrableId: $id }) {
      starrable {
        id
        viewerHasStarred
      }
    }
  }
`;
# leanpub-end-insert

...
~~~~~~~~

> The mutation definition takes the `id` variable as input for the `addStar` mutation. As before, you can decide what should be returned in case of a successful mutation. Now, you can use a Mutation component that represents the previously used Query component, but this time for mutations. You have to pass the mutation prop, but also a variable prop for passing the identifier for the repository.

`addStar` 变更使用变量 `id` 作为它的输入。和之前一样，你可以决定在变更操作成功后返回什么内容。现在，你可以使用 Mutation 组件来替代之前用到的 Query 组件，不同的是这次是用来进行变更操作。另外你必须传入相应的变更 prop，并且也需要为 variable prop 传入代码仓库的标识符。

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import gql from 'graphql-tag';
# leanpub-start-insert
import { Mutation } from 'react-apollo';
# leanpub-end-insert

...

const RepositoryItem = ({
# leanpub-start-insert
  id,
# leanpub-end-insert
  name,
  url,
  descriptionHTML,
  primaryLanguage,
  owner,
  stargazers,
  watchers,
  viewerSubscription,
  viewerHasStarred,
}) => (
  <div>
    <div className="RepositoryItem-title">
      <h2>
        ...
      </h2>

      <div>
# leanpub-start-insert
        <Mutation mutation={STAR_REPOSITORY} variables={{ id }}>
          {addStar => <div>{stargazers.totalCount} Star</div>}
        </Mutation>
# leanpub-end-insert
      </div>
    </div>

    <div className="RepositoryItem-description">
      ...
    </div>
  </div>
);
~~~~~~~~

>Note: The div element surrounding the Mutation component is there for other mutations you will implement in this section.

注意：div 元素所包裹的 Mutation 组件就是你将在本节实现的变更操作

>The `id` for each repository should be available due to previous query result. It has to be used as a variable for the mutation to identify the repository. The Mutation component is used like the Query component, because it implements the render prop pattern as well. The first argument is different, though, as it is the mutation itself instead of the mutation result. Use this function to trigger the mutation before expecting a result. Later, you will see how to retrieve the mutation result; for now, the mutating function can be used in a button element. In this case, it is already in a Button component:

由于之前的查询结果，所以每一个代码仓库的 `id` 应该都是可用的。它必须当做变更操作的变量来标识每一个代码仓库。Mutation 组件的使用方式和 Query 组件类似，因为它也运用了 render prop 模式。但是第一个参数是不同的，它是变更操作本身，而不是变更的结果。在期望得到结果之前，可以使用此函数来触发变更操作。随后，你就可以看到如何获取变更的结果；就目前而言，变更方法可以被用在 button 元素中。在下面这种情况，它已经运用在 Button 组件中了：

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
...

import Link from '../../Link';
# leanpub-start-insert
import Button from '../../Button';
# leanpub-end-insert

...

const RepositoryItem = ({ ... }) => (
  <div>
    <div className="RepositoryItem-title">
      ...

      <div>
        <Mutation mutation={STAR_REPOSITORY} variables={{ id }}>
          {(addStar) => (
# leanpub-start-insert
            <Button
              className={'RepositoryItem-title-action'}
              onClick={addStar}
            >
              {stargazers.totalCount} Star
            </Button>
# leanpub-end-insert
          )}
        </Mutation>
      </div>
    </div>

    ...
  </div>
);
~~~~~~~~

>The styled Button component could be implemented in the *src/Button/index.js* file. It's already extracted, because you will use it in this application later.

这个包含样式的 Button 组件可以在 *src/Button/index.js* 文件中被实现，并且已经被单独提取出来，因为在之后的应用程序中你将会用到它。

{title="src/Button/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

import './style.css';

const Button = ({
  children,
  className,
  color = 'black',
  type = 'button',
  ...props
}) => (
  <button
    className={`${className} Button Button_${color}`}
    type={type}
    {...props}
  >
    {children}
  </button>
);

export default Button;
~~~~~~~~

>Let's get to the mutation result which was left out before. Access it as a second argument in your child function of the render prop.

让我们来看看之前遗漏的变更结果的部分。可以把变更的结果作为 render prop 子函数的第二个参数来访问。

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
const RepositoryItem = ({ ... }) => (
  <div>
    <div className="RepositoryItem-title">
      ...

      <div>
        <Mutation mutation={STAR_REPOSITORY} variables={{ id }}>
# leanpub-start-insert
          {(addStar, { data, loading, error }) => (
# leanpub-end-insert
            <Button
              className={'RepositoryItem-title-action'}
              onClick={addStar}
            >
              {stargazers.totalCount} Star
            </Button>
          )}
        </Mutation>
      </div>
    </div>

    ...
  </div>
);
~~~~~~~~

> A mutation works like a query when using React Apollo. It uses the render prop pattern to access the mutation and the result of the mutation. The mutation can be used as a function in the UI. It has access to the variables that are passed in the Mutation component, but it can also override the variables when you pass them in a configuration object to the function (e.g. `addStar({ variables: { id } })`). That's a general pattern in React Apollo: You can specify information like variables in the Mutation component, or when you call the mutating function to override it.

在使用 React Apollo 时，变更执行起来和查询很像，它也是使用 render prop 模式来访问变更及其结果。变更可以是在 UI 中作为一个函数。它可以访问 Mutation 组件中传入的变量，并且它也可以传入一个配置对象来覆盖之前的变量(例如 `addStar({ variables: { id } })` )。这个是 React Apollo 中的通用模式：你可以在 Mutation 组件中指定 variables 等信息，或者在调用变更函数时覆盖它。
>Note that if you use the `viewerHasStarred` boolean from the query result to show either a "Star" or "Unstar" button, you can do it with a conditional rendering:

请注意，如何你使用查询结果中的 `viewerHasStarred` 布尔值来显示 "Star" 或者 "Unstar" 按钮，那么你可以用条件渲染来执行此操作：

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
const RepositoryItem = ({ ... }) => (
  <div>
    <div className="RepositoryItem-title">
      ...

      <div>
# leanpub-start-insert
        {!viewerHasStarred ? (
# leanpub-end-insert
          <Mutation mutation={STAR_REPOSITORY} variables={{ id }}>
            {(addStar, { data, loading, error }) => (
              <Button
                className={'RepositoryItem-title-action'}
                onClick={addStar}
              >
                {stargazers.totalCount} Star
              </Button>
            )}
          </Mutation>
# leanpub-start-insert
        ) : (
          <span>{/* Here comes your removeStar mutation */}</span>
        )}
# leanpub-end-insert

# leanpub-start-insert
      {/* Here comes your updateSubscription mutation */}
# leanpub-end-insert
      </div>
    </div>

    ...
  </div>
);
~~~~~~~~

> When you star a repository as above, the "Star" button disappears. This is what we want, because it means the `viewerHasStarred` boolean has been updated in Apollo Client's cache for the identified repository. Apollo Client was able to match the mutation result with the repository identifier to the repository entity in Apollo Client's cache, the props were updated, and the UI re-rendered. Yet, on the other side, the count of stargazers who have starred the repository isn't updated because it cannot be retrieved from GitHub's API. The count must be updated in the Apollo Client's cache. You will find out more about this topic in one of the following sections.

当你如上所诉地加星标注一个代码仓库时，"Star" 按钮将消失。这正是我们想要的，因为这意味着在 Apollo Client 的缓存中已经更新了所标识的仓库的 `viewerHasStarred` 布尔值。当 props 更新，UI 重新渲染时，Apollo Client 也能够将标识的代码仓库的变更结果与 Apollo Client 缓存中代码仓库实体相匹配。另一方面，stargazer 的数量并没有更新，这是因为它不能从 Github 的 API 中检索得到，stargazer 的数量必须在 Apollo Client 的缓存中更新。你将会后续章节学习到更多相关主题的知识。
> ### Exercises:
### 练习：
>* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/feb2b794392f9c5b1d2566ed39ad4ca5f650f194)
* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/feb2b794392f9c5b1d2566ed39ad4ca5f650f194)
>* Read more about [mutations with Apollo Client in React](https://www.apollographql.com/docs/react/essentials/mutations.html)
* 延伸阅读：[ Apollo Client 变更](https://www.apollographql.com/docs/react/essentials/mutations.html)
> * Implement other mutations in the RepositoryItem component
>  * Implement the `removeStar` mutation when the `viewerHasStarred` boolean is true
>  * Show a button with the watchers count which should be used to watch/unwatch a repository
>    * Implement the `updateSubscription` mutation from GitHub's GraphQL API to watch/unwatch a repository based on the `viewerSubscription` status
>* Invest three minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5GJQWXC)

* 在 Repository 组件中实现的其他变更
  * 当 `viewerHasStarred` 布尔值为true时，实现 `removeStar` 图标
  * 显示一个带有 watcher 数量的按钮，该按钮用于 watch 或者 unwatch 一个 repository
    * 使用 Github 的 GraphQL API 实现 `updateSubscription` 变更，它根据  `viewerSubscription` 状态实现 watch 或者 unwatch 一个 repository
* 花三分钟的时间进行[测试](https://www.surveymonkey.com/r/5GJQWXC)

> ## GraphQL Query/Mutation with Higher-Order Components in React

## 使用 React 高阶组件来完成 GraphQL 的查询/变更操作
>We've done Query and Mutation components from React Apollo to connect a data-layer (Apollo Client) with a view-layer (React). The Query component executes the query when it is rendered, whereas the Mutation component gives access to a function that triggers the mutation. Both components use the render props pattern to make the results accessible in their child functions.

我们已经在 React Apollo 中使用 Query 和 Mutation 组件完成了数据层（ Apollo Client ）和视图层（ React ）的连接。Query 组件在渲染后就会执行查询操作，而 Mutation 组件允许访问一个函数来触发变更。两个组件都使用的是 render props 模式，以便可以在子函数中访问结果。

>[Higher-Order Components (HOC)](https://www.robinwieruch.de/gentle-introduction-higher-order-components/) is a widely accepted alternative to React's render prop pattern. The React Apollo package implements a Higher-Order Component for queries and mutations as well, though the team behind Apollo doesn't advertise it, and even spoke in favor of render props as their first choice. Nonetheless, this section shows you the alternative, using a Higher-Order Component instead of a Render Prop, though this application will continue to use the render prop pattern afterward. If you already have access to the query result in the Profile component's arguments, there is no Query component needed in the component itself:

[高阶组件](https://www.robinwieruch.de/gentle-introduction-higher-order-components/)是 React 中被广泛接受的用来替换 render prop 模式的方式。React Apollo 包也为查询和变更提供了高阶组件，尽管 Apollo 团队没有宣传它，甚至支持 render props 模式作为他们的首选。尽管如此，本节还是会使用高阶组件来取代 render prop 模式，虽然这个应用之会在后续依然使用 render prop 模式。如果你可以在 Profile 组件的参数中访问查询的结果，那么组件本身并不需要使用 Query 组件：

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
const Profile = ({ data, loading, error }) => {
  if (error) {
    return <ErrorMessage error={error} />;
  }

  const { viewer } = data;

  if (loading || !viewer) {
    return <Loading />;
  }

  return <RepositoryList repositories={viewer.repositories} />;
};
~~~~~~~~

>There is no GraphQL involved here, because all you see is the pure view-layer. Instead, the data-layer logic is extracted into a Higher-Order Component. We import the `graphql` HOC from the React Apollo package in order to apply it on the Profile component, which takes the query definition as argument.

这里并没有涉及到 GraphQL，因为你所看到的只是纯视图层。而数据层的逻辑会被提取到一个高阶组件中。我们将 React Apollo 包中的 `graphql` 高阶组件导入，以便在 Profile 组件中使用它。这个组件会将查询的定义作为一个参数。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import gql from 'graphql-tag';
# leanpub-start-insert
import { graphql } from 'react-apollo';
# leanpub-end-insert

...

const GET_REPOSITORIES_OF_CURRENT_USER = gql`
  {
    viewer {
      ...
    }
  }
`;

const Profile = ({ data, loading, error }) => {
  ...
};

# leanpub-start-insert
export default graphql(GET_REPOSITORIES_OF_CURRENT_USER)(Profile);
# leanpub-end-insert
~~~~~~~~

> I find the HOC approach cleaner than the render props, because it co-locates both the data-layer and view-layer instead of inserting the one into the other. However, the team behind Apollo made the decision to favor render props instead. While I find the HOC approach more concise, the render prop pattern comes with its own advantages for mutating and querying data. For instance, imagine a query depends on a prop used as variable. It would be cumbersome to access the incoming prop in a statically-defined Higher-Order Component, but it can be dynamically used in a render prop because it is used within the Profile component where the props are naturally accessible. Another advantage is the power of composition for render props, which is useful when one query depends on the result of another. It can be achieved with HOCs as well, but again, it is more cumbersome. It boils down to seemingly never ending "Higher-Order Components vs Render Props" discussions.

我发现使用 HOC 比 render props 更清晰，因为它将数据层和视图层协同使用，而不是在一个组件里面使用另一个。然而，Apollo 的支持团队还是更倾向于使用render props 的模式。虽然我发现 HOC 方法更简洁，但是 render prop 模式在变更和查询方面，也有它自己的优势。例如，假设一个查询要使用一个 prop 作为变量。在静态定义的高阶组件中访问传入的 prop 会很麻烦，但是它可以在 render prop 中动态的使用，因为它在 Profile 组件中可以很自然的访问 prop。另一个优势就是 render props 的组合能力，当一个查询依赖于另一个的查询结果时，这种方式会很实用。这种情况虽然也能使用 HOC 来实现，但确实会很麻烦。“高阶组件对比 Render Props” 的讨论似乎永远不会结束。
> ### Exercises:
### 练习：
>* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/694cc4ec8f0d3546c13e0a32cd1f18ba9a990713)
* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/694cc4ec8f0d3546c13e0a32cd1f18ba9a990713)
> * Come up with your own opinion about the advantages and disadvantages of using a Higher-Order Component or Render Prop
* 对使用高阶组件或者 Render Prop 的优缺点提出自己的看法
> * Try to implement one of your mutations with a Higher-Order Component
* 尝试使用高阶组件去实现一个变更
> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5G6QPLY)
* 花三分钟的时间进行[测试](https://www.surveymonkey.com/r/5G6QPLY)
> ## Local State Management with Apollo Client in React
## Apollo 客户端在 React 中使用本地状态管理
>Let's get back to the Repository component. You have experienced that the `viewerHasStarred` boolean updates in the Apollo Client's cache after a mutation was successful. That's great, because Apollo Client handles this for you, based on the mutation result. If you have followed the exercises of the mutation section, you should probably see something like a toggling "Star" and "Unstar" label for the button. All of this happens because you returned the `viewerHasStarred` boolean in your mutation result. Apollo Client is clever enough to update the repository entity, which is normalized and accessible in the cache. That's powerful default behavior, isn't it? You don't need to handle the local state management yourself, since Apollo Client figures it out for you as long as you provide useful information in the mutation's result.

让我们回到 Repository 组件。你已经见过了在变更成功后更改 Apollo 客户端缓存中 `viewerHasStarred` 的值。这里的亮点是 Apollo 客户端会根据变更的结果为你做这样的处理。如果你已经完成了变更操作那一章节的练习，你大概可以看到按钮带有类似于 "Star" 和 "Unstar" 的标签了。你可以看到它们是因为你在变更结果中返回了 `viewerHasStarred`。Apollo 客户端可以很智能地去更新缓存中代码仓库实体。这是一个强大的默认行为，不是吗？你不必自己处理组件本地状态管理，因为只要你在变更操作的结果中提供有用的信息，Apollo 客户端就会为你处理这些问题。

>Apollo Client doesn't update the count of stars after the mutation, though. Normally, it is assumed that the count of stars increments by one when it is starred, with the opposite for unstarring. Since we don't return a count of stargazers in the mutation result, you have to handle the update in Apollo Client's cache yourself. Using Apollo Client's `refetchQueries` option is the naive approach for a mutation call, or a Mutation component to trigger a refetch for all queries, where the query result might be affected by the mutation. But that's not the best way to deal with this problem. It costs another query request to keep the data consistent after a mutation. In a growing application, this approach will eventually become problematic. Fortunately, the Apollo Client offers other functionalities to read/write manually from/to the cache locally without more network requests. The Mutation component offers a prop where you can insert update functionality that has access to the Apollo Client instance for the update mechanism.

Apollo 客户端不会在变更成功后更新 star 的数量，通常情况下，假设 star 的数量在被点加星标识的时候增加一个，相反条件下减一个。因为我们没有在变更的结果中返回 stargazer 的数量，所以你必须自己去更新 Apollo 客户端中的缓存。对于变更操作来说，使用 Apollo 客户端中的 refetchQueries 选项或者使用 Mutation 组件重新触发所有查询是很天真的做法，查询的结果可能会因受到变更的影响而改变。但是这不是最好的处理方式，因为这种方式会在变更之后使用了一个查询来保持数据的一致性。在一个不断壮大的应用程序中，这种方法最终会成为一个问题。幸运的是，Apollo 客户端提供了其它的功能，让我们可以在不使用更多网络请求的情况下，在本地读写操作缓存。Mutation 组件提供了一个 prop，你可以通过这个 prop 插入一个可以访问 Apollo 客户端实例的更新功能，来实现更新机制。
> Before implementing the update functionality for the local state management, let's refactor another piece of code that will be useful for a local state update mechanism. The query definition next to your Profile component has grown to several fields with multiple object nestings. Previously, you learned about GraphQL fragments, and how they can be used to split parts of a query to reuse later. Next, we will split all the field information you used for the repository's node. You can define this fragment in the *src/Repository/fragments.js* file to keep it reusable for other components.

在实现这个更新功能之前，让我们重构一段对本地状态更新机制更有用的代码。在 Profile 组件中运用到的查询定义已经增长到多个字段并且有多个嵌套对象。之前，你已经学习了 GraphQL 片段，以及它们如何用于拆分查询以便在以后重用。接下来，我们将拆分所有用于代码仓库的节点的字段信息，你可以在 *src/Repository/fragments.js* 文件中定义这个片段，以便它可以被其他组件重用。

{title="src/Repository/fragments.js",lang="javascript"}
~~~~~~~~
import gql from 'graphql-tag';

const REPOSITORY_FRAGMENT = gql`
  fragment repository on Repository {
    id
    name
    url
    descriptionHTML
    primaryLanguage {
      name
    }
    owner {
      login
      url
    }
    stargazers {
      totalCount
    }
    viewerHasStarred
    watchers {
      totalCount
    }
    viewerSubscription
  }
`;

export default REPOSITORY_FRAGMENT;
~~~~~~~~

>You split this partial query (fragment), because it is used more often in this application in the next sections for a local state update mechanism, hence the previous refactoring.

你可以拆分这部分的查询（片段），因为之前的重构，它在后续章节会更频繁地用于本地状态的管理机制。
>The fragment shouldn't be imported directly from the *src/Repository/fragments.js* path to your Profile component, because the *src/Repository/index.js* file is the preferred entry point to this module.

这个片段不应该直接的从 *src/Repository/fragments.js* 路径导入到 Profile 组件，因为 *src/Repository/index.js* 文件才是此模块的首选入口点。

{title="src/Repository/index.js",lang="javascript"}
~~~~~~~~
import RepositoryList from './RepositoryList';
# leanpub-start-insert
import REPOSITORY_FRAGMENT from './fragments';
# leanpub-end-insert

# leanpub-start-insert
export { REPOSITORY_FRAGMENT };
# leanpub-end-insert

export default RepositoryList;
~~~~~~~~

>Finally, import the fragment in the Profile component's file to use it again.

最后，在 Profile 组件中导入这个片段来重复使用。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
import RepositoryList, { REPOSITORY_FRAGMENT } from '../Repository';
# leanpub-end-insert
import Loading from '../Loading';
import ErrorMessage from '../Error';

const GET_REPOSITORIES_OF_CURRENT_USER = gql`
  {
    viewer {
      repositories(
        first: 5
        orderBy: { direction: DESC, field: STARGAZERS }
      ) {
        edges {
          node {
# leanpub-start-insert
            ...repository
# leanpub-end-insert
          }
        }
      }
    }
  }

# leanpub-start-insert
  ${REPOSITORY_FRAGMENT}
# leanpub-end-insert
`;

...
~~~~~~~~

>The refactoring is done. Your query is now more concise, and the fragment in its natural repository module can be reused for other places and functionalities. Next, use Mutation component's `update` prop to pass a function which will update the local cache eventually.

重构到此就已经完成了。现在你的查询部分的代码更简洁了。位于代码仓库模块中的片段，也可以在其他的位置和功能重复使用。下一步，我们会使用 Mutation 组件中的 `update` prop 传递一个最终会更新本地缓存的函数。

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const updateAddStar = (client, mutationResult) => {
  ...
};
# leanpub-end-insert

const RepositoryItem = ({ ... }) => (
  <div>
    <div className="RepositoryItem-title">
      ...

      <div>
        {viewerHasStarred ? (
          ...
        ) : (
          <Mutation
            mutation={STAR_REPOSITORY}
            variables={{ id }}
# leanpub-start-insert
            update={updateAddStar}
# leanpub-end-insert
          >
            ...
          </Mutation>
        )}
      </div>
    </div>

    ...
  </div>
);

export default RepositoryItem;
~~~~~~~~

>The function is extracted as its own JavaScript variable, otherwise ends up too verbose in the RepositoryItem component when keeping it inlined in the Mutation component. The function has access to the Apollo Client and the mutation result in its argument, and you need both to update data so you can destructure the mutation result in the function signature. If you don't know how the mutation result looks like, check the `STAR_REPOSITORY` mutation definition again, where you defined all fields that should appear in the mutation result. For now, the `id` of the repository to be updated is the important part.

这个函数被单独提取成一个变量，否则当它内联到 Mutation 组件中时，RepositoryItem 组件会变得过于亢长。该函数可以访问到 Apollo 客户端，及其参数中变更操作的结果，你需要更新数据，以便在函数签名中解构变更的结果。如果你不知道变更结果的结构，请再次查看 `STAR_REPOSITORY` 变更的定义，这里面有所有被定义的字段并且会出现在变更操作的结果中。目前，要更新的代码仓库的 `id` 是重要部分。

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
const updateAddStar = (
# leanpub-start-insert
  client,
  { data: { addStar: { starrable: { id } } } },
# leanpub-end-insert
) => {
  ...
};
~~~~~~~~

>You could have passed the `id` of the repository to the `updateAddStar()` function, which was a higher-order function in the Mutation component's render prop child function. You already have access to the repository's identifier in the Repository component.

你可以在 `updateAddStar()` 函数中传入代码仓库的 `id`，这个函数是 Mutation 组件的 render prop 子函数中的一个高阶函数。你已经可以在 Repository 组件中访问代码仓库的唯一标识了。

>Now comes the most exciting part of this section. You can use the Apollo Client to read data from the cache, but also to write data to it. The goal is to read the starred repository from the cache, which is why we need the `id` to increment its stargazers count by one and write the updated repository back to the cache. You got the repository by its `id` from the cache by extracting the repository fragment. You can use it along with the repository identifier to retrieve the actual repository from Apollo Client's cache without querying all the data with a naive query implementation.

现在到了本节最激动人心的部分了。你可以使用 Apollo 客户端从缓存中读取数据，也可以向其中写入数据。我们的目标是从缓存中读取已经加星标注的代码仓库，这就是为什么我们需要 `id` 来找到相应的代码仓库，将它的 star 数量加1并写入到缓存中。你可以通过提取出来的代码仓库片段来获得它的缓存数据，你可以将该片段和代码仓库的唯一标识一起使用，从而在 Apollo 客户端的缓存中，检索实际需要的代码仓库数据，而无需使用基本的查询来检索所有的数据。

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
import REPOSITORY_FRAGMENT from '../fragments';
# leanpub-end-insert
import Link from '../../Link';
import Button from '../../Button';

...

const updateAddStar = (
  client,
  { data: { addStar: { starrable: { id } } } },
) => {
# leanpub-start-insert
  const repository = client.readFragment({
    id: `Repository:${id}`,
    fragment: REPOSITORY_FRAGMENT,
  });
# leanpub-end-insert

  // update count of stargazers of repository

  // write repository back to cache
};
~~~~~~~~

>The Apollo Client's cache that you set up to initialize the Apollo Client normalizes and stores queried data. Otherwise, the repository would be a deeply nested entity in a list of repositories for the query structure used in the Profile component. Normalization of a data structure makes it possible to retrieve entities by their identifier and their GraphQL `__typename` meta field. The combination of both is the default key, which is called a [composite key](https://en.wikipedia.org/wiki/Compound_key), to read or write an entity from or to the cache. You may find out more about changing this default composite key in the exercises of this section.

你为初始化 Apollo 客户端而创建的 Apollo 客户端缓存，会规范化并存储查询到的数据。否则，对于 Profile 组件中使用的查询结构而言，代码仓库将会是代码仓库列表中的一个深层嵌套的实体。数据结构的标准化使得可以通过它们的标识符和 GraphQL 的 `__typename` 元字段来检索实体。两者的组合是默认键，这个被称为[复合键](https://en.wikipedia.org/wiki/Compound_key)，它用于从缓存中读取和写入实体。你将会在本节的练习中找到更多关于更改默认复合键的信息。
>Furthermore, the resulting entity has all properties specified in the fragment. If there is a field in the fragment not found on the entity in the cache, you may see the following error message: *Can't find field __typename on object ...*. That's why we use the identical fragment to read from the local cache to query the GraphQL API.

此外，生成的实体具有片段中涉及的所有属性。如何在缓存中无法找到该实体上的某些字段，你会看到以下的错误：*Can't find field __typename on object ...*。这就是为什么我们用相同的片段，来读取本地缓存用于查询 GraphQL API 。
> After you have retrieved the repository entity with a fragment and its composite key, you can update the count of stargazers and write back the data to your cache. In this case, increment the number of stargazers.

在你使用片段和它的复合键来检索 repository 实体之后，你可以更新 stargazer 数量，并把更新后的数据写入到缓存中。在这种情况下，就能够增加 stargazer 的数量。

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
const updateAddStar = (
  client,
  { data: { addStar: { starrable: { id } } } },
) => {
  const repository = client.readFragment({
    id: `Repository:${id}`,
    fragment: REPOSITORY_FRAGMENT,
  });

# leanpub-start-insert
  const totalCount = repository.stargazers.totalCount + 1;
# leanpub-end-insert

# leanpub-start-insert
  client.writeFragment({
    id: `Repository:${id}`,
    fragment: REPOSITORY_FRAGMENT,
    data: {
      ...repository,
      stargazers: {
        ...repository.stargazers,
        totalCount,
      },
    },
  });
# leanpub-end-insert
};
~~~~~~~~

>Let's recap all three steps here. First, you have retrieved (read) the repository entity from the Apollo Client using an identifier and the fragment; second, you updated the information of the entity; and third, you wrote back the data with updated information, but kept all remaining information intact using the JavaScript spread operator. This is a manual update mechanism that can be used when a mutation is missing data.

让我们回顾一下这三个步骤。首先，你使用唯一标识和片段从 Apollo 客户端中检索（读取）了代码仓库实体；第二步，你更新这个实体的数据信息；第三步，你将这个更新信息写回到缓存中，同时通过使用 JavaScript 的拓展运算符保证了所有其他信息的完整性。这就是一种手动更新机制，可以在变更操作缺失数据时使用。
>It is a good practice to use an identical fragment for all three parts: the initial query, the `readFragment()`, and `writeFragment()` cache method. Your data structure for the entity stays consistent in your cache. For instance, if you forget to include a property defined by the fragment's fields in data object of the `writeFragment()` method, you get a warning: *Missing field __typename in ...*.

对以下的三个部分，使用相同片段是一个好的实践，这三个部分包括：初始化的查询，`readFragment（）` 以及 `writeFragment（）` 方法。这使得你的实体的数据结构在缓存中始终保持一致。例如，如果在 `writeFragment（）` 方法中的数据对象里，你忘了包含片段所定义的字段，你将会得到一个警告：*Missing field __typename in ...*。
>On an implementation level, you learned about extracting fragments from a query or mutation. Fragments allow you to define your shared entities by GraphQL types. You can reuse those in your queries, mutations or local state management methods to update the cache. On a higher level, you learned that Apollo Client's cache normalizes your data, so you can retrieve entities that were fetched with a deeply nested query using their type and identifier as composite key. Without it, you'd have to perform normalizations for all the fetched data before putting it in your store/state.

在代码实现级别上，你学到了从查询和变更中提取片段，这些片段允许你通过 GraphQL 类型来定义共享实体。你可以在查询，变更或者组件本地状态管理方法中重用这些片段来更新缓存。在更高的级别上，你学到了用 Apollo 客户端的缓存来标准化你的数据，从而你可以使用实体的类型和他们的组合键，来从深层嵌套的查询结果中，检索到你所期望的实体数据。如果没有这个缓存来标准化，在将所有获取的数据存放到 store 或者 state 之前，你必须对这些数据进行标准化。
>### Exercises:
### 练习：
>* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/24bb647ac94f1af1c52b61e41cebba6a6fd95f4f)
* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/24bb647ac94f1af1c52b61e41cebba6a6fd95f4f)
>* Read more about [Local State Management in Apollo Client](https://www.apollographql.com/docs/react/essentials/local-state.html)
* 延伸阅读：[ Apollo 客户端组件本地状态管理](https://www.apollographql.com/docs/react/essentials/local-state.html)
>* Read more about [Fragments in Apollo Client](https://www.apollographql.com/docs/react/advanced/fragments.html)
* 延伸阅读：[ Apollo 客户端中片段](https://www.apollographql.com/docs/react/advanced/fragments.html)

>* Implement local cache updates for all the other mutations from the previous exercises
>  * Implement the identical local cache update, but with decreasing the count of stargazers, for your `removeStar` mutation
>  * Implement the local cache update for the `updateSubscription` mutation
>  * You will see in the next section a working solution for it
>* Read more about [Caching in Apollo Client and the composite key to identify entities](https://www.apollographql.com/docs/react/advanced/caching.html)
>* Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5BSDXF7)

* 为之前练习中的所有其他的变更操作实现更新本地缓存
  * 为 `removeStar` 变更实现相同的本地缓存更新，不过是去减少 stargazer 的数量
  * 为 `updateSubscription` 变更实现本地缓存更新
  * 你将会在下一节看到一个可行的解决方案
* 延伸阅读：[ Apollo 客户端中的缓存以及用于标识实体的组合键](https://www.apollographql.com/docs/react/advanced/caching.html)
* 花三分钟的时间进行[测试](https://www.surveymonkey.com/r/5BSDXF7)

> ## Apollo Client Optimistic UI in React 
## React 中的 Apollo Client 乐观 UI

> We've covered the basics, so now it's time for the advanced topics. One of those topics is the optimistic UI with React Apollo, which makes everything onscreen more synchronous. For instance, when liking a post on Twitter, the like appears immediately. As developers, we know there is a request that sends the information for the like to the Twitter backend. This request is asynchronous and doesn't resolve immediately with a result. The optimistic UI immediately assumes a successful request and mimics the result of such request for the frontend so it can update its UI immediately, before the real response arrives later. With a failed request, the optimistic UI performs a rollback and updates itself accordingly. Optimistic UI improves the user experience by omitting inconvenient feedback (e.g. loading indicators) for the user. The good thing is that React Apollo comes with this feature out of the box.

我们已经介绍了基础知识，现在是学习进阶主题的时候了。其中一个主题是 React Apollo 的乐观 UI，它会让屏幕上的所有内容更加同步。举个例子，当在 Twitter 给一条推文点赞时，这个赞立刻就显示出来了。作为开发者，我们知道这里会有一个请求把点赞的信息发送给后端。这个请求是异步的并且不会立刻决议结果。乐观 UI 立即假定一个成功的请求并模拟前端请求的结果，因此在真实的响应返回之前，它可以立刻更新其 UI。如果请求失败，乐观 UI 会执行回滚并相应地更新自身。乐观 UI 通过省略用户的不方便反馈（例如加载指示符）来提高用户体验。好消息是 React Apollo 提供了这个功能并且开箱即用。

> In this section, you will implement an optimistic UI for when a user clicks the watch/unwatch mutation you implemented in a previous exercise. If you haven't, it's time to implement it now, or you can substitute it with the star or unstar mutation. Either way, completing the optimistic UI behavior for all three mutations is the next exercise. For completeness, this is a possible implementation of the watch mutation as a button next to the "Star"/"Unstar" buttons. First, the mutation:

在本节中，你将实现一个乐观 UI，以便用户点击你在上个练习中实现的 watch 或 unwatch 变更。如果你还没有，现在是时候实现它，或者你可以用 star 或 unstar 变更来代替。无论哪种方式，完成这三个变更操作的乐观 UI 行为就是下一个练习。为了完整起见，可以将 watch 变更实现为 "Star"/"Unstar" 按钮旁边的按钮。首先，变更如下：

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const WATCH_REPOSITORY = gql`
  mutation ($id: ID!, $viewerSubscription: SubscriptionState!) {
    updateSubscription(
      input: { state: $viewerSubscription, subscribableId: $id }
    ) {
      subscribable {
        id
        viewerSubscription
      }
    }
  }
`;
# leanpub-end-insert
~~~~~~~~

> Second, the usage of the mutation with a Mutation render prop component:

其次，Mutation render prop 组件中 mutation 的用法：

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const VIEWER_SUBSCRIPTIONS = {
  SUBSCRIBED: 'SUBSCRIBED',
  UNSUBSCRIBED: 'UNSUBSCRIBED',
};
# leanpub-end-insert

# leanpub-start-insert
const isWatch = viewerSubscription =>
  viewerSubscription === VIEWER_SUBSCRIPTIONS.SUBSCRIBED;
# leanpub-end-insert

# leanpub-start-insert
const updateWatch = () => {
  ...
};
# leanpub-end-insert

const RepositoryItem = ({ ... }) => (
  <div>
    <div className="RepositoryItem-title">
      ...

      <div>
        ...

# leanpub-start-insert
        <Mutation
          mutation={WATCH_REPOSITORY}
          variables={{
            id,
            viewerSubscription: isWatch(viewerSubscription)
              ? VIEWER_SUBSCRIPTIONS.UNSUBSCRIBED
              : VIEWER_SUBSCRIPTIONS.SUBSCRIBED,
          }}
          update={updateWatch}
        >
          {(updateSubscription, { data, loading, error }) => (
            <Button
              className="RepositoryItem-title-action"
              onClick={updateSubscription}
            >
              {watchers.totalCount}{' '}
              {isWatch(viewerSubscription) ? 'Unwatch' : 'Watch'}
            </Button>
          )}
        </Mutation>
# leanpub-end-insert

        ...
      </div>
    </div>

    ...
  </div>
);
~~~~~~~~

> And third, the missing update function that is passed to the Mutation component:

第三，传递给 Mutation 组件缺少的更新函数：

{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const updateWatch = (
  client,
  {
    data: {
      updateSubscription: {
        subscribable: { id, viewerSubscription },
      },
    },
  },
) => {
  const repository = client.readFragment({
    id: `Repository:${id}`,
    fragment: REPOSITORY_FRAGMENT,
  });

  let { totalCount } = repository.watchers;
  totalCount =
    viewerSubscription === VIEWER_SUBSCRIPTIONS.SUBSCRIBED
      ? totalCount + 1
      : totalCount - 1;

  client.writeFragment({
    id: `Repository:${id}`,
    fragment: REPOSITORY_FRAGMENT,
    data: {
      ...repository,
      watchers: {
        ...repository.watchers,
        totalCount,
      },
    },
  });
};
# leanpub-end-insert
~~~~~~~~

> Now let's get to the optimistic UI feature. Fortunately, the Mutation component offers a prop for the optimistic UI strategy called `optimisticResponse`. It returns the same result, which is accessed as argument in the function passed to the `update` prop of the Mutation component. With a watch mutation, only the `viewerSubscription` status changes to subscribed or unsubscribed. This is an optimistic UI.

现在让我们来看一下乐观 UI 特性。幸运的是，Mutation 组件为乐观 UI 策略提供了一个叫做 `optimisticResponse` 的属性。它返回相同的结果，该结果在传递给 Mutation 组件的 `update` 属性的函数中作为参数访问。对于 watch 变更，只有 `viewerSubscription` 状态更改为“已订阅”或者“未订阅”。这就是一个乐观 UI。


{title="src/Repository/RepositoryItem/index.js",lang="javascript"}
~~~~~~~~
const RepositoryItem = ({ ... }) => (
  <div>
    <div className="RepositoryItem-title">
      ...

      <div>
        ...

        <Mutation
          mutation={WATCH_REPOSITORY}
          variables={{
            id,
            viewerSubscription: isWatch(viewerSubscription)
              ? VIEWER_SUBSCRIPTIONS.UNSUBSCRIBED
              : VIEWER_SUBSCRIPTIONS.SUBSCRIBED,
          }}
# leanpub-start-insert
          optimisticResponse={{
            updateSubscription: {
              __typename: 'Mutation',
              subscribable: {
                __typename: 'Repository',
                id,
                viewerSubscription: isWatch(viewerSubscription)
                  ? VIEWER_SUBSCRIPTIONS.UNSUBSCRIBED
                  : VIEWER_SUBSCRIPTIONS.SUBSCRIBED,
              },
            },
          }}
# leanpub-end-insert
          update={updateWatch}
        >
          ...
        </Mutation>

        ...
      </div>
    </div>

    ...
  </div>
);
~~~~~~~~

> When you start your application and watch a repository, the "Watch" and "Unwatch" label of the button changes immediately after clicking it. This is because the optimistic response arrives synchronously, while the real response is pending and resolves later. Since the `__typename ` meta field comes with every Apollo request, include those as well.

当你启动应用程序并 watch 一个代码库时，按钮的 "Watch" 和 "Unwatch" 标签在会在点击之后立刻修改。这是因为乐观响应是同步到达的，而真正的响应还在等待并且稍后才决议。由于 `__typename ` 元字段会随每一次 Apollo 请求一起提供，因此也包括这些。  

> An additional benefit of the optimistic response is that it makes the count of watchers updates optimistic, too. The function used in the `update` prop is called twice now, the first time with the optimistic response, and the second with a response from GitHub's GraphQL API. It makes sense to capture identical information in the optimistic response expected as a mutation result in the function passed to the `update` prop of the Mutation component. For instance, if you don't pass the `id` property in the `optimisticResponse` object, the function passed to the `update` prop throws an error, because it can't retrieve the repository from the cache without an identifier.

乐观响应的另一个好处是让观察者的更新计数更变得乐观。在 `update` 属性中使用的函数现在被调用了两次：第一次调用乐观响应，第二次调用 GitHub 的 GraphQL API。在乐观响应中捕获相同的信息作为变更结果传递给 Mutation 组件的 `update` 属性的函数是有意义的。例如，如果你没有在 `optimisticResponse` 对象中传递 `id` 属性，那么传递给 `update` 属性的函数会抛出一个错误，因为它无法在没有标识符的情况下从缓存中检索代码库。

> At this point, it becomes debatable whether or not the Mutation component becomes too verbose. Using the Render Props pattern co-locates the data layer even more to the view-layer than Higher-Order Components. One could argue it doesn't co-locate the data-layer, but inserts it into the view-layer. When optimizations like the `update` and `optimisticResponse` props are put into the Render Prop Component, it can become too verbose for a scaling application. I advise using techniques you've learned as well as your own strategies to keep your source code concise. I see four different ways to solve this issue:

此时，Mutation 组件是否变得太冗长也变得有争议。使用 Render Props 模式比使用高阶组件更能将数据层共同定位到视图层。有人可能会说它没有共同定位数据层，而是将其插入了视图层中。当一些像 `update` 和 `optimisticResponse` 属性这样的优化被放入 Render Prop 组件时，对于一个缩放应用程序来说它可能变得过于冗长。我建议你使用自己学到的技巧以及自己的策略来保持源代码的简洁。我认为有四种不同方法可以解决这个问题：


> * Keep the declarations inlined (see: `optimisticUpdate`)

* 保持声明内联。（见：`optimisticUpdate`）

> * Extracting the inlined declarations as variable (see: `update`).

* 将内联的声明提取为变量。（见：`update`）

> * Perform a combination of 1 and 2 whereas only the most verbose parts are extracted

* 执行 1 和 2 的组合，但是只提取最冗长的部分。

> * Use Higher-Order Components instead of Render Props to co-locate data-layer, instead of inserting it in the view-layer

* 用高阶组件代替 Render Props 来共同定位数据层，而不是将其插入视图层。

> The first three are about **inserting** a data-layer into the view-layer, while the last is about **co-locating** it. Each comes with drawbacks. Following the second way, you might yourself declaring functions instead of objects, or higher-order functions instead of functions because you need to pass arguments to them. With the fourth, you could encounter the same challenge in keeping HOCs concise. There, you could use the other three ways too, but this time in a HOC rather than a Render Prop.

前三个是关于将数据层 **插入** 视图层，而最后一个是关于 **共同定位**，每一个都有缺点。按照第二种方式，你可以自己声明函数而不是对象，或者声明高阶函数而不是函数，因为你需要将参数传递给它们。第四种方式，你可能会在保持 HOC 简洁方面遇到相同的挑战。在这个问题上，你也可以使用其他三种方式，但这一次是在 HOC 而不是 Render Prop 中。

> ### Exercises:
### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/2fd3f5bad7668655feebe876db7bc9247905c475)

* [查看本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/2fd3f5bad7668655feebe876db7bc9247905c475)

> * Throttle your internet connection (often browsers offers such functionality) and experience how the `optimisticResponse` takes the `update` function into account even though the request is slow

* 限制你互联网连接的流量（通常浏览器会提供此类功能）并体验一下 `optimisticResponse` 是如何考虑 `update` 函数的，即使请求速度很慢。

> * Try different ways of co-locating or inserting your data-layer with render props and higher-order components

* 尝试使用 render props 和高阶组件对数据层进行共同定位或者插入的不同方法。 

> * Implement the optimistic UIs for the star and unstar mutations

* 为 star 和 unstar 变更实现乐观 UI

> * Read more about [Apollo Optimistic UI in React with GraphQL](https://www.apollographql.com/docs/react/features/optimistic-ui.html)

延伸阅读：[React 中使用 GraphQL 的 Apollo 乐观 UI](https://www.apollographql.com/docs/react/features/optimistic-ui.html) 的更多信息

> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5B6D8BX)

* 花三分钟的时间进行[测验](https://www.surveymonkey.com/r/5B6D8BX)

> ## GraphQL Pagination with Apollo Client in React
## React 中 Apollo 客户端的 GraphQL 分页

> Finally, you are going to implement another advanced feature when using a GraphQL API called **pagination**. In this section, you implement a button that allows successive pages of repositories to be queries, a simple "More" button rendered below the list of repositories in the RepositoryList component. When is clicked, another page of repositories is fetched and merged with the previous list as one state into Apollo Client's cache.

最后，你将使用一个叫做 **pagination** 的 GraphQL API，来实现另一个高级特性。在这一节中，你会实现一个按钮，它允许代码库中的后续页面被查询，在 RepositoryList 组件中，一个简单的 "More" 按钮渲染在代码库列表之下。当它被点击时，会获取代码库的另一个页面，并将其与上一个列表合并，作为一个状态保存到 Apollo 客户端的缓存中。


> First, extend the query next for your Profile component with the necessary information to allow pagination for the list of repositories:

首先，使用必要的信息扩展 Profile 组件的查询，以便对代码库列表进行分页：

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
const GET_REPOSITORIES_OF_CURRENT_USER = gql`
# leanpub-start-insert
  query($cursor: String) {
# leanpub-end-insert
    viewer {
      repositories(
        first: 5
        orderBy: { direction: DESC, field: STARGAZERS }
# leanpub-start-insert
        after: $cursor
# leanpub-end-insert
      ) {
        edges {
          node {
            ...repository
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
  ${REPOSITORY_FRAGMENT}
`;
~~~~~~~~

> The `endCursor` can be used as `$cursor` variable when fetching the next page of repositories, but the `hasNextPage` can disable the functionality (e.g. not showing the "More" button) to fetch another page. The initial request to fetch the first page of repositories will have a `$cursor` variable of `undefined`, though. GitHub's GraphQL API will handle this case gracefully and return the first items from the list of repositories without considering the `after` argument. Every other request to fetch more items from the list will send a defined `after` argument with the cursor, which is the `endCursor` from the query.

当获取代码库的下一页时，`endCursor` 可以当做 `$cursor` 变量来使用，但是 `hasNextPage` 可以禁用获取其他页面的功能（例如：不显示 "More" 按钮）。不过，获取代码库首页的初始请求将会有一个值为 `undefined` 的 `$cursor` 变量。GitHub 的 GraphQL API 可以优雅地处理这种情况，返回代码库列表的第一项，而不用考虑 `after` 参数。每一次从列表中获取更多项的请求，都会发送一个使用游标定义的 `after` 参数，即查询中的 `endCursor`。

> Now we have all information to fetch more pages of repositories from GitHub's GraphQL API. The Query component exposes a function to retrieve them in its child function. Since the button to fetch more repositories fits best in the the RepositoryList component, you can pass this function as prop to it.

现在，我们有了从 GitHub 的 GraphQL API 中获取更多代码库页面的所有信息。Query 组件暴露了一个函数，以便在其子函数中检索它们。由于获取更多代码库的按钮最适合于 RepositoryList 组件，因此你可以将这个函数作为 prop 传递给它。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
const Profile = () => (
  <Query query={GET_REPOSITORIES_OF_CURRENT_USER}>
# leanpub-start-insert
    {({ data, loading, error, fetchMore }) => {
# leanpub-end-insert
      ...

      return (
        <RepositoryList
          repositories={viewer.repositories}
# leanpub-start-insert
          fetchMore={fetchMore}
# leanpub-end-insert
        />
      );
    }}
  </Query>
);
~~~~~~~~

> Next, use the function in the RepositoryList component, and add a button to fetch successive pages of repositories that appears when another page is available.

接下来，使用 RepositoryList 组件中的函数，并添加一个按钮，以获取当另一个页面可用时出现的后续代码库页面。

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import React, { Fragment } from 'react';
# leanpub-end-insert

...

# leanpub-start-insert
const RepositoryList = ({ repositories, fetchMore }) => (
  <Fragment>
    {repositories.edges.map(({ node }) => (
# leanpub-end-insert
      ...
# leanpub-start-insert
    ))}
# leanpub-end-insert

# leanpub-start-insert
    {repositories.pageInfo.hasNextPage && (
      <button
        type="button"
        onClick={() =>
          fetchMore({
            /* configuration object */
          })
        }
      >
        More Repositories
      </button>
    )}
  </Fragment>
# leanpub-end-insert
);

export default RepositoryList;
~~~~~~~~

> The `fetchMore()` function performs the query from the initial request, and takes a configuration object, which can be used to override variables. With pagination, this means you pass the `endCursor` of the previous query result to use it for the query as `after` argument. Otherwise, you would perform the initial request again because no variables are specified.

`fetchMore()` 函数从初始请求执行查询，并获取一个配置对象，用以覆盖变量。对于分页，这意味着传递上一个查询结果的 `endCursor`，将其作为 `after` 参数用于查询。否则，你将再次执行初始请求，因为未指定任何变量。

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
const RepositoryList = ({ repositories, fetchMore }) => (
  <Fragment>
    ...

    {repositories.pageInfo.hasNextPage && (
      <button
        type="button"
        onClick={() =>
          fetchMore({
# leanpub-start-insert
            variables: {
              cursor: repositories.pageInfo.endCursor,
            },
# leanpub-end-insert
          })
        }
      >
        More Repositories
      </button>
    )}
  </Fragment>
);
~~~~~~~~

> If you attempt to click the button, you should get the following error message: *Error: updateQuery option is required.*. The `updateQuery` function is needed to tell Apollo Client how to merge the previous result with a new one. Define the function outside of the button, because it would become too verbose otherwise.

如果你尝试点击这个按钮，你应该得到如下的信息：*错误：updateQuery 是必填的。* `updateQuery` 函数需要告诉 Apollo 客户端如何合并上一个结果和新的结果。在按钮的外部定义这个函数，否则它会变得过于冗长。

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const updateQuery = (previousResult, { fetchMoreResult }) => {
  ...
};
# leanpub-end-insert

const RepositoryList = ({ repositories, fetchMore }) => (
  <Fragment>
    ...

    {repositories.pageInfo.hasNextPage && (
      <button
        type="button"
        onClick={() =>
          fetchMore({
            variables: {
              cursor: repositories.pageInfo.endCursor,
            },
# leanpub-start-insert
            updateQuery,
# leanpub-end-insert
          })
        }
      >
        More Repositories
      </button>
    )}
  </Fragment>
);
~~~~~~~~

> The function has access to the previous query result, and to the next result that resolves after the button click:

该函数可以访问上一个查询结果，以及点击按钮后解析的下一个结果：

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
const updateQuery = (previousResult, { fetchMoreResult }) => {
# leanpub-start-insert
  if (!fetchMoreResult) {
    return previousResult;
  }

  return {
    ...previousResult,
    viewer: {
      ...previousResult.viewer,
      repositories: {
        ...previousResult.viewer.repositories,
        ...fetchMoreResult.viewer.repositories,
        edges: [
          ...previousResult.viewer.repositories.edges,
          ...fetchMoreResult.viewer.repositories.edges,
        ],
      },
    },
  };
# leanpub-end-insert
};
~~~~~~~~

> In this function, you can merge both results with the JavaScript spread operator. If there is no new result, return the previous result. The important part is merging the `edges` of both repositories objects to have a merge list of items. The `fetchMoreResult` takes precedence over the `previousResult` in the `repositories` object because it contains the new `pageInfo`, with its `endCursor` and `hasNextPage` properties from the last paginated result. You need to have those when clicking the button another time to have the correct cursor as an argument. If you want to checkout an alternative to the verbose JavaScript spread operator when dealing with deeply nested data, checkout the changes in [this GitHub Pull Request](https://github.com/the-road-to-graphql/react-graphql-github-apollo/pull/14) that uses Lenses from Ramda.js.

在这个函数中，你使用 JavaScript 展开操作符来合并这两个结果。如果没有新的结果，则返回上一个结果。重要的部分是合并两个 repositories 对象的 `edges`，使其拥有一个合并列表。 在 `repositories`  对象中，`fetchMoreResult` 的优先级高于 `previousResult`，因为它包含新的 `pageInfo`，以及最后一个分页结果中的 `endCursor` 和 `hasNextPage` 属性。你需要在再次点击按钮时使用正确的 cursor 作为参数。在处理深层嵌套数据时，如果你想了解详细的 JavaScript 展开操作符，请查看 [这个 GitHub Pull Request](https://github.com/the-road-to-graphql/react-graphql-github-apollo/pull/14) 中的更改，它使用了 Ramda.js 中的 Lenses。

> To add one more small improvement for user friendliness, add a loading indicator when more pages are fetched. So far, the `loading` boolean in the Query component of the Profile component is only true for the initial request, but not for the following requests. Change this behavior with a prop that is passed to the Query component, and the loading boolean will be updated accordingly.

添加一个小的改进，让用户体验更友好：在获取更多页面时添加一个加载指示符。到目前为止，在 Profile 组件的 Query 组件中，`loading` 布尔值对于初始请求来说始终为 true，但不适用于接下来的请求。使用传递给 Query 组件的 prop 更改此行为，loading 布尔值也会相应地更新。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
const Profile = () => (
  <Query
    query={GET_REPOSITORIES_OF_CURRENT_USER}
# leanpub-start-insert
    notifyOnNetworkStatusChange={true}
# leanpub-end-insert
  >
    {({ data, loading, error, fetchMore }) => {
      ...
    }}
  </Query>
);
~~~~~~~~

> When you run your application again and try the "More" button, you should see odd behavior. Every time you load another page of repositories, the loading indicator is shown, but the list of repositories disappears entirely, and the merged list is rendered as assumed. Since the `loading` boolean becomes true with the initial and successive requests, the conditional rendering in the Profile component will always show the loading indicator. It returns from the Profile function early, never reaching the code to render the RepositoryList. A quick change from `||` to `&&` of the condition will allow it to show the loading indicator for the initial request only. Every request after that, where the `viewer` object is available, is beyond this condition, so it renders the RepositoryList component.

当你再次运行你的应用并尝试点击 "More" 按钮时，你应该会看到奇怪的行为。每次加载另一页代码库时，loading 指示符就会显示，但是 代码库列表完全消失，而合并后的列表按照假定的方式渲染。由于 `loading` 布尔值随着初始请求和后续请求变为了 true，因此在 Profile 组件中的条件渲染会始终显示加载指示符。它过早地从 Profile 函数返回，从未到达用于渲染 RepositoryList 的代码。条件从 `||` 到 `&&` 的快速修改将允许它只显示初始请求的加载指示符。此后的每个请求（在 `viewer` 对象可用的情况下）都超出了这个条件，因此它渲染了 RepositoryList 组件。

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
const Profile = () => (
  <Query
    query={GET_REPOSITORIES_OF_CURRENT_USER}
    notifyOnNetworkStatusChange={true}
  >
    {({ data, loading, error, fetchMore }) => {
      ...

      const { viewer } = data;

# leanpub-start-insert
      if (loading && !viewer) {
# leanpub-end-insert
        return <Loading />;
      }

      return (
        <RepositoryList
# leanpub-start-insert
          loading={loading}
# leanpub-end-insert
          repositories={viewer.repositories}
          fetchMore={fetchMore}
        />
      );
    }}
  </Query>
);
~~~~~~~~

> The boolean can be passed down to the RepositoryList component. There it can be used to show a loading indicator instead of the "More" button. Since the boolean never reaches the RepositoryList component for the initial request, you can be sure that the "More" button only changes to the loading indicator when there is a successive request pending.

布尔值向下传递到 RepositoryList 组件。在那里，它可以用来显示一个加载指示符，而不是 "More" 按钮。由于布尔值永远不会到达初始请求的 RepositoryList 组件，因此可以确保 "More" 按钮仅在连续请求被挂起时更改为加载指示符。

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
import React, { Fragment } from 'react';

# leanpub-start-insert
import Loading from '../../Loading';
# leanpub-end-insert
import RepositoryItem from '../RepositoryItem';

...

# leanpub-start-insert
const RepositoryList = ({ repositories, loading, fetchMore }) => (
# leanpub-end-insert
  <Fragment>
    ...

# leanpub-start-insert
    {loading ? (
      <Loading />
    ) : (
# leanpub-end-insert
      repositories.pageInfo.hasNextPage && (
        <button
          ...
        >
          More Repositories
        </button>
      )
# leanpub-start-insert
    )}
# leanpub-end-insert
  </Fragment>
);
~~~~~~~~

> The pagination feature is complete now, and you are fetching successive pages of an initial page, then merging the results in Apollo Client's cache. In addition, you show your user feedback about pending requests for either the initial request or further page requests.

分页功能现在已经完成，你将获取初始页面的后续页面，然后将结果合并到 Apollo 客户端的缓存中。此外，你还可以向用户展示有关挂起请求的反馈，不管是初始请求还是后续页面请求。

> Now we'll take it a step further, making the button used to fetch more repositories reusable. Let me explain why this would be a neat abstraction. In an upcoming section, you have another list field that could potentially implement the pagination feature. There, you have to introduce the `More` button, which could be nearly identical to the `More` button you have in the RepositoryList component. Having only one button in a UI would be a satisfying abstraction, but this abstraction wouldn't work in a real-world coding scenario. You would have to introduce a second list field first, implement the pagination feature for it, and then consider an abstraction for the `More` button. For the sake of the tutorial, we implement this abstraction for the pagination feature only in this section, though you should be aware this is a premature optimization put in place for you to learn it.

现在我们将进一步，让用于获取更多 repositories 的按钮变得可重用。让我来解释为什么这会是一个简洁的抽象。在接下来的部分，你还有另外一个列表字段，它可以实现分页功能。在那里，你必须引入 `More` 按钮，它和你在 RepositoryList 组件中的 `More` 按钮几乎一模一样。在 UI 中，只有一个按钮是令人满意的抽象，但是这个抽象并不能在现实世界的编码场景中工作。首先你必须引入第二个列表字段，为其实现分页功能，然后再考虑 `More` 按钮的抽象。为了教程，我们仅在本节中实现这个分页功能的抽象，尽管你应该意识到这是一个提前优化，以便你学习它。

> For another way, imagine you wanted to extract the functionality of the `More` button into a FetchMore component. The most important thing you would need is the `fetchMore()` function from the query result. The `fetchMore()` function takes an object to pass in the necessary `variables` and `updateQuery` information as a configuration. While the former is used to define the next page by its cursor, the latter is used to define how the results should be merged in the local state. These are the three essential parts: fetchMore, variables, and updateQuery. You may also want to shield away the conditional renderings in the FetchMore component, which happens because of the `loading` or `hasNextPage` booleans. Et voilà! That's how you get the interface to your FetchMore abstraction component.

另一方面，假设你需要将 `More` 按钮的功能提取到 FetchMore 组件中。你需要的最重要的东西就是查询结果中的 `fetchMore()` 函数。`fetchMore()` 函数将对象作为配置传递给必须的 `variables` 和 `updateQuery` 信息。前者用于通过 cursor 定义下一页，后者用于定义如何将结果合并到本地状态。这是三个基本的部分：fetchMore, variables 和 updateQuery。你可能还想屏蔽 FetchMore 组件中的条件渲染，这是由于 `loading` 或 `hasNextPage` 布尔值造成的。我说得没错吧！这就是你如何获得 FetchMore 抽象组件接口的！

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
import React, { Fragment } from 'react';

# leanpub-start-insert
import FetchMore from '../../FetchMore';
# leanpub-end-insert
import RepositoryItem from '../RepositoryItem';

...

const RepositoryList = ({ repositories, loading, fetchMore }) => (
  <Fragment>
    {repositories.edges.map(({ node }) => (
      <div key={node.id} className="RepositoryItem">
        <RepositoryItem {...node} />
      </div>
    ))}

# leanpub-start-insert
    <FetchMore
      loading={loading}
      hasNextPage={repositories.pageInfo.hasNextPage}
      variables={{
        cursor: repositories.pageInfo.endCursor,
      }}
      updateQuery={updateQuery}
      fetchMore={fetchMore}
    >
      Repositories
    </FetchMore>
# leanpub-end-insert
  </Fragment>
);

export default RepositoryList;
~~~~~~~~

> Now this FetchMore component can be used by other paginated lists as well, because every part that can be dynamic is passed as props to it. Implementing a FetchMore component in the *src/FetchMore/index.js* is the next step. First, the main part of the component:

现在 FetchMore 组件也可以被其他分页列表使用，因为每个可以被动态化的部分都作为 prop 传递给它了。下一步就是在 *src/FetchMore/index.js* 中实现 FetchMore 组件。首先，组件的主要部分：

{title="src/FetchMore/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

import './style.css';

const FetchMore = ({
  variables,
  updateQuery,
  fetchMore,
  children,
}) => (
  <div className="FetchMore">
    <button
      type="button"
      className="FetchMore-button"
      onClick={() => fetchMore({ variables, updateQuery })}
    >
      More {children}
    </button>
  </div>
);

export default FetchMore;
~~~~~~~~

> Here, you can see how the `variables` and `updateQuery` are taken as configuration object for the `fetchMore()` function when it's invoked. The button can be made cleaner using the Button component you defined in a previous section. To add a different style, let's define a specialized ButtonUnobtrusive component next to the Button component in the *src/Button/index.js* file:

这里，你可以看到当 `fetchMore()` 函数被调用时， `variables` 和 `updateQuery` 如何作为它的配置对象。使用你在上一节中定义的 Button 组件可以让按钮更整洁。为了添加不同的样式，让我们在 *src/Button/index.js* 文件中的 Button 组件旁定义一个专门的 ButtonUnobtrusive 组件：

{title="src/Button/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

import './style.css';

const Button = ({ ... }) => ...

# leanpub-start-insert
const ButtonUnobtrusive = ({
  children,
  className,
  type = 'button',
  ...props
}) => (
  <button
    className={`${className} Button_unobtrusive`}
    type={type}
    {...props}
  >
    {children}
  </button>
);
# leanpub-end-insert

# leanpub-start-insert
export { ButtonUnobtrusive };
# leanpub-end-insert

export default Button;
~~~~~~~~

> Now the ButtonUnobtrusive component is used as button instead of the button element in the FetchMore component. In addition, the two booleans `loading` and `hasNextPage` can be used for the conditional rendering, to show the Loading component or nothing, because there is no next page which can be fetched.

现在，ButtonUnobtrusive 组件在 FetchMore 组件中代替 button 元素被用作按钮。另外，`loading` 和 `hasNextPage` 这两个布尔值可以用于条件渲染，以显示 Loading 组件或者在没有可获取的下一页时不显示任何内容。

{title="src/FetchMore/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

# leanpub-start-insert
import Loading from '../Loading';
import { ButtonUnobtrusive } from '../Button';
# leanpub-end-insert

import './style.css';

const FetchMore = ({
# leanpub-start-insert
  loading,
  hasNextPage,
# leanpub-end-insert
  variables,
  updateQuery,
  fetchMore,
  children,
}) => (
  <div className="FetchMore">
# leanpub-start-insert
    {loading ? (
      <Loading />
    ) : (
      hasNextPage && (
        <ButtonUnobtrusive
# leanpub-end-insert
          className="FetchMore-button"
          onClick={() => fetchMore({ variables, updateQuery })}
        >
          More {children}
# leanpub-start-insert
        </ButtonUnobtrusive>
      )
    )}
# leanpub-end-insert
  </div>
);

export default FetchMore;
~~~~~~~~

> That's it for the abstraction of the FetchMore button for paginated lists with Apollo Client. Basically, you pass in everything needed by the `fetchMore()` function, including the function itself. You can also pass all booleans used for conditional renderings. You end up with a reusable FetchMore button that can be used for every paginated list.

这就是对 Apollo 客户端分页列表的 FetchMore 按钮的抽象。基本上，你将传入 `fetchMore()` 函数需要的所有东西，包括函数本身。你还可以传递用于条件渲染的所有布尔值。最终得到一个可重用的 FetchMore 按钮，可用于每一个分页列表。

> ### Exercises:
### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/65cb143d605b1c7e9c080f36b5f64805f02aba29)

* [查看本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/65cb143d605b1c7e9c080f36b5f64805f02aba29)

> * Read more about [pagination with Apollo Client in React](https://www.apollographql.com/docs/react/features/pagination.html)

* 延伸阅读：[React 中的 Apollo 客户端分页](https://www.apollographql.com/docs/react/features/pagination.html) 的更多信息

> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5HYMGN7)

* 花三分钟的时间进行[测验](https://www.surveymonkey.com/r/5HYMGN7)

> ## GraphQL Caching of Queries with Apollo Client in React

## React 中通过 Apollo 客户端实现 GraphQL 缓存查询

> In this section, you introduce [React Router](https://github.com/ReactTraining/react-router) to show two separate pages for your application. At the moment, you are only showing one page with a Profile component that displays all your repositories. We want to add another Organization component that shows repositories by an organization, and there could be a search field as well, to lookup individual organizations with their repositories on that page. Let's do this by introducing React Router to your application. If you haven't used React Router before, make sure to conduct the exercises of this section to learn more about it.

在这一小节中，为了在程序里展示两个页面，你需要引入 [React Router](https://github.com/ReactTraining/react-router) 来实现。到目前为止，你还只是在一个页面中用 Profile 组件展示你所有的代码库。 我们想要添加另一个 Organization 组件来按组织展示你的代码库，并且加上搜索框实现在这个页面上查找不同组织的代码库的功能。让我们通过在你的程序引入 React Router 中来实现它吧。如果你之前没有使用过 React Router，请务必进行此小节的练习来更好地了解它。

{title="Command Line",lang="json"}
~~~~~~~~
npm install react-router-dom --save
~~~~~~~~

> In your *src/constants/routes.js* file, you can specify both routes you want to make accessible by React Router. The `ORGANIZATION` route points to the base URL, while the `PROFILE` route points to a more specific URL.

在 *src/constants/routes.js* 文件中你可以指定要通过 React Router 访问的路由。`ORGANIZATION` 路由指向根 URL，`PROFILE` 路由指向更具体的 URL。

{title="src/constants/routes.js",lang="javascript"}
~~~~~~~~
export const ORGANIZATION = '/';
export const PROFILE = '/profile';
~~~~~~~~

> Next, map both routes to their components. The App component is the perfect place to do it because the two routes will exchange the Organization and Profile components based on the URL there.

接下来，将两个路由分别指向它们对应的组件。App 组件是做这个的最佳位置，因为这两个路由将会基于这里写的URL来切换 Organization 组件和 Profile 组件。

{title="src/App/index.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import { BrowserRouter as Router, Route } from 'react-router-dom';

import Profile from '../Profile';
import Organization from '../Organization';

import * as routes from '../constants/routes';

import './style.css';

class App extends Component {
  render() {
    return (
      <Router>
        <div className="App">
          <div className="App-main">
            <Route
              exact
              path={routes.ORGANIZATION}
              component={() => (
                <div className="App-content_large-header">
                  <Organization />
                </div>
              )}
            />
            <Route
              exact
              path={routes.PROFILE}
              component={() => (
                <div className="App-content_small-header">
                  <Profile />
                </div>
              )}
            />
          </div>
        </div>
      </Router>
    );
  }
}

export default App;
~~~~~~~~

> The Organization component wasn't implemented yet, but you can start with a functional stateless component in the *src/Organization/index.js* file, that acts as a placeholder to keep the application working for now.

虽然 Organization 组件还没有实现，但你可以先在 *src/Organization/index.js* 文件中放一个无状态组件作为占位元素来保持程序可以正常运行。

{title="src/Organization/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

const Organization = () => <div>Organization</div>;

export default Organization;
~~~~~~~~

> Since you mapped both routes to their respective components, so you want to implement navigation from one route to another. For this, introduce a **Navigation** component in the App component.

既然你已经将两个路由指向了它们各自的组件，你一定想要实现一个从一个路由到另一个路由的导航吧。在 App 组件中引入 **Navigation** 组件来实现它。

{title="src/App/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
import Navigation from './Navigation';
# leanpub-end-insert
import Profile from '../Profile';
import Organization from '../Organization';

...

class App extends Component {
  render() {
    return (
      <Router>
        <div className="App">
# leanpub-start-insert
          <Navigation />
# leanpub-end-insert

          <div className="App-main">
            ...
          </div>
        </div>
      </Router>
    );
  }
}

export default App;
~~~~~~~~

> Next, we'll implement the Navigation component, which is responsible for displaying the two links to navigate between your routes using React Router's Link component.

接下来，我们将会实现 Navigation 组件，该组件通过使用 React Router 的 Link 组件来显示两个路由之间导航的链接。

{title="src/App/Navigation/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import { Link } from 'react-router-dom';

import * as routes from '../../constants/routes';

import './style.css';

const Navigation = () => (
  <header className="Navigation">
    <div className="Navigation-link">
      <Link to={routes.PROFILE}>Profile</Link>
    </div>
    <div className="Navigation-link">
      <Link to={routes.ORGANIZATION}>Organization</Link>
    </div>
  </header>
);

export default Navigation;
~~~~~~~~

> The Profile page works as before, but the Organization page is empty. In the last step, you defined the two routes as constants, used them in the App component to map to their respective components, and introduced Link components to navigate to them in the Navigation component.

Profile 页面与之前一样，只是 Organization 页面是空的。在最后一步中，你将两个路由定义为常量，在 App 组件中将它们指向它们各自的组件，并引入 Link 组件来在 Navigation 组件中导航到它们。

> Another great feature of the Apollo Client is that it caches query requests. When navigating from the Profile page to the Organization page and back to the Profile page, the results appear immediately because the Apollo Client checks its cache before making the query to the remote GraphQL API. It's a pretty powerful tool.

Apollo 客户端的另一个重要功能就是它的缓存查询请求。当页面从 Profile 页面跳转到 Organization 页面再回到 Profile 页面时，结果会立即显示出来，因为 Apollo 客户端会在查询远程 GraphQL API 之前检查缓存。这是个非常强大的功能。

> The next part of this section is the Organization component. It is the same as the Profile component, except the query differs because it takes a variable for the organization name to identify the organization's repositories.

本小节的下一部分是 Organization 组件。和 Profile 组件基本一样，除了使用不同的查询来将组织名作为变量标识各组织的代码库。

{title="src/Organization/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import gql from 'graphql-tag';
import { Query } from 'react-apollo';

import { REPOSITORY_FRAGMENT } from '../Repository';

const GET_REPOSITORIES_OF_ORGANIZATION = gql`
  query($organizationName: String!) {
    organization(login: $organizationName) {
      repositories(first: 5) {
        edges {
          node {
            ...repository
          }
        }
      }
    }
  }
  ${REPOSITORY_FRAGMENT}
`;

const Organization = ({ organizationName }) => (
  <Query
    query={GET_REPOSITORIES_OF_ORGANIZATION}
    variables={{
      organizationName,
    }}
    skip={organizationName === ''}
  >
    {({ data, loading, error }) => {
      ...
    }}
  </Query>
);

export default Organization;
~~~~~~~~

> The Query component in the Organization component takes a query tailored to the organization being the top level field of the query. It takes a variable to identify the organization, and it uses the newly introduced `skip` prop to skip executing the query if no organization identifier is provided. Later, you will pass an organization identifier from the App component. You may have noticed that the repository fragment you introduced earlier to update the local state in the cache can be reused here. It saves lines of code, and more importantly, ensures the returned list of repositories have identical structures to the list of repositories in the Profile component.

Organization 组件中的 Query 组件实现对组织的定制查询作为查询的顶级字段。它通过一个变量来对组织进行标识，如果这个组织标识符没有被提供，则引入一个新的 `skip` prop 来跳过查询操作。然后，你会从 App 组件中传递组织标识符。你可能已经注意到，之前引入的更新本地状态的 fragment 可以在这里重用。这不仅仅是减少了代码冗余，更重要的是，这样确保了返回的代码库列表和和 Profile 组件中的代码库列表具有相同的结构。

> Next, extend the query to fit the requirements of the pagination feature. It requires the `cursor` argument to identify the next page of repositories. The `notifyOnNetworkStatusChange` prop is used to update the `loading` boolean for paginated requests as well.

接下来，对查询进行扩展以满足分页功能。这里需要 `cursor` 参数来标识代码库的下一页。`notifyOnNetworkStatusChange` prop 用来更新分页请求布尔值的 `loading` 状态。

{title="src/Organization/index.js",lang="javascript"}
~~~~~~~~
...

const GET_REPOSITORIES_OF_ORGANIZATION = gql`
# leanpub-start-insert
  query($organizationName: String!, $cursor: String) {
# leanpub-end-insert
    organization(login: $organizationName) {
# leanpub-start-insert
      repositories(first: 5, after: $cursor) {
# leanpub-end-insert
        edges {
          node {
            ...repository
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
  ${REPOSITORY_FRAGMENT}
`;

const Organization = ({ organizationName }) => (
  <Query
    query={GET_REPOSITORIES_OF_ORGANIZATION}
    variables={{
      organizationName,
    }}
    skip={organizationName === ''}
# leanpub-start-insert
    notifyOnNetworkStatusChange={true}
  >
    {({ data, loading, error, fetchMore }) => {
# leanpub-end-insert
      ...
    }}
  </Query>
);

export default Organization;
~~~~~~~~

> Lastly, the render prop child function needs to be implemented. It doesn't differ much from the Query's content in the Profile component. Its purpose is to handle edge cases like loading and 'no data' errors, and eventually, to show a list of repositories. Because the RepositoryList component handles the pagination feature, this improvement is included in the newly implemented Organization component.

最后一步，实现 render prop 子函数。这和在 Profile 组件中渲染 Query 的内容基本一致。它主要是用于处理像“没有数据”这样的边界情况和错误，并最终显示出代码库列表。由于 RepositoryList 组件处理了分页功能，所以将这个改进放入新实现的 Organization 组件中。

{title="src/Organization/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
import RepositoryList, { REPOSITORY_FRAGMENT } from '../Repository';
import Loading from '../Loading';
import ErrorMessage from '../Error';
# leanpub-end-insert

...

const Organization = ({ organizationName }) => (
  <Query ... >
    {({ data, loading, error, fetchMore }) => {
# leanpub-start-insert
      if (error) {
        return <ErrorMessage error={error} />;
      }

      const { organization } = data;

      if (loading && !organization) {
        return <Loading />;
      }

      return (
        <RepositoryList
          loading={loading}
          repositories={organization.repositories}
          fetchMore={fetchMore}
        />
      );
    }}
# leanpub-end-insert
  </Query>
);

export default Organization;
~~~~~~~~

> Provide a `organizationName` as prop when using the Organization in the App component, and leave it inlined for now. Later, you will make it dynamic with a search field.

在 App 组件中提供一个 `organizationName` 作为 prop 传给 Organization 组件并暂时保留为内联传值，你会用它来动态化搜索字段。

{title="src/App/index.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  render() {
    return (
      <Router>
        <div className="App">
          <Navigation />

          <div className="App-main">
            <Route
              exact
              path={routes.ORGANIZATION}
              component={() => (
                <div className="App-content_large-header">
                  <Organization
# leanpub-start-insert
                    organizationName={'the-road-to-learn-react'}
# leanpub-end-insert
                  />
                </div>
              )}
            />
            ...
          </div>
        </div>
      </Router>
    );
  }
}
~~~~~~~~

> The Organization component should almost work now, as the `More` button is the only incomplete part. The remaining issue is the resolving block for the pagination feature in the `updateQuery` function. It assumes that the nested data structure always starts with a `viewer` object. It does for the Profile page, but not for the Organization page. There the top level object is the `organization` followed by the list of `repositories`. Only the top level object changes from page to page, where the underlying structure stays identical.

现在就差一个 `More` 按钮 Organization 组件就可以完整地正常工作了。剩下的问题是在 `updateQuery` 函数中处理我们分页功能的各个块(页)。假设嵌套的数据结构始终以 `viewer` 对象开头。这样适用于 Profile 页面但不适用于 Organization 页面。所以这里顶层的对象是 `organization`，然后是 `repositories` 的列表。这样在页面切换的时候，只有顶层对象会发生改变，底层的结构保持相同。

 > When the top level object changes from page to page, the ideal next step is to tell the RepositoryList component its top level object from the outside. With the Organization component, its the top-level object `organization`, which could be passed as a string and reused as a dynamic key later:

当顶层对象在页面切换发生改变时，理想的下一步是从外部告诉 RepositoryList 组件它的顶层对象。对于 Organization 组件，顶层对象是 `organization`，可以将它作为字符串传递并在后面作为 key 使用：

{title="src/Organization/index.js",lang="javascript"}
~~~~~~~~
const Organization = ({ organizationName }) => (
  <Query ... >
    {({ data, loading, error, fetchMore }) => {
      ...

      return (
        <RepositoryList
          loading={loading}
          repositories={organization.repositories}
          fetchMore={fetchMore}
# leanpub-start-insert
          entry={'organization'}
# leanpub-end-insert
        />
      );
    }}
  </Query>
);
~~~~~~~~

> With the Profile component, the `viewer` would be the top level object:

对于 Profile 组件，顶层对象则是 `viewer` ：

{title="src/Profile/index.js",lang="javascript"}
~~~~~~~~
const Profile = () => (
  <Query ... >
    {({ data, loading, error, fetchMore }) => {
      ...

      return (
        <RepositoryList
          loading={loading}
          repositories={viewer.repositories}
          fetchMore={fetchMore}
# leanpub-start-insert
          entry={'viewer'}
# leanpub-end-insert
        />
      );
    }}
  </Query>
);
~~~~~~~~

> Now you can handle the new case in the RepositoryList component by passing the entry as [computed property name](https://developer.mozilla.org/my/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names) to the `updateQuery` function. Instead of passing the `updateQuery` function directly to the FetchMore component, it can be derived from a higher-order function needed to pass the new `entry` property.

现在，你可以通过在 RepositoryList 组件中给 `updateQuery` 函数传入参数作为[计算属性名](https://developer.mozilla.org/my/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names)来进行处理。它不是直接将 `updateQuery` 函数传递给 FetchMore 组件，而是从传递的新 `entry` 属性所需的高阶函数派生而来的。

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
const RepositoryList = ({
  repositories,
  loading,
  fetchMore,
# leanpub-start-insert
  entry,
# leanpub-end-insert
}) => (
  <Fragment>
    ...

    <FetchMore
      loading={loading}
      hasNextPage={repositories.pageInfo.hasNextPage}
      variables={{
        cursor: repositories.pageInfo.endCursor,
      }}
# leanpub-start-insert
      updateQuery={getUpdateQuery(entry)}
# leanpub-end-insert
      fetchMore={fetchMore}
    >
      Repositories
    </FetchMore>
  </Fragment>
);
~~~~~~~~

> The higher-order function next to the RepositoryList component is completed as such:

RepositoryList 组件中的这个高阶函数实现如下：

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const getUpdateQuery = entry => (
# leanpub-end-insert
  previousResult,
  { fetchMoreResult },
) => {
  if (!fetchMoreResult) {
    return previousResult;
  }

  return {
    ...previousResult,
# leanpub-start-insert
    [entry]: {
      ...previousResult[entry],
# leanpub-end-insert
      repositories: {
# leanpub-start-insert
        ...previousResult[entry].repositories,
        ...fetchMoreResult[entry].repositories,
# leanpub-end-insert
        edges: [
# leanpub-start-insert
          ...previousResult[entry].repositories.edges,
          ...fetchMoreResult[entry].repositories.edges,
# leanpub-end-insert
        ],
      },
    },
  };
};
~~~~~~~~

> That's how a deeply-nested object is updated with the `fetchMoreResult`, even though the top level component from the query result is not static. The pagination feature should work on both pages now. Take a moment to recap the last implementations again and why these were necessary.

这就是通过 `fetchMoreResult` 函数来更新深层嵌套对象的方式，即使查询结果中的顶层组件不是静态的。现在分页功能应该在两个页面上都生效了。让我们花一点时间再回顾一下最后的实现和它的必要性。

> Next, we'll implement the search function I mentioned earlier. The best place to add the search field would be the Navigation component, but only when the Organization page is active. React Router comes with a useful higher-order component to access to the current URL, which can be used to show a search field.

接下来，我们将会实现之前提到的搜索功能。最好的放置搜索框的地方是 Navigation 组件，但只是在 Organization 页面生效时显示。React Router 附带一个有用的高阶组件来访问当前的 URL ，可以用来显示搜索框。

{title="src/App/Navigation/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import { Link, withRouter } from 'react-router-dom';
# leanpub-end-insert

import * as routes from '../../constants/routes';

import './style.css';

const Navigation = ({
# leanpub-start-insert
  location: { pathname },
# leanpub-end-insert
}) => (
  <header className="Navigation">
    <div className="Navigation-link">
      <Link to={routes.PROFILE}>Profile</Link>
    </div>
    <div className="Navigation-link">
      <Link to={routes.ORGANIZATION}>Organization</Link>
    </div>

# leanpub-start-insert
    {pathname === routes.ORGANIZATION && (
      <OrganizationSearch />
    )}
# leanpub-end-insert
  </header>
);

# leanpub-start-insert
export default withRouter(Navigation);
# leanpub-end-insert
~~~~~~~~

> The OrganizationSearch component is implemented next to the Navigation component in the next steps. Before that can work, there needs to be some kind of initial state for the OrganizationSearch, as well as a callback function to update the initial state in the Navigation component. To accommodate this, the Navigation component becomes a class component.

后续步骤中，NavigationSearch 组件将紧接着 Navigation 组件实现。在此之前，我们需要在 Navigation 组件中设置 OrganizationSearch 的某种初始状态和更新初始状态的回调函数。Navigation 组件需要变成类组件来实现这个需求。

{title="src/App/Navigation/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
class Navigation extends React.Component {
  state = {
    organizationName: 'the-road-to-learn-react',
  };

  onOrganizationSearch = value => {
    this.setState({ organizationName: value });
  };

  render() {
    const { location: { pathname } } = this.props;

    return (
# leanpub-end-insert
      <header className="Navigation">
        <div className="Navigation-link">
          <Link to={routes.PROFILE}>Profile</Link>
        </div>
        <div className="Navigation-link">
          <Link to={routes.ORGANIZATION}>Organization</Link>
        </div>

        {pathname === routes.ORGANIZATION && (
          <OrganizationSearch
# leanpub-start-insert
            organizationName={this.state.organizationName}
            onOrganizationSearch={this.onOrganizationSearch}
# leanpub-end-insert
          />
        )}
      </header>
# leanpub-start-insert
    );
  }
}
# leanpub-end-insert

export default withRouter(Navigation);
~~~~~~~~

> The OrganizationSearch component implemented in the same file would also work with the following implementation. It handles its own local state, the value that shows up in the input field, but uses it as an initial value from the parent component. It also receives a callback handler, which can be used in the `onSubmit()` class method to propagate the search fields value on a submit interaction up the component tree.

在同一文件中实现的 OrganizationSearch 组件也可以采用如下实现方式。它处理自己的本地状态，即输入框中显示的值，不过初始值来自于父组件。它还接受一个回调函数，该回调在 `onSubmit()` 中使用，用来在组件树中向上传递搜索的字段。

{title="src/App/Navigation/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
import Button from '../../Button';
import Input from '../../Input';
# leanpub-end-insert

import './style.css';

const Navigation = ({ ... }) => ...

# leanpub-start-insert
class OrganizationSearch extends React.Component {
  state = {
    value: this.props.organizationName,
  };

  onChange = event => {
    this.setState({ value: event.target.value });
  };

  onSubmit = event => {
    this.props.onOrganizationSearch(this.state.value);

    event.preventDefault();
  };

  render() {
    const { value } = this.state;

    return (
      <div className="Navigation-search">
        <form onSubmit={this.onSubmit}>
          <Input
            color={'white'}
            type="text"
            value={value}
            onChange={this.onChange}
          />{' '}
          <Button color={'white'} type="submit">
            Search
          </Button>
        </form>
      </div>
    );
  }
}
# leanpub-end-insert

export default withRouter(Navigation);
~~~~~~~~

> The Input component is a slightly styled input element that is defined in *src/Input/index.js* as its own component.

Input 组件是一个带有样式的 input 元素，在 *src/Input/index.js*  中定义为一个组件。

{title="src/Input/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

import './style.css';

const Input = ({ children, color = 'black', ...props }) => (
  <input className={`Input Input_${color}`} {...props}>
    {children}
  </input>
);

export default Input;
~~~~~~~~

> While the search field works in the Navigation component, it doesn't help the rest of the application. It only updates the state in the Navigation component when a search request is submitted. However, the value of the search request is needed in the Organization component as a GraphQL variable for the query, so the local state needs to be lifted up from the Navigation component to the App component. The Navigation component becomes a stateless functional component again.

虽然搜索框已经在 Navigation 组件中起生效了，但它对应用的其他部分还没有什么作用。它只是在提交搜索请求时更新 Navigation 组件中的状态。然而，搜索请求的值需要在 Organization 组件中作为查询 GraphQL 的变量，所以本地状态需要从 Navigation 组件传递到 App 组件。Navigation 组件就又变回了无状态组件。

{title="src/App/Navigation/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const Navigation = ({
  location: { pathname },
  organizationName,
  onOrganizationSearch,
}) => (
# leanpub-end-insert
  <header className="Navigation">
    <div className="Navigation-link">
      <Link to={routes.PROFILE}>Profile</Link>
    </div>
    <div className="Navigation-link">
      <Link to={routes.ORGANIZATION}>Organization</Link>
    </div>

    {pathname === routes.ORGANIZATION && (
      <OrganizationSearch
# leanpub-start-insert
        organizationName={organizationName}
        onOrganizationSearch={onOrganizationSearch}
# leanpub-end-insert
      />
    )}
  </header>
# leanpub-start-insert
);
# leanpub-end-insert
~~~~~~~~

> The App component takes over the responsibility from the Navigation component, managing the local state, passing the initial state and a callback function to update the state to the Navigation component, and passing the state itself to the Organization component to perform the query:

App 组件接管 Navigation 组件的职责，来管理本地状态，传递初始状态和回调函数以更新状态传递给 Navigation 组件，并将自身的状态传递给 Organization 组件以执行查询：

{title="src/App/index.js",lang="javascript"}
~~~~~~~~
...

class App extends Component {
# leanpub-start-insert
  state = {
    organizationName: 'the-road-to-learn-react',
  };
# leanpub-end-insert

# leanpub-start-insert
  onOrganizationSearch = value => {
    this.setState({ organizationName: value });
  };
# leanpub-end-insert

  render() {
# leanpub-start-insert
    const { organizationName } = this.state;
# leanpub-end-insert

    return (
      <Router>
        <div className="App">
          <Navigation
# leanpub-start-insert
            organizationName={organizationName}
            onOrganizationSearch={this.onOrganizationSearch}
# leanpub-end-insert
          />

          <div className="App-main">
            <Route
              exact
              path={routes.ORGANIZATION}
              component={() => (
                <div className="App-content_large-header">
# leanpub-start-insert
                  <Organization organizationName={organizationName} />
# leanpub-end-insert
                </div>
              )}
            />
            ...
          </div>
        </div>
      </Router>
    );
  }
}

export default App;
~~~~~~~~

> You have implemented a dynamic GraphQL query with a search field. Once a new `organizationName` is passed to the Organization component from a local state change, the Query component triggers another request due to a re-render. The request is not always made to the remote GraphQL API, though. The Apollo Client cache is used when an organization is searched twice. Also, you have used the well-known technique called lifting state in React to share the state across components.

你已实现了使用搜索字段进行动态的 GraphQL 查询。一旦将新的 `organizationName` 变化从本地状态传递给 Organization 组件，Query 组件就会因为重新渲染而触发另一个请求。但它并不会总是向远程 GraphQL API 发送请求。一个组织被搜索多次时，则是使用的 Apollo 客户端缓存。此外，你也在 React 中使用了众所周知的被称之为状态提升的技巧，来使组件之间的状态共享。

> ### Exercises:
> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/3ab9c752ec0ec8c3e5f7a1ead4519ea3a626785b)
> * If you are not familiar with React Router, try it out in [this pragmatic tutorial](https://www.robinwieruch.de/complete-firebase-authentication-react-tutorial/)
> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5HFQ3TD)

### 练习：
* 查看 [本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/3ab9c752ec0ec8c3e5f7a1ead4519ea3a626785b)
* 如果你还不熟悉 React Router，请练习 [这个实用的教程](https://www.robinwieruch.de/complete-firebase-authentication-react-tutorial/)
* 花三分钟的时间进行[测验](https://www.surveymonkey.com/r/5HFQ3TD)

> ## Implementing the Issues Feature: Setup
## 实现 Issues 功能：准备

> In the previous sections you have implemented most of the common Apollo Client features in your React application. Now you can start implementing extensions for the application on your own. This section showcases how a full-fledged feature can be implemented with Apollo Client in React.

在前面的小节中你已经在你的 React 应用中实现了大部分常用的 Apollo Client 的功能。现在你可以开始自己实现应用的扩展。本节将展示如何在 React 中使用 Apollo Client 实现一个完整的功能。

> So far, you have dealt with GitHub repositories from organizations and your account. This will take that one step further, fetching GitHub issues that are made available using a list field associated to a repository in a GraphQL query. However, this section doesn't only show you how to render a nested list field in your React application.

到目前为止，你已经处理了来自组织和你的账户的 GitHub 代码库。我们将更进一步，获取 GitHub issues，这会使用 GraphQL 查询中与代码库关联的列表字段。然而，本节不仅仅向你展示如何在 React 中渲染嵌套列表。

> The foundation will be rendering the list of issues. You will implement client-side filtering with plain React to show opened, closed, or no issue. Finally, you will refactor the filtering to a server-side filtering using GraphQL queries. We will only fetch the issues by their state from the server rather than filtering the issue's state on the client-side. Implementing pagination for the issues will be your exercise.

最基本的就是渲染 issue 列表，你将使用最简单的 React 来实现客户端的打开，关闭或者不显示 issue 的筛选。最后，你将使用 GraphQL 查询将筛选重构为服务器端筛选。我们只根据 issue 的状态从服务端获取，而不是在客户端去过滤 issue 的状态。你的练习是为这些 issue 列表实现分页。


> First, render a new component called 'Issues' in your RepositoryList component. This component takes two props that are used later in a GraphQL query to identify the repository from which you want to fetch the issues.

首先，在 RepositoryList 组件中渲染一个名叫 Issues 的新组件。这个组件接收两个 props，稍后将用于 GraphQL 查询，以标识从中要从中获取 issue 列表的代码库。

{title="src/Repository/RepositoryList/index.js",lang="javascript"}
~~~~~~~~
...

import FetchMore from '../../FetchMore';
import RepositoryItem from '../RepositoryItem';
# leanpub-start-insert
import Issues from '../../Issue';
# leanpub-end-insert

...

const RepositoryList = ({
  repositories,
  loading,
  fetchMore,
  entry,
}) => (
  <Fragment>
    {repositories.edges.map(({ node }) => (
      <div key={node.id} className="RepositoryItem">
        <RepositoryItem {...node} />

# leanpub-start-insert
        <Issues
          repositoryName={node.name}
          repositoryOwner={node.owner.login}
        />
# leanpub-end-insert
      </div>
    ))}

    ...
  </Fragment>
);

export default RepositoryList;
~~~~~~~~

> In the *src/Issue/index.js* file, import and export the Issues component. Since the issue feature can be kept in a module on its own, it has this *index.js* file again. That's how you can tell other developers to access only this feature module, using the *index.js* file as its interface. Everything else is kept private.

在 src/Issue/index.js 文件中，引入和导出 Issues 组件，由于 issue 功能可以保持在单独的模块中，所以它有这个 *index.js* 文件，这样你可以告诉别的开发人员仅仅使用 *index.js* 访问这个功能模块。*index.js* 文件作为接口，而其他的都保持私有。

{title="src/Issue/index.js",lang="javascript"}
~~~~~~~~
import Issues from './IssueList';

export default Issues;
~~~~~~~~

> Note how the component is named Issues, not IssueList. The naming convention is used to break down the rendering of a list of items: Issues, IssueList and IssueItem. Issues is the container component, where you query the data and filter the issues, and the IssueList and IssueItem are only there as presentational components for rendering. In contrast, the Repository feature module hasn't a Repositories component, because there was no need for it. The list of repositories already came from the Organization and Profile components and the Repository module's components are mainly only there for the rendering. This is only one opinionated approach of naming the components, however.

请注意组件的名字是 Issues，而不是 IssueList。拆分渲染列表的命名约定为：Issues, IssueList 和 IssueItem。Issues 是容器组件，用来查询数据和过滤 issue，IssueList 和 IssueItem 组件仅仅用做内容展示。相反，代码库功能模块并没有一个 Repositories 组件，因为并不需要它，代码库列表已经在 Organization 和 Profile 组件中，并且代码库模块的组件主要用于内容展示。当然，这只是命名组件的一种固执己见的方式。

> Let's start implementing Issues and IssueList components in the *src/Issue/IssueList/index.js* file. You could argue to split both components up into their own files, but for the sake of this tutorial, they are kept together in one file.

让我们开始在 *src/Issue/IssueList/index.js* 文件中实现 Issues 和 IssueList 组件，你可以认为应该将这两个组件拆分到各自的文件中，但由于本教程的缘故，它们被保存在同一个文件中。

> First, there needs to be a new query for the issues. You might wonder: Why do we need a new query here? It would be simpler to include the issues list field in the query at the top next to the Organization and Profile components. That's true, but it comes with a cost. Adding more nested (list) fields to a query often results into performance issues on the server-side. There you may have to make multiple roundtrips to retrieve all the entities from the database.

首先，需要对 issue 进行一个新的查询，你可能会想：为什么这里需要一个新的查询？将 issue 列表字段包含在 Organization 和 Profile 组件顶部的查询会更简单。确实是这样，但也会有代价。向查询添加更多嵌套（列表）字段常常会导致服务器端的性能问题。所以你可能需要多次往返才能从数据库中检索所有的实体。

> * Roundtrip 1: get organization by name
> * Roundtrip 2: get repositories of organization by organization identifier
> * Roundtrip 3: get issues of repository by repository identifier

* 往返1：根据名称获取组织
* 往返2：根据组织标识获取组织下的代码库
* 往返3：根据代码库标识获取 issue 列表

> It is simple to conclude that nesting queries in a naive way solves all of our problems. Whereas it solves the problem of only requesting the data once and not with multiple network request (similar roundtrips as shown for the database), GraphQL doesn't solve the problem of retrieving all the data from the database for you. That's not the responsibility of GraphQL after all. So by having a dedicated query in the Issues component, you can decide **when** to trigger this query. In the next steps, you will just trigger it on render because the Query component is used. But when adding the client-side filter later on, it will only be triggered when the "Filter" button is toggled. Otherwise the issues should be hidden. Finally, that's how all the initial data loading can be delayed to a point when the user actually wants to see the data.

很容易得出一个结论，以一种简单的方式嵌套查询可以解决我们所有的问题。虽然它解决了只请求一次数据而不是用多个网络请求（类似于数据库的往返）的问题，但是 GraphQL 并没有为你解决从数据库检索所有数据的问题，这毕竟不是 GraphQL 的责任。因此，通过在 Issues 组件中有个专门的查询，你可以决定**何时**触发这个查询。在接下来的步骤中，你只需要在渲染时触发它，因为使用了 Query 组件。但当稍后添加了客户端过滤器时，只有切换“过滤”按钮才会触发查询，否则 issue 列表应该被隐藏。最终，这就是如何将所有初始数据加载延迟到用户真正想要查看数据的时候。

> First, define the Issues component which has access to the props which were passed in the RepositoryList component. It doesn't render much yet.

首先，定义一个能获取从 RepositoryList 组件传递下来 props 的 Issues 组件，它暂时还没渲染太多内容。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

import './style.css';

const Issues = ({ repositoryOwner, re positoryName }) =>
  <div className="Issues">
  </div>

export default Issues;
~~~~~~~~

> Second, define the query in the *src/Issue/IssueList/index.js* file to retrieve issues of a repository. The repository is identified by its owner and name. Also, add the `state` field as one of the fields for the query result. This is used for client-side filtering, for showing issues with an open or closed state.

然后，在 *src/Issue/IssueList/index.js* 文件中定义查询用来获取代码库 issue。代码库的所有者和名字作为唯一标识，另外，添加 `state` 字段作为查询结果的字段之一。这个用于客户端的筛选，用来显示打开或关闭状态的 issue。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import gql from 'graphql-tag';
# leanpub-end-insert

import './style.css';

# leanpub-start-insert
const GET_ISSUES_OF_REPOSITORY = gql`
  query($repositoryOwner: String!, $repositoryName: String!) {
    repository(name: $repositoryName, owner: $repositoryOwner) {
      issues(first: 5) {
        edges {
          node {
            id
            number
            state
            title
            url
            bodyHTML
          }
        }
      }
    }
  }
`;
# leanpub-end-insert

...
~~~~~~~~

> Third, introduce the Query component and pass it the previously defined query and the necessary variables. Use its render prop child function to access the data, to cover all edge cases and to render a IssueList component eventually.

第三步，引入 Query 组件，并将前面定义的查询和必要的变量传递给它。使用 children 函数作为 render prop 访问数据来覆盖所有边缘情况并最终渲染一个 IssueList 组件。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import { Query } from 'react-apollo';
# leanpub-end-insert
import gql from 'graphql-tag';

# leanpub-start-insert
import IssueItem from '../IssueItem';
import Loading from '../../Loading';
import ErrorMessage from '../../Error';
# leanpub-end-insert

import './style.css';

const Issues = ({ repositoryOwner, repositoryName }) => (
  <div className="Issues">
# leanpub-start-insert
    <Query
      query={GET_ISSUES_OF_REPOSITORY}
      variables={{
        repositoryOwner,
        repositoryName,
      }}
    >
      {({ data, loading, error }) => {
        if (error) {
          return <ErrorMessage error={error} />;
        }

        const { repository } = data;

        if (loading && !repository) {
          return <Loading />;
        }

        if (!repository.issues.edges.length) {
          return <div className="IssueList">No issues ...</div>;
        }

        return <IssueList issues={repository.issues} />;
      }}
    </Query>
# leanpub-end-insert
  </div>
);

# leanpub-start-insert
const IssueList = ({ issues }) => (
  <div className="IssueList">
    {issues.edges.map(({ node }) => (
      <IssueItem key={node.id} issue={node} />
    ))}
  </div>
);
# leanpub-end-insert

export default Issues;
~~~~~~~~

> Finally, implement a basic IssueItem component in the *src/Issue/IssueItem/index.js* file. The snippet below shows a placeholder where you can implement the Commenting feature, which we'll cover later.

最后，在 *src/Issue/IssueItem/index.js* 中实现一个基本的 IssueItem 组件。下面的代码片段展示了一个占位符，你可以在其中实现评论的功能，稍后我们将会进行介绍。

{title="src/Issue/IssueItem/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';

import Link from '../../Link';

import './style.css';

const IssueItem = ({ issue }) => (
  <div className="IssueItem">
    {/* 稍后添加一个 显示/隐藏 评论按钮的占位符 */}

    <div className="IssueItem-content">
      <h3>
        <Link href={issue.url}>{issue.title}</Link>
      </h3>
      <div dangerouslySetInnerHTML={{ __html: issue.bodyHTML }} />

      {/* 稍后用于渲染评论列表的占位符 */}
    </div>
  </div>
);

export default IssueItem;
~~~~~~~~

> Once you start your application again, you should see the initial page of paginated issues rendered below each repository. That's a performance bottleneck. Worse, the GraphQL requests are not bundled in one request, as with the issues list field in the Organization and Profile components. In the next steps you are implementing client-side filtering. The default is to show no issues, but it can toggle between states of showing none, open issues, and closed issues using a button, so the issues will not be queried before toggling one of the issue states.

当你再次启动程序时，你应该会看到每个代码库下面渲染出了分页后 issue 的初始页面。这是个性能瓶颈，更糟糕的是，和 Organization 和 Profile 组件中的 issue 列表字段一样，GraphQL 请求没有合并在一个请求中。在接下来的步骤中，你将实现客户端的筛选。默认情况不显示任何 issue，但是它可以使用一个按钮在不显示，打开 issue 和关闭 issue 的状态之间切换，因此在切换 issue 状态之前不会查询 issue。

> ### Exercises:
### 练习

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/6781b487d6799e55a4deea48dfe706253b373f0a)
> * Read more about [the rate limit when using a (or in this case GitHub's) GraphQL API](https://developer.github.com/v4/guides/resource-limitations/)

* 确认[最后一节的代码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/6781b487d6799e55a4deea48dfe706253b373f0a)
* 阅读有关[使用（或者在本例中是 Github 的）GraphQL API 时的速率限制](https://developer.github.com/v4/guides/resource-limitations/)的更多信息

> ## Implementing the Issues Feature: Client-Side Filter

## 实现 Issue 功能: 客户端过滤

> In this section, we enhance the Issue feature with client-side filtering. It prevents the initial issue querying because it happens with a button, and it lets the user filter between closed and open issues.

在本节中，我们将用客户端筛选增强 issue 功能。它可以防止初始的 issue 查询，因为将由一个按钮触发，并且允许用户在已关闭和打开 issue 之间进行筛选。

> First, let's introduce our three states as enumeration next to the Issues component. The `NONE` state is used to show no issues; otherwise, the other states are used to show open or closed issues.

首先，让我介绍在 Issues 组件旁边的三个枚举状态，`NONE` 状态用于不显示 issue 列表，否则，其他状态用于显示打开或者关闭的 issue。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const ISSUE_STATES = {
  NONE: 'NONE',
  OPEN: 'OPEN',
  CLOSED: 'CLOSED',
};
~~~~~~~~

> Second, let's implement a short function that decides whether it is a state to show the issues or not. This function can be defined in the same file.

其次，让我们实现一个简短函数来决定 issue 是否显示的状态。这个函数可以在同一个文件中进行定义。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const isShow = issueState => issueState !== ISSUE_STATES.NONE;
~~~~~~~~

> Third, the function can be used for conditional rendering, to either query the issues and show the IssueList, or to do nothing. It's not clear yet where the `issueState` property comes from.

第三，该函数可以用于条件渲染，查询 issue 并显示 IssueList，或者什么也不做，目前还不清楚 `issueState` 属性的来源。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const Issues = ({ repositoryOwner, repositoryName }) => (
  <div className="Issues">
# leanpub-start-insert
    {isShow(issueState) && (
# leanpub-end-insert
      <Query ... >
        ...
      </Query>
    )}
  </div>
);
~~~~~~~~

> The `issueState` property must come from the local state to toggle it via a button in the component, so the Issues component must be refactored to a class component to manage this state.

`issueState` 属性必须来自组件内状态，才能通过组件内的按钮切换它，因此必须将 Issues 组件重构为一个类组件来管理这个状态。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
class Issues extends React.Component {
  state = {
    issueState: ISSUE_STATES.NONE,
  };

  render() {
    const { issueState } = this.state;
    const { repositoryOwner, repositoryName } = this.props;

    return (
      <div className="Issues">
        {isShow(issueState) && (
          <Query ... >
            ...
          </Query>
        )}
      </div>
    );
  }
}
# leanpub-end-insert
~~~~~~~~

> The application should be error-free now, because the initial state is set to `NONE` and the conditional rendering prevents the query and the rendering of a result. However, the client-side filtering is not done yet, as you still need to toggle the `issueState` property with React's local state. The ButtonUnobtrusive component has the appropriate style, so we can reuse it to implement this toggling behavior to transition between the three available states.

程序现在应该是无错的，因为初始状态被设置为 `NONE`，条件渲染阻止查询和结果渲染。但是，客户端的筛选还未完成，因为你仍然需要使用 React 的组件状态来切换 `issueState` 属性。ButtonUnobtrusive 组件具有适合的样式，因此我们可以复用它来实现这种切换行为，以便在三种可用状态中进行转换。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
...

import IssueItem from '../IssueItem';
import Loading from '../../Loading';
import ErrorMessage from '../../Error';
# leanpub-start-insert
import { ButtonUnobtrusive } from '../../Button';
# leanpub-end-insert

class Issues extends React.Component {
  state = {
    issueState: ISSUE_STATES.NONE,
  };

# leanpub-start-insert
  onChangeIssueState = nextIssueState => {
    this.setState({ issueState: nextIssueState });
  };
# leanpub-end-insert

  render() {
    const { issueState } = this.state;
    const { repositoryOwner, repositoryName } = this.props;

    return (
      <div className="Issues">
# leanpub-start-insert
        <ButtonUnobtrusive
          onClick={() =>
            this.onChangeIssueState(TRANSITION_STATE[issueState])
          }
        >
          {TRANSITION_LABELS[issueState]}
        </ButtonUnobtrusive>
# leanpub-end-insert

        {isShow(issueState) && (
          <Query ... >
            ...
          </Query>
        )}
      </div>
    );
  }
}
~~~~~~~~

> In the last step, you introduced the button to toggle between the three states. You used two enumerations, `TRANSITION_LABELS` and `TRANSITION_STATE`, to show an appropriate button label and to define the next state after a state transition. These enumerations can be defined next to the `ISSUE_STATES` enumeration.

在最后一步中，你引入了用于在三种状态中切换的按钮，你使用了两个枚举，`TRANSITION_LABELS` 和 `TRANSITION_STATE` 来显示合适的按钮标签，并定义状态转换后的下一个状态。这些枚举可以在 `ISSUE_STATES` 旁边定义。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const TRANSITION_LABELS = {
  [ISSUE_STATES.NONE]: 'Show Open Issues',
  [ISSUE_STATES.OPEN]: 'Show Closed Issues',
  [ISSUE_STATES.CLOSED]: 'Hide Issues',
};

const TRANSITION_STATE = {
  [ISSUE_STATES.NONE]: ISSUE_STATES.OPEN,
  [ISSUE_STATES.OPEN]: ISSUE_STATES.CLOSED,
  [ISSUE_STATES.CLOSED]: ISSUE_STATES.NONE,
};
~~~~~~~~

> As you can see, whereas the former enumeration only matches a label to a given state, the latter enumeration matches the next state to a given state. That's how the toggling to a next state can be made simple. Last but not least, the `issueState` from the local state has to be used to filter the list of issues after they have been queried and should be rendered.

正如你所看到的，前一个枚举只将标签匹配到给定状态，而下一个枚举将下一个状态匹配到给定状态，这就是如何简单的切换到下一个状态的方法。最后一个重点是，在 issue 列表被查询并且渲染之后，必须使用组件内状态 `issueState` 来过滤 issue 列表。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
class Issues extends React.Component {
  ...

  render() {
    ...

    return (
      <div className="Issues">
        ...

        {isShow(issueState) && (
          <Query ... >
            {({ data, loading, error }) => {
              if (error) {
                return <ErrorMessage error={error} />;
              }

              const { repository } = data;

              if (loading && !repository) {
                return <Loading />;
              }

# leanpub-start-insert
              const filteredRepository = {
                issues: {
                  edges: repository.issues.edges.filter(
                    issue => issue.node.state === issueState,
                  ),
                },
              };
# leanpub-end-insert

# leanpub-start-insert
              if (!filteredRepository.issues.edges.length) {
# leanpub-end-insert
                return <div className="IssueList">No issues ...</div>;
              }

              return (
# leanpub-start-insert
                <IssueList issues={filteredRepository.issues} />
# leanpub-end-insert
              );
            }}
          </Query>
        )}
      </div>
    );
  }
}
~~~~~~~~

> You have implemented client-side filtering. The button is used to toggle between the three states managed in the local state of the component.  The issues are only queried in filtered and rendered states. In the next step, the existing client-side filtering should be advanced to a server-side filtering, which means the filtered issues are already requested from the server and not filtered afterward on the client.

你已经实现了客户端筛选，这个按钮用于在组件内状态中管理三个状态之间的切换。issue 只有在过滤和渲染状态下才会被查询。在下一步中，应该将现有的客户端过滤提升到服务端过滤，这意味着过滤后的 issue 是从服务端请求，而不是在客户端上过滤。

> ### Exercises:
### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/0f261b13696046832ad65f1909266957d6275d6c)
> * Install the [recompose](https://github.com/acdlite/recompose) library which implements many higher-order components
> * Refactor the Issues component from class component to functional stateless component
> * Use the `withState` HOC for the Issues component to manage the `issueState`

* 确查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/0f261b13696046832ad65f1909266957d6275d6c)
* 安装实现了很多高阶组件的 [recompose](https://github.com/acdlite/recompose) 库
* 将 Issues 组件从类组件重构为无状态组件
* 用 `withState` 高阶组件来管理 Issues 组件的 `issueState`

> ## Implementing the Issues Feature: Server-Side Filter
## 实现 Issues 功能: 服务端过滤

> Before starting with the server-side filtering, let's recap the last exercise in case you had difficulties with it. Basically you can perform the refactoring in three steps. First, install recompose as package for your application on the command line:

开始学习服务端过滤前，以防你遇到困难，让我们先回顾最近一次的练习。基本来说，我们可以分3个步骤来开始我们的重构。首先，在命令行为你的应用安装 recompose 依赖包。

{title="Command Line",lang="json"}
~~~~~~~~
npm install recompose --save
~~~~~~~~

> Second, import the `withState` higher-order component in the *src/Issue/IssueList/index.js* file and use it to wrap your exported Issues component, where the first argument is the property name in the local state, the second argument is the handler to change the property in the local state, and the third argument is the initial state for that property.

其次，导入位于 *src/Issue/IssueList/index.js* 的高阶组件 `withState`，然后用它包裹你导出的 Issues 组件，高阶组件第一个参数是当前 state 下的属性名字，第二个参数是改变当前 state 属性的 handler，第三个参数是属性的初始 state。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import { Query } from 'react-apollo';
import gql from 'graphql-tag';
# leanpub-start-insert
import { withState } from 'recompose';
# leanpub-end-insert

...

# leanpub-start-insert
export default withState(
  'issueState',
  'onChangeIssueState',
  ISSUE_STATES.NONE,
)(Issues);
# leanpub-end-insert
~~~~~~~~

>  Finally, refactor the Issues component from a class component to a functional stateless component. It accesses the `issueState` and `onChangeIssueState()` function in its props now. Remember to change the usage of the `onChangeIssueState` prop to being a function and not a class method anymore.

最后，将 Issues 组件从类组件重构为函数式无状态组件。 现在，它可以访问 `props` 中的 `issueState` 和 `onChangeissueState()` 函数。记得将 `onChangeIssueState` 的使用方式修改为函数而不再是类的方法调用。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const Issues = ({
  repositoryOwner,
  repositoryName,
  issueState,
  onChangeIssueState,
}) => (
# leanpub-end-insert
  <div className="Issues">
    <ButtonUnobtrusive
# leanpub-start-insert
      onClick={() => onChangeIssueState(TRANSITION_STATE[issueState])}
# leanpub-end-insert
    >
      {TRANSITION_LABELS[issueState]}
    </ButtonUnobtrusive>

    ...
  </div>
# leanpub-start-insert
);
# leanpub-end-insert

...
~~~~~~~~

>  The previous section makes writing stateful components, where the state is much more convenient. Next, advance the filtering from client-side to server-side. We use the defined GraphQL query and its arguments to make a more exact query by requesting only open or closed issues. In the *src/Issue/IssueList/index.js* file, extend the query with a variable to specify the issue state:

在上一节使编写状态组件变得更加方便。接下来，改进过滤从客户端到服务端。我们使用已定义的  GraphQL  查询和它的参数通过请求仅打开或关闭的 issues 来构造更加精确的查询。在文件 *src/Issue/IssueList/index.js* 中，通过指定 issue 状态扩展查询属性。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = gql`
  query(
    $repositoryOwner: String!
    $repositoryName: String!
# leanpub-start-insert
    $issueState: IssueState!
# leanpub-end-insert
  ) {
    repository(name: $repositoryName, owner: $repositoryOwner) {
# leanpub-start-insert
      issues(first: 5, states: [$issueState]) {
# leanpub-end-insert
        edges {
          node {
            id
            number
            state
            title
            url
            bodyHTML
          }
        }
      }
    }
  }
`;
~~~~~~~~

> Next, you can use the `issueState` property as variable for your Query component. In addition, remove the client-side filter logic from the Query component's render prop function.

接下来，你可以使用 `issueState`  作为你的 Query 组件的查询属性。此外，从 Query 组件的 render props 函数中移除客户端过滤逻辑。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const Issues = ({
  repositoryOwner,
  repositoryName,
  issueState,
  onChangeIssueState,
}) => (
  <div className="Issues">
    ...

    {isShow(issueState) && (
      <Query
        query={GET_ISSUES_OF_REPOSITORY}
        variables={{
          repositoryOwner,
          repositoryName,
# leanpub-start-insert
          issueState,
# leanpub-end-insert
        }}
      >
        {({ data, loading, error }) => {
          if (error) {
            return <ErrorMessage error={error} />;
          }

          const { repository } = data;

          if (loading && !repository) {
            return <Loading />;
          }

          return <IssueList issues={repository.issues} />;
        }}
      </Query>
    )}
  </div>
);
~~~~~~~~

> You are only querying open or closed issues. Your query became more exact, and the filtering is no longer handled by the client.

你仅在查询打开或关闭状态的issues。你的查询变得更加精确，客户端也不再处理筛选。

> ### Exercises:

### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/df737276a4bc8d2d889d182937b77ba9e474e70c)
> * Implement the pagination feature for the Issue feature
>   * Add the pageInfo information to the query
>   * Add the additional cursor variable and argument to the query
>   * Add the FetchMore component to the IssueList component

* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/df737276a4bc8d2d889d182937b77ba9e474e70c)
* 为 issue 实现分页
  * 为查询新增 pageInfo 信息
  * 为查询新增额外的锚点属性和参数
  * 新增 FetchMore 组件到 IssueList 组件中

> ##  Apollo Client Prefetching in React
## React 中 Apollo 客户端预加载
> This section is all about prefetching data, though the user doesn't need it immediately. It is another UX technique that can be deployed to the optimistic UI technique you used earlier. You will implement the prefetching data feature for the list of issues, but feel free to implement it for other data fetching later as your exercise.

本节全是关于数据预加载的内容，虽然用户并不一定会立刻用到。它是另一种 UX 技术，可以应用在你之前使用过的乐观 UI 上。你将能在 issues 列表中实现数据预加载的特性，不过作为你的练习，稍后你可以随意实现其他的数据预加载。

> When your application renders for the first time, there no issues fetched, so no issues are rendered. The user has to toggle the filter button to fetch open issues, and do it again to fetch closed issues. The third click will hide the list of issues again. The goal of this section is to prefetch the next bulk of issues when the user hovers the filter button. For instance, when the issues are still hidden and the user hovers the filter button, the issues with the open state are prefetched in the background. When the user clicks the button, there is no waiting time, because the issues with the open state are already there. The same scenario applies for the transition from open to closed issues. To prepare this behavior, split out the filter button as its own component in the *src/Issue/IssueList/index.js* file:

当你的应用首次渲染的时候，没有已经请求到的 issues，因此不会渲染任何 issues。用户必须切换筛选按钮来请求打开的 issues，然后再次切换筛选按钮来请求关闭的 issues。第三次点击将会再次隐藏列表的 issues。本章节的目标是当用户悬浮在筛选按钮上时预加载下一批的 issues。例如，当 issues 仍然是在隐藏时，用户悬浮在筛选按钮上，打开状态的 issues 将会在后台预加载。当用户点击筛选按钮，由于打开状态的 issues 已经获取到，因此不会再有等待时间。同样的情况适用于 issues 打开到关闭的过渡。为了给这种行为做准备，在文件 *src/Issue/IssueList/index.js* 中，分离筛选按钮作为自己单独的组件。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const Issues = ({
  repositoryOwner,
  repositoryName,
  issueState,
  onChangeIssueState,
}) => (
  <div className="Issues">
# leanpub-start-insert
    <IssueFilter
      issueState={issueState}
      onChangeIssueState={onChangeIssueState}
    />
# leanpub-end-insert

    {isShow(issueState) && (
      ...
    )}
  </div>
);

# leanpub-start-insert
const IssueFilter = ({ issueState, onChangeIssueState }) => (
  <ButtonUnobtrusive
    onClick={() => onChangeIssueState(TRANSITION_STATE[issueState])}
  >
    {TRANSITION_LABELS[issueState]}
  </ButtonUnobtrusive>
);
# leanpub-end-insert
~~~~~~~~

> Now it is easier to focus on the IssueFilter component where most of the logic for data prefetching is implemented. Like before, the prefetching should happen when the user hovers over the button. There needs to be a prop for it, and a callback function which is executed when the user hovers over it. There is such a prop (attribute) for a button (element). We are dealing with HTML elements here.

现在，当大部分关于数据预加载的逻辑已经实现，更容易将注意力集中在 IssueFilter 组件。像之前一样，预加载应该发生在用户悬浮在按钮上的时候。它需要一个属性和一个回调函数，当用户悬停在它上面时执行该函数。这是按钮元素的属性。我们将在这里处理 HTML 元素。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const prefetchIssues = () => {};
# leanpub-end-insert

...

const IssueFilter = ({ issueState, onChangeIssueState }) => (
  <ButtonUnobtrusive
    onClick={() => onChangeIssueState(TRANSITION_STATE[issueState])}
# leanpub-start-insert
    onMouseOver={prefetchIssues}
# leanpub-end-insert
  >
    {TRANSITION_LABELS[issueState]}
  </ButtonUnobtrusive>
);
~~~~~~~~

> The `prefetchIssue()` function has to execute the identical GraphQL query executed by the Query component in the Issues component, but this time it is done in an imperative way instead of declarative. Rather than using the Query component for it, use the the Apollo Client instance directly to execute a query. Remember, the Apollo Client instance is hidden in the component tree, because you used React's Context API to provide the Apollo Client instance the component tree's top level. The Query and Mutation components have access to the Apollo Client, even though you have never used it yourself directly. However, this time you use it to query the prefetched data. Use the ApolloConsumer component from the React Apollo package to expose the Apollo Client instance in your component tree. You have used the ApolloProvider somewhere to provide the client instance, and you can use the ApolloConsumer to retrieve it now. In the *src/Issue/IssueList/index.js* file, import the ApolloConsumer component and use it in the IssueFilter component. It gives you access to the Apollo Client instance via its render props child function.

函数 `prefetchIssue() ` 必须通过 Issues 组件中的 Query 组件，执行同样的 GraphQL 查询，但这一次它是以命令式的方式完成，而不是声明式的。与其使用 Query 组件，不如直接使用 Apollo 客户端实例来执行查询。记住， Apollo 客户端实例隐藏在组件树中，因为你使用了React的 Context API 在组件树的顶层为其提供 Apollo 客户端实例。查询和突变组件可以访问 Apollo 客户端，即使您从未直接使用过它。但是，这次你可以使用它来查询预加载的数据。使用 React Apollo 包中的 ApolloConsumer 组件在组件树中暴露 Apollo 客户端实例。你已经在某个地方使用了 ApolloProvider 来提供客户端实例，现在可以使用 ApolloConsumer 来取到它。在文件 *src/Issue/IssueList/index.js* 中，导入 ApolloConsumer 组件并在 IssueFilter 组件中使用它。它允许你通过其渲染 props 子函数访问 Apollo 客户端实例。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import { Query, ApolloConsumer } from 'react-apollo';
# leanpub-end-insert
import gql from 'graphql-tag';
import { withState } from 'recompose';

...

const IssueFilter = ({ issueState, onChangeIssueState }) => (
# leanpub-start-insert
  <ApolloConsumer>
    {client => (
# leanpub-end-insert
      <ButtonUnobtrusive
        onClick={() =>
          onChangeIssueState(TRANSITION_STATE[issueState])
        }
# leanpub-start-insert
        onMouseOver={() => prefetchIssues(client)}
# leanpub-end-insert
      >
        {TRANSITION_LABELS[issueState]}
      </ButtonUnobtrusive>
# leanpub-start-insert
    )}
  </ApolloConsumer>
# leanpub-end-insert
);
~~~~~~~~

> Now you have access to the Apollo Client instance to perform queries and mutations, which will enable you to query GitHub's GraphQL API imperatively. The variables needed to perform the prefetching of issues are the same ones used in the Query component. You need to pass those to the IssueFilter component, and then to the `prefetchIssues()` function.

现在，您可以访问 Apollo 客户端实例来执行查询和变更，这将使您能够强制查询 Github 的 GraphQL API。执行 issues 预加载所需的变量与查询组件中使用的变量相同。你需要将这些传递到 IssueFilter 组件，然后传递给函数 `prefetchIssues()`。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
...

const Issues = ({
  repositoryOwner,
  repositoryName,
  issueState,
  onChangeIssueState,
}) => (
  <div className="Issues">
    <IssueFilter
# leanpub-start-insert
      repositoryOwner={repositoryOwner}
      repositoryName={repositoryName}
# leanpub-end-insert
      issueState={issueState}
      onChangeIssueState={onChangeIssueState}
    />

    {isShow(issueState) && (
      ...
    )}
  </div>
);

const IssueFilter = ({
# leanpub-start-insert
  repositoryOwner,
  repositoryName,
# leanpub-end-insert
  issueState,
  onChangeIssueState,
}) => (
  <ApolloConsumer>
    {client => (
      <ButtonUnobtrusive
        onClick={() =>
          onChangeIssueState(TRANSITION_STATE[issueState])
        }
        onMouseOver={() =>
          prefetchIssues(
            client,
# leanpub-start-insert
            repositoryOwner,
            repositoryName,
            issueState,
# leanpub-end-insert
          )
        }
      >
        {TRANSITION_LABELS[issueState]}
      </ButtonUnobtrusive>
    )}
  </ApolloConsumer>
);

...
~~~~~~~~

> Use this information to perform the prefetching data query. The Apollo Client instance exposes a `query()` method for this. Make sure to retrieve the next `issueState`, because when prefetching open issues, the current `issueState` should be `NONE`.

使用此信息执行数据预加载查询。Apollp 客户端实例为此暴露了一个 `query()` 方法。确保得到下一个 `issueState`，因为预加载打开的 issues 时，当前的  `issueState`  应为 `NONE`。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const prefetchIssues = (
# leanpub-start-insert
  client,
  repositoryOwner,
  repositoryName,
  issueState,
# leanpub-end-insert
) => {
# leanpub-start-insert
  const nextIssueState = TRANSITION_STATE[issueState];

  if (isShow(nextIssueState)) {
    client.query({
      query: GET_ISSUES_OF_REPOSITORY,
      variables: {
        repositoryOwner,
        repositoryName,
        issueState: nextIssueState,
      },
    });
  }
# leanpub-end-insert
};
~~~~~~~~

> That's it. Once the button is hovered, it should prefetch the issues for the next `issueState`. The Apollo Client makes sure that the new data is updated in the cache like it would do for the Query component. There shouldn't be any visible loading indicator in between except when the network request takes too long and you click the button right after hovering it. You can verify that the request is happening in your network tab in the developer development tools of your browser. In the end, you have learned about two UX improvements that can be achieved with ease when using Apollo Client: optimistic UI and prefetching data.

就是这样。一旦该按钮悬停，它将预加载下一个 `issueState` 的 issues。Apollo 客户端确保新数据更新到缓存，就像对 Query 组件所做的那样。中间不应该有任何可见的加载指示器，除非网络请求花费了太长时间，并且你在悬停之后单击了按钮。你可以验证请求是否在浏览器的开发者工具的网络选项卡中发生。在最后，你已经了解了使用 Apollo 客户端可以轻松实现的两个 UX 改进：乐观 UI 和数据预加载。

> ### Exercises:
### 练习：
> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/87dc6eee7948dad6e1eb4c15078063337eff94db)
> * Read more about [Apollo Prefetching and Query Splitting in React](https://www.apollographql.com/docs/react/recipes/performance.html)
> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5PLMBR3)

* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/87dc6eee7948dad6e1eb4c15078063337eff94db)

* 延伸阅读：[Apollo 在 React 中预加载和查询分离](https://www.apollographql.com/docs/react/recipes/performance.html)

* 花三分钟的时间来做一个[测验](https://www.surveymonkey.com/r/5PLMBR3)

> ## Exercise: Commenting Feature
## 练习：评论功能

> This last section is for hands-on experience with the application and implementing features yourself. I encourage you to continue implementing features for the application and improving it. There are a couple of guiding points to help you implementing the Commenting feature. In the end it should be possible to show a list of paginated comments per issue on demand. Finally, a user should be able to leave a comment. The source code of the implemented feature can be found [here](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/c689a90d43272bbcb64c05f85fbc84ad4fe4308d).

最后一部分是亲自实践这个应用并实现其相关功能。我鼓励你继续实现并改进该应用的功能。这里有一些引导点可以帮助你来实现评论功能。在末尾，这个功能应该按要求展示出每个 issue 的评论分页列表。最后，用户可以留言。你可以在[这里](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/c689a90d43272bbcb64c05f85fbc84ad4fe4308d)找到已实现功能的源码。

> * Introduce components for fetching a list of comments (e.g. Comments), rendering a list of comments (e.g. CommentList), and rendering a single comment (e.g. CommentItem). They can render sample data for now.

* 介绍获取评论列表的组件(例如：Comments)，渲染评论列表的组件(例如：CommentList)，渲染一条评论的组件(例如：CommentItem)，这些组件现在可以渲染样本数据。

> * Use the top level comments component (e.g. Comments), which will be your container component that is responsible to query the list of comments, in the *src/Issue/IssueItem/index.js* file. In addition, add a toggle to either show or hide comments. The IssueItem component has to become a class component or needs to make use of the `withState` HOC from the recompose library.

* 使用 *src/Issue/IssueItem/index.js* 文件中最顶层的评论组件(例如：Comments)，它会是负责查询评论列表的容器组件。此外，添加一个切换展示或隐藏评论的按钮。IssueItem 组件必须转为类组件或利用 recompose 库中的 `withState` 高阶组件。

> * Use the Query component from React Apollo in your container Comments component to fetch a list of comments. It should be similar to the query that fetches the list of issues. You only need to identify the issue for which the comments should be fetched.

* 在你的容器组件：评论组件中用 React Apollo 的 Query 组件来获取评论列表。这跟查询获取 issues 列表很相似。你只需确定哪些评论是应该被获取的。

> * Handle all edge cases in the Comments to show loading indicator, no data, or error messages. Render the list of comments in the CommentList component and a single comment in the CommentItem component.

* 处理评论组件( Comments )中的所有边界情况：展示加载标识、无数据、或错误信息。在 CommentList 组件中渲染评论列表，在 CommentItem 组件中渲染单条评论。

> * Implement the pagination feature for comments. Add the necessary fields in the query, the additional props and variables to the Query component, and the reusable FetchMore component. Handle the merging of the state in the `updateQuery` prop.

* 实现评论的分页功能。在查询操作中添加必要的字段，在 Query 组件和可复用的 FetchMore 组件中添加附加的属性和变量。在 `updateQuery` 属性中处理状态的合并。

> * Enable prefetching of the comments when hovering the "Show/Hide Comments" button.

* 允许在鼠标停留于 "Show/Hide Comments" 按钮时对评论数据的预加载。

> * Implement an AddComment component that shows a `textarea` and a `submit` button to enable user comments. Use the `addComment` mutation from GitHub's GraphQL API and the Mutation component from React Apollo to execute the mutation with the submit button.

* 实现一个 AddComment 组件， 这个组件需要为用户提供一个 `textarea` 和 `submit` 按钮来留言。使用 GitHub 中 GraphQL API 的 `addComment` 变更和 React Apollo 中的 Mutation 组件来执行提交按钮的变更。

> * Improve the AddComment component with the optimistic UI feature (perhaps read again the [Apollo documentation about the optimistic UI with a list of items](https://www.apollographql.com/docs/react/features/optimistic-ui.html)). A comment should show up in the list of comments, even if the request is pending.

* 用乐观 UI 特性(你或许想再读一遍[乐观 UI 结合列表项的 Apollo 文档](https://www.apollographql.com/docs/react/features/optimistic-ui.html))来优化 AddComment 组件。哪怕是请求还在继续，评论列表中也要展示出评论。

> I hope this section, building your own feature in the application with all the learned tools and techniques, matched your skills and challenged you to implement React applications with Apollo and GraphQL. I would recommend working to improve and extend the existing application. If you haven't implemented a GraphQL server yet, find other third-party APIs that offer a GraphQL API and build your own React with Apollo application by consuming it. Keep yourself challenged to grow your skills as a developer.

在这部分中，我希望你通过掌握的工具及方法并结合你的技能与遇到的挑战，去实现 Apollo 和 GraphQL 结合 React 的应用，在其中构建你自己的特性。我会建议你提升并且拓展当前已有的应用。如果你到目前为止还没有实现 GraphQL 服务端，就找一个提供 GraphQL API 的第三方 API 库，用它来构建一个你自己的 React 结合 Apollo 的应用。作为一个开发者，不断挑战自己，提升自己的技能吧。

> ## Appendix: CSS Files and Styles
## 附录: CSS 文件与样式 

> This section has all the CSS files as well as their content and locations, to give your React with GraphQL and Apollo Client application a nice touch. It even makes it responsive for mobile and tablet devices. These are only recommendations, though; you can experiment with them, or come up with your own styles.

这个部分涵盖了所有 CSS 文件的内容与文件存放路径，给你的集成了 GraphQL 和 Apollo 客户端的 React 应用之旅带来一场绝妙体验。这些样式甚至可以在手机和平板设备上适配。当然了，这不过是些建议，你也可以完全自己写样式来达到你想要的效果。

{title="src/style.css",lang="css"}
~~~~~~~~
#root,
html,
body {
  height: 100%;
}

body {
  margin: 0;
  padding: 0;
  font-family: 'Source Sans Pro', sans-serif;
  font-weight: 200;
  text-rendering: optimizeLegibility;
}

h2 {
  font-size: 24px;
  font-weight: 600;
  line-height: 34px;
  margin: 5px 0;
}

h3 {
  font-size: 20px;
  font-weight: 400;
  line-height: 27px;
  margin: 5px 0;
}

ul,
li {
  list-style: none;
  padding-left: 0;
}

a {
  text-decoration: none;
  color: #000;
  opacity: 1;
  transition: opacity 0.25s ease-in-out;
}

a:hover {
  opacity: 0.35;
  text-decoration: none;
}

a:active {
  text-decoration: none;
}

pre {
  white-space: pre-wrap;
}
~~~~~~~~

{title="src/App/style.css",lang="css"}
~~~~~~~~
.App {
  min-height: 100%;
  display: flex;
  flex-direction: column;
}

.App-main {
  flex: 1;
}

.App-content_large-header,
.App-content_small-header {
  margin-top: 54px;
}

@media only screen and (max-device-width: 480px) {
  .App-content_large-header {
    margin-top: 123px;
  }

  .App-content_small-header {
    margin-top: 68px;
  }
}
~~~~~~~~

{title="src/App/Navigation/style.css",lang="css"}
~~~~~~~~
.Navigation {
  overflow: hidden;
  position: fixed;
  top: 0;
  width: 100%;
  z-index: 1;
  background-color: #24292e;
  display: flex;
  align-items: baseline;
}

@media only screen and (max-device-width: 480px) {
  .Navigation {
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
}

.Navigation-link {
  font-size: 12px;
  letter-spacing: 3.5px;
  font-weight: 500;
  text-transform: uppercase;
  padding: 20px;
  text-decoration: none;
}

.Navigation-link a {
  color: #ffffff;
}

.Navigation-search {
  padding: 0 10px;
}

@media only screen and (max-device-width: 480px) {
  .Navigation-link {
    padding: 10px;
  }

  .Navigation-search {
    padding: 10px 10px;
  }
}
~~~~~~~~

{title="src/Button/style.css",lang="css"}
~~~~~~~~
.Button {
  padding: 10px;
  background: none;
  cursor: pointer;
  transition: color 0.25s ease-in-out;
  transition: background 0.25s ease-in-out;
}

.Button_white {
  border: 1px solid #fff;
  color: #fff;
}

.Button_white:hover {
  color: #000;
  background: #fff;
}

.Button_black {
  border: 1px solid #000;
  color: #000;
}

.Button_black:hover {
  color: #fff;
  background: #000;
}

.Button_unobtrusive {
  padding: 0;
  color: #000;
  background: none;
  border: none;
  cursor: pointer;
  opacity: 1;
  transition: opacity 0.25s ease-in-out;
  outline: none;
}

.Button_unobtrusive:hover {
  opacity: 0.35;
}

.Button_unobtrusive:focus {
  outline: none;
}
~~~~~~~~

{title="src/Error/style.css",lang="css"}
~~~~~~~~
.ErrorMessage {
  margin: 20px;
  display: flex;
  justify-content: center;
}
~~~~~~~~

{title="src/FetchMore/style.css",lang="css"}
~~~~~~~~
.FetchMore {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.FetchMore-button {
  margin: 20px 0;
}
~~~~~~~~

{title="src/Input/style.css",lang="css"}
~~~~~~~~
.Input {
  border: none;
  padding: 10px;
  background: none;
  outline: none;
}

.Input:focus {
  outline: none;
}

.Input_white {
  border-bottom: 1px solid #fff;
  color: #fff;
}

.Input_black {
  border-bottom: 1px solid #000;
  color: #000;
}
~~~~~~~~

{title="src/Issue/IssueItem/style.css",lang="css"}
~~~~~~~~
.IssueItem {
  margin-bottom: 10px;
  display: flex;
  align-items: baseline;
}

.IssueItem-content {
  margin-left: 10px;
  padding-left: 10px;
  border-left: 1px solid #000;
}
~~~~~~~~

{title="src/Issue/IssueList/style.css",lang="css"}
~~~~~~~~
.Issues {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 0 20px;
}

.Issues-content {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
}

.IssueList {
  margin: 20px 0;
}

@media only screen and (max-device-width: 480px) {
  .Issues-content {
    align-items: center;
  }
}
~~~~~~~~

{title="src/Loading/style.css",lang="css"}
~~~~~~~~
.LoadingIndicator {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 20px 0;
}

.LoadingIndicator_center {
  margin-top: 30%;
}
~~~~~~~~

{title="src/Repository/style.css",lang="css"}
~~~~~~~~
.RepositoryItem {
  padding: 20px;
  border-bottom: 1px solid #000;
}

.RepositoryItem-title {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
}

@media only screen and (max-device-width: 480px) {
  .RepositoryItem-title {
    flex-direction: column;
    align-items: center;
  }
}

.RepositoryItem-title-action {
  margin-left: 10px;
}

.RepositoryItem-description {
  margin: 10px 0;
  display: flex;
  justify-content: space-between;
}

@media only screen and (max-device-width: 480px) {
  .RepositoryItem-description {
    flex-direction: column;
    align-items: center;
  }
}

.RepositoryItem-description-info {
  margin-right: 20px;
}

@media only screen and (max-device-width: 480px) {
  .RepositoryItem-description-info {
    text-align: center;
    margin: 20px 0;
  }
}

.RepositoryItem-description-details {
  text-align: right;
  white-space: nowrap;
}

@media only screen and (max-device-width: 480px) {
  .RepositoryItem-description-details {
    text-align: center;
  }
}
~~~~~~~~

| |

> You can find the final [repository on GitHub](https://github.com/rwieruch/react-graphql-github-apollo) that showcases most of the exercise tasks. The application is not feature-complete and it doesn't cover all edge cases, but it should give insight into using GraphQL with Apollo in React applications. If you want to dive more deeply into different topics like testing and state management with GraphQL on the client-side, you can start here: [A minimal Apollo Client in React Example](https://www.robinwieruch.de/react-apollo-client-example). Try to apply what you've learned in this application (e.g. testing, state management). Otherwise, I encourage you to try to build your own GraphQL client library, which helps you understand more of the GraphQL internals: [How to build a GraphQL client library for React](https://www.robinwieruch.de/react-graphql-client-library). Whichever you decide, keep tinkering on this application, or start with another GraphQL client application to fortify your skill set. You have finished all the GraphQL client chapters now.

你可以在 [Github的代码库](https://github.com/rwieruch/react-graphql-github-apollo)中发现，绝大部分的练习任务都已被陈列出来。尽管这些案例的功能并不完善，也没有覆盖到涉及边界的所有情况，但它应该表达出了对 React 应用中与 Apollo 一起使用 GraphQL 的深刻理解。假如你想钻研更多类似于在客户端中使用 GraphQL 进行测试和状态管理等深层次主题的话，你可以从这里开始：[Apollo 客户端在 React 中的小例子](https://www.robinwieruch.de/react-apollo-client-example)。试着在这个应用结合你所学到的东西(例如：测试、状态管理)，否则的话，我更支持你去尝试着构建一个自己的 GraphQL 客户端库，[如何为 React 构建一个 GraphQL 客户端库](https://www.robinwieruch.de/react-graphql-client-library) 可以让你了解到更多 GraphQL 的内部构件，不管你最后怎么决定，记住要对这个应用反复锤炼，或者再另启一个 GraphQL 客户端应用来加强你的综合技能，做到学以致用。至此，你已经完成了 GraphQL 客户端的所有章节。