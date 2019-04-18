# GraphQL

> When it comes to network requests between client and server applications, [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) is one of the most popular choices to connect both worlds. In REST, everything evolves around the idea of having resources that are accessible by URLs. You can read a resource with a HTTP GET request, create a resource with a HTTP POST request, and update or delete it with HTTP PUT and DELETE requests. These are called CRUD (Create, Read, Update, Delete) operations. Resources can be anything from authors, articles, or users. The format for transferring data is not opinionated with REST, but most often people will use JSON for it. In the end, REST enables applications to communicate with each other by using plain HTTP with URLs and HTTP methods.

对于客户端和服务器应用间的网络请求来说，[REST](https://en.wikipedia.org/wiki/Representational_state_transfer) 是用来连接两端最流行的选择。在 REST 的世界中，一切都围绕着“可被 URL 访问的资源”来发展。你可以通过 HTTP GET 请求来读取资源，也可以通过 HTTP POST 请求新建资源，亦或者通过 HTTP PUT 和 DELETE 请求更新或删除资源。这些操作被称为增删查改（CRUD  Create，Read，Update，Delete）。资源可以是来自 authors，articles 或者 users 的任何东西。REST 并不限制用于传输数据的格式，但多数情况下人们会用 JSON。总之，REST 使应用之间能通过用 URL 和 HTTP 方法进行交流。

{title="Code Playground",lang="json"}
~~~~~~~~
// a RESTful request with HTTP GET
https://api.domain.com/authors/7

// the response in JSON
{
  "id": "7",
  "name": "Robin Wieruch",
  "avatarUrl": "https://domain.com/authors/7",
  "firstName": "Robin",
  "lastName": "Wieruch"
}
~~~~~~~~

> Though REST was the status quo for a long time, a Facebook technology called GraphQL has recently emerged as a potential successor. The following sections introduce GraphQL's advantages and disadvantages, as well as possible alternatives for developers who need options.

尽管 REST 已经成为现状很长一段时间了， 但 Facebook 的一项名为 GraphQL 的技术最近成为了一种潜在的继承者。接下来的部分将介绍 GraphQL 的优劣势，以便为需要的开发人员提供可能的替代方案。

## What is GraphQL? 什么是 GraphQL？

> In short, GraphQL is an open source **query language** created by Facebook, a company that unsurprisingly remains at the pinnacle of web-based software development. Before GraphQL went open source in 2015, Facebook used it internally for their mobile applications since 2012, as an alternative to the common REST architecture. It allows requests for specific data, giving clients more control over what information is sent. This is more difficult with a RESTful architecture because the backend defines what data is available for each resource on each URL, while the frontend always has to request all the information in a resource, even if only a part of it is needed. This problem is called overfetching. In the worst case scenario, a client application has to read multiple resources through multiple network requests. This is overfetching, but also adds the need for waterfall network requests. A query language like GraphQL on the server-side and client-side lets the client decide which data it needs by making a single request to the server. Network usage was reduced dramatically for Facebook's mobile applications as a result, because GraphQL made it more efficient with data transfers.

简要来说，GraphQL 是一种由 Facebook 开源的**查询语言**，Facebook 毫无疑问是一家处于 web 软件开发顶尖的公司。早在2015年 GraphQL 开源之前，Facebook 从2012年开始就在内部的移动开发应用中使用它，并把它当做 REST 架构的替代方案。它允许请求声明想要的数据，这让客户端能更好的控制（服务器）发什么样的数据回来。这点对于 RESTful 架构来说就很难实现了，因为后端在每一个 URL 上都定义了固定数据的资源，所以即使前端只需要一部分数据，它也不得不请求一个资源上的所有数据。这个问题被称之为数据冗余（overfetching）。在最糟的情况下，一个客户端应用不得不通过多次请求去获取多个资源。这也是数据冗余，它同时还增加了对瀑布网络的请求数。如果将一个像 GraphQL 这样的查询语言应用在服务器端和客户端上的话，客户端发送一个请求就可以获取所有需要的数据了。最终，Facebook 移动端应用的网络流量显著减少，因为 GraphQL 使数据传输变得更加高效了。

> Facebook open-sourced the GraphQL specification and its reference implementation in JavaScript, and multiple major programming languages have implemented the specification since then. The ecosystem around GraphQL is growing horizontally by offering multiple programming languages, but also vertically, with libraries on top of GraphQL like Apollo and Relay.

Facebook 开源了 GraphQL 的规范以及它的在 JavaScript 上的实现范例，之后多个主流语言也根据这个规范实现了 GraphQL。GraphQL 的生态系统通过对多种语言的实现得到了水平发展，但同时，随着像 Apollo 和 Relay 这样的基于 GraphQL 的库的出现，它也得到了垂直发展。

> A GraphQL operation is either a query (read), mutation (write), or subscription (continuous read). Each of those operations is only a string that needs to be constructed according to the GraphQL query language specification. Fortunately, GraphQL is evolving all the time, so there may be other operations in the future.

一个 GraphQL 操作可以是查询（query），变更（mutation）或者订阅（subscription）。其实每一个操作只是一个符合 GraphQL 查询语言规范的字符串而已。幸运的是，GraphQL 还在不停的发展，所以在未来可能会出现更多的操作。

> Once this GraphQL operation reaches the backend application, it can be interpreted against the entire GraphQL schema there, and resolved with data for the frontend application. GraphQL is not opinionated about the network layer, which is often HTTP, nor about the payload format, which is usually  JSON. It isn't opinionated about the application architecture at all. It is only a query language.

一旦 GraphQL 的操作到达后端，GraphQL schema 就会针对它进行匹配，然后为前端解析数据。GraphQL 不指定特殊的网络协议，但通常会使用 HTTP，它也不指定特殊的负载（payload），但通常会使用 JSON。它也完全与应用的架构无关。总之它就只是个查询语言。

{title="Code Playground",lang="json"}
~~~~~~~~
// a GraphQL query
author(id: "7") {
  id
  name
  avatarUrl
  articles(limit: 2) {
    name
    urlSlug
  }
}

// a GraphQL query result
{
  "data": {
    "author": {
      "id": "7",
      "name": "Robin Wieruch",
      "avatarUrl": "https://domain.com/authors/7",
      "articles": [
        {
          "name": "The Road to learn React",
          "urlSlug": "the-road-to-learn-react"
        },
        {
          "name": "React Testing Tutorial",
          "urlSlug": "react-testing-tutorial"
        }
      ]
    }
  }
}
~~~~~~~~

> One query already requests multiple resources (author, article), called fields in GraphQL, and only a particular set of nested fields for these fields (name, urlSlug for article), even though the entity itself offers more data in its GraphQL schema (e.g. description, releaseData for article). A RESTful architecture needs at least two waterfall requests to retrieve the author entity and its articles, but the GraphQL query made it happen in one. In addition, the query only selected the necessary fields instead of the whole entity.

在 GraphQL 的调用中，一次查询就已经请求了多个资源（author，article），并且仅仅查询了 article 的特定的字段（name，urlSlug），即使在它的 schema 中提供了更多的数据。RESTful 架构的话至少需要两次瀑布请求分别去获取 author 和 articles，但是 GraphQL 实现了一步到位。此外，这个请求仅仅选择了必要的字段而非整个实体（entity）。

> That's GraphQL in a nutshell. The server application offers a GraphQL schema, where it defines all available data with its hierarchy and types, and a client application only queries the required data.

下面是对 GraphQL 的概括。服务器应用提供一个 GraphQL schema，这个 schema 定义了所有可以利用的数据的结构和类型，而客户端应用只需请求必要的数据。

## GraphQL Advantages GraphQL 的优势

> The following list shows the major advantages of using GraphQL in an application.

下面将列出在应用中使用 GraphQL 的主要优势。

### Declarative Data Fetching 声明式数据获取

> As you've seen, GraphQL embraces declarative data fetching with its queries. The client selects data along with its entities with fields across relationships in one query request. GraphQL decides which fields are needed for its UI, and it almost acts as UI-driven data fetching, like how Airbnb uses it. A search page at Airbnb usually has a search result for homes, experiences, and other domain-specific things. To retrieve all data in one request, a GraphQL query that selects only the part of the data for the UI makes perfect sense. It offers a great separation of concerns: a client knows about the data requirements; the server knows about the data structure and how to resolve the data from a data source (e.g. database, microservice, third-party API).

正如前文所示，GraphQL 可以在查询中声明想要获取的数据，并且客户端在一次查询中可以跨资源的选择实体和字段。举个例子，Airbnb 是这样使用 GraphQL 的，由 GraphQL 决定为 UI 获取哪些字段，这样的方式就像 UI 去驱动数据的获取 （UI-driven data fetching）。在 Airbnb 上，一个搜索页面的搜索结果通常包括房屋、入住体验以及其他特定领域的信息。要想通过一次请求就获取所有数据，只为 UI 定制化数据的 GraphQL 查询是完美的选择。这样的方式也让前后端很好的各司其职：客户端知道自己对数据的需求；服务器知道数据结构并且知道如何从数据源（如：数据库，微服务，第三方 API）去解析数据。

### No Overfetching with GraphQL         没有数据冗余

> There is no overfetching in GraphQL. A mobile client usually overfetches data when there is an identical API as the web client with a RESTful API. With GraphQL, the mobile client can choose a different set of fields, so it can fetch only the information needed for what's onscreen.

对于 GraphQL 来说不存在数据冗余的情况。如果移动端使用为 web 端设计的 RESTful API，通常是会出现数据冗余的。但是用 GraphQL 的话，移动端就可以选择不同的字段组合，也就是说它可以只获取需要用来显示在屏幕上的信息。

### GraphQL for React, Angular, Node and Co.       独立性 

> GraphQL is not just exciting for React developers, though. While Facebook showcased GraphQL on a client-side application with React, it is decoupled from any frontend or backend solution. The reference implementation of GraphQL is written in JavaScript, so the usage of GraphQL in Angular, Vue, Express, Hapi, Koa and other JavaScript libraries on the client-side and server-side is possible, and that's just the JavaScript ecosystem. GraphQL does mimic REST's programming language-agnostic interface between two entities, such as client or server.

GraphQL 不仅仅是为了 React 开发者而存在的。虽然 Facebook 使用 React 客户端应用展示了 GraphQL，但它独立于任何前端或者后端的解决方案。由于 GraphQL 的范例是用 JavaScript 写的，所以像 Angular，Vue，Express，Hapi，Koa 等 JavaScript 的前后端库都可以使用 GraphQL，而且这仅仅只是 JavaScript 的生态系统。就在两个实体间（比如客户端和服务器）无关编程语言这一点上，GraphQL 确实模仿了 REST。

### Who is using GraphQL?      谁在使用 GraphQL

> Facebook is the driving company behind the GraphQL specification and reference implementation in JavaScript, but other well-known companies are also using it for their applications. They are invested in the GraphQL ecosystem due to the huge demand for modern applications. Beyond Facebook, GraphQL has also been used by these well-known companies:

虽然 GraphQL 规范以及它在 JavaScript 上实现的背后驱动者是 Facebook，但还有许多其它的著名公司也正在将它运用在各自的应用上。由于来自于现代应用的强烈需求，这些公司也在 GraphQL 上投入了人力物力。除了 Facebook 以外，以下这些著名公司都使用了 GraphQL：

* GitHub [[1]](https://githubengineering.com/the-github-graphql-api/) [[2]](https://youtu.be/lj41qhtkggU)
* Shopify [[1]](https://shopifyengineering.myshopify.com/blogs/engineering/solving-the-n-1-problem-for-graphql-through-batching) [[2]](https://youtu.be/2It9NofBWYg)
* [Twitter](https://www.youtube.com/watch?v=Baw05hrOUNM)
* [Coursera](https://youtu.be/F329W0PR6ds)
* [Yelp](https://youtu.be/bqcRQYTNCOA)
* [Wordpress](https://youtu.be/v3xY-rCsUYM)
* [The New York Times](https://youtu.be/W-u-vZUSnIk)
* [Samsara](https://youtu.be/g-asVW9JFPw)
* > and [more](https://graphql.org/users/) ...

* [查看更多](https://graphql.org/users/)

> When GraphQL was developed and open sourced by Facebook, other companies ran into similar issues for their mobile applications. That's how Netflix came up with [Falcor](https://github.com/Netflix/falcor), an alternative to GraphQL. It shows again that modern applications demanded solutions like GraphQL and Falcor.

当 GraphQL 被 Facebook 开源时，其它公司在移动应用上也面临着同样的问题。此时 Netflix 提出了 [Falcor](https://github.com/Netflix/falcor)，一种 GraphQL 的备选方案。这再次证明了现代应用需要像 GraphQL 和 Falcor 这样的解决方案。

### Single Source of Truth   单一数据源

> The GraphQL schema is the single source of truth in GraphQL applications. It provides a central location, where all available data is described. The GraphQL schema is usually defined on server-side, but clients can read (query) and write (mutation) data based on the schema. Essentially, the server-side application offers all information about what is available on its side, and the client-side application asks for part of it by performing GraphQL queries, or alters part of it using GraphQL mutations.

GraphQL schema 是 GraphQL 应用中的单一数据源。它提供一个中心位置，在这个中心位置上描述了所有可用的数据。GraphQL schema 通常在服务端定义，但是客户端可用基于 schema 对数据进行读（查询）写（变更）操作。总的来说，服务端应用提供它所有可以使用的数据，然后客户端应用通过 GraphQL 查询请求部分数据，或者通过 GraphQL 变更修改部分数据。

### GraphQL embraces modern Trends    GraphQL 符合现在发展趋势

> GraphQL embraces modern trends on how applications are built. You may only have one backend application, but multiple clients on the web, phones, and smartwatches depending on its data. GraphQL can be used to connect both worlds, but also to fulfil the requirements of each client application--network usage requirements, nested relationships of data, fetching only the required data--without a dedicated API for each client. On the server side, there might be one backend, but also a group of microservices that offer their specific functionalities. This defines the perfect use for GraphQL schema stitching, which lets you aggregate all functionalities into one GraphQL schema.

GraphQL 符合现代应用开发的趋势。你可能只有一个后端应用，但有多个与之对应的客户端，例如 web 上，手机上以及智能手表上，这些客户端都依赖于这一个后端应用的数据。GraphQL 不仅可以用来连接前后端，也能满足每一个客户端应用的需求：网络流量的需求，有嵌套关系的数据，只请求需要的数据；并且不需要为每一个客户端定制化 API。在服务器端，它可能是一个完整的后端，但也可能是一组提供不同功能的微服务。在微服务的情况下可以将 GraphQL schema 的拼接特性运用到极致——你可以将所有的功能聚合到一个 GraphQL schema 中。

### GraphQL Schema Stitching     拼接性

> Schema stitching makes it possible to create one schema out of multiple schemas. Think about a microservices architecture for your backend where each microservice handles the business logic and data for a specific domain. In this case, each microservice can define its own GraphQL schema, after which you'd use schema stitching to weave them into one that is accessed by the client. Each microservice can have its own GraphQL endpoint, where one GraphQL API gateway consolidates all schemas into one global schema.

Schema 拼接（Schema stitching）特性让多个 schema 组合成一个 schema 成为可能。设想一下，你的后端是一个微服务架构，而每一个微服务都处理特定领域的数据和逻辑。在这种情况下，每一个微服务都会定义他自己的 GraphQL schema，之后你可以使用 schema 拼接将它们组合成一个客户端可以访问的 schema。每个微服务都可以拥有自己的 GraphQL 端点，然后由一个 GraphQL API 网关将所有的 schema 组合起来形成一个全局的 schema。

### GraphQL Introspection 内省系统

> A GraphQL introspection makes it possible to retrieve the GraphQL schema from a GraphQL API. Since the schema has all the information about data available through the GraphQL API, it is perfect for autogenerating API documentation. It can also be used to mock the GraphQL schema client-side, for testing or retrieving schemas from multiple microservices during schema stitching.

通过 GraphQL 的内省系统可以从 GraphQL API 获取 GraphQL schema。因为 schema 包含了所有 GraphQL API 上可以使用的数据，所以它可以很好地自动化生成 API 文档。在测试的时候或者在从多个微服务获取 schema 进行拼接的时候，它也可以用来模拟 GraphQL 客户端。

### Strongly Typed GraphQL    GraphQL 是强类型的

> GraphQL is a strongly typed query language because it is written in the expressive GraphQL Schema Definition Language (SDL). Being strongly-typed makes GraphQL less error prone, can be validated during compile-time and can be used for supportive IDE/editor integrations such as auto-completion and validation.

GraphQL 是强类型的查询语言因为它是由富有表现力的 GraphQL 模式定义语言（Schema Definition Language）来写的。强类型使 GraphQL 更少出错，在编译时能被校验，并且可以使它支持与编译器和 IDE 的集成，例如自动补全和校验。

### GraphQL Versioning        版本控制

> In GraphQL there are no API versions as there used to be in REST. In REST it is normal to offer multiple versions of an API (e.g. api.domain.com/v1/, api.domain.com/v2/), because the resources or the structure of the resources may change over time. In GraphQL it is possible to deprecate the API on a field level. Thus a client receives a deprecation warning when querying a deprecated field. After a while, the deprecated field may be removed from the schema when not many clients are using it anymore. This makes it possible to evolve a GraphQL API over time without the need for versioning.

在 GraphQL 中并没有像 REST 使用的 API 版本控制规则。在 REST 中通常提供多个 API 的版本（比如 api.domain.com/v1/， api.domain.com/v2/）进行版本控制，因为资源或者资源的结构随着时间在不断的改变。但对于 GraphQL 来说，它可以声明废弃一个字段。之后，客户端在查询这个废弃的字段时将收到一个废弃警告。一段时间过后当大部分客户端不再使用这个废弃的字段时，它可能会从 schema 中被移除。这使得 GraphQL API 在不需要多个版本共存的情况下就可以随着时间不断的演进。



### A growing GraphQL Ecosystem          GraphQL 的生态系统正在不断成长

> The GraphQL ecosystem is growing. There are not only integrations for the strongly typed nature of GraphQL for editors and IDEs, but also standalone applications for GraphQL itself. What you may remember as [Postman](https://www.getpostman.com) for REST APIs is now [GraphiQL](https://github.com/graphql/graphiql) or [GraphQL Playground](https://github.com/prismagraphql/graphql-playground) for GraphQL APIs. There are various libraries like [Gatsby.js](https://www.gatsbyjs.org/), a static website generator for React using GraphQL. With Gatsby.js, you can build a blog engine by providing your blog content at build-time with a GraphQL API, and you have headless content management systems (CMS) (e.g. [GraphCMS](https://graphcms.com)) for providing (blog) content with a GraphQL API. More than just technical aspects are evolving; there are conferences, meetups, and communities forming for GraphQL, as well as newsletters and podcasts.

GraphQL 的生态系统在不断成长。现在不仅仅有为 GraphQL 强类型特性在编辑器 和 IDE 上集成的插件，还有针对 GraphQL 本身独立的应用。可能你知道 REST API 有 [Postman](https://www.getpostman.com)，类似的，现在 GraphQL API 也有  [GraphiQL](https://github.com/graphql/graphiql) 和 [GraphQL Playground](https://github.com/prismagraphql/graphql-playground)。还有像 [Gatsby.js](https://www.gatsbyjs.org/) 这样的库，它是为了能在 React 上使用 GraphQL 而产生的静态网站生成器。使用 Gatsby.js，你就能够在构建时用 GraphQL API 来创建一个提供博客内容的博客引擎，并且你将拥有一个无头字段的内容管理系统（CMS content management systems），这个管理系统通过 GraphQL API 来提供（博客）内容。GraphQL 不仅在技术方面不断发展壮大，许多关于 GraphQL 的会议，聚会，社区以及新闻报道和播客也在不断涌现。

### Should I go all in GraphQL?        我该在所有技术栈上使用 GraphQL 吗？

> Adopting GraphQL for an existing tech stack is not an "all-in" process. Migrating from a monolithic backend application to a microservice architecture is the perfect time to offer a GraphQL API for new microservices. With multiple microservices, teams can introduce a GraphQL gateway with schema stitching to consolidate a global schema. The API gateway is also used for the monolithic REST application. That's how APIs are bundled into one gateway and migrated to GraphQL.

在现有的技术栈上采用 GraphQL 不是一个"一刀切"的过程。从单一的后端应用迁移到微服务架构时，是在新的微服务上使用 GraphQL API 的最佳时机。在多个微服务情况下，团队可以引进 GraphQL 网关，通过 schema 拼接来合并一个全局的 schema。同时 API 网关也用于之前的 REST 应用，而这就是如何通过将 API 捆绑在网关上来使其迁移到 GraphQL 上的过程。

## GraphQL Disadvantages       GraphQL 的劣势

> The following topics show you some of the disadvantages of using GraphQL.

下面的内容将介绍一些 GraphQL 的劣势。

### GraphQL Query Complexity             查询的复杂性

> People often mistake GraphQL as a replacement for server-side databases, but it's just a query language. Once a query needs to be resolved with data on the server, a GraphQL agnostic implementation usually performs database access. GraphQL isn't opinionated about that. Also, GraphQL doesn't take away performance bottlenecks when you have to access multiple fields (authors, articles, comments) in one query. Whether the request was made in a RESTful architecture or GraphQL, the varied resources and fields still have to be retrieved from a data source. As a result, problems arise when a client requests too many nested fields at once. Frontend developers are not always aware of the work a server-side application has to perform to retrieve data, so there must be a mechanism like maximum query depths, query complexity weighting, avoiding recursion, or persistent queries for stopping inefficient requests from the other side.

人们常常误以为 GraphQL 是服务器端数据库的替代品，但实际上它就只是一个查询语言而已。在服务器上，一次需要解析的数据查询，通常由与 GraphQL 无关的实现去执行数据库的访问。同时，当你需要在一次查询中访问多个字段时（authors，articles，comments），GraphQL 也不能解决性能瓶颈的问题。不论是一个 RESTful 架构的请求还是 GraphQL 的，各种资源和字段仍然需要从数据源获取。这样，当客户端一次请求过多的嵌套字段时，问题就会出现。前端开发者不是总能意识到服务器端需要执行获取数据的工作量，所以后端通常会有像最大查询深度（maximum query depths），查询复杂度加权（query complexity weighting），递归避免（avoiding recursion），或者持久性查询（persistent queries）这样的机制去阻止来自其它端的低效的查询请求。

### GraphQL Rate Limiting             速率限制（Rate Limiting）

> Another problem is rate limiting. Whereas in REST it is simpler to say "we allow only so many resource requests in one day", it becomes difficult to make such a statement for individual GraphQL operations, because it can be everything between a cheap or expensive operation. That's where companies with [public GraphQL APIs come up with their specific rate limiting calculations](https://developer.github.com/v4/guides/resource-limitations/) which often boil down to the previously mentioned maximum query depths and query complexity weighting.

另一个问题就是速率限制。对于 REST，它能很轻易的说出"我们一天只允许这么多的资源访问"这样的话，相反，对于单独的 GraphQL 操作就很难去做这样的声明，因为一个 GraphQL 操作即可以是一个昂贵的操作也可以是一个廉价的操作。这就是[拥有公共 GraphQL API 的公司会用特别的计算方法来限制速率](https://developer.github.com/v4/guides/resource-limitations/)的原因，这些方法通常就是前文提到的最大查询深度和查询复杂度加权。

### GraphQL Caching            缓存

> Implementing a simplified cache with GraphQL is more complex than implementing it in REST. In REST, resources are accessed with URLs, so you can cache on a resource level because you have the resource URL as identifier. In GraphQL, this becomes complex because each query can be different, even though it operates on the same entity. You may only request just the name of an author in one query, but want to know the email address in the next. That's where you need a more fine-grained cache at field level, which can be difficult to implement. However, most of the libraries built on top of GraphQL offer caching mechanisms out of the box.

在 GraphQL 中实现一个简单的缓存比在 REST 中实现要复杂许多。因为在 REST 中资源用 URL 进行访问，所以你可以用 URL 作为标识符在资源级别上缓存数据。而在 GraphQL 中，这就变得复杂了，因为就算是操作同一个实体每一次查询都可能是不同的。你可能在一次查询中只请求了作者的名字，但是在下次查询中想要知道邮件地址，即使他们都在查询同一个实体。这种情况下你就需要一个更加精细的在字段级别的缓存了，而这实现起来可能会比较困难。然而，大多数建立在 GraphQL 上的库都已经实现了开箱即用的缓存机制。

## Why not REST?  为什么不用 REST

> GraphQL is an alternative to the commonly used RESTful architecture that connects client and server applications. Since REST comes with a URL for each resource, it often leads to inefficient waterfall requests. For instance, imagine you want to fetch an author entity identified by an id, and then you fetch all the articles by this author using the author's id. In GraphQL, this is a single request, which is more efficient. If you only want to fetch the author's articles without the whole author entity, GraphQL lets you to select only the parts you need. In REST, you would overfetch the entire author entity.

GraphQL 通常可以作为 RESTful 架构的代替方案去连接客户端和服务器。因为 REST 用了 URL 来标识资源，所以它常常会导致低效的瀑布请求。举个例子，假设你想要获取带有 id 的 author 实例，然后你想通过 author 的 id 来获取它所有的 article。在 GraphQL 中，只需一个请求就可以完成所有操作，这比 RESTful 高效多了。如果你只想要获取 author 的 article 而不是整个 author 实体，GraphQL 也可以让你只选择你想要的部分。但用 REST 的话，你就会拿到整个 author 实体造成数据冗余的现象。

> Today, client applications are not made for RESTful server applications. The search result on Airbnb's platform shows homes, experiences, and other related things. Homes and experiences would already be their own RESTful resources, so in REST you would have to execute multiple network requests. Using a GraphQL API instead, you can request all entities in one GraphQL query, which can request entities side by side (e.g. homes and experiences) or in nested relationships (e.g. articles of authors). GraphQL shifts the perspective to the client, which decides on the data it needs rather than the server. This is the primary reason GraphQL was invented in the first place, because a Facebook's mobile client required different data than their web client.

如今，客户端应用不是为 RESTful 服务器应用而生的。在 Airbnb 平台上的搜索结果要显示房间，入住体验以及其他相关的信息。而房间和入住体验都来自于不同的资源，所以在 REST 中你不得不执行多次网络请求。而用 GraphQL 的话，你可以在一次查询中请求所有的实体，这些实体可以是并列的（例如房间和入住体验），也可是嵌套的（如 author 的 article）。GraphQL 将数据的选择权交给了客户端而不是服务器。这就是 GraphQL 被发明出来的首要原因——Facebook 的移动端应用与 web 端所需求的数据不同。

> There are still cases where REST is a valuable approach for connecting client and server applications, though. Applications are often resource-driven and don't need a flexible query language like GraphQL. However, I recommend you to give GraphQL a shot when developing your next client server architecture to see if it fits your needs.

当然，在某些案例中，使用 REST 去连接客户端与服务器应用的价值仍然存在。这些应用通常是资源驱动的，并且不需要像 GraphQL 这样一个灵活的查询语言作为连接方式。即使这样，我建议你在你的下一个客户端—服务器架构中尝试一下 GraphQL，然后观察它是否满足你的需求。

## GraphQL Alternatives            GraphQL 的替代方案

> REST is the most popular alternative for GraphQL, as it is still the most common architecture for connecting client and server applications. It became more popular than networking technologies like [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call) and [SOAP](https://simple.wikipedia.org/wiki/SOAP_(protocol)) because it used the native features of HTTP, where other protocols like SOAP tried to build their own solution on top of it.

对于 GraphQL 来说，REST 是最热门的替代方案，而且它仍然是当下连接客户端服务器应用最流行的架构。REST 之所以能够变得比 [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call) 和 [SOAP](https://simple.wikipedia.org/wiki/SOAP_(protocol)) 这样的网络技术更加流行是因为它用了 HTTP 原生的特性。而像 SOAP 这样的协议试图在它之上去构建自己的网络传输协议。

> Falcor by Netflix is another alternative, and it was developed at the same time as GraphQL. Netflix ran into similar issues as Facebook, and eventually open-sourced their own solution. There isn't too much traction around Falcor, maybe because GraphQL got so popular, but developers at Netflix have shown great engineering efforts in the past, so it may be worth looking into it.

Netflix 的 Falcor 是另一个可供选择的方案，并且它是与 GraphQL 在同一个时代开发的产品。Netflix 当时遇到了和 Facebook 同样的问题，并且最终也开源了他们的解决方案。可能由于 GraphQL 太受欢迎了，所以 Falcor 没有得到太多的关注。但是在过去一段时间中，Netflix 的开发者们也为之付出了很多工程上的努力，所以它可能还是值得去研究一下的。

| |

> There are plenty of reasons to adopt GraphQL for your JavaScript applications instead of implementing yet another RESTful architecture. It has many advantages, and plays nicely with modern software architecture. This book will introduce how it can be used for many practical, real-life solutions, so you should have an idea if it works for you by the time you've read through the chapters.

有大量的理由让你采用 GraphQL 去开发你的 JavaScript 应用，而不是去继续实现另一个 RESTful 架构的应用。GraphQL 拥有很多优点，并且与现代软件架构结合得很好。本书接下来将介绍 GraphQL 是如何运用在许多实际解决方案中的，看完这些章节之后你应该就会明白它对你来说是否有用了。