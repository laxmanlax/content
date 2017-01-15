---
alias: oiviev0xi7
path: /docs/reference/relay-api/generated-queries
layout: REFERENCE
shorttitle: Generated Queries
description: Use queries to fetch data. Generated queries in the GraphQL schema of your project are derived from models and relations that you defined.
simple_relay_twin: nia9nushae
tags:
  - relay-api
  - queries
related:
  further:
    - vah0igucil
    - aizoong9ah
  more:
    - cahzai2eur
---

# Generated Queries in the Relay API

A *query* enables you to declare data requirements in your app by supplying multiple [fields](!alias-teizeit5se).
All queries are automatically generated. To explore them, use the [playground](!alias-uh8shohxie#playground) inside your project.

After you send a query to your [endpoint](!alias-uh8shohxie#client-api-endpoints) you will receive the *query response*. It contains the actual data for all fields that were specified in the query:

```graphql
query {
  viewer {
    <query> {
      <subselection of fields>
    }
  }
}
---
{
  "data": {
    "viewer": {
      <response data>
    }
  }
}
```

There are different categories of generated queries. Depending on the query category, different *query arguments* are available that allow you to further modify the query response.

## Querying one node

For each [model](!alias-ij2choozae) in your project there is a query to fetch [one specific node](!alias-ga4chied8m) of that model.

## Querying multiple nodes

For each [model](!alias-ij2choozae) in your project there is a query to fetch [multiple nodes](!alias-uu4ohnaih7) of that model.

## Session user

To get more information on the currently signed in user, there is a query for the [session user](!alias-peyaaph9vi)