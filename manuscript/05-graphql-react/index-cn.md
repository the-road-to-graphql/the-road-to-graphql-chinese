# React 和 GraphQL 的结合

我们将一起开发一个 GraphQL 的客户端程序，从中你将了解如何将 React 与 GraphQL 结合起来。目前我们不会使用 [Apollo Client](https://github.com/apollographql/apollo-client) 或 [Relay](https://github.com/facebook/relay) 这样强大的工具来帮助你快速上手，而是使用基本的 HTTP 请求执行 GraphQL 查询和变更。在之后的下一个应用程序中，我们将引入 Apollo 作为你的 React.js 应用的 GraphQL 客户端。现阶段开发的应用只展示如何在 React 中基于 HTTP 使用 GraphQL。

在此过程中，你将构建一个类似 GitHub 的问题跟踪器，通过执行 GraphQL 的查询和变更来读写数据，基于 [GitHub's GraphQL API](https://developer.github.com/v4/) 的简单 GitHub 客户端。最终实现一个能够在 React 示例中展示 GraphQL的案例，也可以将其作为其他开发人员的学习工具，最终实现的应用程序可以参考 [GitHub 的代码库](https://github.com/rwieruch/react-graphql-github-vanilla)。

## 编写你的第一个 React GraphQL 客户端

在上一节之后，你应该准备好了如何在 React 应用程序中使用查询和变更。在本节中，你将创建一个使用 GitHub GraphQL API 的 React 应用程序。它是一个简单的问题跟踪器，需要显示在 GitHub 代码库中的一些 open issues，如果你对 React 缺乏经验，可以阅读 [React 学习之道](https://www.robinwieruch.de/the-road-to-learn-react)，了解更多相关知识，为接下来的部分做好充分的准备。

现阶段的应用程序，不需要复杂的 React 配置。你只需使用 [create-react-app](https://github.com/facebook/create-react-app) 就可以创建无需额外配置的 React 应用程序。在命令行中输入以下指令，用npm安装它：`npm install -g create-react-app`。如果你想学习详细的 React 设置，请参考 [React 的 Webpack 设置指南](https://www.robinwieruch.de/minimal-react-webpack-babel-setup/)。

现在，让我们使用 create-react-app 创建应用程序。进入你常规的项目文件夹中，输入如下命令：

{title="Command Line",lang="json"}
~~~~~~~~
create-react-app react-graphql-github-vanilla
cd react-graphql-github-vanilla
~~~~~~~~

创建应用程序之后，可以使用 `npm start` 和 `npm test` 对其进行测试。在学习了简单的 React 之后，你应该熟悉 npm，create-react-app 和 React。

接下来我们将主要关注在 src/App.js 文件上，你也可以将组件、配置或函数拆分到它们自己的文件夹和文件中。我们从上述文件中的 App 组件开始，为了简化它，你可以将其更改为如下内容：

{title="src/App.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';

const TITLE = 'React GraphQL GitHub Client';

class App extends Component {
  render() {
    return (
      <div>
        <h1>{TITLE}</h1>
      </div>
    );
  }
}

export default App;
~~~~~~~~

现阶段组件只会渲染一个 `title` 作为标题。在实现其他的 React 组件之前，我们先安装一个使用 HTTP POST 方法执行查询和变更的第三方库来处理 GraphQL 请求。推荐使用 [axios](https://github.com/axios/axios)。在命令行中，输入如下命令在项目文件夹中安装 axios：

{title="Command Line",lang="json"}
~~~~~~~~
npm install axios --save
~~~~~~~~

接下来，可以在你的 App 组件中导入并配置它。它可以简化后续的开发步骤，因为在某种程度上，你只需要用你的 access token 和 GitHub 的 GraphQL API 配置它一次。

首先，在从 axios 创建配置实例时，为它定义一个基本的 URL。如前所述，你不需要在每次发出请求时都定义 GitHub 的 URL 端点，因为所有查询和变更都指向 GraphQL 中的相同 URL 端点。你可以使用对象和字段灵活地进行查询和变更结构。

{title="src/App.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import axios from 'axios';

const axiosGitHubGraphQL = axios.create({
  baseURL: 'https://api.github.com/graphql',
});
# leanpub-end-insert

...

export default App;
~~~~~~~~

接下来，将 access token 作为 header 传递到配置中。使用这个 axios 实例发出的每个请求都会使用这个 header。

{title="src/App.js",lang="javascript"}
~~~~~~~~
...

const axiosGitHubGraphQL = axios.create({
  baseURL: 'https://api.github.com/graphql',
# leanpub-start-insert
  headers: {
    Authorization: 'bearer YOUR_GITHUB_PERSONAL_ACCESS_TOKEN',
  },
# leanpub-end-insert
});

...
~~~~~~~~

用你的 access token 替换 `YOUR_GITHUB_PERSONAL_ACCESS_TOKEN` 字符串。为了避免将访问令牌直接剪切和粘贴到源代码中，你可以创建一个 *.env* 文件来保存项目文件夹中命令行上的所有环境变量。如果不想在公开的 GitHub 代码库中共享个人令牌，可以将该文件添加到 .gitignore。

{title="Command Line",lang="json"}
~~~~~~~~
touch .env
~~~~~~~~

将环境变量定义在这个 *.env* 文件中。在使用 create-react-app 时，确保遵循正确的命名约束，它使用 `REACT_APP` 作为每个 key 的前缀。将下面的键值对粘贴在你的 *.env* 文件中。密钥必须有 `REACT_APP` 前缀，并且值必须是你 GitHub 的 access token。

{title=".env",lang="javascript"}
~~~~~~~~
REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN=xxxXXX
~~~~~~~~

现在，你可以将 access token 作为环境变量传递给 axios 配置，并使用字符串插值（[模板字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)创建一个配置好的axios实例。

{title="src/App.js",lang="javascript"}
~~~~~~~~
...

const axiosGitHubGraphQL = axios.create({
  baseURL: 'https://api.github.com/graphql',
  headers: {
# leanpub-start-insert
    Authorization: `bearer ${
      process.env.REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN
    }`,
# leanpub-end-insert
  },
});

...
~~~~~~~~

axios 的初始设置基本上与我们之前使用 GraphiQL 应用程序访问 GitHub 的 GraphQL API 时所完成的设置相同，那时你还必须设置带有 access token 和端点 URL 的头文件。

接下来需要添加一个表单，用于从用户处获得关于 GitHub 组织和代码库的详细信息。这个表单包含一个输入栏，可以为特定的 GitHub 代码库请求一个可以分页的 issues 列表。首先，需要一个带有输入栏的表单来输入组织和代码库。其次，表单需要一个提交按钮来请求用户在输入字段中提供的组织和代码库的数据，并且将这些数据存放在于组件的本地状态中。接下来，当组件第一次挂载时，组织和代码库有一个初始的本地状态来请求初始数据。

我们可以分两步来实现这个场景。render 方法需要渲染一个带有输入栏的表单。表单必须有一个 `onSubmit` 处理方法，而输入栏需要一个 `onChange` 处理方法。输入栏使用组件中 state 的 `path` 作为值成为一个受控组件。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  render() {
    return (
      <div>
        <h1>{TITLE}</h1>

        <form onSubmit={this.onSubmit}>
          <label htmlFor="url">
            Show open issues for https://github.com/
          </label>
          <input
            id="url"
            type="text"
            onChange={this.onChange}
            style={{ width: '300px' }}
          />
          <button type="submit">Search</button>
        </form>

        <hr />

        {/* Here comes the result! */}
      </div>
    );
  }
}
~~~~~~~~

声明要在 render 方法中使用的类方法。`componentDidMount()` 生命周期方法可用于在应用程序组件挂载时发出初始请求。在这个生命周期方法中，需要发出初始化请求为输入栏设置一个初始值。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  state = {
    path: 'the-road-to-learn-react/the-road-to-learn-react',
  };

  componentDidMount() {
    // fetch data
  }

  onChange = event => {
    this.setState({ path: event.target.value });
  };

  onSubmit = event => {
    // fetch data

    event.preventDefault();
  };

  render() {
    ...
  }
}
~~~~~~~~

你可能在前面的实现中使用了以前未使用过的 React 类组件语法。如果你不熟悉它，请查看这个 [GitHub 代码库](https://github.com/the-road-to-learn-react/react-alternative-class-component-syntax)以获得更多的理解。使用**类字段声明**可以省略初始化本地状态的构造函数语句，而且不需要绑定类方法。相反，箭头函数将处理所有绑定行为。

依照 React 的最佳实践，将输入栏设置为受控组件。输入元素不应使用原生的 HTML 行为处理其内部状态；而应该是自然反应。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  render() {
# leanpub-start-insert
    const { path } = this.state;
# leanpub-end-insert

    return (
      <div>
        <h1>{TITLE}</h1>

        <form onSubmit={this.onSubmit}>
          <label htmlFor="url">
            Show open issues for https://github.com/
          </label>
          <input
            id="url"
            type="text"
# leanpub-start-insert
            value={path}
# leanpub-end-insert
            onChange={this.onChange}
            style={{ width: '300px' }}
          />
          <button type="submit">Search</button>
        </form>

        <hr />

        {/* Here comes the result! */}
      </div>
    );
  }
}
~~~~~~~~

为表单提前设置——可使用的输入栏、一个提交按钮、`onChange()` 和 `onSubmit()` 类方法——是 React 中实现表单的一种常见方法。唯一增加的是 `componentDidMount()` 生命周期方法中的初始数据获取，通过为查询提供一个初始状态来从后端请求数据，从而改进用户体验。它是 [React 获取第三方 API 数据](https://www.robinwieruch.de/react-fetching-data/)的基础。

When you start the application on the command line, you should see the initial state for the `path` in the input field. You should be able to change the state by entering something else in the input field, but nothing happens with `componentDidMount()` and submitting the form yet.

当你在命令行上启动应用程序时，应该会在输入栏中看到 `path` 的初始状态。并且能够通过在输入栏中输入其他内容来更改状态，但是在 `componentDidMount()` 方法中以及提交表单时什么都不会发生。

You might wonder why there is only one input field to grab the information about the organization and repository. When opening up a repository on GitHub, you can see that the organization and repository are encoded in the URL, so it becomes a convenient way to show the same URL pattern for the input field. You can also split the `organization/repository` later at the `/` to get these values and perform the GraphQL query request.

你可能想知道为什么只需要一个输入字段来获取关于组织和仓库的信息。在 GitHub 上打开代码库时，你可以看到组织和代码库都是在 URL 中编码的，因此可以方便地用输入的字段来匹配相似的 URL 模式。你之后也可以以 `/` 分割 `organization/repository`，以获取这些值并执行 GraphQL 查询请求。

### 练习：

* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/ca7b278b8f602c46dfac64a1304d39a8e8e0006b)

* 如果你不熟悉 React，可以阅读 *React 学习之道*

## 在 React 中执行 GraphQL 查询

在本章节中，你将在 React 中实现第一个 GraphQL 查询，从组织的一个代码库中获取 issues，但不是一次获取所有问题，先获取一个组织。我们先将查询定义为 App 组件上面的一个变量。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ORGANIZATION = `
  {
    organization(login: "the-road-to-learn-react") {
      name
      url
    }
  }
`;
~~~~~~~~

使用 JavaScript 中的模板字符串将查询语句定义为多行的字符串。与之前在 GraphiQL 或 GitHub Explorer 中使用的查询相同。现在你就可以使用 axios 向 GitHub 的 GraphiQL API 发出 POST 请求。axios 的配置已经指向正确的 API 端点，并使用的是你的 access token。唯一剩下的事情是在 POST 请求期间将查询作为有效负载传递给它。端点的参数可以是空字符串，因为你在配置中定义了端点。当 App 组件生命周期执行到 `componentDidMount()` 时，就会执行请求。在 axios 的 promise 被解析之后，结果将会保留在控制台日志中。

{title="src/App.js",lang="javascript"}
~~~~~~~~
...

const axiosGitHubGraphQL = axios.create({
  baseURL: 'https://api.github.com/graphql',
  headers: {
    Authorization: `bearer ${
      process.env.REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN
    }`,
  },
});

const GET_ORGANIZATION = `
  {
    organization(login: "the-road-to-learn-react") {
      name
      url
    }
  }
`;

class App extends Component {
  ...

  componentDidMount() {
# leanpub-start-insert
    this.onFetchFromGitHub();
# leanpub-end-insert
  }

  onSubmit = event => {
    // fetch data

    event.preventDefault();
  };

# leanpub-start-insert
  onFetchFromGitHub = () => {
    axiosGitHubGraphQL
      .post('', { query: GET_ORGANIZATION })
      .then(result => console.log(result));
  };
# leanpub-end-insert

  ...
}
~~~~~~~~

你只需要使用 axios 执行一个 HTTP POST 请求，并将 GraphQL 查询作为有效负载。因为 axios 使用 promises，promise 最终会被解析后，你就掌握了 GraphQL API 的结果。这没什么惊讶的。它只是使用普通 JavaScript 实现的，用 axios 作为 HTTP 客户端，使用普通的 HTTP 执行 GraphQL 请求。

再次启动你的应用程序，并确认你已经在开发者模式下的控制台日志中得到了结果。如果得到了  [401 HTTP 状态码](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)，说明你没有正确设置你的 access token。否则，你应该会在开发人员控制台日志中看到类似的结果。

{title="Developer Tools",lang="json"}
~~~~~~~~
{
  "config": ...,
  "data":{
    "data":{
      "organization":{
        "name":"The Road to learn React",
        "url":"https://github.com/the-road-to-learn-react"
      }
    }
  },
  "headers": ...,
  "request": ...,
  "status": ...,
  "statusText": ...
}
~~~~~~~~

最外层信息是 axios 的元信息返回给你所发请求的所有内容。它都是 axios，与 GraphQL 还没有任何关系，这就是为什么它的大部分被占位符所替代。Axios 有一个 `data` 属性，代表 Axios 请求的结果。里面包裹了一个反映 GraphQL 结果的 `data` 属性。首先，`data` 属性在第一个结果中看起来是冗余的，但是一旦理解了它，你就会知道一个 `data` 属性来自 axios，而另一个来自 GraphQL 数据结构。最后，在第二个 `data` 属性中找到 GraphQL 查询的结果。在里面也可以找到包含已解析的名称和 url 字段作为字符串属性的组织。

下一步，你将把包含组织信息的结果存储在 React 的本地状态中。如果发生任何错误，还需要将潜在的错误存储在状态中。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  state = {
    path: 'the-road-to-learn-react/the-road-to-learn-react',
# leanpub-start-insert
    organization: null,
    errors: null,
# leanpub-end-insert
  };

  ...

  onFetchFromGitHub = () => {
    axiosGitHubGraphQL
      .post('', { query: GET_ORGANIZATION })
      .then(result =>
# leanpub-start-insert
        this.setState(() => ({
          organization: result.data.data.organization,
          errors: result.data.errors,
        })),
# leanpub-end-insert
      );
  }

  ...

}
~~~~~~~~

接下来，你可以在 App 组件的 `render()` 方法中展示 Organization 的信息:

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  render() {
# leanpub-start-insert
    const { path, organization } = this.state;
# leanpub-end-insert

    return (
      <div>
        <h1>{TITLE}</h1>

        <form onSubmit={this.onSubmit}>
          ...
        </form>

        <hr />

# leanpub-start-insert
        <Organization organization={organization} />
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

将组织组件作为一个新的功能无状态组件引入，保持 App 组件的渲染方法简洁。因为这个应用程序只是一个简单的 GitHub 问题跟踪器，你已经可以在一小段话里提到它了。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...
}

# leanpub-start-insert
const Organization = ({ organization }) => (
  <div>
    <p>
      <strong>Issues from Organization:</strong>
      <a href={organization.url}>{organization.name}</a>
    </p>
  </div>
);
# leanpub-end-insert
~~~~~~~~

在最后一步中，你需要决定在还没有获取任何东西时界面应该渲染什么，以及在发生错误时应该渲染什么。可以在 React 中使用[条件呈现](https://www.robinwieruch.de/conditional-rendering-react/)解决这些边界情况。对于第一个边缘情况，只需检查 `organization` 是否存在。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  render() {
# leanpub-start-insert
    const { path, organization, errors } = this.state;
# leanpub-end-insert

    return (
      <div>
        ...

        <hr />

# leanpub-start-insert
        {organization ? (
          <Organization organization={organization} errors={errors} />
        ) : (
          <p>No information yet ...</p>
        )}
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

对于第二种边缘情况，你已经将错误传递给了 Organization 组件。如果出现了错误，它应该简单地将每个错误的错误消息渲染出来。否则，它应该渲染组织信息。对于 GraphQL 中的不同字段和环境，可能会有多个错误。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const Organization = ({ organization, errors }) => {
  if (errors) {
    return (
      <p>
        <strong>Something went wrong:</strong>
        {errors.map(error => error.message).join(' ')}
      </p>
    );
  }

  return (
    <div>
      <p>
        <strong>Issues from Organization:</strong>
        <a href={organization.url}>{organization.name}</a>
      </p>
    </div>
  );
};
# leanpub-end-insert
~~~~~~~~

你已经在 React 应用程序中执行了第一个 GraphQL 查询，虽然这只是一个以查询语句作为有效负载的普通 HTTP POST 请求。使用了配置好的 axios client 实例。之后，你可以将结果存储在 React 的本地状态中，以便稍后显示。

> ### GraphQL Nested Objects in React

### React 中 GraphQL 嵌套对象的使用

> Next, we'll request a nested object for the organization. Since the application will eventually show the issues in a repository, you should fetch a repository of an organization as the next step. Remember, a query reaches into the GraphQL graph, so we can nest the `repository` field in the `organization` when the schema defined the relationship between these two entities.

接下来，我们需要为组织添加一个嵌套对象。由于应用程序最终会在代码库中显示 issue，因此下一步你应该获取组织的代码库。请记住，查询操作会进入 GraphQL 图，因此我们可以在架构定义这两个实体之间的关系时将 `repository` 字段嵌套在 `organization` 中。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const GET_REPOSITORY_OF_ORGANIZATION = `
# leanpub-end-insert
  {
    organization(login: "the-road-to-learn-react") {
      name
      url
# leanpub-start-insert
      repository(name: "the-road-to-learn-react") {
        name
        url
      }
# leanpub-end-insert
    }
  }
`;

class App extends Component {
  ...

  onFetchFromGitHub = () => {
    axiosGitHubGraphQL
# leanpub-start-insert
      .post('', { query: GET_REPOSITORY_OF_ORGANIZATION })
# leanpub-end-insert
      .then(result =>
          ...
      );
  };

  ...
}
~~~~~~~~

> In this case, the repository name is identical to the organization. That's okay for now. Later on, you can define an organization and repository on your own dynamically. In the second step, you can extend the Organization component with another Repository component as child component. The result for the query should now have a nested repository object in the organization object.

虽然现在代码库与组织的名称相同，但是没关系，因为稍后你可以动态地定义组织和代码库。接下来，你可以使用另一个代码库组件作为子组件扩展组织组件。现在，查询操作的结果应该在组织对象中具有一个嵌套的代码库对象。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Organization = ({ organization, errors }) => {
  if (errors) {
    ...
  }

  return (
    <div>
      <p>
        <strong>Issues from Organization:</strong>
        <a href={organization.url}>{organization.name}</a>
      </p>
# leanpub-start-insert
      <Repository repository={organization.repository} />
# leanpub-end-insert
    </div>
  );
};

# leanpub-start-insert
const Repository = ({ repository }) => (
  <div>
    <p>
      <strong>In Repository:</strong>
      <a href={repository.url}>{repository.name}</a>
    </p>
  </div>
);
# leanpub-end-insert
~~~~~~~~

> The GraphQL query structure aligns perfectly with your component tree. It forms a natural fit to continue extending the query structure like this, by nesting other objects into the query, and extending the component tree along the structure of the GraphQL query. Since the application is an issue tracker, we need to add a list field of issues to the query.

GraphQL 查询结构与组件树完全一致。它通过将其他对象嵌套到查询中，并沿着 GraphQL 查询的结构扩展组件树，这样就可以继续拓展查询结构。由于应用程序是一个问题跟踪器，我们需要向查询添加 issue 列表字段。

> If you want to follow the query structure more thoughtfully, open the "Docs" sidebar in GraphiQL to learn out about the types `Organization`, `Repository`, `Issue`. The paginated issues list field can be found there as well. It's always good to have an overview of the graph structure.

如果你想更详细地了解查询结构，请打开 GraphiQL 中的 "Docs" 侧栏以了解 `Organization`, `Repository`, `Issue`，也可以在那里找到分页 issue 列表字段。对图表结构有一个大概的了解总是好的。

> Now let's extend the query with the list field for the issues. These issues are a paginated list in the end. We will cover these more later; for now, nest it in the `repository` field with a `last` argument to fetch the last items of the list.

现在让我们用 issue 列表字段扩展查询操作。这些 issue 最终是一个分页列表，我们稍后会介绍这些内容；现在，将它嵌套在 `repository` 字段中，并使用 `last` 参数来获取列表的最后一项。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const GET_ISSUES_OF_REPOSITORY = `
# leanpub-end-insert
  {
    organization(login: "the-road-to-learn-react") {
      name
      url
      repository(name: "the-road-to-learn-react") {
        name
        url
# leanpub-start-insert
        issues(last: 5) {
          edges {
            node {
              id
              title
              url
            }
          }
        }
# leanpub-end-insert
      }
    }
  }
`;
~~~~~~~~

> You can also request an id for each issue using the `id` field on the issue's `node` field, to use a `key` attribute for your list of rendered items in the component, which is considered [best practice in React](https://reactjs.org/docs/lists-and-keys.html). Remember to adjust the name of the query variable when its used to perform the request.

你还可以使用问题的 `node` 字段上的 `id` 字段为每个 issue 请求一个 id，以便于在组件中使用在列表中呈现的 `key` 属性，这是在 React 中的最佳实践。记得要在执行请求时调整查询变量的名称。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  onFetchFromGitHub = () => {
    axiosGitHubGraphQL
# leanpub-start-insert
      .post('', { query: GET_ISSUES_OF_REPOSITORY })
# leanpub-end-insert
      .then(result =>
          ...
      );
  };

  ...
}
~~~~~~~~

> The component structure follows the query structure quite naturally again. You can add a list of rendered issues to the Repository component. It is up to you to extract it to its own component as a refactoring to keep your components concise, readable, and maintainable.

组件结构非常自然地遵循了查询结构。你可以向代码库组件中添加已呈现的 issue 列表。你也可以将其作为重构提取到自己的组件中，让你的组件变得更加简洁，增加可读性和可维护性。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({ repository }) => (
  <div>
    <p>
      <strong>In Repository:</strong>
      <a href={repository.url}>{repository.name}</a>
    </p>

# leanpub-start-insert
    <ul>
      {repository.issues.edges.map(issue => (
        <li key={issue.node.id}>
          <a href={issue.node.url}>{issue.node.title}</a>
        </li>
      ))}
    </ul>
# leanpub-end-insert
  </div>
);
~~~~~~~~

> That's it for the nested objects, fields, and list fields in a query. Once you run your application again, you should see the last issues of the specified repository rendered in your browser.

这就是查询操作中的嵌套对象，字段和列表字段。当你再次运行你的应用程序，你会在浏览器中看到指定代码库的最后提的 issue。

> ### GraphQL Variables and Arguments in React

### React-GraphQL 变量和参数的使用

> Next we'll make use of the form and input elements. They should be used to request the data from GitHub's GraphQL API when a user fills in content and submits it. The content is also used for the initial request in `componentDidMount()` of the App component. So far, the organization `login` and repository `name` were inlined arguments in the query. Now, you should be able to pass in the `path` from the local state to the query to define dynamically an organization and repository. That's where variables in a GraphQL query came into play, do you remember?

接下来我们将使用表单和输入元素。当用户填写内容并提交内容时，它们应该用于从 Github GraphQL API 请求数据。该内容还用于 App 组件的 `componentDidMount()` 中的初始请求。目前，组织 login 和代码库 name 应该是查询操作中的内联参数。现在，你应该能够通过从本地状态到查询的 path 来动态定义组织和代码库。这就是 GraphQL 查询中的变量发挥作用的地方，还记得吗？

> First, let's use a naive approach by performing string interpolation with JavaScript rather than using GraphQL variables. To do this, refactor the query from a template literal variable to a function that returns a template literal variable. By using the function, you should be able to pass in an organization and repository.

首先，我们先使用一种简单的方法，通过 JavaScript 来执行字符串插值，注意不是 GraphQL 变量。为此，请将查询操作从模板文字变量重构为返回模板文字变量的函数。通过使用该函数，你可以动态地传入组织和代码库。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const getIssuesOfRepositoryQuery = (organization, repository) => `
# leanpub-end-insert
  {
# leanpub-start-insert
    organization(login: "${organization}") {
# leanpub-end-insert
      name
      url
# leanpub-start-insert
      repository(name: "${repository}") {
# leanpub-end-insert
        name
        url
        issues(last: 5) {
          edges {
            node {
              id
              title
              url
            }
          }
        }
      }
    }
  }
`;
~~~~~~~~

> Next, call the `onFetchFromGitHub()` class method in the submit handle, but also when the component mounts in `componentDidMount()` with the initial local state of the `path` property. These are the two essential places to fetch the data from the GraphQL API on initial render, and on every other manual submission from a button click.

接下来，可以在提交句柄中调用 `onFetchFromGitHub()` 这个方法，也可以在组件挂载 `componentDidMount()` 时， `path` 属性的本地状态初始化后调用。这些是在初始渲染时从 GraphQL API 获取数据的两个基本位置，以及从按钮单击获取每个手动提交的数据。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  state = {
    path: 'the-road-to-learn-react/the-road-to-learn-react',
    organization: null,
    errors: null,
  };

  componentDidMount() {
# leanpub-start-insert
    this.onFetchFromGitHub(this.state.path);
# leanpub-end-insert
  }

  onChange = event => {
    this.setState({ path: event.target.value });
  };

  onSubmit = event => {
# leanpub-start-insert
    this.onFetchFromGitHub(this.state.path);
# leanpub-end-insert

    event.preventDefault();
  };

  onFetchFromGitHub = () => {
    ...
  }

  render() {
    ...
  }
}
~~~~~~~~

> Lastly, call the function that returns the query instead of passing the query string directly as payload. Use the [JavaScript's split method on a string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split) to retrieve the prefix and suffix of the `/` character from the path variable where the prefix is the organization and the suffix is the repository.

最后，调用返回查询的函数，而不是直接将查询字符串作为有效负载传递。对字符串使用 JavaScript 的 [split](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split) 方法从路径变量中检索 `/` 字符的前缀和后缀，其中前缀是组织，后缀是代码库。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

# leanpub-start-insert
  onFetchFromGitHub = path => {
    const [organization, repository] = path.split('/');
# leanpub-end-insert

    axiosGitHubGraphQL
      .post('', {
# leanpub-start-insert
        query: getIssuesOfRepositoryQuery(organization, repository),
# leanpub-end-insert
      })
      .then(result =>
        this.setState(() => ({
          organization: result.data.data.organization,
          errors: result.data.errors,
        })),
      );
  };

  ...
}
~~~~~~~~

> Since the split returns an array of values and it is assumed that there is only one slash in the path, the array should consist of two values: the organization and the repository. That's why it is convenient to use a JavaScript array destructuring to pull out both values from an array in the same line.

由于 split 方法的返回值是一个数组，假设路径中只有一个斜杠，那么该数组应包含两个值:组织和代码库。所以，使用 JavaScript 数组解构从同一行中的数组中提取两个值很方便。

> Note that the application is not built for to be robust, but is intended only as a learning experience. It is unlikely anyone will ask a user to input the organization and repository with a different pattern than *organization/repository*, so there is no validation included yet. Still, it is a good foundation for you to gain experience with the concepts.

注意，此程序仅作为学习体验，未考虑其健壮性。任何人都不可能要求用户使用与 组织/代码库 不同的模式输入组织和代码库，所以这里没有包含任何验证。尽管如此，这也能为你学习概念打好良好的基础。

> If you want to go further, you can extract the first part of the class method to its own function, which uses axios to send a request with the query and return a promise. The promise can be used to resolve the result into the local state, using `this.setState()` in the `then()` resolver block of the promise.

如果你想更进一步，可以将类方法的第一部分提取到它自己的函数中，该函数使用 axios 发送带有查询的请求并返回一个 promise，可用于将结果解析为本地状态，然后你可以在 `then()` 函数中使用 `this.setState()` 方法来解析程序块中的 promise.

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const getIssuesOfRepository = path => {
  const [organization, repository] = path.split('/');

  return axiosGitHubGraphQL.post('', {
    query: getIssuesOfRepositoryQuery(organization, repository),
  });
};
# leanpub-end-insert

class App extends Component {
  ...

  onFetchFromGitHub = path => {
# leanpub-start-insert
    getIssuesOfRepository(path).then(result =>
# leanpub-end-insert
      this.setState(() => ({
        organization: result.data.data.organization,
        errors: result.data.errors,
      })),
    );
  };

  ...
}
~~~~~~~~

> You can always split your applications into parts, be they functions or components, to make them concise, readable, reusable and [testable](https://www.robinwieruch.de/react-testing-tutorial/). The function that is passed to `this.setState()` can be extracted as higher-order function. It has to be a higher-order function, because you need to pass the result of the promise, but also provide a function for the `this.setState()` method.

你也可以将应用程序的功能或者组件拆分为多个部分，使它们简洁，可读，可重用和[可测试](https://www.robinwieruch.de/react-testing-tutorial/)。传递给 `this.setState()` 的函数可以作为高阶函数提取。它必须是高阶函数，因为你需要传递 promise 的结果，但也为 `this.setState()` 方法提供函数。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const resolveIssuesQuery = queryResult => () => ({
  organization: queryResult.data.data.organization,
  errors: queryResult.data.errors,
});
# leanpub-start-insert

class App extends Component {
  ...

  onFetchFromGitHub = path => {
# leanpub-start-insert
    getIssuesOfRepository(path).then(queryResult =>
      this.setState(resolveIssuesQuery(queryResult)),
# leanpub-end-insert
    );
  };

  ...
}
~~~~~~~~

> Now you've made your query flexible by providing dynamic arguments to your query. Try it by starting your application on the command line and by filling in a different organization with a specific repository (e.g. *facebook/create-react-app*).

现在，你已经通过在查询中提供动态参数灵活的进行查询操作。在终端尝试启动你的程序，在特定代码库（例如 facebook/create-react-app）中填充不同的组织。

> It's a decent setup, but there was nothing to see about variables yet. You simply passed the arguments to the query using a function and string interpolation with template literals. Now we'll use GraphQL variables instead, to refactor the query variable again to a template literal that defines inline variables.

这是一个很好的设置，但还没有什么可见的变量。你只需使用一个函数和带有模板文字的字符串插值将参数传递给查询操作。现在我们将使用 GraphQL 变量，将查询变量重新调整到定义内联变量的模板文本中。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const GET_ISSUES_OF_REPOSITORY = `
  query ($organization: String!, $repository: String!) {
    organization(login: $organization) {
# leanpub-end-insert
      name
      url
# leanpub-start-insert
      repository(name: $repository) {
# leanpub-end-insert
        name
        url
        issues(last: 5) {
          edges {
            node {
              id
              title
              url
            }
          }
        }
      }
    }
  }
`;
~~~~~~~~

> Now you can pass those variables as arguments next to the query for the HTTP POST request:

现在，你可以将这些变量作为 HTTP POST 请求的查询参数：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const getIssuesOfRepository = path => {
  const [organization, repository] = path.split('/');

  return axiosGitHubGraphQL.post('', {
# leanpub-start-insert
    query: GET_ISSUES_OF_REPOSITORY,
    variables: { organization, repository },
# leanpub-end-insert
  });
};
~~~~~~~~

> Finally, the query takes variables into account without detouring into a function with string interpolation. I strongly suggest practicing with the exercises below before continuing to the next section. We've yet to discuss features like fragments or operation names, but we'll soon cover them using Apollo instead of plain HTTP with axios.

最后，查询操作将变量纳入考虑，而不用字符串插值的方法去检测函数。我强烈建议在继续下一节之前先练习下面的练习。我们还没有讨论像碎片或操作名称这样的特性，但是我们很快就会使用 Apollo 代替普通的带 axios 的 HTTP 来覆盖它们。

> ### Exercises:

### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/c08126a9ec91dde4198ae85bb2f194fa7767c683)
* 查看[本节源码](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/c08126a9ec91dde4198ae85bb2f194fa7767c683)
> * Explore and add fields to your organization, repository and issues
* 探索你的组织、代码库和 issues 中并尝试添加字段
>   * Extend your components to display the additional information
  * 扩展你的组件以显示附加信息
> * Read more about [serving a GraphQL API over HTTP](http://graphql.org/learn/serving-over-http/)
* 延伸阅读：[通过 HTTP 提供 GraphQL API 服务](http://graphql.org/learn/serving-over-http/)

> ## GraphQL Pagination in React

## React 中的 GraphQL 分页

> Last section you implemented a list field in your GraphQL query, which fit into the flow of structuring the query with nested objects and a list responsible for showing partial results of the query in React.

上一节中，你在 React 应用上实现了一个用于 GraphQL 查询的列表字段，它能得到一个包含部分查询结果的列表，其中的元素满足指定的嵌套结构。

> In this section, you will explore pagination with list fields with GraphQL in React in more detail. Initially, you will learn more about the arguments of list fields. Further, you will add one more nested list field to your query. Finally, you will fetch another page of the paginated `issues` list with your query.

本节将继续在使用 React 的基础上，更为深入地探索基于 GraphQL 列表字段实现的分页功能。你将首先了解列表字段的参数使用，而后在查询中添加另一个嵌套列表字段，从而能够在查询中实现获取下一页 `issues` 的功能。

> Let's start by extending the `issues` list field in your query with one more argument:

我们首先为这个 `issues` 列表字段添加一个额外参数：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query ($organization: String!, $repository: String!) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        name
        url
# leanpub-start-insert
        issues(last: 5, states: [OPEN]) {
# leanpub-end-insert
          edges {
            node {
              id
              title
              url
            }
          }
        }
      }
    }
  }
`;
~~~~~~~~

> If you read the arguments for the `issues` list field using the "Docs" sidebar in GraphiQL, you can explore which arguments you can pass to the field. One of these is the `states` argument, which defines whether or not to fetch open or closed issues. The previous implementation of the query has shown you how to refine the list field, in case you only want to show open issues. You can explore more arguments for the `issues` list field, but also for other list fields, using the documentation from Github's API.

如果你在 GraphiQL 工具中通过 "Docs" 边栏查看过这个 `issues` 列表字段的参数信息，就能找到可供使用的参数信息。其中的一个参数便是 `states`，用于决定获取处在开启状态或关闭状态的 issues，亦或两者同时。如果仅需要展示处于开启状态的 issues，那么可以进一步提炼这个列表字段，方法如上述代码所示。通过 GitHub 的 API 文档，你能够查看关于这个 `issues` 列表字段的更多参数信息，其它的列表字段也是一样。

> Now we'll implement another nested list field that could be used for pagination. Each issue in a repository can have reactions, essentially emoticons like a smiley or a thumbs up. Reactions can be seen as another list of paginated items. First, extend the query with the nested list field for reactions:

现在我们将实现另一个列表字段，以便用于完善分页功能。我们知道，仓库中的每个 issue 都可能具有 reactions，也就是像笑脸或者大拇指之类的表情。这些 reactions 可以被视作另一项分页内容，为了展示它们，首先在查询中添加一个嵌套的 reactions 列表字段：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query ($organization: String!, $repository: String!) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        name
        url
        issues(last: 5, states: [OPEN]) {
          edges {
            node {
              id
              title
              url
# leanpub-start-insert
              reactions(last: 3) {
                edges {
                  node {
                    id
                    content
                  }
                }
              }
# leanpub-end-insert
            }
          }
        }
      }
    }
  }
`;
~~~~~~~~

> Second, render the list of reactions in one of your React components again. Implement dedicated List and Item components, such as ReactionsList and ReactionItem for it. As an exercise, try keeping the code for this application readable and maintainable.

之后，我们继续通过 React 组件来渲染 reactions 列表。这里本应当为列表和条目定义专门的组件，例如叫做 ReactionsList 和 ReactionItem，可以将此当作课后练习，从而提升应用代码的可读性和可维护性。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({ repository }) => (
  <div>
    ...

    <ul>
      {repository.issues.edges.map(issue => (
        <li key={issue.node.id}>
          <a href={issue.node.url}>{issue.node.title}</a>

# leanpub-start-insert
          <ul>
            {issue.node.reactions.edges.map(reaction => (
              <li key={reaction.node.id}>{reaction.node.content}</li>
            ))}
          </ul>
# leanpub-end-insert
        </li>
      ))}
    </ul>
  </div>
);
~~~~~~~~

> You extended the query and React's component structure to render the result. It's a straightforward implementation when you are using a GraphQL API as your data source which has a well defined underlying schema for these field relationships.

通过扩展查询结构以及 React 组件内容，你已经成功渲染出相应结果。由于作为数据源的 GraphQL API 为其字段定义了清晰的结构和关联性，这里的实现方式直截了当。

> Lastly, you will implement real pagination with the `issues` list field, as there should be a button to fetch more issues from the GraphQL API to make it a function of a completed application. Here is how to implement a button:

最后，为了做出完整的应用，你需要实现一个对 `issues` 列表字段的实际分页功能，通过点击一个按钮从 GraphQL API 获取更多的 issues。以下是按钮部分的实现代码：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({
  repository,
# leanpub-start-insert
  onFetchMoreIssues,
# leanpub-end-insert
}) => (
  <div>
    ...

    <ul>
      ...
    </ul>

# leanpub-start-insert
    <hr />
# leanpub-end-insert

# leanpub-start-insert
    <button onClick={onFetchMoreIssues}>More</button>
# leanpub-end-insert
  </div>
);
~~~~~~~~

> The handler for the button passes through all the components to reach the Repository component:

该按钮的处理函数经由更外层传入 Repository 组件：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Organization = ({
  organization,
  errors,
# leanpub-start-insert
  onFetchMoreIssues,
# leanpub-end-insert
}) => {
  ...

  return (
    <div>
      <p>
        <strong>Issues from Organization:</strong>
        <a href={organization.url}>{organization.name}</a>
      </p>
      <Repository
        repository={organization.repository}
# leanpub-start-insert
        onFetchMoreIssues={onFetchMoreIssues}
# leanpub-end-insert
      />
    </div>
  );
};
~~~~~~~~

> Logic for the function is implemented in the App component as class method. It passes to the Organization component as well.

该函数在 App 组件中作为类方法实现，同样也作为属性传入 Organization 组件。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

# leanpub-start-insert
  onFetchMoreIssues = () => {
    ...
  };
# leanpub-end-insert

  render() {
    const { path, organization, errors } = this.state;

    return (
      <div>
        ...

        {organization ? (
          <Organization
            organization={organization}
            errors={errors}
# leanpub-start-insert
            onFetchMoreIssues={this.onFetchMoreIssues}
# leanpub-end-insert
          />
        ) : (
          <p>No information yet ...</p>
        )}
      </div>
    );
  }
}
~~~~~~~~

> Before implementing the logic for it, there needs to be a way to identify the next page of the paginated list. To extend the inner fields of a list field with fields for meta information such as the `pageInfo` or the `totalCount` information, use `pageInfo` to define the next page on button-click. Also, the `totalCount` is only a nice way to see how many items are in the next list:

开始实现功能之前，首先需要考虑如何标明这个分页列表中的下一页。在列表字段中，可以通过添加额外的内部字段来获取相关元信息，例如 `pageInfo` 和 `totalCount`，前者可用于按钮点击的事件处理中定义下一页的相关信息。此外，`totalCount` 可辅助判断下一页中的条目数量：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query ($organization: String!, $repository: String!) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        ...
        issues(last: 5, states: [OPEN]) {
          edges {
            ...
          }
# leanpub-start-insert
          totalCount
          pageInfo {
            endCursor
            hasNextPage
          }
# leanpub-end-insert
        }
      }
    }
  }
`;
~~~~~~~~

> Now, you can use this information to fetch the next page of issues by providing the cursor as a variable to your query. The cursor, or the `after` argument, defines the starting point to fetch more items from the paginated list.

通过这些信息，现在你能将这个游标（cursor）作为变量传入查询中，从而获取下一页的 issues 内容。这个游标，也就是 `after` 参数，在分页列表中定义了获取更多条目所需的起始位置。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

  onFetchMoreIssues = () => {
# leanpub-start-insert
    const {
      endCursor,
    } = this.state.organization.repository.issues.pageInfo;
# leanpub-end-insert

# leanpub-start-insert
    this.onFetchFromGitHub(this.state.path, endCursor);
# leanpub-end-insert
  };

  ...
}
~~~~~~~~

> The second argument wasn't introduced  to the `onFetchFromGitHub()` class method yet. Let's see how that turns out.

名为 `onFetchFromGitHub` 的类方法尚未定义第二个参数，现在让我们来定义它。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const getIssuesOfRepository = (path, cursor) => {
# leanpub-end-insert
  const [organization, repository] = path.split('/');

  return axiosGitHubGraphQL.post('', {
    query: GET_ISSUES_OF_REPOSITORY,
# leanpub-start-insert
    variables: { organization, repository, cursor },
# leanpub-end-insert
  });
};

class App extends Component {
  ...

# leanpub-start-insert
  onFetchFromGitHub = (path, cursor) => {
    getIssuesOfRepository(path, cursor).then(queryResult =>
      this.setState(resolveIssuesQuery(queryResult, cursor)),
# leanpub-end-insert
    );
  };

  ...
}
~~~~~~~~

> The argument is simply passed to the `getIssuesOfRepository()` function, which makes the GraphQL API request, and returns the promise with the query result. Check the other functions that call the `onFetchFromGitHub()` class method, and notice how they don't make use of the second argument, so the cursor parameter will be `undefined` when it's passed to the GraphQL API call. Either the query uses the cursor as argument to fetch the next page of a list, or it fetches the initial page of a list by having the cursor not defined at all:

新的参数被直接传入 `getIssuesOfRepository()` 函数，后者用于构造 GraphQL 的 API 请求，并返回对应查询结果的 Promise。通过观察其它对 `onFetchFromGitHub()` 类方法的调用，能够注意到它们并未使用第二个参数，因此实际传入 GraphQL API 调用中的参数是 `undefined`。最终，这个查询要么使用了传入的游标信息作为参数来获取下一页的列表，要么由于游标未定义获取到第一页的列表。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query (
    $organization: String!,
    $repository: String!,
# leanpub-start-insert
    $cursor: String
# leanpub-end-insert
  ) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        ...
# leanpub-start-insert
        issues(first: 5, after: $cursor, states: [OPEN]) {
# leanpub-end-insert
          edges {
            ...
          }
          totalCount
          pageInfo {
            endCursor
            hasNextPage
          }
        }
      }
    }
  }
`;
~~~~~~~~

> In the previous template string, the `cursor` is passed as variable to the query and used as `after` argument for the list field. The variable is not enforced though, because there is no exclamation mark next to it, so it can be `undefined`. This happens for the initial page request for a paginated list, when you only want to fetch the first page. Further, the argument `last` has been changed to `first` for the `issues` list field, because there won't be another page after you fetched the last item in the initial request. Thus, you have to start with the first items of the list to fetch more items until you reach the end of the list.

上面的模板字符串中，`cursor` 作为变量传入到该列表字段的 `after` 参数中。因为没有后置的的感叹号声明强制要求，所以可以是 `undefined`，这种场景出现在分页列表中获取第一页的情况下。此外，`issues` 列表字段中的 `last` 参数改成了 `first`，因为获取到最后一个条目之后就不存在下一页了，因此必须从第一个条目开始不断获取更多内容，直到到达列表中的最后一个条目为止。

> That's it for fetching the next page of a paginated list with GraphQL in React, except one final step. Nothing updates the local state of the App component about a page of issues yet, so there are still only the issues from the initial request. You want to merge the old pages of issues with the new page of issues in the local state of the App component, while keeping the organization and repository information in the deeply nested state object intact. The perfect time for doing this is when the promise for the query resolves. You already extracted it as a function outside of the App component, so you can use this place to handle the incoming result and return a result with your own structure and information. Keep in mind that the incoming result can be an initial request when the App component mounts for the first time, or after a request to fetch more issues happens, such as when the "More" button is clicked.

以上就基本是在 React 应用中使用 GraphQL 获取分页列表所需的过程，不过还差最后一步。目前为止，App 组件中关于 issues 的状态并未得到更新，因此仅显示第一个请求中的结果。为了在保持组织和仓库基本信息不变的情况下合并 issues 的分页列表，最佳的操作时间就是查询得到的 Promise 完成时。由于这个过程已经被提取为 App 组件之外的独立函数，因此可以在该函数中对输入数据进行必要的数据处理。需要注意的是，这里的输入数据既可以是应用挂载时的第一次请求结果，也可以是点击 "More" 按钮时获取的后续结果。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const resolveIssuesQuery = (queryResult, cursor) => state => {
  const { data, errors } = queryResult.data;

  if (!cursor) {
    return {
      organization: data.organization,
      errors,
    };
  }

  const { edges: oldIssues } = state.organization.repository.issues;
  const { edges: newIssues } = data.organization.repository.issues;
  const updatedIssues = [...oldIssues, ...newIssues];

  return {
    organization: {
      ...data.organization,
      repository: {
        ...data.organization.repository,
        issues: {
          ...data.organization.repository.issues,
          edges: updatedIssues,
        },
      },
    },
    errors,
  };
};
# leanpub-end-insert
~~~~~~~~

> The function is a complete rewrite, because the update mechanism is more complex now. First, you passed the `cursor` as an argument to the function, which determines whether it was an initial query or a query to fetch another page of issues. Second, if the `cursor` is `undefined`, the function can return early with the state object that encapsulates the plain query result, same as before. There is nothing to keep intact in the state object from before, because it is an initial request that happens when the App component mounts or when a user submits another request which should overwrite the old state anyway. Third, if it is a fetch more query and the cursor is there, the old and new issues from the state and the query result get merged in an updated list of issues. In this case, a JavaScript destructuring alias is used to make naming both issue lists more obvious. Finally, the function returns the updated state object. Since it is a deeply nested object with multiple levels to update, use the JavaScript spread operator syntax to update each level with a new query result. Only the `edges` property should be updated with the merged list of issues.

由于更新机制变得更复杂，该函数也经历了全面重写。首先，`cursor` 被作为函数参数传入，用于确定是首次查询还是获取下一页内容的查询；随后，当 `cursor` 是 `undefined` 时函数会提前结束并将查询结果简单包装后直接返回，与原先的逻辑相同，这里对应于 App 组件挂载或者用户提交新的输入的场景，应当覆盖之前的状态；其次，当 `cursor` 存在也就是加载更多的查询时，查询结果中的 issues 列表会与已有的列表进行合并，为了显得更为直观而使用了 JavaScript 解构别名语法；最后，该函数返回了更新后的对象，由于存在多层嵌套而使用了 JavaScript 扩展（Spread）语法逐层更新，不过只有 `edges` 属性需要更新为合并后的 issues 列表。

> Next, use the `hasNextPage` property from the `pageInfo` that you requested to show a "More" button (or not). If there are no more issues in the list, the button should disappear.

之后，根据 `pageInfo` 对象中的 `hasNextPage` 来决定是否显示 "More" 按钮，当没有更多页面时按钮消失。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({ repository, onFetchMoreIssues }) => (
  <div>
    ...

    <hr />

# leanpub-start-insert
    {repository.issues.pageInfo.hasNextPage && (
# leanpub-end-insert
      <button onClick={onFetchMoreIssues}>More</button>
# leanpub-start-insert
    )}
# leanpub-end-insert
  </div>
);
~~~~~~~~

> Now you've implemented pagination with GraphQL in React. For practice, try more arguments for your issues and reactions list fields on your own. Check the "Docs" sidebar in GraphiQL to find out about arguments you can pass to list fields. Some arguments are generic, but have arguments that are specific to lists. These arguments should show you how finely-tuned requests can be with a GraphQL query.

至此你已经成功地在 React 应用中实现了基于 GraphQL 的分页功能，如果需要，可以自行练习更多关于 issues 和 reactions 的参数使用。通过 GraphiQL 中地 "Docs" 边栏可以查看列表字段的参数信息，有些参数是公共的，而有些是部分列表所特有的，这些参数将帮助你实现精致的 GraphQL 查询。

> ### Exercises:

### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/060677346e8955fb1a6c7579859ce92e62e1f406)
* 确认[本节源代码](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/060677346e8955fb1a6c7579859ce92e62e1f406)
> * Explore further arguments, generic or specific for the type, on the `issues` and `reactions` list fields
  > * Think about ways to beautify the updating mechanism of deeply nested state objects and [contribute your thoughts to it](https://github.com/rwieruch/react-graphql-github-apollo/pull/14)
* 探索更多关于 `issues` 和 `reactions` 列表字段的参数信息，包括公共和类型独有的参数
  * 思考对深度嵌套状态对象的更新机制的优化方案，并[在此贡献你的想法](https://github.com/rwieruch/react-graphql-github-apollo/pull/14)

> ## GraphQL Mutation in React

## React 中的 GraphQL 变更操作

> You fetched a lot of data using GraphQL in React, the larger part of using GraphQL. However, there are always two sides to such an interface: read and write. That's where GraphQL mutations complement the interface. Previously, you learned about GraphQL mutations using GraphiQL without React. In this section, you will implement such a mutation in your React GraphQL application.

你已经尝试了在 React 应用中通过 GraphQL 获取大量数据，这也是 GraphQL 的主要应用场景。不过，这种接口永远存在两个方向的操作：读取和写入。在 GraphQL 中写入操作被称为变更。此前，你已经在没有 React 的情况下通过 GraphiQL 使用过 GraphQL 变更操作，本节中你将在 React 应用中实现 GraphQL 变更操作。

> You have executed GitHub's `addStar` mutation before in GraphiQL. Now, let's implement this mutation in React. Before implementing the mutation, you should query additional information about the repository, which is partially required to star the repository in a mutation.

你已经在 GraphiQL 执行过 GitHub 的 `addStar` 变更操作了，现在将通过 React 实现同样的操作。开始实现 star 某仓库这个变更操作之前，你需要查询关于这个仓库所需的部分额外信息。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query (
    $organization: String!,
    $repository: String!,
    $cursor: String
  ) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
# leanpub-start-insert
        id
# leanpub-end-insert
        name
        url
# leanpub-start-insert
        viewerHasStarred
# leanpub-end-insert
        issues(first: 5, after: $cursor, states: [OPEN]) {
          ...
        }
      }
    }
  }
`;
~~~~~~~~

> The `viewerHasStarred` field returns a boolean that tells whether the viewer has starred the repository or not. This boolean helps determine whether to execute a `addStar` or `removeStar` mutation in the next steps. For now, you will only implement the `addStar` mutation. The `removeStar` mutation will be left off as part of the exercise. Also, the `id` field in the query returns the identifier for the repository, which you will need to clarify the target repository of your mutation.

通过 `viewerHasStarred` 字段可以知晓当前用户是否已经 star 了这个仓库，从而帮助确定下一步的变更操作是 `addStar` 还是 `removeStar`。现在你只需要实现 `addStar` 变更操作，而 `removeStar` 部分将作为本节的练习。另外，查询操作中返回的 `id` 字段返回了该仓库的标识符，能够用于在后续变更操作中指明目标。

> The best place to trigger the mutation is a button that stars or unstars the repository. That's where the `viewerHasStarred` boolean can be used for a conditional rendering to show either a "Star" or "Unstar" button. Since you are going to star a repository, the Repository component is the best place to trigger the mutation.

触发这个变更操作的最佳场景，莫过于一个对仓库 star 或取消 star 的按钮，通过 `viewerHasStarred` 这个布尔值进行条件渲染，得到一个显示内容为 "Star" 或 "Unstar" 的按钮。鉴于 star 的目标是一个仓库，因此在 Repository 组件中触发该变更操作最合适不过。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({
  repository,
  onFetchMoreIssues,
# leanpub-start-insert
  onStarRepository,
# leanpub-end-insert
}) => (
  <div>
    ...

# leanpub-start-insert
    <button
      type="button"
      onClick={() => onStarRepository()}
    >
      {repository.viewerHasStarred ? 'Unstar' : 'Star'}
    </button>
# leanpub-end-insert

    <ul>
      ...
    </ul>
  </div>
);
~~~~~~~~

> To identify the repository to be starred, the mutation needs to know about the `id` of the repository. Pass the `viewerHasStarred` property as a parameter to the handler, since you'll use the parameter to determine whether you want to execute the star or unstar mutation later.

为了确定用户 star 的是哪一个仓库，变更操作必须知晓该仓库的 `id` 信息，这里通过 `viewerHasStarred` 属性作为参数传入，以便后续能够区分 star 或取消 star 的场景。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({ repository, onStarRepository }) => (
  <div>
    ...

    <button
      type="button"
      onClick={() =>
# leanpub-start-insert
        onStarRepository(repository.id, repository.viewerHasStarred)
# leanpub-end-insert
      }
    >
      {repository.viewerHasStarred ? 'Unstar' : 'Star'}
    </button>

    ...
  </div>
);
~~~~~~~~

> The handler should be defined in the App component. It passes through each component until it reaches the Repository component, also reaching through the Organization component on its way.

对应的事件处理函数应该定义在 App 组件中，向下传递至 Repository 组件，中途也经过了 Organization 组件。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Organization = ({
  organization,
  errors,
  onFetchMoreIssues,
# leanpub-start-insert
  onStarRepository,
# leanpub-end-insert
}) => {
  ...

  return (
    <div>
      ...
      <Repository
        repository={organization.repository}
        onFetchMoreIssues={onFetchMoreIssues}
# leanpub-start-insert
        onStarRepository={onStarRepository}
# leanpub-end-insert
      />
    </div>
  );
};
~~~~~~~~

> Now it can be defined in the App component. Note that the `id` and the `viewerHasStarred` information can be destructured from the App's local state, too. This is why you wouldn't need to pass this information in the handler, but use it from the local state instead. However, since the Repository component knew about the information already, it is fine to pass the information in the handler, which also makes the handler more explicit. It's also good preparation for dealing with multiple repositories and repository components later, since the handler will need to be more specific in these cases.

现在可以尝试在 App 组件中定义这个方法，很容易发现 `id` 和 `viewerHasStarred` 的信息可以从应用的本地状态中解构得到，因此无需在函数中传入便可直接使用。不过，鉴于 Repository 已经知晓这些信息，作为函数参数中传入仍然是合理的选择，并且能够使得数据流更加直观。特别是考虑到之后有通过多个 Repository 组件处理多个仓库的需求，这种场景下函数需要具备更加充分的信息。

{title="src/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  ...

# leanpub-start-insert
  onStarRepository = (repositoryId, viewerHasStarred) => {
    ...
  };
# leanpub-end-insert

  render() {
    const { path, organization, errors } = this.state;

    return (
      <div>
        ...

        {organization ? (
          <Organization
            organization={organization}
            errors={errors}
            onFetchMoreIssues={this.onFetchMoreIssues}
# leanpub-start-insert
            onStarRepository={this.onStarRepository}
# leanpub-end-insert
          />
        ) : (
          <p>No information yet ...</p>
        )}
      </div>
    );
  }
}
~~~~~~~~

> Now, you can implement the handler. The mutation can be outsourced from the component. Later, you can use the `viewerHasStarred` boolean in the handler to perform a `addStar` or `removeStar` mutation. Executing the mutation looks similar to the GraphQL query from before. The API endpoint is not needed, because it was set in the beginning when you configured axios. The mutation can be sent in the `query` payload, which we'll cover later. The `variables` property is optional, but you need to pass the identifier.

现在便可以开始实现这个事件处理函数，整个变更操作的执行过程可以从组件外包出去作为独立函数，之后就可以通过 `viewerHasStarred` 这个布尔值判断是执行 `addStar` 还是 `removeStar` 变更操作。GraphQL 中变更操作的执行十分类似于之前使用过的查询操作，API 地址的配置已经在最初的章节通过 axios 完成，并不需要重复进行。变更的内容可以通过 `query` 属性传输，这点我们会在之后详述，虽然 `variables` 属性在 API 中是可选的，但当前需要用于传入仓库标识符。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const addStarToRepository = repositoryId => {
  return axiosGitHubGraphQL.post('', {
    query: ADD_STAR,
    variables: { repositoryId },
  });
};
# leanpub-end-insert

class App extends Component {
  ...

  onStarRepository = (repositoryId, viewerHasStarred) => {
# leanpub-start-insert
    addStarToRepository(repositoryId);
# leanpub-end-insert
  };

  ...
}
~~~~~~~~

> Before you define the `addStar` mutation, check GitHub's GraphQL API again. There, you will find all information about the structure of the mutation, the required arguments, and the available fields for the result. For instance, you can include the `viewerHasStarred` field in the returned result to get an updated boolean of a starred or unstarred repository.

定义 `addStar` 变更操作之前，先再次检查 GitHub 的 GraphQL API，从中你可以找到它的所有结构信息以及必须参数，还有可用的结果字段。例如，你可以在返回结果中包含 `viewerHasStarred` 字段来取得是否 star 该仓库的更新后结果。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const ADD_STAR = `
  mutation ($repositoryId: ID!) {
    addStar(input:{starrableId:$repositoryId}) {
      starrable {
        viewerHasStarred
      }
    }
  }
`;
~~~~~~~~

> You could already execute the mutation in the browser by clicking the button. If you haven't starred the repository before, it should be starred after clicking the button. You can visit the repository on GitHub to get visual feedback, though you won't see any results reflected yet. The button still shows the "Star" label when the repository wasn't starred before, because the `viewerHasStarred` boolean wasn't updated in the local state of the App component after the mutation. That's the next thing you are going to implement. Since axios returns a promise, you can use the `then()` method on the promise to resolve it with your own implementation details.

现在已经能够通过在浏览器中点击该按钮来执行变更操作，如果之前你没有 star 过这个仓库，在点击按钮之后就会进行 star。虽然可以在 GitHub 网站上得到可视化的反馈，不过在应用中并不会产生任何反应，即便已经进行 star 之后该按钮仍然会显示 "Star" 标签，这是由于 App 组件中的 `viewerHasStarred` 状态在变更操作后并未得到更新。这也是你下一步的工作，鉴于 axios 返回了一个 Promise，你可以在 `then()` 方法的回调中添加相关实现细节。

{title="src/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const resolveAddStarMutation = mutationResult => state => {
  ...
};
# leanpub-end-insert

class App extends Component {
  ...

  onStarRepository = (repositoryId, viewerHasStarred) => {
# leanpub-start-insert
    addStarToRepository(repositoryId).then(mutationResult =>
      this.setState(resolveAddStarMutation(mutationResult)),
    );
# leanpub-end-insert
  };

  ...
}
~~~~~~~~

> When resolving the promise from the mutation, you can find out about the `viewerHasStarred` property in the result. That's because you defined this property as a field in your mutation. It returns a new state object for React's local state, because you used the function in `this.setState()`. The spread operator syntax is used here, to update the deeply nested data structure. Only the `viewerHasStarred` property changes in the state object, because it's the only property returned by the resolved promise from the successful request. All other parts of the local state stay intact.

在变更操作的 Promise 结果中，你能看到一个 `viewerHasStarred` 属性，这是由于它被定义为变更操作中的一个字段。调用 `resolveAddStarMutation` 函数后产生了一个新的状态对象，用于 `this.setState()` 方法中。这里同样使用了扩展语法来更新深度嵌套结构，这个函数中除了 `viewerHasStarred` 属性之外的内容都不会发生变化，因为它是从响应中唯一能够得到的内容。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const resolveAddStarMutation = mutationResult => state => {
# leanpub-start-insert
  const {
    viewerHasStarred,
  } = mutationResult.data.data.addStar.starrable;

  return {
    ...state,
    organization: {
      ...state.organization,
      repository: {
        ...state.organization.repository,
        viewerHasStarred,
      },
    },
  };
# leanpub-end-insert
};
~~~~~~~~

> Now try to star the repository again. You may have to go on the GitHub page and unstar it first. The button label should adapt to the updated `viewerHasStarred` property from the local state to show a "Star" or "Unstar" label. You can use what you've learned about starring repositories to implement a `removeStar` mutation.

现在可以重新尝试 star 这个仓库，虽然可能需要先到 GitHub 页面上取消 star。按钮中的文本将根据本地状态中 `viewerHasStarred` 更新后的值来决定显示 "Star" 或是 "Unstar" 标签，你可以根据目前习得的成果实现出 `removeStar` 变更操作。

 > We also want to show the current number of people who have starred the repository, and update this count in the `addStar` and `removeStar` mutations. First, retrieve the total count of stargazers by adding the following fields to your query:

我们想要更进一步显示出 star 了该仓库的当前人数，并在 `addStar` 或 `removeStar` 操作后更新，为此，首先需要在查询操作中添加 stargazers 字段来获取总人数：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const GET_ISSUES_OF_REPOSITORY = `
  query (
    $organization: String!,
    $repository: String!,
    $cursor: String
  ) {
    organization(login: $organization) {
      name
      url
      repository(name: $repository) {
        id
        name
        url
# leanpub-start-insert
        stargazers {
          totalCount
        }
# leanpub-end-insert
        viewerHasStarred
        issues(first: 5, after: $cursor, states: [OPEN]) {
          ...
        }
      }
    }
  }
`;
~~~~~~~~

> Second, you can show the count as a part of your button label:

随后，你可以将人数作为按钮标签的一部分：

{title="src/App.js",lang="javascript"}
~~~~~~~~
const Repository = ({
  repository,
  onFetchMoreIssues,
  onStarRepository,
}) => (
  <div>
    ...

    <button
      type="button"
      onClick={() =>
        onStarRepository(repository.id, repository.viewerHasStarred)
      }
    >
# leanpub-start-insert
      {repository.stargazers.totalCount}
      {repository.viewerHasStarred ? ' Unstar' : ' Star'}
# leanpub-end-insert
    </button>

    <ul>
      ...
    </ul>
  </div>
);
~~~~~~~~

>  Now we want the count to update when you star (or unstar) a repository. It is the same issue as the missing update for the `viewerHasStarred` property in the local state of the component after the `addStar` mutation succeeded. Return to your mutation resolver and update the total count of stargazers there as well. While the stargazer object isn't returned as a result from the mutation, you can increment and decrement the total count after a successful mutation manually using a counter along with the `addStar` mutation.

现在我们希望这个数量随着 star（或取消 star）一个仓库而更新，而目前的问题就和在没有更新 `addStar` 变更操作成功后没有更新 `viewerHasStarred` 属性一样，为此在回调函数中一同更新本地的 stargazers 总数。虽然 stargazer 对象并没有作为变更结果返回，但仍然可以在 `addStar` 变更操作成功后人工增加或者减少计数。

{title="src/App.js",lang="javascript"}
~~~~~~~~
const resolveAddStarMutation = mutationResult => state => {
  const {
    viewerHasStarred,
  } = mutationResult.data.data.addStar.starrable;

# leanpub-start-insert
  const { totalCount } = state.organization.repository.stargazers;
# leanpub-end-insert

  return {
    ...state,
    organization: {
      ...state.organization,
      repository: {
        ...state.organization.repository,
        viewerHasStarred,
# leanpub-start-insert
        stargazers: {
          totalCount: totalCount + 1,
        },
# leanpub-end-insert
      },
    },
  };
};
~~~~~~~~

> You have implemented your first mutation in React with GraphQL. So far, you have just implemented the `addStar` mutation. Even though the button already reflects the `viewerHasStarred` boolean by showing a "Star" or "Unstar" label, the button showing "Unstar" should still execute the `addStar` mutation. The `removeStar` mutation to unstar the repository is one of the practice exercises mentioned below.

你已经在 React 应用中实现了首个 GraphQL 的变更操作，虽然目前为止，只有 `addStar` 一个操作。尽管按钮仍然能够通过显示 "Star" 或 "Unstar" 标签来反映 `viewerHasStarred` 的值，但在按钮显示为 "Unstar" 时仍然执行的是 `addStar` 操作，而用来取消 star 某仓库的 `removeStar` 变更操作就作为下面的练习之一。

> ### Exercises:

### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/3dcd95e32ef24d9e716a1e8ac144b62c0f41ca3c)
* 确认[本节源代码](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/3dcd95e32ef24d9e716a1e8ac144b62c0f41ca3c)
> * Implement the `removeStar` mutation, which is used analog to the `addStar` mutation.
  > * The `onStarRepository` class method has already access to the `viewerHasStarred` property.
  > * Conditionally execute a `addStar` or `removeStar` mutation in the class handler.
  > * Resolve the new state after removing a star from a repository.
  > * Align your final thoughts with [this implementation](https://github.com/rwieruch/react-graphql-github-vanilla).
* 实现 `removeStar` 变更操作，应当与 `addStar` 操作类似。
  * 仍然通过 `onStarRepository` 类方法访问 `viewerHasStarred` 属性。
  * 在类方法中根据条件执行 `addStar` 或 `removeStar` 变更操作。
  * 取消 star 一个仓库后更新本地状态。
  * 保持你的最终思路与[该实现](https://github.com/rwieruch/react-graphql-github-vanilla)一致。
> * Implement the `addReaction` mutation for an issue
* 为特定 issue 实现 `addReaction` 变更操作
> * Implement more fine-grained components (e.g. IssueList, IssueItem, ReactionList, ReactionItem)
  > * Extract components to their own files and use import and export statements to use them again in the App or other extracted components
* 将组件进一步细分（例如 IssueList、IssueItem、ReactionList 和 ReactionItem）

> ## Shortcomings of GraphQL in React without Apollo

## React 中不借助 Apollo 使用 GraphQL 的缺点

> We implemented a simple GitHub issue tracker that uses React and GraphQL without a dedicated library for GraphQL, using only axios to communicate with the GraphQL API with HTTP POST methods. I think it is important to work with raw technologies, in this case GraphQL, using plain HTTP methods, before introducing another abstraction. The Apollo library offers an abstraction that makes using GraphQL in React much easier, so you will use Apollo for your next application. For now, using GraphQL with HTTP has shown you two important things before introducing Apollo:

我们基于 React 和 GraphQL 实现了一个简单的 GitHub issue 追踪器，并未使用任何专用的 GraphQL 封装库，而是仅仅通过 axios 发送 HTTP POST 请求实现与 GraphQL API 的通信。我认为掌握底层技术是十分重要的，也就是能够在不引入额外抽象的情况下，使用普通 HTTP 请求实现 GraphQL 交互。不过，Apollo 库的抽象能够极大简化在 React 中使用 GraphQL 的成本，因此在下一个应用将使用 Apollo 进行开发。引入 Apollo 之前，使用 HTTP 与 GraphQL 的过程体现出了两项重要内容：

> * How GraphQL works when using a puristic interface such as HTTP.
> * The shortcomings of using no sophisticated GraphQL Client library in React, because you have to do everything yourself.

* GraphQL 如何结合 HTTP 这样简洁的接口工作。
* 缺乏成熟的 GraphQL 客户端工具情况下的缺点，因为你要将一切从头做起。

> Before we move on, I want to address the shortcomings of using puristic HTTP methods to read and write data to your GraphQL API in a React application:

在继续下一章内容之前，我希望总结一下现有实现存在的缺点，即 React 应用中，直接使用 HTTP 方法来实现对 GraphQL API 读写数据的方式：

> * **Complementary:** To call a GraphQL API from your client application, use HTTP methods. There are several quality libraries out there for HTTP requests, one of which is axios. That's why you have used axios for the previous application. However, using axios (or any other HTTP client library) doesn't feel like the best fit to complement a GraphQL centred interface. For instance, GraphQL doesn't use the full potential of HTTP. It's just fine to default to HTTP POST and only one API endpoint. It doesn't use resources and methods on those resources like a RESTful interface, so it makes no sense to specify a HTTP method and an API endpoint with every request, but to set it up once in the beginning instead. GraphQL comes with its own constraints. You could see it as a layer on top of HTTP when it's not as important for a developer to know about the underlying HTTP.

* **互补性：**为了从客户端应用中通过 HTTP 请求调用 GraphQL API，有很多优质的库可供选择，其中之一就是 axios，这也是为什么在上述应用中使用它的原因。然而，使用 axios（或者任何其它 HTTP 客户端）并不能很好填补 GraphQL 接口的需求。例如，GraphQL 并不需要使用全部 HTTP 功能，仅需要自动应用 POST 方法和唯一 API 地址即可。由于不需要 RESTful 接口中那样的资源路径和方法定义，因此对每个请求重复输入 HTTP 方法和 API 地址是毫无意义的，而是应当作为一劳永逸的配置。GraphQL 有着自己的约束，可以将其视作 HTTP 的上层抽象，因此底层的 HTTP 实现对于开发人员并不是必要的。

> * **Declarative:** Every time you make a query or mutation when using plain HTTP requests, you have to make a dedicated call to the API endpoint using a library such as axios. It's an imperative way of reading and writing data to your backend. However, what if there was a declarative approach to making queries and mutations? What if there was a way to co-locate queries and mutations to your view-layer components? In the previous application, you experienced how the query shape aligned perfectly with your component hierarchy shape. What if the queries and mutations would align in the same way? That's the power of co-locating your data-layer with your view-layer, and you will find out more about it when you use a dedicated GraphQL client library for it.

* **声明式：**每当需要使用 HTTP 请求实现一个查询或变更，你都需要使用 axios 之类的库对专门的 API 地址进行调用，这样是命令式地对后端读取或写入数据。然而，要是存在声明式的手段来构造查询和变更呢？要是可以在视图层组件的协同定义查询和变更呢？上述应用中，你亲身体验了组件的层次结构与查询的结构高度是如何得高度相似，要是查询和变更也同样能够达到此等一致呢？这就是协同定义数据层与视图层带来的强大力量，通过使用一个专用的 GraphQL 客户端库，你将发现更多的简化方式。

> * **Feature Support:** When using plain HTTP requests to interact with your GraphQL API, you are not leveraging the full potential of GraphQL. Imagine you want to split your query from the previous application into multiple queries that are co-located with their respective components where the data is used. That's when GraphQL would be used in a declarative way in your view-layer. But when you have no library support, you have to deal with multiple queries on your own, keeping track of all of them, and trying to merge the results in your state-layer. If you consider the previous application, splitting up the query into multiple queries would add a whole layer of complexity to the application. A GraphQL client library deals with aggregating the queries for you.

* **功能支持：**使用普通 HTTP 请求与 GraphQL API 交互时，你将无法发掘 GraphQL 的全部可能性。设想你希望把上述应用中的查询拆分，并且协同定义于实际使用数据的相应组件中，这时 GraphQL 将在视图层中以声明式的方式出现。但是当你缺少工具支持，只能人工处理多个查询，追踪各个查询的关系并且在状态层中合并查询结果。以上述应用为例，拆分查询内容会极大提升应用复杂度，而一个 GraphQL 客户端工具可以高效地实现查询间的级联。

> * **Data Handling:** The naive way for data handling with puristic HTTP requests is a subcategory of the missing feature support for GraphQL when not using a dedicated library for it. There is no one helping you out with normalizing your data and caching it for identical requests. Updating your state-layer when resolving fetched data from the data-layer becomes a nightmare when not normalizing the data in the first place. You have to deal with deeply nested state objects which lead to the verbose usage of the JavaScript spread operator. When you check the implementation of the application in the GitHub repository again, you will see that the updates of React's local state after a mutation and query are not nice to look at. A normalizing library such as [normalizr](https://github.com/paularmstrong/normalizr) could help you to improve the structure of your local state. You learn more about normalizing your state in the book [Taming the State in React](https://roadtoreact.com). In addition to a lack of caching and normalizing support, avoiding libraries means missing out on functionalities for pagination and optimistic updates. A dedicated GraphQL library makes all these features available to you.

* **数据处理：**使用基础 HTTP 请求时的原生数据处理可以认为是缺少 GraphQL 功能支持的子集，没有专用工具的情况下，没有其他人会帮你进行数据规范化或者缓存相同请求。如果没有在第一时间对数据进行规范化，那么整个数据层的更新都将变成噩梦，你将不得不面对深层的嵌套对象，导致大量不必要的使用 JavaScript 的展开语法。当你在 GitHub 仓库中查看该应用的实现，你会发现在变更或者查询后 React 本地状态的更新过程并不美观，可以使用像 [normalizr](https://github.com/paularmstrong/normalizr) 这样强大的规范化工具来帮助你改善本地状态结构，在 [Taming the State in React](https://roadtoreact.com) 中可以了解更多关于规范化状态的信息。除了缺乏缓存于规范化过程之外，缺少工具意味着缺乏像分页和乐观更新这样的功能，而一个专用的 GraphQL 工具能够保证所有这些功能可用。

> * **GraphQL Subscriptions:** While there is the concept of a query and mutation to read and write data with GraphQL, there is a third concept of a GraphQL **subscription** for receiving real-time data in a client-sided application. When you would have to rely on plain HTTP requests as before, you would have to introduce [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) next to it. It enables you to introduce a long-lived connection for receiving results over time. In conclusion, introducing GraphQL subscriptions would add another tool to your application. However, if you would introduce a GraphQL library for it on the client-side, the library would probably implement GraphQL subscriptions for you.

* **GraphQL 订阅：** GraphQL 中读写数据的概念除了查询和变更外，还存在第三个概念——**订阅**——用于在客户端应用中接收实时数据。当你依赖于基础 HTTP 请求进行读写时，如果需要实现订阅的等效功能，还需要依赖于 [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) 来接收实时数据。后者引入了长链接机制来持续接收结果。总而言之，使用 GraphQL 订阅能够为你的应用再添加一个工具，通过在客户端使用 GraphQL 工具，订阅功能可能已经被工具所自动实现。

> I am looking forward to introducing Apollo as a GraphQL client library to your React application. It will help with the aforementioned shortcomings. However, I do strongly believe it was good to learn about GraphQL in React without a GraphQL library in the beginning.

我十分期待能够为这个 React 应用引入 Apollo 作为 GraphQL 客户端工具库，它会帮助解决上述提到的所有缺点。不过，我仍然坚信在不使用工具库的情况下入门 GraphQL 是一个很好的学习方式。

| |

> You can find the final [repository on GitHub](https://github.com/rwieruch/react-graphql-github-vanilla). The repository showcases most of the exercise tasks too. The application is not feature complete since it doesn't cover all edge cases and isn't styled. However, I hope the implementation walkthrough with plain GraphQL in React has helped you to understand using only GraphQL client-side in React using HTTP requests. I feel it's important to take this step before using a sophisticated GraphQL client library such as Apollo or Relay.

你能在[这个 GitHub 仓库](https://github.com/rwieruch/react-graphql-github-vanilla)中找到应用的最终版本，它还包含了绝大多数练习的答案。鉴于仍有大量边界场景未覆盖，同时缺少样式，这个应用并不能算开发完成。不过，我希望在 React 中通过 HTTP 手动实现能够帮助你了解客户端的 GraphQL 概念，我认为在使用一个像 Apollo 和 Relay 这样成熟的 GraphQL 客户端工具之前这个步骤是十分重要的。

> I've shown how to implement a React application with GraphQL and HTTP requests without using a library like Apollo. Next, you will continue learning about using GraphQL in React using Apollo instead of basic HTTP requests with axios. The Apollo GraphQL Client makes caching your data, normalizing it, performing optimistic updates, and pagination effortless. That's not all by a long shot, so stay tuned for the next applications you are are going to build with GraphQL.

我已经演示了如何在 React 应用中通过普通的 HTTP 请求实现 GraphQL 交互，并未用到 Apollo 等工具库。之后，你将继续学习在 React 应用中通过 Apollo 来使用 GraphQL，而非基于 axios 的基础 HTTP 请求。借助 Apollo GraphQL 客户端能够零成本地实现数据缓存、数据规范化、乐观更新以及分页，甚至远不止这些，使用希望你能为后续 GraphQL 应用的开发做好准备。
