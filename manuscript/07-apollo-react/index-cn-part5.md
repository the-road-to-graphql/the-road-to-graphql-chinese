> ## Implementing the Issues Feature: Setup
## 实现 Issues 功能：准备

> In the previous sections you have implemented most of the common Apollo Client features in your React application. Now you can start implementing extensions for the application on your own. This section showcases how a full-fledged feature can be implemented with Apollo Client in React.

在前面的小节中你已经在你的 React 程序中实现了大部分常用的 Apollo Client 的功能。现在你可以开始自己实现程序的扩展。本节将展示如何在 React 中使用 Apollo Client 实现一个完整的功能。

> So far, you have dealt with GitHub repositories from organizations and your account. This will take that one step further, fetching GitHub issues that are made available using a list field associated to a repository in a GraphQL query. However, this section doesn't only show you how to render a nested list field in your React application.

到目前为止，你已经处理了来自组织和你的账户的 GitHub 代码库。我们将更进一步，用 GraphQL 查询中的列表字段关联来获取代码库可用的问题列表。然而，本节不仅仅向你展示如何在 React 中渲染嵌套列表。

> The foundation will be rendering the list of issues. You will implement client-side filtering with plain React to show opened, closed, or no issue. Finally, you will refactor the filtering to a server-side filtering using GraphQL queries. We will only fetch the issues by their state from the server rather than filtering the issue's state on the client-side. Implementing pagination for the issues will be your exercise.

最基本的将会渲染出问题列表，你将使用纯粹的 React 来实现客户端侧的打开，关闭或者不显示问题的筛选。最后，你将使用 GraphQL 查询将筛选重构为服务器端筛选。我们只根据问题列表的状态从服务端获取，而不是在客户端侧去过滤 issue 的状态。你的练习是为这些列表实现分页。


> First, render a new component called 'Issues' in your RepositoryList component. This component takes two props that are used later in a GraphQL query to identify the repository from which you want to fetch the issues.

首先，在 RepositoryList 组件中渲染一个名叫 Issues 的新组件。这个组件接收两个 props，用于稍后在 GraphQL 查询获取问题列表的代码库的标识。

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

在 src/Issue/index.js 文件中，引入和导出 Issues 组件，由于问题功能保持在单独的模块中，所以它有这个 *index.js* 文件，这样你可以告诉别的开发人员仅仅使用 *index.js* 使用这个功能模块。*index.js* 作为接口文件，而其他的都保持私有。

{title="src/Issue/index.js",lang="javascript"}
~~~~~~~~
import Issues from './IssueList';

export default Issues;
~~~~~~~~

> Note how the component is named Issues, not IssueList. The naming convention is used to break down the rendering of a list of items: Issues, IssueList and IssueItem. Issues is the container component, where you query the data and filter the issues, and the IssueList and IssueItem are only there as presentational components for rendering. In contrast, the Repository feature module hasn't a Repositories component, because there was no need for it. The list of repositories already came from the Organization and Profile components and the Repository module's components are mainly only there for the rendering. This is only one opinionated approach of naming the components, however.

请注意组件的名字是 Issues，而不是 IssueList。拆分渲染列表的命名约定为：Issues, IssueList 和 IssueItem。Issues 是容器组件，用来获取数据和过滤问题，IssueList 和 IssueItem 组件仅仅用做内容展示。相反，代码库功能模块并没有一个 Repositories 组件，因为并不需要它，代码库列表已经在 Organization 和 Profile 组件中，并且代码库模块的组件主要用于内容展示。当然，这只是一种命名组件的方式。

> Let's start implementing Issues and IssueList components in the *src/Issue/IssueList/index.js* file. You could argue to split both components up into their own files, but for the sake of this tutorial, they are kept together in one file.

让我们开始在 *src/Issue/IssueList/index.js* 文件中实现 Issues 和 IssueList 组件，你可能会认为应该将这两个组件拆分到各自的文件中，但由于本教程的缘故，它们被保存在同一个文件中。

> First, there needs to be a new query for the issues. You might wonder: Why do we need a new query here? It would be simpler to include the issues list field in the query at the top next to the Organization and Profile components. That's true, but it comes with a cost. Adding more nested (list) fields to a query often results into performance issues on the server-side. There you may have to make multiple roundtrips to retrieve all the entities from the database.

首先，需要对问题进行一个新的查询，你可能会想：为什么这里需要一个新的查询？将问题列表字段包含在 Organization 和 Profile 组件顶部的查询会更简单。确实是这样，但也会有代价。向查询添加更多嵌套（列表）字段常常会导致服务器端的性能问题。所以你可能需要多次往返才能从数据库中检索所有的数据实体。

> * Roundtrip 1: get organization by name
> * Roundtrip 2: get repositories of organization by organization identifier
> * Roundtrip 3: get issues of repository by repository identifier

* 往返1：根据名称获取组织
* 往返2：根据组织标识获取组织下的代码库
* 往返3：根据代码库标识获取问题列表

> It is simple to conclude that nesting queries in a naive way solves all of our problems. Whereas it solves the problem of only requesting the data once and not with multiple network request (similar roundtrips as shown for the database), GraphQL doesn't solve the problem of retrieving all the data from the database for you. That's not the responsibility of GraphQL after all. So by having a dedicated query in the Issues component, you can decide **when** to trigger this query. In the next steps, you will just trigger it on render because the Query component is used. But when adding the client-side filter later on, it will only be triggered when the "Filter" button is toggled. Otherwise the issues should be hidden. Finally, that's how all the initial data loading can be delayed to a point when the user actually wants to see the data.

很容易得出一个结论，以一种简单的方式嵌套查询可以解决我们所有的问题。虽然它解决了只请求一次数据而不是用多个网络请求（类似于数据库的往返）的问题，但是 GraphQL 并没有为你解决从数据库检索所有数据的问题，这毕竟不是 GraphQL 的责任。因此，通过在 Issues 组件中有个专门的查询，你可以决定**何时**触发这个查询。在接下来的步骤中，你将会在渲染时触发，因为使用了 Query 组件。但当稍后添加了客户端测的过滤器时，只有在“过滤”按钮被切换时才会触发查询，否则问题列表应该被隐藏。最终，这就是如何将所有初始数据加载延迟到用户真正想要查看数据的时候。

> First, define the Issues component which has access to the props which were passed in the RepositoryList component. It doesn't render much yet.

首先，定义一个能获取从 RepositoryList 组件传递下来 props 的 Issues 组件，它暂时还没渲染什么内容。

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

然后，在 *src/Issue/IssueList/index.js* 文件中定义查询用来获取代码库问题。代码库的所有者和名字作为唯一标识，另外，添加 `state` 字段作为查询结果的字段之一，这个用于客户端测的筛选，用来显示打开或关闭状态的问题。

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

第三步，引入 Query 组件，并将前面定义的查询和必要的变量传递给它。在 children 中使用一个函数来访问数据来覆盖所有边缘情况并最终渲染一个 IssueList 组件。

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

最后，在 *src/Issue/IssueItem/index.js* 中实现一个基本的 IssueItem 组件。下面的代码片段展示了一个占位符，你可以在其中实现评论的功能，稍后我们将会进行讨论。

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

当你再次启动程序时，你应该会看到每个代码库下面渲染出了分页后问题的初始页面。这是个性能瓶颈，更糟糕的是，和 Organization 和 Profile 组件中的问题列表字段一样，GraphQL 请求没有合并在一个请求中。在接下来的步骤中，你将实现客户端侧的筛选。默认情况不显示任何问题，但是它可以使用一个按钮在不显示，打开问题和关闭问题的状态之间切换，因此在切换问题状态之前不会查询问题。

> ### Exercises:
### 练习

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/6781b487d6799e55a4deea48dfe706253b373f0a)
> * Read more about [the rate limit when using a (or in this case GitHub's) GraphQL API](https://developer.github.com/v4/guides/resource-limitations/)

* 确认[最后一节的代码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/6781b487d6799e55a4deea48dfe706253b373f0a)
* [在使用（或者在本例中是 Github 的）GraphQL API 时，请阅读有关速率限制](https://developer.github.com/v4/guides/resource-limitations/)的更多信息

> ## Implementing the Issues Feature: Client-Side Filter

## 实现 Issue 功能: 客户端过滤

> In this section, we enhance the Issue feature with client-side filtering. It prevents the initial issue querying because it happens with a button, and it lets the user filter between closed and open issues.

在本节中，我们将用客户端侧筛选增强问题功能。它可以阻止初始的问题查询，因为将由一个按钮触发，并且允许用户在关闭和打开问题之间进行筛选。

> First, let's introduce our three states as enumeration next to the Issues component. The `NONE` state is used to show no issues; otherwise, the other states are used to show open or closed issues.

首先，让我介绍在 Issues 组件旁边的三个枚举状态，`NONE` 状态用于不显示问题列表，否则，其他状态用于显示打开或者关闭的问题。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const ISSUE_STATES = {
  NONE: 'NONE',
  OPEN: 'OPEN',
  CLOSED: 'CLOSED',
};
~~~~~~~~

> Second, let's implement a short function that decides whether it is a state to show the issues or not. This function can be defined in the same file.

其次，让我们实现一个短函数来决定问题是否显示的状态。这个函数可以在同一个文件中进行定义。

{title="src/Issue/IssueList/index.js",lang="javascript"}
~~~~~~~~
const isShow = issueState => issueState !== ISSUE_STATES.NONE;
~~~~~~~~

> Third, the function can be used for conditional rendering, to either query the issues and show the IssueList, or to do nothing. It's not clear yet where the `issueState` property comes from.

第三，该函数可以用于条件渲染，查询问题并显示 IssueList，或者什么也不做，目前还不清楚 `issueState` 属性的来源。

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

程序现在应该是无错的，因为初始状态被设置为 `NONE`，条件渲染阻止查询和结果渲染。但是，客户端侧的筛选还未完成，因为你仍然需要使用 React 的组件状态来切换 `issueState` 属性。ButtonUnobtrusive 组件具有适合的样式，因此我们可以复用它来实现这种切换行为，以便在三种可用状态中进行切换。

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

正如你所看到的，前一个枚举只将标签匹配到给定状态，而下一个枚举将下一个状态匹配到给定状态，这就是如何简单的切换到下一个状态的方法。最后一个重点是，组件内的 `issueState` 需要使用来过滤问题列表，然后查询并渲染它们。

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

你已经实现了客户端筛选，这个按钮用于在组件内状态中管理三个状态之间的切换。问题只会在有筛选状态时才会查询和渲染。在下一步中，应该将现有的客户端侧过滤提升到服务端侧过滤，这意味着过滤后的问题是从服务端请求，而不是在客户端上过滤。

> ### Exercises:
### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/0f261b13696046832ad65f1909266957d6275d6c)
> * Install the [recompose](https://github.com/acdlite/recompose) library which implements many higher-order components
> * Refactor the Issues component from class component to functional stateless component
> * Use the `withState` HOC for the Issues component to manage the `issueState`

* 确认[最后一节的代码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/0f261b13696046832ad65f1909266957d6275d6c)
* 安装实现了很多高阶组件的 [recompose](https://github.com/acdlite/recompose) 库
* 将 Issues 组件从类组件重构为无状态组件
* 用 `withState` 高阶组件来管理 Issues 组件的 `issueState`