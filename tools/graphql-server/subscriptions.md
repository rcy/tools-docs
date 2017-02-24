---
title: Subscriptions
order: 204
description: How to set up subscriptions with GraphQL Server
---

<h2 id="overview">Overview</h2>

This section details how to set up a GraphQL server to support subscriptions based on `graphql-subscriptions` and `subscriptions-transport-ws`. `graphql-subscriptions` is a transport-agnostic JavaScript utility that helps you execute GraphQL subscription on your NodeJS server. `subscriptions-transport-ws` is a WebSocket server and client library for GraphQL subscriptions that complements `graphql-subscriptions` and can be used directly in a JavaScript app or wired up to a fully-featured GraphQL client like Apollo or Relay.

<h2 id="setup">Setup</h2>

To get started, install the required packages:

```bash
npm install --save subscriptions-transport-ws graphql-subscriptions
```

Or, with Yarn:
```bash
yarn add subscriptions-transport-ws graphql-subscriptions
```

Now create a new simple PubSub instance and a `SubscriptionManager`:

```js
import { PubSub, SubscriptionManager } from 'graphql-subscriptions';
import mySchema from './schema';

const pubsub = new PubSub();

const subscriptionManager = new SubscriptionManager({
  schema: mySchema,
  pubsub: pubsub
});
```

Next you need to create `SubscriptionsServer` to handle the network connection. We will use the WebSocket transport from `subscriptions-transport-ws` here:

```js
import { createServer } from 'http';
import { SubscriptionServer } from 'subscriptions-transport-ws';

const WS_PORT = 5000;

// Create WebSocket listener server
const websocketServer = createServer((request, response) => {
  response.writeHead(404);
  response.end();
});

// Bind it to port and start listening
websocketServer.listen(WS_PORT, () => console.log(
  `Websocket Server is now running on http://localhost:${WS_PORT}`
));

const subscriptionsServer = new SubscriptionServer(
  {
    subscriptionManager: subscriptionManager
  },
  {
    server: websocketServer
  }
);
```

<h3 id="express">Using subscriptions with GraphQL-Server-Express</h3>

If you already have an existing Express HTTP server (created with `createServer`), you can add subscriptions on a specific path.

For example: if your server is already running on port 3000 and accepts GraphQL HTTP connections (POST) on the `/graphql` endpoint, you can expose `/subscriptions` as your WebSocket subscriptions endpoint:

```js
import express from 'express';
import bodyParser from 'body-parser';
import { graphqlExpress } from 'graphql-server-express';
import { createServer } from 'http';

const PORT = 3000;
const app = express();

app.use('/graphql', bodyParser.json(), graphqlExpress({ schema: myGraphQLSchema }));

const pubsub = new PubSub();
const subscriptionManager = new SubscriptionManager({
  schema: myGraphQLSchema,
  pubsub: pubsub,
});

const server = createServer(app);

server.listen(PORT, () => {
    new SubscriptionServer({
      subscriptionManager: subscriptionManager,
    }, {
      server: server,
      path: '/subscriptions',
    });
});
```

<h3 id="meteor">Using with Meteor</h3>

Meteor exposes `httpServer` server over a `meteor/web` package, so you can use it the same way as any other http server:

```js
import { WebApp } from 'meteor/webapp';

const pubsub = new PubSub();
const subscriptionManager = new SubscriptionManager({
  schema: myGraphQLSchema,
  pubsub: pubsub,
});

new SubscriptionServer({
  subscriptionManager: subscriptionManager,
}, {
  server: WebApp.httpServer,
  path: '/subscriptions',
});
```

<h2 id="adding-subscriptions">Adding Subscriptions to the schema</h2>

Adding GraphQL subscriptions to your GraphQL schema is simple, since subscriptions definition is the same as Query and Mutation: your can customize the publication data with a selection set and add arguments:

You need to create a root schema definition and a root resolver for your `Subscription` root, just like with Query/Mutation:

```
type Comment {
    id: String
    content: String
}

type Subscription {
  commentAdded(repoFullName: String!): Comment
}

schema {
  subscription: Subscription
  query: Query
  mutation: Mutation
}
```

And create a resolver if you need to perform a logic over published data:

```js
const rootResolver = {
    Query: () => { ... },
    Mutation: () => { ... },
    Subscription: {
        commentAdded: comment => {
          // the subscription payload is the comment.
          return comment;
        },
    },
};
```

<h2 id="pubsub">PubSub</h2>

`PubSub` is a class that exposes a simple `publish` and `subscribe` API.

`graphql-subscriptions` exposes a default `PubSub` class you can use for a simple usage of data publication.

Use your `PubSub` instance for publish new data over your subscriptions transport, for example:

```js
import { PubSub } from `graphql-subscriptions`;

const pubsub = new PubSub();

// ... Later in your code, when you want to publish data over subscription, run:

const payload = {
    id: '1',
    content: 'Hello!',
};

pubsub.publish('commentAdded', payload);
```

<h3 id="external-pubsub">Using an external PubSub Engine</h3>

`graphql-subscriptions` also supports any external Pub/Sub system that implements the subscriptions interface of [`PubSubEngine`](https://github.com/apollographql/graphql-subscriptions/blob/master/src/pubsub.ts#L21-L25).

By default `graphql-subscriptions` uses an in-memory event system to re-run subscriptions. This is not suitable for running in a serious production app, because there is no way to share subscriptions and publishes across many running servers.

There are implementations for the following PubSub systems:

* Redis PubSub using [`graphql-redis-subscriptions`](https://www.npmjs.com/package/graphql-redis-subscriptions)
* MQTT using [`graphql-mqtt-subscriptions`](https://www.npmjs.com/package/graphql-mqtt-subscriptions)

<h2 id="setup-functions">Specifying a Setup Function</h2>

`setupFunctions` is a part of `SubscriptionManager` API - it maps from subscription name to a map of channel names and their filter functions.

Let's see an example - for the following server-side subscription:

```
subscription($repoName: String!){
  commentAdded(repoName: $repoName) { # <-- this is the actual GraphQL subscription name
    id
    content
  }
}
```

And the following definition of `setupFunctions`:

```js
const subscriptionManager = new SubscriptionManager({
  schema,
  pubsub,
  setupFunctions: {
    commentAdded: (options, args) => ({
      newCommentsChannel: {
        filter: comment => comment.repository_name === args.repoFullName,
      },
    }),
  },
});
```

In this case, the subscription name is `commentAdded`, but the publication channel is `commentAddedChannel` - so when will use our PubSub instance to publish over `commentAddedChannel` channel - it will re-run the subscription query every time a new comment is posted whose repository name matches `args.repoFullName`.

Our publication looks as follows:

```js
pubsub.publish('newCommentsChannel', {
  id: 123,
  content: 'Test',
  repoFullName: 'apollostack/GitHunt-API',
});
```

<h2 id="events">Lifecycle Events</h2>

`SubscriptionServer` exposes lifecycle hooks you can use to manage your subscription and clients:

* `onConnect` - called upon client connection, with the `connectionParams` passed to `SubscriptionsClient` - you can return a Promise and reject the connection by throwing an exception. The resolved return value will be appended to the GraphQL `context` of your subscriptions.
* `onDisconnect` - called when the client disconnects.
* `onSubscribe` - called when the client subscribes to GraphQL subscription - use this method to create custom params that will be used when resolving this subscription.
* `onUnsubscribe` - called when client unsubscribes from a GraphQL subscription.

```js
const subscriptionsServer = new SubscriptionServer(
  {
    subscriptionManager: subscriptionManager,
    onConnect: (connectionParams, webSocket) => {
        // ...
    },
    onSubscribe: (message, params, webSocket) => {
        // ...
    },
    onUnsubsribe: (webSocket) => {
        // ...
    },
    onDisconnect: (webSocket) => {
        // ...
    }
  },
  {
    server: websocketServer
  }
);
```

<h2 id="authentication">Authentication</h2>

You can use `SubscriptionServer` lifecycle hooks to create an authenticated transport, by using `onConnect` to validate the connection.

`SubscriptionsClient` supports `connectionParams` ([example available here](/react/subscriptions.html#authentication)) that will be sent with the first WebSocket message. All GraphQL subscriptions are delayed until the connection has been fully established.

You can use those connection parameters with `onConnect` callback, and handle the user authentication, and even extend the GraphQL context with the current user object.

```js
const validateToken = (authToken) => {
    // ... validate token and return a Promise, rejects in case of an error
}

const findUser = (authToken) => {
    // ... finds user by auth token and return a Promise, rejects in case of an error
}

const subscriptionsServer = new SubscriptionServer(
  {
    subscriptionManager: subscriptionManager,
    onConnect: (connectionParams, webSocket) => {
       if (connectionParams.authToken) {
            return validateToken(connectionParams.authToken)
                .then(findUser(connectionParams.authToken))
                .then((user) => {
                    return {
                        currentUser: user,
                    };
                });
       }

       throw new Error('Missing auth token!');
    }
  },
  {
    server: websocketServer
  }
);
```

The example above validates the user's token that is sent with the first initialization message on the transport, then looks up the user and return it in a Promise. The user object found will be available as `content.currentUser` in your GraphQL resolvers.

In case of an authentication error, the Promise will be rejected, and the client's connection will be rejected.
