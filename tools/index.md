---
title: Overview
order: 0
---

In addition to a set of fully-featured GraphQL clients, the Apollo community maintains a set of tools for building GraphQL servers, and some utilities that make it easier to work with GraphQL in general.

If you're looking for a simple way to get a GraphQL.js server up and running, start with [graphql-tools](/tools/graphql-tools/).

## GraphQL server & tools

These are libraries designed to make it easy to build a JavaScript GraphQL server using [GraphQL.js](https://github.com/graphql/graphql-js), Facebook's reference implementation of a GraphQL type system and execution engine.

- [graphql-tools](/tools/graphql-tools), a package that enables you to build a production-ready GraphQL.js schema using the GraphQL schema language, rather than using the GraphQL.js type constructors directly. Schemas built with this package are compatible with any GraphQL servers, including our own `apollo-server` and Facebook's `express-graphql`. This allows you to build on the [getting started guide for GraphQL.js](http://graphql.org/graphql-js/) with additional support for resolvers, unions, interfaces, custom scalars, modularizing your schema, and more.
- [graphql-server](/tools/graphql-server), a production-ready Node.js GraphQL server library that supports **Express**, **Connect**, **Hapi**, **Koa**, and other popular Node HTTP servers, with built-in features like persisted queries, batching, and more. Apollo Server works with any GraphQL client, like Apollo, Relay, Lokka, and more.

## GraphQL subscriptions

The Apollo community is excited about enabling people to add realtime functionality to their applications, and GraphQL subscriptions are the first step on that path. You can use the packages below to add subscription support to any Node.js GraphQL server.

- [graphql-subscriptions](https://github.com/apollostack/graphql-subscriptions), a transport-agnostic JavaScript utility that helps you execute GraphQL subscriptions and attach them to event sources.
- [subscriptions-transport-ws](https://github.com/apollostack/subscriptions-transport-ws), a WebSocket server and client for GraphQL subscriptions that works with `graphql-subscriptions` and can be easily used directly in a JavaScript app or wired up to a fully-featured GraphQL client like Apollo or Relay.

You can red more about subscription server [here]().

## GraphQL client tools

- [graphql-anywhere](https://github.com/apollostack/graphql-anywhere), the core schema-less GraphQL execution engine of the Apollo JavaScript Client, which lets you build your own powerful GraphQL tools and cache implementations, and works anywhere you can run JavaScript code.
- [graphql-tag](https://github.com/apollostack/graphql-tag), a simple library for parsing and printing GraphQL queries, as well as putting together multiple fragments into one query. It helps power [apollo-client](https://github.com/apollostack/apollo-client) and works great with [eslint-plugin-graphql](https://github.com/apollostack/eslint-plugin-graphql).

## Developer tools

- [eslint-plugin-graphql](https://github.com/apollostack/eslint-plugin-graphql), an ESLint plugin that will check your GraphQL query strings for syntax errors and schema compliance, and works with any JavaScript GraphQL client including Apollo, Relay, Lokka, and more.

