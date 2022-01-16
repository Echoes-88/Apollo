# Apollo

## Query
```JS
import gql from 'graphql-tag';
import { useQuery } from '@apollo/react-hooks';

const GET_DOGS = gql`
  {
    dogs {
      id
      breed
    }
  }
`;
```

```JS
function Dogs({ onDogSelected }) {
  const { loading, error, data } = useQuery(GET_DOGS);

  if (loading) return 'Loading...';
  if (error) return `Error! ${error.message}`;

  return (
    <select name="dog" onChange={onDogSelected}>
      {data.dogs.map(dog => (
        <option key={dog.id} value={dog.breed}>
          {dog.breed}
        </option>
      ))}
    </select>
  );
}
```

**Caching query results**

Whenever Apollo Client fetches query results from your server, it automatically caches those results locally. This makes subsequent executions of the same query extremely fast.

## Local State Management

A la manière de redux il est possible d'utiliser le cache Apollo comme un store et d'y stocker du contenu

Source : https://www.apollographql.com/docs/react/v2/data/local-state/

**Initializing the cache**

Often, you'll need to write an initial state to the cache so any components querying data before a mutation is triggered don't error out.

```JS
import { ApolloClient } from 'apollo-client';
import { InMemoryCache } from 'apollo-cache-inmemory';

const cache = new InMemoryCache();
const client = new ApolloClient({
  cache,
  resolvers: { /* ... */ },
});

cache.writeData({
  data: {
    todos: [],
    visibilityFilter: 'SHOW_ALL',
    networkStatus: {
      __typename: 'NetworkStatus',
      isConnected: false,
    },
  },
});
```

**Reset store with onResetStore**

Sometimes you may need to reset the store in your application, when a user logs out for example. See below of this code.

```JS
import { ApolloClient } from 'apollo-client';
import { InMemoryCache } from 'apollo-cache-inmemory';

const cache = new InMemoryCache();
const client = new ApolloClient({
  cache,
  resolvers: { /* ... */ },
});

const data = {
  todos: [],
  visibilityFilter: 'SHOW_ALL',
  networkStatus: {
    __typename: 'NetworkStatus',
    isConnected: false,
  },
};

cache.writeData({ data });

client.onResetStore(() => cache.writeData({ data }));
```

**Write in cache**

```JS
import React from "react";
import { useApolloClient } from "@apollo/react-hooks";

import Link from "./Link";

function FilterLink({ filter, children }) {
  const client = useApolloClient();
  return (
    <Link
      onClick={() => client.writeData({ data: { visibilityFilter: filter } })}
    >
      {children}
    </Link>
  );
}
```

**Get data from cache**

Querying for local data is very similar to querying your GraphQL server. The only difference is that you add a @client directive on your local fields to indicate they should be resolved from the Apollo Client cache or a local resolver function. Let's look at an example:

```JS
import React from "react";
import { useQuery } from "@apollo/react-hooks";
import gql from "graphql-tag";

import Todo from "./Todo";

const GET_TODOS = gql`
  {
    todos @client {
      id
      completed
      text
    }
    visibilityFilter @client
  }
`;

function TodoList() {
  const { data: { todos, visibilityFilter } } = useQuery(GET_TODOS);
  return (
    <ul>
      {getVisibleTodos(todos, visibilityFilter).map(todo => (
        <Todo key={todo.id} {...todo} />
      ))}
    </ul>
  );
}
```

## Mutations
```TS
const ADD_TODO = gql`
  mutation AddTodo($text: String!) {
    addTodo(text: $text) {
      id
      text
    }
  }
`;

export const Mutation: React.FC<Props> = () => {

const [addTodo, { data, loading, error }] = useMutation(ADD_TODO);

  return (
      <button
        onClick={() => 
          addTodo({ variables: {id: "2212b", text: "nouveau texte" } });
        }
      >
        Add todo
      </button>
  );
  
}
```

On peut également passer en deuxieme paramètre les variables
```TS
const [addCount, { data, loading, error }] = useMutation(INCREMENT_COUNTER, {
  variables: {
    id,
    text: "nouveau texte"
  },
});
```

## MISE A JOUR DU FRONT APRES UNE MUTATION

Il existe plusieurs méthodes pour mettre à jour le front suite à la mutation (modification du backend)

- En effectuant un refetch 

```TS
// Refetches two queries after mutation completes
const [addTodo, { data, loading, error }] = useMutation(ADD_TODO, {
  refetchQueries: [
    GET_POST, // DocumentNode object parsed with gql
    'GetComments' // Query name
  ],
});
````

- En faisant un update du cache directement (évite une requête vers le serveur)

```TS

// Mutation + update du cache

const GET_TODOS = gql`
  query GetTodos {
    todos {
      id
    }
  }
`;

function AddTodo() {
  let input;
  const [addTodo] = useMutation(ADD_TODO, {
    update(cache, { data: { addTodo } }) {
      cache.modify({
        fields: {
          todos(existingTodos = []) {
            const newTodoRef = cache.writeFragment({
              data: addTodo,
              fragment: gql`
                fragment NewTodo on Todo {
                  id
                  type
                }
              `
            });
            return [...existingTodos, newTodoRef];
          }
        }
      });
    }
  });
```

OU

```TS

// update du cache seul si la mutation est effectué ailleurs

    const updateCache = (user: Unpack<GetUsersQuery['users']['data']>) => {
        client.cache.updateQuery<GetUsersQuery, GetUsersQueryVariables>(
            { query: GetUsersDocument, variables: { filter } },
            (cached) => {
                if (cached === null) return;
                return {
                    ...cached,
                    users: {
                        ...cached.users,
                        count: cached.users.count + 1,
                        data: [...cached.users.data, user]
                    }
                };
            }
        );
    };
```

! Parfois on veut s'assurer que le refetch a bien été effectué partout ou necessaire : voir "Refetching after update" dans la doc
