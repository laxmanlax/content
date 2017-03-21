# Relay vs Apollo - Comparing GraphQL Clients

[Relay](https://facebook.github.io/relay/) and [Apollo](http://dev.apollodata.com/) are the most popular and sophisticated GraphQL clients available at the moment. But how do you know which one to choose? 

In this article, we are going to shed some light on the commonalities and differences between Relay and Apollo and want to help you in making an informed decision on which GraphQL client is the best for your next project!

> This article assumes some familiarity with basic GraphQL concepts such as queries, mutations or fragments. If you're just getting started with GraphQL, you can read up on these concepts in the [GraphQL docs](http://graphql.org/). Also, if you're keen in learning about Relay and Apollo in more detail, you can visit [Learn Relay](www.learnrelay.org) and [Learn Apollo](www.learnapollo.com) for comprehensive tutorials.


## Why using a GraphQL client? 

GraphQL backends commonly expose their API over HTTP where queries and mutations can be sent in the _body_ of a POST request. When building a JavaScript app, this means that you can get quite far by using `fetch` or `request` and thus interact with the GraphQL API directly.

However, there are a number of recurring challenges and use cases in working with a GraphQL backend that are agnostic to the app that's being built and that would have to be reimplemented with every new project. A few examples are:

- caching data that is returned by the server
- making received data available to the UI
- keeping the local cache consistent after a mutation
- setting up websockets to implement subscriptions for realtime updates
- pagination for collections

A GraphQL client should come with that kind of functionality so that you don't have to reimplement these behaviours yourself. Instead, you can completely focus on your application domain and on implementing the specific requirements of your app.



## Relay vs Apollo - Overview

In the following, we are going to explore how to solve common and recurring client-side tasks when working with a GraphQL API, both from a Relay and an Apollo perspective. 

To give you a broad overview on what you can expect from either client, here is a high-level comparison before we dive into more details.

| | Relay | Apollo | 
| --- | --- | --- | 
| Built by | [Facebook](https://code.facebook.com/projects/) (Check out the project on [GitHub](https://github.com/facebook/relay)) | [Meteor](https://www.meteor.com/) (Check out the project on [GitHub](https://github.com/apollographql)) | 
| Frontend Technologies | requires [React](https://facebook.github.io/react/) / [React Native](https://facebook.github.io/react-native/); needs configuration of [Babel plugin](https://facebook.github.io/relay/docs/guides-babel-plugin.html#content) | framework and platform agnostic (works with any JS framework such as React, Angular or Vue as well as on the native mobile platforms) |
| GraphQL API | Requires a certain structure in the GraphQL schema | Works with any GraphQL schema |
| Complexity | Slow learning curve: Lots of powerful _magic_ happening behind the scenes | Low entrance barrier: Let's you get started quickly and involves more manual work for certain features |
| Flexibility | Almost no flexibility, strict rules how to integrate Relay with React components | Lots of flexibility how Apollo is used throughout a whole project, easy to adopt incrementally |
| Usage of GraphQL [fragments](https://facebook.github.io/graphql/#sec-Language.Fragments) | Relies on fragments as an essential tool to express data requirements per component | Fragments are a convenience to improve structuring of GraphQL queries and mutations that generally enable reusablity of GraphQL code | 
| GraphQL Subscription | No explicit means to integrate subscriptions | Subscriptions supported through [`subscriptions-transport-ws`](https://github.com/apollographql/subscriptions-transport-ws) | 

When reading about the differences of Relay and Apollo, you'll notice that a major difference lies in the flexibility of the two approaches. While Relay is very opiniated and doesn't give you a lot of freedom in how you want to structure your application, Apollo has a variety of options that range from lightweight integrations to much more sophisticated approaches. 

In short, Relay lends itself well for large-scale applications that have complex data requirements and many dependencies between different parts of the application where maintaining these dependencies by hand would be very tedious and error-prone. Apollo on the other hand provides a much more lightweight and flexible approach that works in any environment. Many tasks such as keeping the local cache consistent can also be achieved with the Apollo client but require a little more manual work upfront.

In the following, we are going to to compare Apollo and Relay by practical examples and show how each of them can be used to deal with a certain use case. We are going to use the data model for the Pokedex application that is being used on [Learn Relay](www.learnrelay.org) and [Learn Apollo](www.learnapollo.com).

This is what the data model looks like:

```graphql
type Pokemon {
  id: ID!
  name: String
  trainer: Trainer @relation(name: "OwnedPokemons")
  url: String
}

type Trainer {
  id: ID!
  name: String
  ownedPokemons: [Pokemon!]! @relation(name: "OwnedPokemons")
}
```


## Relay vs Apollo - Environment and setup

### Server-side requirements

In contrast to the Apollo client that works with any GraphQL schema, Relay actually has a few requirements when it comes to the structure of the GraphQL schema that's implemented on the server.

Basically, with Relay the GraphQL server is expected to expose two kinds of capabilities:

- query one particular resource by its ID using a root field in the GraphQL schema called `node` that takes an `ID` as an argument
- query the data graph using a root field called `viewer` that contains all other fields as children


### Modelling relationships

With Relay, creating relations between model objects in the GraphQL schema also needs to follow a specific pattern whereas in the Apollo client it doesn't matter what the relation between two model types looks like. 

The Relay approach again contains some additional complexity but comes with nice benefits when implementing features such as pagination for collections. The concept that is used to model a relationship in Relay is called [connection](https://facebook.github.io/relay/docs/graphql-connections.html#content). It requires that a relation in the data model is expressed using `edges` that each contain a `node`.

If you wanted to access all the pokemons in the database, this would look as follows with Relay:

```graphql
{
  viewer {
    allPokemons(first: 1000) {
      edges {
        node {
          id
          name
        }
      }
    }
  }
}
```

Notice that the `edges` field requires an indication of how many objects you want to fetch, here this is done using the `first` parameter but more options such as `last`, `after` or `before` are avilable as well. This allows for easy slicing of the list and implement pagination easily.

A more intuitive but less powerful model of relationships is implemented in the Graphcool [Simple API](!alias-heshoov3ai/) which is optimized for usage with Apollo. Here, querying the same info would look like this:

```graphql
{
  allPokemons {
    id
    name
  }
}
```

### Using GraphQL with higher-order components in React

While Apollo can be used in [any client-side environment](http://dev.apollodata.com/) (such as React, Angular, Vue or plain JS as well as on iOS and Android), Relay is restricted to be used with React or React Native. 

However, they actually both follow a similar approach when being used in a React application: With both clients, the general idea is that React components are wrapped using a [higher-order component](https://facebook.github.io/react/docs/higher-order-components.html) which takes care of fetching the data and making it available to the component through its props. Data requirements for a component are specified in a declarative manner and all actual networking logic is completely abstracted away and hidden from the developer. A major benefit of this approach is that the container can manage data fetching and resolution logic without interfering with the state of the inner component.


## Relay vs Apollo - Fetching data using GraphQL queries

A major responsibility of any GraphQL client is the ability to fetch data and make it available to the view layer of the app. In Relay, there is only one way how to get access to data in a React component and that is through a higher-order component called [`Relay.Container`](https://facebook.github.io/relay/docs/guides-containers.html#content). With Apollo, it is possible to use a similar approach with the [`graphql`](http://dev.apollodata.com/react/api.html#graphql) higher-order component. Another way to obtain data from the server would be to directly send queries using the [`ApolloClient`](http://dev.apollodata.com/core/apollo-client-api.html#apollo-client) and process the returned data in a callback function. In the following, we are going to dive into what data fetching with Relay and with Apollo looks like.


### Data fetching with Relay

##### Co-located queries

Relay heavily relies on GraphQL fragments to express data requirements. In fact, when working with Relay, we're not actually writing full-blown GraphQL queries but only specify the data that is needed for each component in terms of a fragment. Relay is then taking care of composing the fragments and building the actual queries that are getting sent to the server. As developers, this process is completely hidden from us and we don't have an easy way to influence it. 

As mentioned above, React components that need data from the server need to be wrapped with a `Relay.Container`. This higher-order component does not directly take care of fetching the data but rather provides the wrapped component with the ability to define its data requirements in the form of a fragment and then guarantees that this data is available before the component is rendered.

Let's take a look at some example code that creates and exports a Relay container which wants to display all Pokemons of the trainer called `Ash Ketchum` in a `Pokedex` component.

The `Pokedex` component renders a list of `PokemonPreview` components where each `PokemonPreview` presents information for one particular Pokemon. 

```js
// wrap Pokedex class with a RelayContainer and inject the data requirements via fragments
export default Relay.createContainer(
  // the wrapped component
  Pokedex, 
  {
    // specify data requirements in terms of a fragment
    fragments: {
      viewer: () => Relay.QL`
        fragment on Viewer {
          allPokemons (
            filter: {
              trainer: {
                name: "Ash Ketchum"
              }
            },
            first: 1000
          ) {
            edges {
              node {
                id
                # include fields that appear in the `pokemon` fragment in `PokemonPreview`
                ${PokemonPreview.getFragment('pokemon')}
              }
            }
          }
          id
        }
      `,
    },
  },
)
```

In the wrapped React component, we have to specify props according to the defined fragments on the Relay container. In this case, we need to specify `viewer` that includes the `allPokemons` object.

```js
// the Pokedex class is responsible to display multiple pokemons
class Pokedex extends React.Component {
  // require the viewer prop type
  static propTypes = {
    viewer: React.PropTypes.shape({
      allPokemons: React.PropTypes.object,
    }),
  }

  render () {
    return (
      <div>
        <div>
          // iterate the edges and nodes in the allPokemons connection
          {this.props.viewer.allPokemons.edges.map((edge) => edge.node).map((pokemon) =>
            <PokemonPreview key={pokemon.id} pokemon={pokemon} />)
          }
        </div>
      </div>
    )
  }
}
```

##### Data normalization with nodes 

Relay requires the unique `id` field on every node in the GraphQL backend. This ID is heavily used by Relay to normalize the data and make sure that all components rerender when there is new data for a certain node. No more configuration is needed to make the cache consistent from the client side - however, there is also no possibility to change this behaviour.

For example, for a GraphQL backend that only has unique IDs per model, Relay's cache mechanisms would break. Graphcool uses [cuids](https://github.com/graphcool/cuid-java) to generate unique IDs across all nodes in your project, so this is not an issue.

To read more about GraphQL queries in Relay, refer to the [Learn Relay](https://www.learnrelay.org/queries/what-is-a-query).


### Data fetching with Apollo

With Apollo, it is possible to fetch data in two different ways:

1. Wrapping the React component with the `graphql` higher-order component and make the data available through the props
2. Send a query directly using the `ApolloClient` and handle the return data in a promise

##### 1. Using `graphql` to wrap a React component

When a React component requires some data, it can be wrapped with a query that expresses these data requirements and then the data will be available through the component's props. Apollo further injects a `loading` and `error` property into the wrapped component. This allows the component to render in situations where the data has not yet arrived in the client (where `loading` will be `true`) or where the network request failed (so `error` will contain some info about what went wrong).

Taking the same scenario as above where the `Pokedex` component displays all Pokemons owned by the trainer called `Ash Ketchum`, the code to wrap the `Pokedex` component looks as follows:

```js
const allPokemonsQuery = gql`
  query {
    allPokemons(filter: {
      trainer: {
        name: "Ash Ketchum"
      }
    }) {
      id
      name
      url
    }
  }
`

export default graphql(allPokemonsQuery)(Pokedex)
```

The response for the query as well as the `loading` and `error` properties are all part of an object called `data` in the props of the wrapped component. Note that it is possible to rename this object, but it's called `data` by default. Consequently, in our scenario the implementation of the `Pokedex` component could be done like so:

```js
// the Pokedex class is responsible to display multiple pokemons
class Pokedex extends React.Component {

  // require the data prop type
  static propTypes = {
    data: React.PropTypes.shape({
      loading: React.PropTypes.bool,
      error: React.PropTypes.object,
      allPokemons: React.PropTypes.array,
    }).isRequired,
  }

  render () {
    // data.loading informs about loading state - we can easily render a loading animation or a text
    if (this.props.data.loading) {
      return (<div>Loading</div>)
    }

    // in the case of an error with the query, data.error contains more information
    if (this.props.data.error) {
      console.log(this.props.data.error)
      return (<div>An unexpected error occurred</div>)
    }

    return (
      <div className='w-100 bg-light-gray min-vh-100'>
        <div className='flex flex-wrap justify-center center w-75'>
          {this.props.data.allPokemons.map((pokemon) =>
            <PokemonPreview key={pokemon.id} pokemon={pokemon} />
          )}
        </div>
      </div>
    )
  }
}
```

##### 2. Directly send queries using `ApolloClient`

The second option is to send a query using the `ApolloClient` and processing the result as a promise. In that case, the [`query`](http://dev.apollodata.com/core/apollo-client-api.html#ApolloClient\.query) method of the `ApolloClient` can be used:

```js
client.query({
  query: gql`
    {
    allPokemons(filter: {
      trainer: {
        name: "Ash Ketchum"
      }
    }) {
      id
      name
      url
    }
  }
  `
})
.then(result => console.log(result))
```

This approach leads to more flexibility if you require certain data for something other than displaying it in the UI.


##### Controlling the Apollo Store

Apollo uses a [normalized store](https://dev-blog.apollodata.com/building-a-graphql-store-from-first-principles-413e19ea65b4#.kzjmjvfo9) for its local cache. That means that the data that is being returned by the server and (potentially) has a nested shape will be flattened before being put into the store. All objects will be stored on the first level of the cache uniquely identified by their ID.

Let's consider the following example query:

```graphql
query {
  Trainer(name: "Ash Ketchum"){
    id
    name
    ownedPokemons {
      id
      name
    }
  }
}
```  

And assume it returns the following JSON data:

```js
{
  "data": {
    "Trainer": {
      "id": "ciwj0dw5dm6aj01632pto44t0",
      "name": "Nikolas",
      "ownedPokemons": [
        {
          "id": "ciwj0dw6zpeuj0148xvsgc3hr",
          "name": "Mewtwo"
        },
        {
          "id": "ciwj0dw8gm6c80163wh8zku78",
          "name": "Pikachu"
        }
      ]
    }
  }
}
```

Apollo would now _flatten_ the nested Pokemons and put them on the first level of the store, so the contents of the store would look somewhat similar to this:

```js
{
    "ciwj0dw5dm6aj01632pto44t0": {
      "id": "ciwj0dw5dm6aj01632pto44t0",
      "name": "Nikolas",
      "ownedPokemons": [ciwj0dw6zpeuj0148xvsgc3hrm, ciwj0dw8gm6c80163wh8zku78]
    },
    "ciwj0dw6zpeuj0148xvsgc3hr": {
      "id": "ciwj0dw6zpeuj0148xvsgc3hr",
      "name": "Mewtwo"
    },
    "ciwj0dw8gm6c80163wh8zku78": {
      "id": "ciwj0dw8gm6c80163wh8zku78",
      "name": "Pikachu"
    }
}
```

In order for this approach to work, the Apollo store needs to be able to uniquely identify an object. It therefore provides the `dataIdFromObject` function that can be provided upon initialization of the `ApolloClient`. In this method, we can specify how an object can be uniquely identified. With Graphcool, this is done using the `id` property, so the implementation would look as follows:

```js
const client = new ApolloClient({
  networkInterface: createNetworkInterface({ uri: 'https://api.graph.cool/simple/v1/__PROJECT_ID__'}),
  dataIdFromObject: o => o.id
})
```

With another GraphQL backend, your IDs might only be unique per model. In this case, you can use the following setup:

```js
const client = new ApolloClient({
  networkInterface: createNetworkInterface({ uri: 'https://api.graph.cool/simple/v1/__PROJECT_ID__'}),
  dataIdFromObject: o => o.__typename + ' ' + o.id
})
```

In both cases, you have to make sure to include the `id` in all queries and mutations whose results should be normalized. To read more about GraphQL queries in Apollo Client, refer to the [Learn Apollo](https://ww.learnapollo.com/tutorial-react/react-02/). There's also an [excursion on the Apollo Store](https://www.learnapollo.com/excursions/excursion-02).


## Relay vs Apollo - Creating, updating and deleting data with mutations

Sending mutations is a core feature of any GraphQL client allowing you to create, modify or delete data in a GraphQL backend.

While calling mutations in both Relay and Apollo is done with mutation strings where GraphQL variables are injected, the two clients handle cache consistency in combination with mutations in completely different manners.

Next, we want to explore how we can send the following GraphQL mutation with Relay and Apollo:

```graphql
mutation {
  createPokemon(
    name: "Zapdos",
  	 url: "http://assets.pokemon.com/assets/cms2/img/pokedex/full/145.png"
  ) {
    id
    name
  }
}
```

### Mutations with Relay

Mutations in Relay are very verbose and not trivial to understand initially. That is because we need to provide a lot of info that Relay uses behind the scenes to make sure all places in our app where the mutated data is used are getting updated accordingly. The major upside of this approach is that we are getting cache consistency for free and without any further manual work, we just need to understand what Relay needs. With Apollo, achieving cache constistency requires a bit more manual work.

Generally, a mutation needs to be defined as a subclass of the `Relay.Mutation` class. There are two methods on the `Relay` object that can be used to perform a mutation to the server:

1. [`commitUpdate(mutation)`](https://facebook.github.io/relay/docs/api-reference-relay-store.html#commitupdate-static-method) send the mutation right away
2. [`applyUpdate(mutation)`](https://facebook.github.io/relay/docs/api-reference-relay-store.html#applyupdate-static-method) returns a _transaction_ that can be sent later on by simply calling `commit()` on it

When subclassing `Relay.Mutation`, the following methods must be implemented:

* `getMutation()`: Specify the name of the mutation (from the GraphQL schema).
* `getFatQuery()`: Specify all nodes, edges and connections in our data graph that may change after this mutation.
* `getConfigs()`: Tell Relay what exactly happens with this mutation. The configuration `RANGE_ADD` needs a `parentName`, a `parentID` (that's what we need the `viewer` in the fragments for!), a `connectionName` and an `edgeName`. Additionally we need a list of `rangeBehaviors`, but typically `'': 'append'` is enough.
* `getVariables()`: Specify the variables needed for this mutation.
* `getOptimisticResponse()` (optional): Specify optimistic data for the query response. It's only possible to return data specified in the fat query and `viewer`.

We also need to provide that static property `fragments`:

* `fragments`: Specify the data needed by this mutation that is already a part of our data graph. In our case, we need the `viewer` ID so that we're able to append a `pokemon` node to the `allPokemons` connection (see `getConfigs`).

This is what our mutation subclass would look like:

```js
import Relay from 'react-relay'
export default class CreatePokemonMutation extends Relay.Mutation {

  // Specifies required data for this mutation.
  // Needs to be supplied together with `getVariables` defined below when calling this mutation.
  static fragments = {
    viewer: () => Relay.QL`
      fragment on Viewer {
        id
      }
    `,
  }

  // Specifies name of the mutation from the GraphQL schema.
  getMutation () {
    return Relay.QL`mutation{createPokemon}`
  }

  // Specifies what data may have changed due to this mutation.
  getFatQuery () {
    return Relay.QL`
      fragment on CreatePokemonPayload {
        pokemon
        edge
        viewer {
          allPokemons
        }
      }
    `
  }
  
  // Uses RANGE_ADD type to add `pokemon` node to `allPokemons` edge
  getConfigs () {
    return [{
      type: 'RANGE_ADD',
      parentName: 'viewer',
      parentID: this.props.viewer.id,
      connectionName: 'allPokemons',
      edgeName: 'edge',
      rangeBehaviors: {
        '': 'append',
      },
    }]
  }

  // Required variables for this mutation.
  // Needs to be supplied together with objects defined in `fragments` above when calling this mutation
  getVariables () {
    return {
      name: this.props.name,
      url: this.props.url,
    }
  }

  // Defines an optimistic response to instantly update the UI. This will be overwritten if the mutation fails.
  getOptimisticResponse () {
    return {
      edge: {
        node: {
          name: this.props.name,
          url: this.props.url,
        },
      },
      viewer: {
        id: this.props.viewer.id,
      },
    }
  }

}
```

In this mutation, we describe what Relay needs to know about the `createPokemon` mutation to first execute the mutation and then update the client side store to keep its data consistent.

We can then actually send the mutation to the server using the `commitUpdate()` method that was mentioned above:

```js
Relay.Store.commitUpdate(
  new CreatePokemonMutation({name: this.state.name, url: this.state.url, viewer: this.props.viewer}),
  {
    onSuccess: (response) => console.log('created pokemon', response),
    onFailure: (transaction) => console.log('error', transaction),
  },
)
```

To find out more, you can read about other [mutation configurations](https://www.learnrelay.org/mutations/mutation-types) and [optimistic responses](https://www.learnrelay.org/mutations/optimistic-updates) or read the [official mutation documentation](https://facebook.github.io/relay/docs/guides-mutations.html).


### Mutations with Apollo

Calling mutations with the Apollo client follows the same approach as sending queries. We again have two options:

1. Wrapping the React component that should send the mutation with the `graphql` higher-order component 
2. Send a mutation directly using the `ApolloClient` and handle the return data in a promise

In contrast to Relay however, Apollo doesn't take care of cache consistency with every mutation that is sent, so we have to manually specify how Apollo should update the cache after the mutation has been performed. 

##### 1. Wrapping the component with `graphql`

First, we define a mutation with variables using the `gql` syntax and wrap a React component with `graphql` to later call the mutation:

```js
const createPokemonMutation = gql`
  mutation createPokemon($name: String!, $url: String!) {
    createPokemon(name: $name, url: $url) {
      id
      name
      url
    }
  }
`
const AddPokemonComponentWithMutation = graphql(createPokemonMutation)(AddPokemonComponent)
export default AddPokemonComponentWithMutation
```

When we used the same approach for fetching data, the query was sent behind the scenes and the returned data was made available to our component as a `data` object in its props. This time, since we're dealing with a mutation where we want keep control as to _when_ that mutation is fired, Apollo actually injects a new function called `mutate` into the component's props that we can use explicitly to send the mutation to the server:

```js
const {name, url} = this.state
this.props.mutate({variables: {name, url}})
  .then((data) => {
    console.log(data)
  })
```

However, to ensure cache consistency in that scenario, we have to configure our `AddPokemonComponentWithMutation` with an [`updateQueries`](http://dev.apollodata.com/react/cache-updates.html#updateQueries) object.

> From the Apollo [docs](http://dev.apollodata.com/react/cache-updates.html): _Most of the time, your UI will update automatically based on mutation results, as long as the object IDs in the result match up with the IDs you already have in your store. See the [`dataIdFromObject`](http://dev.apollodata.com/react/cache-updates.html#dataIdFromObject) documentation above for more information about how to take advantage of this feature. However, if you are removing or adding items to a list with a mutation or can’t assign object identifiers to some of your objects, you’ll have to use `updateQueries` to make sure that your UI reflects the change correctly._

In essence, the `updateQueries` object is a map from the name of a query (that we used previously to fetch some data) to a function that will specify how the new data that's returned from the mutation should be incorporated into the former result of the query. We thus avoid having to refetch the data for the query but instead specify how the cache should be changed after the mutation. The function that we specify follows the same principle as a [Redux reducer](http://redux.js.org/docs/basics/Reducers.html) in that it receives the previous result of the query and the mutation data and merges the two into the new query result.

Taking the example from above, we can add the `updateQueries` configuration like so:

```js
const AddPokemonComponentWithMutation = graphql(createPokemonMutation, {
  props({ ownProps, mutate }) {
    return {
      createPokemon({ variables }) {
        return mutate({
          variables: { ...variables },
          updateQueries: {
            allPokemonsQuery: (prev, { mutationResult }) => {
              const newPokemon = mutationResult.data.createPokemon
              return {
                ...prev,
                allPokemons: [...prev.allPokemons, newPokemon]
              }
            },
          },
        })
      },
    }
  },
})(AddPokemonComponent)
```

What happens here with `updateQueries` is that we tell the Apollo client to update all places where we used the `allPokemonsQuery` to also include the new pokemon. Once the mutation is sent and the data is returned from the server, the function we specified for the `allPokemonsQuery` will be called and the cache will be updated as specified. The resulting change is made available through the props of all components that were using data from that query and the UI will be rerendered.

Note that if we have multiple queries that are affected by a mutation, all of them need to included in `updateQueries` individually. For example, consider this query:

```js
const TrainerQuery = gql`
  query TrainerQuery {
    Trainer(name: "Ash Ketchum") {
      ownedPokemons
    }
  }
`
```

We would have to include a `TrainerQuery` as part of the returned object in `updateQueries` as well.

To find our more, you can read about other [advanced mutations](https://www.learnrelay.org/mutations/mutation-types), [managing Apollo Store](https://www.learnapollo.com/excursions/excursion-02) or read the [official mutation documentation](http://dev.apollodata.com/react/cache-updates.html).


##### 2. Using `ApolloCient` to directly send a mutation

Rather than making the mutation available through the props of a component, we can also use an instance of the `ApolloClient` to directly send a mutation. The code for that would look similar to sending a query but using the [`mutate`](http://dev.apollodata.com/core/apollo-client-api.html#ApolloClient\.mutate) method of the client:

```js
mutate({
  mutation: gql`
     mutation createPokemon($name: String!, $url: String!) {
       createPokemon(name: $name, url: $url) {
        id
        name
        url
      }
    }
  `,
  variables: { 
    name: "Zapdos", 
    url: "http://assets.pokemon.com/assets/cms2/img/pokedex/full/145.png" 
  },
  updateQueries: {
    allPokemonsQuery: (prev, { mutationResult }) => {
      const newPokemon = mutationResult.data.createPokemon
        return {
          ...prev,
          allPokemons: [...prev.allPokemons, newPokemon]
        }
      },
    }
})
.then(result => console.log(result))
```

Note that we can specify how Apollo should be updating the local cache after the mutation in the same way as before using `updateQueries`.


## Relay vs Apollo - Realtime updates with subscriptions

GraphQL offers the ability for clients to [subscribe](http://graphql.org/blog/subscriptions-in-graphql-and-relay/) to changes that are caused by mutations in a GraphQL backend. This allows the client to implement realtime functionality in an application and always keep the UI up to date with the current server-side state.

With Relay, there is not a lot of support for handling subscriptions on the client. You can use this [helper](https://github.com/taion/graphql-relay-subscription) package to ease up integration of subscriptions in Relay. Other than that there is not a lot of support that comes with Relay.

Apollo on the other hand offers a relatively sophisticated support for subscriptions through an additional package called [`subscriptions-transport-ws`](https://github.com/apollographql/subscriptions-transport-ws). If you're keen on learning more about how subscriptions work with the Apollo client, you can read up on it in our comprehensive [tutorial](!alias-ui0eizishe/) and checkout the [example project](https://demos.graph.cool/worldchat).
 






