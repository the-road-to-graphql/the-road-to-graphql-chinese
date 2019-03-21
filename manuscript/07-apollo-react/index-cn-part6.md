> ## Implementing the Issues Feature: Server-Side Filter
## 实现 Issues 功能: 服务端过滤

> Before starting with the server-side filtering, let's recap the last exercise in case you had difficulties with it. Basically you can perform the refactoring in three steps. First, install recompose as package for your application on the command line:

开始学习服务端过滤前, 以防你遇到困难, 让我们先回顾最近一次的练习. 基本来说, 我们可以分3个步骤来开始我们的重构. 首先, 在命令行安装 recompose 作为你应用的依赖包.

{title="Command Line",lang="json"}
~~~~~~~~
npm install recompose --save
~~~~~~~~

> Second, import the `withState` higher-order component in the *src/Issue/IssueList/index.js* file and use it to wrap your exported Issues component, where the first argument is the property name in the local state, the second argument is the handler to change the property in the local state, and the third argument is the initial state for that property.

其次, 导入位于`src/Issue/IssueList/index.js`的高阶组件`withState`, 然后用它包裹你导出的Issues组件, 高阶组件第一个参数是当前state下的属性名字, 第二个参数是改变当前state的属性的handler方法名，第三个参数是属性的初始state。

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

最后，将 Issues 组件从类组件重构为函数式无状态组件。 现在，它可以访问 `props` 中的 `issueState` 和 `onChangeissueState()` 函数。记住  `onChangeIssueState` 的使用方式为函数而不再是类方法。

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

在之前的章节实现了更加方便的有状态组件。接下来，改进过滤从客户端到服务端。我们使用有定义的 `GraphQL` 查询和它的参数通过请求仅打开或关闭的`issues`来构造更加精确的查询。在文件 `src/Issue/IssueList/index.js` 中，通过指定 `issue state` 扩展查询属性。

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

接下来，你可以使用`issueState` 特性作为你的组件查询的属性。此外, 从`Query`组件的属性中移除客户端过滤逻辑。

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

你仅在查询打开或关闭的issues。你的查询变得更加精确，过滤操作也不再依赖客户端。

> ### Exercises:

### 练习：

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/df737276a4bc8d2d889d182937b77ba9e474e70c)
> * Implement the pagination feature for the Issue feature
>   * Add the pageInfo information to the query
>   * Add the additional cursor variable and argument to the query
>   * Add the FetchMore component to the IssueList component

* 确认[最近章节的源代码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/df737276a4bc8d2d889d182937b77ba9e474e70c)
* 为issue实现分页
  * 新增`pageInfo`信息到查询中
  * 新增额外的锚点属性和参数到查询中
  * 新增`FetchMore`组件到`IssueList`组件中

> ##  Apollo Client Prefetching in React
## Apollo 客户端 React 中预先获取
> This section is all about prefetching data, though the user doesn't need it immediately. It is another UX technique that can be deployed to the optimistic UI technique you used earlier. You will implement the prefetching data feature for the list of issues, but feel free to implement it for other data fetching later as your exercise.

本章节全部关于数据预先获取，虽然用户并不会立刻需要。它是另一种 UX 技术，可以部署在你之前使用过的乐观 UI 上。你将能在issues列表中实现数据预先获取的特性，不过作为你的练习，稍后你可以随意实现其他的数据预先获取。

> When your application renders for the first time, there no issues fetched, so no issues are rendered. The user has to toggle the filter button to fetch open issues, and do it again to fetch closed issues. The third click will hide the list of issues again. The goal of this section is to prefetch the next bulk of issues when the user hovers the filter button. For instance, when the issues are still hidden and the user hovers the filter button, the issues with the open state are prefetched in the background. When the user clicks the button, there is no waiting time, because the issues with the open state are already there. The same scenario applies for the transition from open to closed issues. To prepare this behavior, split out the filter button as its own component in the *src/Issue/IssueList/index.js* file:

当你的应用首次渲染的时候，没有已经请求到的issues，因此不会渲染任何issues。用户必须切换筛选按钮来请求打开的issues，然后再次切换筛选按钮来请求关闭的issues。第三次点击将会再次影藏列表的issues。本章节的目标是：当用户悬浮在筛选按钮上时预先获取近期大量的issues。例如，当issues仍然是在隐藏时，用户悬浮在筛选按钮上，打开状态的issues将会在后台预先获取。当用户点击筛选按钮，由于打开状态的issues已经获取到，因此不会再有等待时间。同样的情况适用于issues打开到关闭的过渡。为了给这种行为做准备，在文件`src/Issue/IssueList/index.js`中，分离筛选按钮作为自己单独的组件。

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

现在，当大部分关于数据预先获取的逻辑已经实现，更容易将注意力集中在issueFilter 组件。像之前一样，预先获取应该发生在用户悬浮在按钮上。这里应该作为它的props，当用户悬浮在按钮上的时候执行回调函数。这是按钮元素的prop属性。我们在这里处理HTML元素。

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

函数`prefetchIssue()`必须通过Issues组件中的Query组件，执行同样的GraphQL查询，但这一次它是以命令式的方式完成的，而不是声明式的。与其使用Query组件，不如直接使用Apollo客户端实例来执行查询。记住，Apollo客户端实例隐藏在组件树中，因为你使用了React的 `Context` API 在组件树的顶层提供Apollo客户端实例。查询和突变组件可以访问Apollo客户端，即使您从未直接使用过它。但是，这次你使用它来查询预取的数据。使用  `React Apollo` 包中的 `ApolloConsumer` 组件在组件树中暴露Apollo客户端实例。你已经在某个地方使用了`ApolloProvider` 来提供客户端实例，现在可以使用 `ApolloConsumer` 来恢复它。在文件 `src/Issue/IssueList/index.js` 中，导入 `ApolloConsumer` 组件并在 `IssueFilter` 组件中使用它。它允许你通过其渲染 props 子函数访问Apollo客户端实例。

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

现在，您可以访问Apollo客户端实例来执行查询和突变，这将使您能够强制查询Github的 ` GraphQL` API。执行issues预取所需的变量与查询组件中使用的变量相同。你需要将这些传递到`IssueFilter`组件，然后传递给函数 `prefetchIssues()`。

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

使用此信息执行预取数据查询。Apollp客户端实例为此暴露了一个`query()`方法。确保恢复下一个 `issueState`,因为预取打开的issues时，当前的  `issueState`  应为 `NONE`。

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

就是这样。一旦该按钮悬停，它将预取下一个 `issueState` 的issues。Apollo客户端确保新数据更新到缓存，就像对 Query组件所做的那样。中间不应该有任何可见的加载指示器，除非网络请求花费了太长时间，并且你在悬停之后单击了按钮。你可以验证请求是否在浏览器的开发者工具的 network 选项卡中发生。在最后，你已经了解了使用Apollo客户端可以轻松实现的两个UX改进：乐观UI和预取数据。

> ### Exercises:
### 练习：
> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/87dc6eee7948dad6e1eb4c15078063337eff94db)
> * Read more about [Apollo Prefetching and Query Splitting in React](https://www.apollographql.com/docs/react/recipes/performance.html)
> * Invest 3 minutes of your time and take the [quiz](https://www.surveymonkey.com/r/5PLMBR3)

* 确认 [最近章节的代码](https://github.com/the-road-to-graphql/react-graphql-github-apollo/tree/87dc6eee7948dad6e1eb4c15078063337eff94db)

* 阅读更多关于[Apollo 在React中预取和查询分离](https://www.apollographql.com/docs/react/recipes/performance.html)

* 花三分钟的时间来做一个小[测验](https://www.surveymonkey.com/r/5PLMBR3)