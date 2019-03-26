# Apollo

> Finding the right solution for a given problem is not always simple, and web applications build with GraphQL are a good example of how changing times make for constantly evolving challenges. Moreover, evolving challenges create a scenario where the solutions must also evolve, so even the number of choices becomes a task. This article will decipher the pros and cons of one such solution: Apollo for GraphQL, with alternative solutions in case you decide against it.

​	在给定问题上找到正确的解决方案并不总是那么简单，使用 GrahpQL 构建 Web 应用程序就是一个很好的例子来说明如何应对挑战与时俱进。此外，进化的挑战也造成了解决方案也必须不断进化的，因此即使选择也成了一项任务。这篇文章将会解读一个这样的解决方案的优缺点：用于 GraphQL 的 Apollo ，以及你决定使用它时的替代解决方案

> GraphQL is only the query language that has a reference implementation in JavaScript, and Apollo builds its ecosystem on top to make GraphQL available for a wider audience. This includes the client-side as well as the server-side, because they provide a large ecosystem of libraries for both. The libraries provide an intermediate layer too: Apollo Engine, which is a GraphQL gateway. Essentially there's a reason Apollo is one of the most popular choices for using GraphQL in JavaScript applications.

​	GraphQL 只是在 JavaScript 中具有参考实现的查询语言，Apollo 在其之上构建了它的生态系统，以使 GraphQL 可供更广泛的用户使用。它包括客户端和服务端，因为他们为两端都提供了一个庞大的库生态系统。这些库也提供了一个中间层 — Apollo Engine，它是一个 GraphQL 的网关。本质上讲，Apollo 能成为在 JavaScript 应用中使用 GraphQL 的最流行选择之一也是有原因的。

## Apollo Advantages

> The following topics show you some of the advantages of using Apollo, to provide a well-rounded pro and con list. Feel free to contact me if you think anything is missing from either list.

​	以下话题将向您展示使用 Apollo 的一些优势，以提供一个全面的好处和坏处列表。如果你认为这些列表里有任何遗漏，请随时联系我。

### Apollo's Ecosystem

> While GraphQL is in its early stages, the Apollo ecosystem offers solutions for many of its challenges. Beyond that we can see how much the ecosystem is growing, because the company announces an update for Apollo or another library that can be used with Apollo's tech stack at every other technology conference. Apollo isn't just covering GraphQL, though; they also have effort invested in REST interfaces for backward compatibility to RESTful architectures. This even takes GraphQL beyond the network layer and remote data, offering a state management solution for local data, too.

​	虽然 GraphQL 还处在早期阶段，但 Apollo 生态圈为其他许多挑战提供了解决方案。除此之外，我们可以看到生态圈增长了多少。因为该公司宣布了为 Apollo 和其他库的更新，可以使 Apollo 的技术栈在其他所有技术会议上使用。然而，Apollo 不只是涵盖了 GraphQL；他们还花费精力投入 REST 接口，以便向后兼容 RESTful 架构。这甚至使 GraphQL 超越了网络层和远程数据，也为本地数据提供了状态管理解决方案。

### The Company and Community behind Apollo

> The company behind Apollo is pouring lots of resources into its success. They are also active in open source, offering in-depth articles about their products, supported by an established presence at the conferences. In general, the GraphQL ecosystem [seems to be in good shape for the future](https://techcrunch.com/2018/05/15/prisma). The community behind GraphQL is growing, as more developers adopt it and use Apollo for client and server-side JavaScript applications.

​	Apollo 背后的公司正在为其成功倾注大量资源。他们也积极参与开源，为其产品提供更深入的文章，并得到了会议的支持。总的来说，GraphQL 生态系统[似乎未来状况良好](https://techcrunch.com/2018/05/15/prisma)。随着越来越多的开发人员的接受和使用，并将 Apollo 用于客户端和服务端 JavaScript 应用程序，GraphQL 背后的社区正在成长。

### Who is using Apollo?

> Tech-savvy companies are taking advantage of Apollo already. Many were familiar with the popular Meteor framework before, but new and extremely popular companies like Airbnb and Twitch are using it. These are just a few of their stories:

​	技术精通的公司已经在充分利用 Apollo 了。许多人之前熟悉流行的 Meteor 框架，但是像 Airbnb 和 Twitch 这样的新兴和极受欢迎的公司已经在使用它了。以下这些事部分用例：

* Airbnb [[1]](https://medium.com/airbnb-engineering/reconciling-graphql-and-thrift-at-airbnb-a97e8d290712) [[2]](https://youtu.be/oBOSJFkrNqc)
* [Twitch](https://about.sourcegraph.com/graphql/twitch-our-graphql-transformation)
* [The New York Times](https://open.nytimes.com/the-new-york-times-now-on-apollo-b9a78a5038c)
* [KLM](https://youtu.be/T2njjXHdKqw)
* [Medium](https://www.infoq.com/news/2018/05/medium-reactjs-graphql-migration)

### Apollo's Documentation

> While Apollo continues to evolve, the team and community behind it keeps the documentation up to date, and they have plenty of insight about how to build applications. In fact, they cover so many areas it can be overwhelming for beginners.

​	虽然 Apollo 持续在发展，但它背后的团队和社区仍然使文档保持最新，他们非常洞悉如何去构建应用程序。事实上，他们已经覆盖了很多领域，对于初学者可能是压倒性的。

### Apollo Libraries

> Apollo offers plenty of libraries for implementing an effective GraphQL tech stack for JavaScript applications, and their libraries are open-sourced to be more manageable. For instance, [Apollo Link](https://www.apollographql.com/docs/link/) provides an API for chaining different features into a GraphQL control flow. This makes it possible for automatic network retries or RESTful API endpoints instead of a GraphQL endpoints (the endpoints can be used together, too).

​	Apollo 提供了大量的库，用于为 JavaScript 应用程序实现更高效的 GraphQL 技术栈，并且他们的库都是开源的，以便更易于管理。例如：[Apollo Link](https://www.apollographql.com/docs/link/) 提供了一个 API 用于将不同功能链接到 GraphQL 的控制流中。这使得自动的网络重试或 RESTful API endpoints 替代 GraphQL endpoints 成为可能（endpoints 也可以一起使用）。

> Apollo is also offering exchangeable libraries which can be seen in the Apollo Client Cache. The Apollo Client itself is not biased toward its cache, where the data is stored, as any cache advertised by Apollo or its community works. There are already caches available that can be used to setup a Apollo Client instance.

​	Apollo 还提供了可交换的库，可以在 Apollo 客户端缓存中看到。Apollo 客户端本身是不会偏向其存储数据的缓存的，就像 Apollo 或者其社区宣传的任何缓存一样。已有可用缓存可以用于设置 Apollo 客户端实例。

### Apollo's Features

> Apollo comes with built-in features to pull all the complexity out of applications and handle the intersection between client and server applications. For instance, Apollo Client caches requests, which are not made twice when the result is already in the cache. The function provides a performance boost for applications, saving valuable network traffic. Also, Apollo Client normalizes data, so nested data from a GraphQL query is stored in a normalized data structure in the Apollo Client Cache. Data can be read from the Apollo Client Cache by an identifier, without looking up an "article" entity in an "author" entity. Beyond caching and normalization, Apollo Client comes with many more features like error management, support for pagination and optimistic UI, prefetching of data, and connection of the data layer (Apollo Client) to the view layer (e.g. React).

​	Apollo 具有内置功能，可以将复杂性从应用程序中解脱出来，并且处理客户端和服务端应用程序之间的交叉。例如： Apollo Client 缓存请求，从而使当缓存中已经存在结果时，请求不会被发送两次。该功能可为应用程序提供性能提升，节省宝贵的网络流量。并且, Apollo 客户端会规范化数据，从来使 GraphQL 查询中嵌套的数据被存储在 Apollo Client Cache中的规范化数据结构中。数据可以通过标识符从 Apollo Client Cache中被读取，而不需要在"作者"实体中查找"文章"实体。除了缓存和规范化， Apollo Client 还有很多新功能，比如： 错误管理， 支持分页和乐观UI， 预获取数据，将数据层（Apollo Client）连接到视图层（如：React）。

### Interoperability with other Frameworks

> One of Apollo's libraries makes it possible to connect Apollo Client to React. Just like libraries like Redux and MobX, the React-Apollo library has higher-order and render prop components to connect both worlds. However, there are other libraries out there that bridge not only Apollo Client to React, but also Apollo to Angular or Apollo to Vue. That's what makes Apollo Client view layer agnostic, which is great for the growing JavaScript ecosystem.

​	Apollo 中有一个库可以使 Apollo Client 连接 React。 就像 Redux 和 MobX 这样的库一样。React-Apollo 有高阶组件和 render prop 组件来连接彼此。但是，还有其他的库不仅可以连接 Apollo 客户端和 React，还可以连接 Apollo 到 Angular 或者 Vue。这就是使 Apollo 客户端视图层没有限制的原因，这对 JavaScript 生态圈的增长非常有用。

> Apollo is also library agnostic on the server-side, and it offers several solutions to connect with Node.js libraries. Apollo Server for Express.js is one of the most popular choices among developers and companies, and there are other solutions for Koa and Hapi on Node.js for Apollo Server as well.

​	Apollo 在服务端也是与库无关的，它提供了多种解决方案去连接 Node.js 的库。适用于 Express.js 的 Apollo Server是开发人员和公司中最受欢迎的选择之一，而且还有其他针对 Apollo Server 的解决方案，如基于 Node.js 的 Koa 和 Hapi。

### Modern Data Handling with Apollo

> Remember back when we had to trigger data fetching in a component's lifecycle methods imperatively? Apollo Client solves this, because its data queries are declarative. React often employs a higher-order component or render prop to trigger a query automatically when a component renders. The GraphQL mutations are triggered imperatively, but that's only because a higher-order component or render prop grants access to the function which executes the mutation (e.g. on a button click). Essentially, Apollo embraces declarative programming over imperative programming.

​	还记得我们必须命令式的在组件的生命周期方法中触发数据获取么？Apollo Client 解决了这个问题，因为它的数据查询是声明式的。React 经常使用高阶组件或者 render prop 的方式在组件渲染时去自动触发查询。GraphQL mutations 是强制触发的，但是那只是因为高阶组件或者 render prop 授予了执行 mutation 的函数权限（比如：在点击按钮时）。从本质上讲，Apollo 信奉声明式编程多过命令式编程。

### Modern State Management with GraphQL and Apollo

> With the rise of GraphQL in JavaScript applications, state management entered another state of confusion. Even though lots of pain points are eliminated using a GraphQL library like Apollo Client, since it takes care of state management for remote data, some developers are confused about where to put state management libraries like Redux or MobX now. However, it can be made simple using these libraries for local data only and leaving the remote data to Apollo. There is no longer a need to fetch data with asynchronous actions in Redux, so it becomes a predictable state container for all the remaining application state (e.g. local data/view data/UI data). In fact, the remaining application state may be simple enough to be managed by React's local state instead of Redux.

​	随着 JavaScript 应用程序中 GraphQL 的兴起， 状态管理进入了另一种混乱状态。即使使用像 Apollo Client 一样的 GraphQL 库消除了很多痛点，因为它负责远程数据的状态管理，一些开发者会对在哪里放置像 Redux 或者 MobX 这样的库的位置感到困惑。但是，只需要将这些库用于本地数据管理并将远端数据留给 Apollo 即可。不再需要在 Redux 中通过异步 actions 获取数据了，因此它成了一个预测所有剩余应用状态（如：本地数据／视图数据／UI 数据）的容器。事实上，剩余应用状态可能足够简单到只需要去使用 React 本地状态管理而不是 Redux。

> Meanwhile, Apollo has already released their own solution to manage local state--which is supposed to be managed by React's local state, Redux or MobX--by embracing GraphQL for everything. The Apollo Link State library lets us manage local data with GraphQL operations, except on the client-side in Apollo Client. It's Apollo saying: "You don't need any other state management library, we take care of your data." These are exciting times for developing JavaScript applications.

​	与此同时，Apollo 已经发布了它自己的解决方案，即通过 GraphQL 来处理所有事情来管理原本应该由 React，Redux 或者 MobX 来管理的本地状态。Apollo 链接状态库使我们可以通过 GraphQL 的操作来管理本地数据，但 Apollo Client 中的客户端除外。Apollo 说："你不在需要其他的状态管理库，我们会处理你的数据"。这些是开发JavaScript应用程序的激动人心的时刻。

### Convenient Development Experience

> Using Apollo for JavaScript applications is becoming easier every day. The community is pushing out tools for implementation. There are development tools available as browser extensions, third-party tools to perform GraphQL operations such as GraphiQL, and libraries to simplify developing Apollo applications. For instance, the Apollo Boost library provides an almost zero-configuration Apollo Client setup to get started with GraphQL for client-side applications. Apollo takes away all the boilerplate implementation that comes with the GraphQL reference implementation in JavaScript.

​	使用 Apollo 来写 JavaScript 应用程序变得越来越容易。社区正在推出实施工具。有很多开发工具可以用作浏览器插件，第三方工具来进行 GraphQL 的操作，比如： GraphiQL，以及用语简化开发 Apollo 应用程序的库。例如：Apollo Boost 库提供了一个几乎零配置的 Apollo Client 设置，以便客户端应用程序开始使用 GraphQL。Apollo 删除了JavaScript 中 GraphQL 参考实现附带的所有样板实现。

## Apollo Disadvantages

> The following topics show you some of the disadvantages of using Apollo, to provide a well-rounded pro and con list. Feel free to contact me if you think anything is missing from either list.

​	接下来的内容会像您展示一些使用 Apollo 的缺点，以提供全面的优劣势列表。如果您认为任何一个列表里缺少了任何内容，请随时与我联系。

### Bleeding Edge

> GraphQL is in its early stages. Apollo users and all early GraphQL adopters are working with brand new technology. The Apollo team is developing a rich ecosystem around GraphQL, providing the basics as well as advanced features like caching and monitoring. This comes with pitfalls, however, mainly because everything isn't set in stone. There are sporadic changes that can pose challenges when you are updating GraphQL-related libraries. In contrast, some libraries in GraphQL might be more conservative than the Apollo team, but the features usually aren't as powerful.

​	GraphQL 仍处在早期阶段。Apollo 的用户和所有早期 GraphQL 的采用者都在使用全新的技术。Apollo 团队正在围绕 GraphQL 开发一个丰富的生态系统，提供基础和像缓存以及监控一样的高级功能。这伴随着未知的困难，主要因为一切都不是一尘不变的。当您更新 GraphQL 相关库时，有些零星的改动可能会带来挑战。相比之下，GraphQL 中的库可能比 Apollo 团队更加保守，但是提供的这些功能也通常没那么强大。

> The ability for developers to continue learning is also hindered by fast-pace development. Tutorials for GraphQL and Apollo are sometimes outdated, and finding an answer may require external resources. The same is true for most new technology, though.

​	快速发展也阻碍了开发者继续学习的能力。GraphQL 和 Apollo 的教程有时已经过期，寻找答案可能需要外部资源。不过，大多数的新技术都是这样。

### Under Construction

> The Apollo team and community implements many new features in a rapid pace, but going so fast comes with a price. Searching for solutions often leads to GitHub, because there is little other information on the subject. While you may indeed find a GitHub issue for your problem, there is often no solution for it.

​	Apollo 团队和社区快速实现了很多新功能，但是速度如此之快也需要付出代价。搜索解决方案经常会导向 GitHub， 因为关于该主题的其他信息很少。虽然你确实可以找到关于你的问题的 GitHub issue，但是通常没有解决方案。

> Rapid development also comes with the price of neglecting obsolete earlier versions. In my experience, [people seemed confused when Apollo abandoned Redux](https://github.com/apollographql/apollo-client/issues/2593) as their internal state management solution. Apollo isn't opinionated about how Redux should be used side by side with it, but since it has been abandoned as internal state management solution, many people didn't know how to proceed when Apollo 2.0 was released. I think the team behind Apollo might be struggling to keep up with the fast-paced GraphQL ecosystem, and it's not always easy to heed all voices in open source development.

​	快速开发还伴随着忽视过早期版本的代价。据我的经验，[人们对于 Apollo 放弃 Redux 作为外部状态管理的解决方案感到困惑](https://github.com/apollographql/apollo-client/issues/2593)。Apollo 对于 Redux 如何与它并排使用并不武断，但是因为它的内部已经决定放弃 Redux 作为外部状态管理的解决方案，当 Apollo 2.0 发布时，许多人并不知道如何继续。我认为Apollo背后的团队可能正在努力跟上快节奏 GraphQL 生态系统，但是能够注意到开源开发中的所有声音并不是那么容易。

### It is Bold and Fashionable

> Apollo is bold, because it is moving beyond being a network layer ecosystem between client and server for GraphQL in JavaScript, but positioning itself as the data management solution of tomorrow. It connects client and backend applications with GraphQL, apollo-link-rest for RESTful APIs, and apollo-link-state for local state management. Some experts are skeptical about the "GraphQL everything" mentality, but time will tell if it corners that market.

​	Apollo 很大胆，因为它不仅是 JavaScript 中 GraphQL 的客户端和服务端间的网络层生态系统，还将自己定位为未来的数据管理解决方案。它将客户端和后端应用程序与 GraphQL，用于 RESTful APIs 的 apollo-link-rest， 和用于本地状态管理的 apollo-link-state 连接起来。一些专家对于 "GraphQL 所有"持怀疑态度，但是时间会证明它是否影响市场。

> Apollo is fashionable, because it keeps up with the latest trends. In React, the latest trend was render prop components. Because of this, and arguably the benefits of render prop components over higher-order components, the React Apollo library introduced [render prop components](https://www.robinwieruch.de/react-render-props-pattern/) next to higher-order components. It was a smart move to offer multiple solutions since both higher-order and render prop components come with their own sets of pros and cons. However, Apollo does advocate for render props over higher-order components, and it's not clear if this was hype-driven development or marketing or if they truly believe that this is the way of the future. Render props are relatively new in React, so it will take time for developers to realize they come with their own pitfalls (see: higher-order components). I have seen React applications become too verbose by using multiple render prop components in one React component, even though one render prop didn't depend on another render prop, rather than having those co-located to the React component by using higher-order components. After all, Apollo offers both solutions, render props and higher-order components, so the developer decides on a case by case basis for their applications. It's a good sign for users that the Apollo team is keeping up with the recent trends from other libraries, and not confining themselves to a bubble.

​	Apollo 很时尚，因为它能跟得上最新趋势。在 React 中，最新趋势是 render prop 组件。正应如此，可以说 render prop 组件的好处是多于高阶组件的，React Apollo 库紧挨着高阶组件介绍了 [render prop 组件](https://www.robinwieruch.de/react-render-props-pattern/)。提供多种解决方案是个非常明智之举，因为高阶和 render prop 组件都有各自的优缺点。但是，Apollo 确实提倡 render props 多于高阶组件，不清楚这是否是炒作驱动开发或营销或是他们真的相信这是未来之路。Render props 在 React 中相对较新，所以会开发人员需要花时间才能意识到他们自己带来的缺陷（参见：高阶组件）。我已经看见 React 应用程序因为在一个 React 组件中使用多个 render prop 组件而变得越来越冗长，尽管一个 render prop 不会依赖另一个 render prop，而不是通过使用高阶组件来协同 React Component。毕竟，Apollo 提供了两种解决方案， rende props 和高阶组件，所以开发者可以根据具体情况决定其应用程序。对于用户来说，这是一个好兆头，Apollo 团队正在跟上其他库的最新趋势，而不是局限于泡沫。

### Missing Competition

> Most of these concerns are about the newness of GraphQL, concerns that could be applied to virtually any other open source solution in the same field. One major concern, though, is the missing competition in the GraphQL in JavaScript domain. A couple of alternatives to Apollo are listed in the next section, but they are limited compared to the Apollo ecosystem. While it is possible to write your own library for GraphQL (e.g. [a simple GraphQL in React client](https://github.com/rwieruch/react-graphql-client)), not many developers have attempted it yet. Some problems solved by Apollo are not trivial, but I think competition would be a healthy push for GraphQL in JavaScript ecosystem. There is huge potential in GraphQL now, and open source developers would be wise to take advantage.

​	这些问题大都与 GraphQL 的新特性有关，这些问题可以应用于同一领域的几乎任何其他开源解决方案。但是，其中主要的一个问题是 GraphQL 在 JavaScript 领域中缺少竞争。下一节将会罗列几种 Apollo 的替代方案，但是与 Apollo 的生态系统相比，它们是很有限的。虽然可以为 GraphQL 写你自己的库（比如： React 客户端中一个简单的 GraphQL），但是并没有很多开发者尝试过它。Apollo 解决的问题并非微不足道，但是我认为竞争对于 JavaScript 生态圈中的 GraphQL 来说是一个健康的推动。现在 GraphQL 有巨大的潜力，开源开发者使用它非常的明智。

## Apollo Alternatives for JavaScript, React and Node.js

> Some disadvantages stem from using GraphQL as an alternative to a RESTful-driven architecture. There are some alternatives for Apollo Client and Apollo Server that can consume GraphQL APIs in JavaScript. The following list should grant insights about solutions in the JavaScript ecosystem, used for React on the client-side and Node.js on the server-side.

​	有一些缺点源于使用 GraphQL 作为 RESTful驱动架构的替代方案。在 JavaScript 中也有一些替代 Apollo Client 和 Apollo Server 去消费 GraphQL APIs 的方案。以下列表应该会提供有关 JavaScript 生态系统中解决方案的见解，用于React 在客户端和 Node.js 在服务端的场景。

### Apollo Client Alternatives for React

> When it comes to [Apollo Client](https://github.com/apollographql/apollo-client) for React, Angular, Vue, or similar applications, there are several alternatives to check out. Like Apollo, these come with their own advantages and disadvantages.

​	对于 React，Angular，Vue 或者类似应用程序的 [Apollo Client]((https://github.com/apollographql/apollo-client) )，有一些替代选择。像 Apollo 一样，他们都有各自的优缺点。

* > plain HTTP request: Even though sophisticated GraphQL libraries can be used to perform your GraphQL operations, GraphQL itself isn't opinionated about the network layer. So it is possible for you to use GraphQL with plain HTTP methods using only one endpoint with an opinionated payload structure for GraphQL queries and mutations.

* 普通的 HTTP 请求：尽管可以使用复杂的 GraphQL 库来操作 GraphQL，GraphQL 并不关注它自身的网络层。因此你可以将 GraphQL 和普通的 HTTP 方法结合使用，只使用一个 endpoint ，其中包括固定的针对 GraphQL 查询和改变的 payload 结构。

* > [Relay](https://github.com/facebook/relay): Relay is Facebook's library for consuming GraphQL on the client-side in React applications. It was among the first GraphQL client libraries before Apollo emerged.

* [Relay]((https://github.com/facebook/relay)): Relay 是 Facebook 的一个在 React 应用程序客户端中使用 GraphQL 的库。它是出现在 Apollo 之前的第一批 GraphQL 客户端库。

* > [urql](https://github.com/FormidableLabs/urql): urql is a GraphQL client library from Formidable Labs for consuming GraphQL in React applications. It was open-sourced as minimalistic alternative to the growing Apollo behemoth.

* [urql]((https://github.com/FormidableLabs/urql)): urql 是来自 Formidable Labs 的 GraphQL 客户端库，用于在 React 应用程序中使用 GraphQL。它是开源的，是强大的 Apollo 的简化替代品。

* > [graphql.js](https://github.com/f/graphql.js/): graphql.js shouldn't be mistaken for the GraphQL reference implementation. It's a simple GraphQL client for applications without powerful libraries such as Vue, React, or Angular.

* [graphql.js](https://github.com/f/graphql.js/): graphql.js 不应该被误解为 GraphQL 的参考实现。对于不使用像 Vue，React 或 Angular 这样的强大的库的应用程序，它就是个简单版的 GraphQL 客户端

* > [AWS Amplify - GraphQL Client](https://github.com/aws/aws-amplify): The AWS Amplify family offers libraries for cloud-enabled applications. One of the modules is a GraphQL client used for general GraphQL servers or AWS AppSync APIs.

* [AWS Amplify - GraphQL Client](https://github.com/aws/aws-amplify): AWS Amplify 系列为支持云的应用提供库。其中一个模块被用于常规 GraphQL 服务或 AWS AppSync APIs 的 GraphQL 客户端。

### Apollo Server Alternatives for Node.js

> When it comes to [Apollo Server](https://github.com/apollographql/apollo-server) for Node.js with Express, Koa, Hapi or something else, there are several alternatives you can check out. Obviously these come with their own advantages and disadvantages whereas these things are not covered here.

​	对于 Express，Koa，Hapi 或者其他 Node.js 库的  Apollo Server，还有一些替代方案可以查看。显然它们都有自己的优劣势，然而并没有在这里讨论。

* > [express-graphql](https://github.com/graphql/express-graphql): The library provides a lower-level API to connect GraphQL layers to Express middleware. It takes the pure GraphQL.js reference implementation for defining GraphQL schemas, where Apollo Server simplifies it for developers.

* [express-graphql](https://github.com/graphql/express-graphql): 这个库提供了一个低级 API 去连接 GraphQL 层到 Express 的中间件。它使用纯 GraphQL.js 参考实现来定义 GraphQL 的概要，而 Apollo Server 却为开发者简单化了它。

* > [graphql-yoga](https://github.com/prisma/graphql-yoga): A fully-featured GraphQL Server with focus on easy setup, performance & great developer experience. It builds on top of other GraphQL libraries to take away even more boilerplate code from you.

* [graphql-yoga](https://github.com/prisma/graphql-yoga): 一个完整功能的 GraphQL 服务并且关注轻松启动，性能和出色的开发者体验。它建立在其他的GraphQL 库之上去为我们减少样板代码。

| |

> There are many reasons to use Apollo and its striving ecosystem for JavaScript applications, when you want to use a GraphQL interface over a RESTful interface. Their libraries are framework agnostic, so they can be used with a wide variety of frameworks on the client-side like React, Angular, Vue, and server-side applications like Express, Koa, Hapi.

当你想用 GraphQL 接口多于 RESTful 接口时，有很多原因去使用 Apollo 及其为 JavaScript 应用程序开发的生态系统。它们的库与框架无关，因此可以在客户端使用各种框架，如：React， Angular， Vue 和服务端应用程序，如：Express，Koa，Hapi。