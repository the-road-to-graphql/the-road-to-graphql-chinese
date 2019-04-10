
# React 和 GraphQL 的结合

我们将一起开发一个 GraphQL 的客户端程序，从中你将了解如何将 React 与 GraphQL 结合起来。目前我们不会使用 [Apollo Client](https://github.com/apollographql/apollo-client) 或 [Relay](https://github.com/facebook/relay) 这样强大的工具来帮助你快速上手，而是使用基本的 HTTP 请求执行 GraphQL 查询和修改。在之后的下一个应用程序中，我们将引入 Apollo 作为你的 React.js 应用的 GraphQL 客户端。现阶段开发的应用只展示如何在 React 中基于 HTTP 使用 GraphQL。

在此过程中，你将构建一个类似 GitHub 的问题跟踪器，通过执行 GraphQL 的查询和修改来读写数据，基于 [GitHub's GraphQL API](https://developer.github.com/v4/) 的简单 GitHub 客户端。最终实现一个能够在 React 示例中展示 GraphQL的案例，也可以将其作为其他开发人员的学习工具，最终实现的应用程序可以参考 [GitHub 的代码库](https://github.com/rwieruch/react-graphql-github-vanilla)。

## 编写你的第一个 React GraphQL 客户端

在上一节之后，你应该准备好了如何在 React 应用程序中使用查询和修改。在本节中，你将创建一个使用 GitHub GraphQL API 的 React 应用程序。它是一个简单的问题跟踪器，需要显示在 GitHub 代码库中的一些 open issues，如果你对 React 缺乏经验，可以阅读 [React 学习之道](https://www.robinwieruch.de/the-road-to-learn-react)，了解更多相关知识，为接下来的部分做好充分的准备。

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

现阶段组件只会渲染一个 `title` 作为标题。在实现其他的 React 组件之前，我们先安装一个使用 HTTP POST 方法执行查询和修改的第三方库来处理 GraphQL 请求。推荐使用 [axios](https://github.com/axios/axios)。在命令行中，输入如下命令在项目文件夹中安装 axios：

{title="Command Line",lang="json"}
~~~~~~~~
npm install axios --save
~~~~~~~~

接下来，可以在你的 App 组件中导入并配置它。它可以简化后续的开发步骤，因为在某种程度上，你只需要用你的 access token 和 GitHub 的 GraphQL API 配置它一次。

首先，在从 axios 创建配置实例时，为它定义一个基本的 URL。如前所述，你不需要在每次发出请求时都定义 GitHub 的 URL 端点，因为所有查询和修改都指向 GraphQL 中的相同 URL 端点。你可以使用对象和字段灵活地进行查询和修改结构。

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

用你的 access token 替换 `YOUR_GITHUB_PERSONAL_ACCESS_TOKEN` 字符串。为了避免将访问令牌直接剪切和粘贴到源代码中，你可以创建一个 *.env* 文件来保存项目文件夹中命令行上的所有环境变量。如果不想在公开的GitHub代码库中共享个人令牌，可以将该文件添加到.gitignore。

{title="Command Line",lang="json"}
~~~~~~~~
touch .env
~~~~~~~~

将环境变量定义在这个 *.env* 文件中。在使用 create-react-app 时，确保遵循正确的命名约束，它使用 `REACT_APP` 作为每个 key 的前缀。将下面的键值对粘贴在你的 *.env* 文件中。密钥必须有 `REACT_APP` 前缀，并且值必须是你 GitHub 的 access token 。

{title=".env",lang="javascript"}
~~~~~~~~
REACT_APP_GITHUB_PERSONAL_ACCESS_TOKEN=xxxXXX
~~~~~~~~

现在，你可以将 access token 作为环境变量传递给 axios 配置，并使用字符串插值（ [模板字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) ）创建一个配置好的axios实例。

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

我们可以分两步来实现这个场景。render方法需要渲染一个带有输入栏的表单。表单必须有一个 `onSubmit` 处理方法，而输入栏需要一个 `onChange` 处理方法。输入栏使用组件中 state 的 `path` 作为值成为一个受控组件。

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

你可能在前面的实现中使用了以前未使用过的 React 类组件语法。如果你不熟悉它，请查看这个 [GitHub 代码库](https://github.com/the-road-to-learn-react/react-alternative-class-component-syntax) 以获得更多的理解。使用**类字段声明**可以省略初始化本地状态的构造函数语句，而且不需要绑定类方法。相反，箭头函数将处理所有绑定行为。

依照 React 的最佳实践，将输入栏设置为受控组件。输入元素不应使用原生的 HTML 行为处理其内部状态；而应该是自然反应。（译者注：原文为：The input element shouldn't be used to handle its internal state using native HTML behavior; it should be React.）

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

为表单提前设置——可使用的输入栏、一个提交按钮、`onChange()` 和 `onSubmit()` 类方法——是React中实现表单的一种常见方法。唯一增加的是 `componentDidMount()` 生命周期方法中的初始数据获取，通过为查询提供一个初始状态来从后端请求数据，从而改进用户体验。它是 [React 获取第三方API数据](https://www.robinwieruch.de/react-fetching-data/) 的基础。

When you start the application on the command line, you should see the initial state for the `path` in the input field. You should be able to change the state by entering something else in the input field, but nothing happens with `componentDidMount()` and submitting the form yet.

当你在命令行上启动应用程序时，应该会在输入栏中看到 `path` 的初始状态。并且能够通过在输入栏中输入其他内容来更改状态，但是在 `componentDidMount()` 方法中以及提交表单时什么都不会发生。

You might wonder why there is only one input field to grab the information about the organization and repository. When opening up a repository on GitHub, you can see that the organization and repository are encoded in the URL, so it becomes a convenient way to show the same URL pattern for the input field. You can also split the `organization/repository` later at the `/` to get these values and perform the GraphQL query request.

你可能想知道为什么只需要一个输入字段来获取关于组织和仓库的信息。在 GitHub 上打开代码库时，你可以看到组织和代码库都是在 URL 中编码的，因此可以方便地用输入的字段来匹配相似的 URL模式。你之后也可以以 `/` 分割 `organization/repository`，以获取这些值并执行 GraphQL 查询请求。

### 练习：

* 查看 [本节源码](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/ca7b278b8f602c46dfac64a1304d39a8e8e0006b)

* 如果你不熟悉React，可以阅读 *React 学习之道*

## 在 React 中执行 GraphQL 查询

在本章节中，你将在 React 中实现第一个 GraphQL 查询，从组织的代码库中获取 issues，但不是一次获取所有问题，先获取一个组织。我们先将查询定义为 App 组件上面的一个变量。

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

使用 JavaScript 中的模板字符串将查询语句定义为多行的字符串。与之前在 GraphiQL 或 GitHub Explorer 中使用的查询相同。现在你就可以使用 axios 向 GitHub 的 GraphiQL API 发出 POST 请求。axios 的配置已经指向正确的 API 端点，并使用的是你的 access token 。唯一剩下的事情是在 POST 请求期间将查询作为有效负载传递给它。端点的参数可以是空字符串，因为你在配置中定义了端点。当 App 组件生命周期执行到 `componentDidMount()` 时，就会执行请求。在 axios 的 promise 被解析之后，结果将会保留在控制台日志中。

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

再次启动你的应用程序，并确认你已经在开发者模式下的控制台日志中得到了结果。如果得到了  [401 HTTP 状态码](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)，说明你没有正确设置你的 access token 。否则，你应该会在开发人员控制台日志中看到类似的结果。

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

最外层信息是 axios 的元信息返回给你所发请求的所有内容。它都是 axios，与 GraphQL 还没有任何关系，这就是为什么它的大部分被占位符所替代。Axios 有一个 `data` 属性，代表 Axios 请求的结果。里面包裹了一个反映GraphQL结果的 `data` 属性。首先，`data` 属性在第一个结果中看起来是冗余的，但是一旦理解了它，你就会知道一个 `data` 属性来自 axios，而另一个来自GraphQL数据结构。最后，在第二个 `data` 属性中找到GraphQL查询的结果。在里面也可以找到包含已解析的名称和 url 字段作为字符串属性的组织。

下一步，你将把包含组织信息的结果存储在 React的本地状态中。如果发生任何错误，还需要将潜在的错误存储在状态中。

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

接下来，你可以在App组件的 `render()` 方法中展示组织信息:

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

将组织组件作为一个新的功能无状态组件引入，保持App组件的渲染方法简洁。因为这个应用程序只是一个简单的GitHub 问题跟踪器，你已经可以在一小段话里提到它了（译者注：原文为 Because this application is going to be a simple GitHub issue tracker, you can already mention it in a short paragraph.）。

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

在最后一步中，你需要决定在还没有获取任何东西时界面应该渲染什么，以及在发生错误时应该渲染什么。可以在 React 中使用 [条件呈现](https://www.robinwieruch.de/conditional-rendering-react/) 解决这些边界情况。对于第一个边缘情况，只需检查 `organization` 是否存在。

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

对于第二种边缘情况，你已经将错误传递给了组织组件。如果出现了错误，它应该简单地将每个错误的错误消息渲染出来。否则，它应该渲染组织信息。对于 GraphQL 中的不同字段和环境，可能会有多个错误。

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

### GraphQL Nested Objects in React

Next, we'll request a nested object for the organization. Since the application will eventually show the issues in a repository, you should fetch a repository of an organization as the next step. Remember, a query reaches into the GraphQL graph, so we can nest the `repository` field in the `organization` when the schema defined the relationship between these two entities.

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

In this case, the repository name is identical to the organization. That's okay for now. Later on, you can define an organization and repository on your own dynamically. In the second step, you can extend the Organization component with another Repository component as child component. The result for the query should now have a nested repository object in the organization object.

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

The GraphQL query structure aligns perfectly with your component tree. It forms a natural fit to continue extending the query structure like this, by nesting other objects into the query, and extending the component tree along the structure of the GraphQL query. Since the application is an issue tracker, we need to add a list field of issues to the query.

If you want to follow the query structure more thoughtfully, open the "Docs" sidebar in GraphiQL to learn out about the types `Organization`, `Repository`, `Issue`. The paginated issues list field can be found there as well. It's always good to have an overview of the graph structure.

Now let's extend the query with the list field for the issues. These issues are a paginated list in the end. We will cover these more later; for now, nest it in the `repository` field with a `last` argument to fetch the last items of the list.

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

You can also request an id for each issue using the `id` field on the issue's `node` field, to use a `key` attribute for your list of rendered items in the component, which is considered [best practice in React](https://reactjs.org/docs/lists-and-keys.html). Remember to adjust the name of the query variable when its used to perform the request.

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

The component structure follows the query structure quite naturally again. You can add a list of rendered issues to the Repository component. It is up to you to extract it to its own component as a refactoring to keep your components concise, readable, and maintainable.

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

That's it for the nested objects, fields, and list fields in a query. Once you run your application again, you should see the last issues of the specified repository rendered in your browser.

### GraphQL Variables and Arguments in React

Next we'll make use of the form and input elements. They should be used to request the data from GitHub's GraphQL API when a user fills in content and submits it. The content is also used for the initial request in `componentDidMount()` of the App component. So far, the organization `login` and repository `name` were inlined arguments in the query. Now, you should be able to pass in the `path` from the local state to the query to define dynamically an organization and repository. That's where variables in a GraphQL query came into play, do you remember?

First, let's use a naive approach by performing string interpolation with JavaScript rather than using GraphQL variables. To do this, refactor the query from a template literal variable to a function that returns a template literal variable. By using the function, you should be able to pass in an organization and repository.

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

Next, call the `onFetchFromGitHub()` class method in the submit handle, but also when the component mounts in `componentDidMount()` with the initial local state of the `path` property. These are the two essential places to fetch the data from the GraphQL API on initial render, and on every other manual submission from a button click.

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

Lastly, call the function that returns the query instead of passing the query string directly as payload. Use the [JavaScript's split method on a string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split) to retrieve the prefix and suffix of the `/` character from the path variable where the prefix is the organization and the suffix is the repository.

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

Since the split returns an array of values and it is assumed that there is only one slash in the path, the array should consist of two values: the organization and the repository. That's why it is convenient to use a JavaScript array destructuring to pull out both values from an array in the same line.

Note that the application is not built for to be robust, but is intended only as a learning experience. It is unlikely anyone will ask a user to input the organization and repository with a different pattern than *organization/repository*, so there is no validation included yet. Still, it is a good foundation for you to gain experience with the concepts.

If you want to go further, you can extract the first part of the class method to its own function, which uses axios to send a request with the query and return a promise. The promise can be used to resolve the result into the local state, using `this.setState()` in the `then()` resolver block of the promise.

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

You can always split your applications into parts, be they functions or components, to make them concise, readable, reusable and [testable](https://www.robinwieruch.de/react-testing-tutorial/). The function that is passed to `this.setState()` can be extracted as higher-order function. It has to be a higher-order function, because you need to pass the result of the promise, but also provide a function for the `this.setState()` method.

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

Now you've made your query flexible by providing dynamic arguments to your query. Try it by starting your application on the command line and by filling in a different organization with a specific repository (e.g. *facebook/create-react-app*).

It's a decent setup, but there was nothing to see about variables yet. You simply passed the arguments to the query using a function and string interpolation with template literals. Now we'll use GraphQL variables instead, to refactor the query variable again to a template literal that defines inline variables.

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

Now you can pass those variables as arguments next to the query for the HTTP POST request:

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

Finally, the query takes variables into account without detouring into a function with string interpolation. I strongly suggest practicing with the exercises below before continuing to the next section. We've yet to discuss features like fragments or operation names, but we'll soon cover them using Apollo instead of plain HTTP with axios.

### Exercises:

* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/c08126a9ec91dde4198ae85bb2f194fa7767c683)
* Explore and add fields to your organization, repository and issues
  * Extend your components to display the additional information
* Read more about [serving a GraphQL API over HTTP](http://graphql.org/learn/serving-over-http/)

## GraphQL Pagination in React

Last section you implemented a list field in your GraphQL query, which fit into the flow of structuring the query with nested objects and a list responsible for showing partial results of the query in React.

In this section, you will explore pagination with list fields with GraphQL in React in more detail. Initially, you will learn more about the arguments of list fields. Further, you will add one more nested list field to your query. Finally, you will fetch another page of the paginated `issues` list with your query.

Let's start by extending the `issues` list field in your query with one more argument:

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

If you read the arguments for the `issues` list field using the "Docs" sidebar in GraphiQL, you can explore which arguments you can pass to the field. One of these is the `states` argument, which defines whether or not to fetch open or closed issues. The previous implementation of the query has shown you how to refine the list field, in case you only want to show open issues. You can explore more arguments for the `issues` list field, but also for other list fields, using the documentation from Github's API.

Now we'll implement another nested list field that could be used for pagination. Each issue in a repository can have reactions, essentially emoticons like a smiley or a thumbs up. Reactions can be seen as another list of paginated items. First, extend the query with the nested list field for reactions:

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

Second, render the list of reactions in one of your React components again. Implement dedicated List and Item components, such as ReactionsList and ReactionItem for it. As an exercise, try keeping the code for this application readable and maintainable.

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

You extended the query and React's component structure to render the result. It's a straightforward implementation when you are using a GraphQL API as your data source which has a well defined underlying schema for these field relationships.

Lastly, you will implement real pagination with the `issues` list field, as there should be a button to fetch more issues from the GraphQL API to make it a function of a completed application. Here is how to implement a button:

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

The handler for the button passes through all the components to reach the Repository component:

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

Logic for the function is implemented in the App component as class method. It passes to the Organization component as well.

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

Before implementing the logic for it, there needs to be a way to identify the next page of the paginated list. To extend the inner fields of a list field with fields for meta information such as the `pageInfo` or the `totalCount` information, use `pageInfo` to define the next page on button-click. Also, the `totalCount` is only a nice way to see how many items are in the next list:

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

Now, you can use this information to fetch the next page of issues by providing the cursor as a variable to your query. The cursor, or the `after` argument, defines the starting point to fetch more items from the paginated list.

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

The second argument wasn't introduced  to the `onFetchFromGitHub()` class method yet. Let's see how that turns out.

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

The argument is simply passed to the `getIssuesOfRepository()` function, which makes the GraphQL API request, and returns the promise with the query result. Check the other functions that call the `onFetchFromGitHub()` class method, and notice how they don't make use of the second argument, so the cursor parameter will be `undefined` when it's passed to the GraphQL API call. Either the query uses the cursor as argument to fetch the next page of a list, or it fetches the initial page of a list by having the cursor not defined at all:

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

In the previous template string, the `cursor` is passed as variable to the query and used as `after` argument for the list field. The variable is not enforced though, because there is no exclamation mark next to it, so it can be `undefined`. This happens for the initial page request for a paginated list, when you only want to fetch the first page. Further, the argument `last` has been changed to `first` for the `issues` list field, because there won't be another page after you fetched the last item in the initial request. Thus, you have to start with the first items of the list to fetch more items until you reach the end of the list.

That's it for fetching the next page of a paginated list with GraphQL in React, except one final step. Nothing updates the local state of the App component about a page of issues yet, so there are still only the issues from the initial request. You want to merge the old pages of issues with the new page of issues in the local state of the App component, while keeping the organization and repository information in the deeply nested state object intact. The perfect time for doing this is when the promise for the query resolves. You already extracted it as a function outside of the App component, so you can use this place to handle the incoming result and return a result with your own structure and information. Keep in mind that the incoming result can be an initial request when the App component mounts for the first time, or after a request to fetch more issues happens, such as when the "More" button is clicked.

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

The function is a complete rewrite, because the update mechanism is more complex now. First, you passed the `cursor` as an argument to the function, which determines whether it was an initial query or a query to fetch another page of issues. Second, if the `cursor` is `undefined`, the function can return early with the state object that encapsulates the plain query result, same as before. There is nothing to keep intact in the state object from before, because it is an initial request that happens when the App component mounts or when a user submits another request which should overwrite the old state anyway. Third, if it is a fetch more query and the cursor is there, the old and new issues from the state and the query result get merged in an updated list of issues. In this case, a JavaScript destructuring alias is used to make naming both issue lists more obvious. Finally, the function returns the updated state object. Since it is a deeply nested object with multiple levels to update, use the JavaScript spread operator syntax to update each level with a new query result. Only the `edges` property should be updated with the merged list of issues.

Next, use the `hasNextPage` property from the `pageInfo` that you requested to show a "More" button (or not). If there are no more issues in the list, the button should disappear.

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

Now you've implemented pagination with GraphQL in React. For practice, try more arguments for your issues and reactions list fields on your own. Check the "Docs" sidebar in GraphiQL to find out about arguments you can pass to list fields. Some arguments are generic, but have arguments that are specific to lists. These arguments should show you how finely-tuned requests can be with a GraphQL query.

### Exercises:

* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/060677346e8955fb1a6c7579859ce92e62e1f406)
* Explore further arguments, generic or specific for the type, on the `issues` and `reactions` list fields
  * Think about ways to beautify the updating mechanism of deeply nested state objects and [contribute your thoughts to it](https://github.com/rwieruch/react-graphql-github-apollo/pull/14)

## GraphQL Mutation in React

You fetched a lot of data using GraphQL in React, the larger part of using GraphQL. However, there are always two sides to such an interface: read and write. That's where GraphQL mutations complement the interface. Previously, you learned about GraphQL mutations using GraphiQL without React. In this section, you will implement such a mutation in your React GraphQL application.

You have executed GitHub's `addStar` mutation before in GraphiQL. Now, let's implement this mutation in React. Before implementing the mutation, you should query additional information about the repository, which is partially required to star the repository in a mutation.

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

The `viewerHasStarred` field returns a boolean that tells whether the viewer has starred the repository or not. This boolean helps determine whether to execute a `addStar` or `removeStar` mutation in the next steps. For now, you will only implement the `addStar` mutation. The `removeStar` mutation will be left off as part of the exercise. Also, the `id` field in the query returns the identifier for the repository, which you will need to clarify the target repository of your mutation.

The best place to trigger the mutation is a button that stars or unstars the repository. That's where the `viewerHasStarred` boolean can be used for a conditional rendering to show either a "Star" or "Unstar" button. Since you are going to star a repository, the Repository component is the best place to trigger the mutation.

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

To identify the repository to be starred, the mutation needs to know about the `id` of the repository. Pass the `viewerHasStarred` property as a parameter to the handler, since you'll use the parameter to determine whether you want to execute the star or unstar mutation later.

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

The handler should be defined in the App component. It passes through each component until it reaches the Repository component, also reaching through the Organization component on its way.

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

Now it can be defined in the App component. Note that the `id` and the `viewerHasStarred` information can be destructured from the App's local state, too. This is why you wouldn't need to pass this information in the handler, but use it from the local state instead. However, since the Repository component knew about the information already, it is fine to pass the information in the handler, which also makes the handler more explicit. It's also good preparation for dealing with multiple repositories and repository components later, since the handler will need to be more specific in these cases.

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

Now, you can implement the handler. The mutation can be outsourced from the component. Later, you can use the `viewerHasStarred` boolean in the handler to perform a `addStar` or `removeStar` mutation. Executing the mutation looks similar to the GraphQL query from before. The API endpoint is not needed, because it was set in the beginning when you configured axios. The mutation can be sent in the `query` payload, which we'll cover later. The `variables` property is optional, but you need to pass the identifier.

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

Before you define the `addStar` mutation, check GitHub's GraphQL API again. There, you will find all information about the structure of the mutation, the required arguments, and the available fields for the result. For instance, you can include the `viewerHasStarred` field in the returned result to get an updated boolean of a starred or unstarred repository.

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

You could already execute the mutation in the browser by clicking the button. If you haven't starred the repository before, it should be starred after clicking the button. You can visit the repository on GitHub to get visual feedback, though you won't see any results reflected yet. The button still shows the "Star" label when the repository wasn't starred before, because the `viewerHasStarred` boolean wasn't updated in the local state of the App component after the mutation. That's the next thing you are going to implement. Since axios returns a promise, you can use the `then()` method on the promise to resolve it with your own implementation details.

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

When resolving the promise from the mutation, you can find out about the `viewerHasStarred` property in the result. That's because you defined this property as a field in your mutation. It returns a new state object for React's local state, because you used the function in `this.setState()`. The spread operator syntax is used here, to update the deeply nested data structure. Only the `viewerHasStarred` property changes in the state object, because it's the only property returned by the resolved promise from the successful request. All other parts of the local state stay intact.

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

Now try to star the repository again. You may have to go on the GitHub page and unstar it first. The button label should adapt to the updated `viewerHasStarred` property from the local state to show a "Star" or "Unstar" label. You can use what you've learned about starring repositories to implement a `removeStar` mutation.

 We also want to show the current number of people who have starred the repository, and update this count in the `addStar` and `removeStar` mutations. First, retrieve the total count of stargazers by adding the following fields to your query:

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

Second, you can show the count as a part of your button label:

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

Now we want the count to update when you star (or unstar) a repository. It is the same issue as the missing update for the `viewerHasStarred` property in the local state of the component after the `addStar` mutation succeeded. Return to your mutation resolver and update the total count of stargazers there as well. While the stargazer object isn't returned as a result from the mutation, you can increment and decrement the total count after a successful mutation manually using a counter along with the `addStar` mutation.

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

You have implemented your first mutation in React with GraphQL. So far, you have just implemented the `addStar` mutation. Even though the button already reflects the `viewerHasStarred` boolean by showing a "Star" or "Unstar" label, the button showing "Unstar" should still execute the `addStar` mutation. The `removeStar` mutation to unstar the repository is one of the practice exercises mentioned below.

### Exercises:

* Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-vanilla/tree/3dcd95e32ef24d9e716a1e8ac144b62c0f41ca3c)
* Implement the `removeStar` mutation, which is used analog to the `addStar` mutation.
  * The `onStarRepository` class method has already access to the `viewerHasStarred` property.
  * Conditionally execute a `addStar` or `removeStar` mutation in the class handler.
  * Resolve the new state after removing a star from a repository.
  * Align your final thoughts with [this implementation](https://github.com/rwieruch/react-graphql-github-vanilla).
* Implement the `addReaction` mutation for an issue
* Implement more fine-grained components (e.g. IssueList, IssueItem, ReactionList, ReactionItem)
  * Extract components to their own files and use import and export statements to use them again in the App or other extracted components

## Shortcomings of GraphQL in React without Apollo

We implemented a simple GitHub issue tracker that uses React and GraphQL without a dedicated library for GraphQL, using only axios to communicate with the GraphQL API with HTTP POST methods. I think it is important to work with raw technologies, in this case GraphQL, using plain HTTP methods, before introducing another abstraction. The Apollo library offers an abstraction that makes using GraphQL in React much easier, so you will use Apollo for your next application. For now, using GraphQL with HTTP has shown you two important things before introducing Apollo:

* How GraphQL works when using a puristic interface such as HTTP.
* The shortcomings of using no sophisticated GraphQL Client library in React, because you have to do everything yourself.

Before we move on, I want to address the shortcomings of using puristic HTTP methods to read and write data to your GraphQL API in a React application:

* **Complementary:** To call a GraphQL API from your client application, use HTTP methods. There are several quality libraries out there for HTTP requests, one of which is axios. That's why you have used axios for the previous application. However, using axios (or any other HTTP client library) doesn't feel like the best fit to complement a GraphQL centred interface. For instance, GraphQL doesn't use the full potential of HTTP. It's just fine to default to HTTP POST and only one API endpoint. It doesn't use resources and methods on those resources like a RESTful interface, so it makes no sense to specify a HTTP method and an API endpoint with every request, but to set it up once in the beginning instead. GraphQL comes with its own constraints. You could see it as a layer on top of HTTP when it's not as important for a developer to know about the underlying HTTP.

* **Declarative:** Every time you make a query or mutation when using plain HTTP requests, you have to make a dedicated call to the API endpoint using a library such as axios. It's an imperative way of reading and writing data to your backend. However, what if there was a declarative approach to making queries and mutations? What if there was a way to co-locate queries and mutations to your view-layer components? In the previous application, you experienced how the query shape aligned perfectly with your component hierarchy shape. What if the queries and mutations would align in the same way? That's the power of co-locating your data-layer with your view-layer, and you will find out more about it when you use a dedicated GraphQL client library for it.

* **Feature Support:** When using plain HTTP requests to interact with your GraphQL API, you are not leveraging the full potential of GraphQL. Imagine you want to split your query from the previous application into multiple queries that are co-located with their respective components where the data is used. That's when GraphQL would be used in a declarative way in your view-layer. But when you have no library support, you have to deal with multiple queries on your own, keeping track of all of them, and trying to merge the results in your state-layer. If you consider the previous application, splitting up the query into multiple queries would add a whole layer of complexity to the application. A GraphQL client library deals with aggregating the queries for you.

* **Data Handling:** The naive way for data handling with puristic HTTP requests is a subcategory of the missing feature support for GraphQL when not using a dedicated library for it. There is no one helping you out with normalizing your data and caching it for identical requests. Updating your state-layer when resolving fetched data from the data-layer becomes a nightmare when not normalizing the data in the first place. You have to deal with deeply nested state objects which lead to the verbose usage of the JavaScript spread operator. When you check the implementation of the application in the GitHub repository again, you will see that the updates of React's local state after a mutation and query are not nice to look at. A normalizing library such as [normalizr](https://github.com/paularmstrong/normalizr) could help you to improve the structure of your local state. You learn more about normalizing your state in the book [Taming the State in React](https://roadtoreact.com). In addition to a lack of caching and normalizing support, avoiding libraries means missing out on functionalities for pagination and optimistic updates. A dedicated GraphQL library makes all these features available to you.

* **GraphQL Subscriptions:** While there is the concept of a query and mutation to read and write data with GraphQL, there is a third concept of a GraphQL **subscription** for receiving real-time data in a client-sided application. When you would have to rely on plain HTTP requests as before, you would have to introduce [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) next to it. It enables you to introduce a long-lived connection for receiving results over time. In conclusion, introducing GraphQL subscriptions would add another tool to your application. However, if you would introduce a GraphQL library for it on the client-side, the library would probably implement GraphQL subscriptions for you.

I am looking forward to introducing Apollo as a GraphQL client library to your React application. It will help with the aforementioned shortcomings. However, I do strongly believe it was good to learn about GraphQL in React without a GraphQL library in the beginning.

| |

You can find the final [repository on GitHub](https://github.com/rwieruch/react-graphql-github-vanilla). The repository showcases most of the exercise tasks too. The application is not feature complete since it doesn't cover all edge cases and isn't styled. However, I hope the implementation walkthrough with plain GraphQL in React has helped you to understand using only GraphQL client-side in React using HTTP requests. I feel it's important to take this step before using a sophisticated GraphQL client library such as Apollo or Relay.

I've shown how to implement a React application with GraphQL and HTTP requests without using a library like Apollo. Next, you will continue learning about using GraphQL in React using Apollo instead of basic HTTP requests with axios. The Apollo GraphQL Client makes caching your data, normalizing it, performing optimistic updates, and pagination effortless. That's not all by a long shot, so stay tuned for the next applications you are are going to build with GraphQL.
