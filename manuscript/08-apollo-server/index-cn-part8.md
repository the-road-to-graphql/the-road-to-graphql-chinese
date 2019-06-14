> ### Cursor-based Pagination: Page Info, Connections and Hashes

### 基于游标的分页:页面信息、连接和哈希

> In this last section about pagination in GraphQL, we advance the cursor-based pagination with a few improvements. Currently, you have to query all creation dates of the messages to use the creation date of the last message for the next page as a cursor. GraphQL connections add only a structural change to your list fields in GraphQL that allow you to pass meta information. Let's add a GraphQL connection in the *src/schema/message.js* file:

在 GraphQL 分页的上一章节中，我们对基于游标的分页进行一些改进。目前，你必须查询所有消息的创建时间，以便将最后一条消息的创建时间用作查询下一页的游标。GraphQL 连接只修改列表字段结构，就能支持传递元信息。让我们在 *src/schema/message.js* 文件中添加一个 GraphQL 链接:

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
# leanpub-start-insert
    messages(cursor: String, limit: Int): MessageConnection!
# leanpub-end-insert
    message(id: ID!): Message!
  }

  extend type Mutation {
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

# leanpub-start-insert
  type MessageConnection {
    edges: [Message!]!
    pageInfo: PageInfo!
  }
# leanpub-end-insert

# leanpub-start-insert
  type PageInfo {
    endCursor: Date!
  }
# leanpub-end-insert

  type Message {
    id: ID!
    text: String!
    createdAt: Date!
    user: User!
  }
`;
~~~~~~~~

> You introduced an intermediate layer that holds meta information with the PageInfo type with the list of items in an edges field. In the intermediate layer, you can introduce the new information such as an  `endCursor`( `createdAt` of the last message in the list). Then, you won't need to query every `createdAt` date of every message, only the `endCursor`. Place these changes in the *src/resolvers/message.js* file:

你引入了一个拥有元信息的中间层，其中包含 PageInfo 类型，以及代表列项的 edges 字段。在中间层，你可以引入新的信息，比如 `endCursor` (列表中最后一条消息的 `createdAt`)。然后，你将不需要查询每条消息的 `createdAt` 时间，只需要查询 `endCursor`。在 *src/resolvers/message.js* 文件中进行以下调整:

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

export default {
  Query: {
    messages: async (parent, { cursor, limit = 100 }, { models }) => {
      const cursorOptions = cursor
        ? {
            where: {
              createdAt: {
                [Sequelize.Op.lt]: cursor,
              },
            },
          }
        : {};

# leanpub-start-insert
      const messages = await models.Message.findAll({
# leanpub-end-insert
        order: [['createdAt', 'DESC']],
        limit,
        ...cursorOptions,
      });

# leanpub-start-insert
      return {
        edges: messages,
        pageInfo: {
          endCursor: messages[messages.length - 1].createdAt,
        },
      };
# leanpub-end-insert
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> You gave the result a new structure with the intermediate `edges` and `pageInfo` fields. The `pageInfo` field now has the cursor of the last message in the list, and you should be able to query the first page the following way:

你返回了一个带有 `edges` 和 `pageInfo` 中间字段的新结构。`pageInfo` 字段现在拥有列表中最后一条消息的游标，你应该能够通过以下方式查询第一页:

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  messages(limit: 2) {
    edges {
      text
    }
    pageInfo {
      endCursor
    }
  }
}
~~~~~~~~

> The result may look like the following:

结果如下：

{title="GraphQL Playground",lang="json"}
~~~~~~~~
{
  "data": {
    "messages": {
      "edges": [
        {
          "text": "Published a complete ..."
        },
        {
          "text": "Happy to release ..."
        }
      ],
      "pageInfo": {
        "endCursor": "2018-10-25T08:29:56.771Z"
      }
    }
  }
}
~~~~~~~~

> Use the last cursor to query the next page:

通过最后一个游标查询下一页信息

{title="GraphQL Playground",lang="json"}
~~~~~~~~
query {
  messages(limit: 2, cursor: "2018-10-25T08:29:56.771Z") {
    edges {
      text
    }
    pageInfo {
      endCursor
    }
  }
}
~~~~~~~~

> Again, this will only return the remaining last message in the list. You are no longer required to query the creation date of every message, only to query the cursor for the last message. The client application doesn't need the details for the cursor of the last message, as it just needs `endCursor` now.:

同样，这将返回列表中剩余的最后一条消息。你不再需要查询每条消息的创建时间，只需要查询最后一条消息的游标。客户端应用程序不需要最后一条消息的游标的详细信息，因为它现在只需要 `endCursor`。

> You can add relevant information in the intermediate GraphQL connection layer. Sometimes, a GraphQL client needs to know whether there are more pages of a list to query, because every list is finite. Let's add this information to the schema for the message's connection in the *src/schema/message.js* file

你可以在 GraphQL 连接层中添加相关信息。由于每个列表数目都是有限的，有时，GraphQL 客户端需要知道列表中是否还有更多可以查询的页面。让我们在 *src/schema/message.js* 文件中将这个信息添加到消息连接的 schema 中:

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
    messages(cursor: String, limit: Int): MessageConnection!
    message(id: ID!): Message!
  }

  extend type Mutation {
    createMessage(text: String!): Message!
    deleteMessage(id: ID!): Boolean!
  }

  type MessageConnection {
    edges: [Message!]!
    pageInfo: PageInfo!
  }

  type PageInfo {
# leanpub-start-insert
    hasNextPage: Boolean!
# leanpub-end-insert
    endCursor: Date!
  }

  ...
`;
~~~~~~~~

> In the resolver in the *src/resolvers/message.js* file, you can find this information with the following:

在 *src/resolvers/message.js* 文件的解析器中，你可以看到以下信息

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

export default {
  Query: {
    messages: async (parent, { cursor, limit = 100 }, { models }) => {
      ...

      const messages = await models.Message.findAll({
        order: [['createdAt', 'DESC']],
# leanpub-start-insert
        limit: limit + 1,
# leanpub-end-insert
        ...cursorOptions,
      });

# leanpub-start-insert
      const hasNextPage = messages.length > limit;
      const edges = hasNextPage ? messages.slice(0, -1) : messages;
# leanpub-end-insert

      return {
# leanpub-start-insert
        edges,
# leanpub-end-insert
        pageInfo: {
# leanpub-start-insert
          hasNextPage,
          endCursor: edges[edges.length - 1].createdAt,
# leanpub-end-insert
        },
      };
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> You only retrieve one more message than defined in the limit. If the list of messages is longer than the limit, there is a next page; otherwise, there is no next page. You return the limited messages, or all messages if there is no next page. Now you can include the `hasNextPage` field in the `pageInfo` field. If you query messages with a limit of 2 and no cursor, you get true for the `hasNextPage` field. If query messages with a limit of more than 2 and no cursor, the `hasNextPage` field becomes false. Then, your GraphQL client application knows that the list has reached its end.

你只比定义的 limit 多检索一条消息。如果消息列表数量大于 limit，则有下一页;否则，就没有下一页。要么返回限定的 limit 数量的消息，要么没有下一页时，返回所有消息。现在，你可以在 `pageInfo` 类型中添加 `hasNextPage` 字段。如果查询消息的条件为 limit=2,且没有游标，返回结果中，`hasNextPage` 为 true。如果查询消息的条件为 limit>2且没有游标，返回结果中，`hasNextPage` 为 false。据此，GraphQL 客户端应用程序知道列表已经查询到最后了。

> The last improvements gave your GraphQL client application a more straightforward GraphQL API. The client doesn't need to know about the cursor being the last creation date of a message in a list. It only uses the `endCursor` as a `cursor` argument for the next page. However, the cursor is still a creation date property, which may lead to confusion on the GraphQL client side. The client shouldn't care about the format or the actual value of the cursor, so we'll ask the cursor with a hash function that uses a base64 encoding:

最后的改进是给 GraphQL 客户端应用程序提供一个更加直观的 GraphQL API。客户端不需要知道游标代表列表中最后一条消息的创建时间。它仅使用 `endCursor` 作为下一页的 `cursor` 参数。但是，游标仍然是一个创建时间属性，这可能会引起 GraphQL 客户端的困惑。客户端不应该关心游标的格式或实际值，所以我们将用一个使用 base64 编码的哈希函数来处理游标:

{title="src/resolvers/message.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const toCursorHash = string => Buffer.from(string).toString('base64');
# leanpub-end-insert

# leanpub-start-insert
const fromCursorHash = string =>
  Buffer.from(string, 'base64').toString('ascii');
# leanpub-end-insert

export default {
  Query: {
    messages: async (parent, { cursor, limit = 100 }, { models }) => {
      const cursorOptions = cursor
        ? {
            where: {
              createdAt: {
# leanpub-start-insert
                [Sequelize.Op.lt]: fromCursorHash(cursor),
# leanpub-end-insert
              },
            },
          }
        : {};

      ...

      return {
        edges,
        pageInfo: {
          hasNextPage,
# leanpub-start-insert
          endCursor: toCursorHash(
            edges[edges.length - 1].createdAt.toString(),
          ),
# leanpub-end-insert
        },
      };
    },
    message: async (parent, { id }, { models }) => {
      return await models.Message.findById(id);
    },
  },

  Mutation: {
    ...
  },

  ...
};
~~~~~~~~

> The returned cursor as meta information is hashed by the new utility function. Remember to stringify the date before hashing it. In addition, the `endCursor` in the *src/schema/message.js* file isn't a Date anymore, but a String scalar again.

作为元信息返回的游标被这个新的通用函数哈希化。记住，在哈希化之前要把时间转化为字符串格式。此外，在 *src/schema/message.js* 文件中 `endCursor` 不再是一个时间，而是一个字符串标量。

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  ...

  type MessageConnection {
    edges: [Message!]!
    pageInfo: PageInfo!
  }

  type PageInfo {
    hasNextPage: Boolean!
# leanpub-start-insert
    endCursor: String!
# leanpub-end-insert
  }

  ...
`;
~~~~~~~~

> The GraphQL client receives a hashed `endCursor` field. The hashed value can be used as a cursor to query the next page. In the resolver, the incoming cursor is reverse hashed to the actual date, which is used for the database query.

GraphQL 客户端接收哈希化的 `endCursor` 字段。这个哈希值可以用作查询下一页的游标。在解析器中，用于数据库查询时，需要把传入的游标反哈希化成实际时间。

> Hashing the cursor is a common approach for cursor-based pagination because it hides the details from the client. The (GraphQL) client application only needs to use the hash value as a cursor to query the next paginated page.

哈希化游标是基于游标的分页的一种常见方法，因为它对客户端隐藏了详细信息。(GraphQL)客户端应用程序只需要使用哈希值作为查询下一页的游标。

> ### Exercises:

### 练习:

> * Confirm your [source code for the last section](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/810907cde43b460231b9ed3a2172e62528f81ba4)
> * Read more about [GraphQL pagination](https://graphql.github.io/learn/pagination/)

* 查看 [上一节代码](https://github.com/the-road-to-graphql/fullstack-apollo-react-express-boilerplate-project/tree/810907cde43b460231b9ed3a2172e62528f81ba4)
* 阅读更多关于 [GraphQL 分页](https://graphql.github.io/learn/pagination/)


> ## GraphQL Subscriptions

## GraphQL 订阅

> So far, you used GraphQL to read and write data with queries and mutations. These are the two essential GraphQL operations to get a GraphQL server ready for CRUD operations. Next, you will learn about GraphQL Subscriptions for real-time communication between GraphQL client and server.

到目前为止，你使用 GraphQL 的查询(Query)和变更(Mutation)来读写数据。这是使 GraphQL 服务端为 CRUD（增删改查）操作做好准备的两个基本 GraphQL 操作。接下来，你将了解 GraphQL 订阅，以便在 GraphQL 客户端和服务端之间进行实时通信。

> Next, you will implement real-time communication for created messages. If a user creates a message, another user should get this message in a GraphQL client application as a real-time update. To start, we add the Subscription root level type to the *src/schema/message.js* schema:

接下来，你将为创建的消息实现实时通信。如果一个用户创建了一条消息，那么在 GraphQL 客户端应用程序中，另一个用户应当实时获得这条新消息。首先，我们在 *src/schema/message.js* 文件中，添加根级别的订阅类型:

{title="src/schema/message.js",lang="javascript"}
~~~~~~~~
import { gql } from 'apollo-server-express';

export default gql`
  extend type Query {
    ...
  }

  extend type Mutation {
    ...
  }

  ...

  type Message {
    id: ID!
    text: String!
    createdAt: Date!
    user: User!
  }

# leanpub-start-insert
  extend type Subscription {
    messageCreated: MessageCreated!
  }
# leanpub-end-insert

# leanpub-start-insert
  type MessageCreated {
    message: Message!
  }
# leanpub-end-insert
`;
~~~~~~~~

> As a naive GraphQL consumer, a subscription works like a GraphQL query. The difference is that the subscription emits changes (events) over time. Every time a message is created, the subscribed GraphQL client receives the created message as payload. A subscription from a GraphQL client for the schema would look like this:

作为一个简单的 GraphQL 用户，订阅的工作方式类似于 GraphQL 查询(query)。不同之处在于订阅会实时触发变更(事件)。每次创建消息时，订阅的 GraphQL 客户端都会将创建的消息作为有效负载接收。来自 GraphQL 客户端的订阅 schema 将如下所示:

{title="GraphQL Playground",lang="json"}
~~~~~~~~
subscription {
  messageCreated {
    message {
      id
      text
      createdAt
      user {
        id
        username
      }
    }
  }
}
~~~~~~~~

> In the first part, you'll set up the subscription architecture for your application; then, you'll add the implementation details for the created message subscription. The first step need only be completed once, but the latter will be a recurring when more GraphQL subscriptions are added to your application.

在第一部分中，你将为应用程序设置订阅结构;然后，你将为创建的消息订阅添加实现细节。第一步只需要做一次，但是当向应用程序添加更多的 GraphQL 订阅时，这一步将需要再次设置。

> ### Apollo Server Subscription Setup

### Apollo Server 订阅配置

> Because we are using Express as middleware, expose the subscriptions with an advanced HTTP server setup in the *src/index.js* file:

由于我们使用 Express 作为中间件，所以在 *src/index.js* 文件中可以使用高级 HTTP 服务设置公开订阅:

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import http from 'http';
# leanpub-end-insert

...

server.applyMiddleware({ app, path: '/graphql' });

# leanpub-start-insert
const httpServer = http.createServer(app);
server.installSubscriptionHandlers(httpServer);
# leanpub-end-insert

const eraseDatabaseOnSync = true;

sequelize.sync({ force: eraseDatabaseOnSync }).then(async () => {
  if (eraseDatabaseOnSync) {
    createUsersWithMessages(new Date());
  }

# leanpub-start-insert
  httpServer.listen({ port: 8000 }, () => {
# leanpub-end-insert
    console.log('Apollo Server on http://localhost:8000/graphql');
  });
});

...
~~~~~~~~

> For the context passed to the resolvers, you can distinguish between HTTP requests (GraphQL mutations and queries) and subscriptions in the same file. HTTP requests come with a req and res object, but the subscription comes with a connection object, so you can pass the models as a data access layer for the subscription's context.

根据传递给解析器的上下文，你可以在同一个文件中区分 HTTP 请求(GraphQL 变更和查询）和订阅。HTTP 请求附带一个 req 和 res 对象，但是订阅附带一个连接对象，因此可以将模型(models)作为订阅上下文的数据访问层。

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

const server = new ApolloServer({
  typeDefs: schema,
  resolvers,
  ...
# leanpub-start-insert
  context: async ({ req, connection }) => {
    if (connection) {
      return {
        models,
      };
    }
# leanpub-end-insert

# leanpub-start-insert
    if (req) {
# leanpub-end-insert
      const me = await getMe(req);

      return {
        models,
        me,
        secret: process.env.SECRET,
      };
# leanpub-start-insert
    }
# leanpub-end-insert
  },
});

...
~~~~~~~~

> To complete the subscription setup, you'll need to use one of the available [PubSub engines](https://www.apollographql.com/docs/apollo-server/v2/features/subscriptions.html#PubSub-Implementations) for publishing and subscribing to events. Apollo Server comes with its own by default, but there are links for other options should you find it lacking. In a new *src/subscription/index.js* file, add the following:

要完成订阅设置，你需要使用一个可用的 [PubSub 引擎](https://www.apollographql.com/docs/apollo-server/v2/features/subscriptions.html#PubSub-Implementations)来发布和订阅事件。默认情况下，Apollo Server 自带 PubSub 引擎，但是你可能会发现它缺少一些其他的配置。新建一个 *src/subscription/index.js* 文件，配置以下内容:

{title="src/subscription/index.js",lang="javascript"}
~~~~~~~~
import { PubSub } from 'apollo-server';

export default new PubSub();
~~~~~~~~

> This PubSub instance is your API which enables subscriptions in your application. The overarching setup for subscriptions is done now.

这个 PubSub 实例是你的 API，它支持应用程序中的订阅。现在我们完成了订阅的总体设置。
