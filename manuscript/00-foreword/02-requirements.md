## 基本要求

要想充分理解本书的内容，你应该熟悉基本的 Web 开发知识，其中包括 HTML，CSS 和 JavaScript。除此之外你也需要熟悉 [API](https://www.robinwieruch.de/what-is-an-api-javascript/) 这个术语，因为它会经常被提到。同时，我建议你加入本书的官方 [Slack 讨论组](https://slack-the-road-to-learn-react.wieruch.com/)，在那里你可以帮助他人或者寻求帮助。

### 编辑器和终端

对于开发环境，你需要一个可用的编辑器或者IDE，以及一个终端（命令行工具），然后[参照我的环境搭建指南](https://www.robinwieruch.de/developer-setup/)配置环境。该指南适用于 MacOS 用户，但你也可以在上面找到一个 Windows 的搭建指南。

### React

在客户端上，本书使用了 React 来讲授 JavaScript 中的 GraphQL。我的另一本书《The Road to learn React》(《React 学习之道》)讲授了所有关于 React 的基础知识。它同时还讲了如何从 JavaScript ES5 迁移到 JavaScript ES6。那本书可以免费阅读，在看完它之后，你应该就能掌握本书中实现 GraphQL 客户端应用程序的所有基础知识。

### Node

至于服务器端上，本书使用了 Node 加 Express 作为库来讲授 JavaScript 中的 GraphQL。在将这些技术用于你的第一个支持 GraphQL 的应用程序之前，你不必了解太多关于它们的知识。本书将指导你完成用 Express 设置 Node 应用的过程，并且像你展示如何将 GraphQL 穿插其中。之后，在你的前端应用中就能够消费你的后端应用提供的 GraphQL API 了。

### Node and NPM

你需要安装 [node 和 npm](https://nodejs.org/en/)，这两个工具都是用来管理你在本书中所用到的各种库的。本书中提及的 node 包需要通过 npm （node package manager，Node 包管理器）来安装，这些包可能是一些库，或者是集成在一起的框架。你可以在命令行验证 node 和 npm 的安装版本。如果没有看到任何输出结果，那就说明你需要安装他们，下面是我在写这本书的时候使用的版本：

{title="Command Line",lang="text"}
~~~~~~~~
node --version
*v10.11.0
npm --version
*v6.4.1
~~~~~~~~

如果你看过《React 学习之道》，你应该已经熟悉整个设置过程了，因为那本书中也采用了 npm 生态系统。