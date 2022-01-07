# Apollo

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

## Update query cache

Après une mutation il est possible de faire un update da la query dans le cache afin d'éviter un refetch vers le serveur

```TS
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
Autre exemple plus générique :
```TS
// Query to fetch all todo items
const query = gql`
  query MyTodoAppQuery {
    todos {
      id
      text
      completed
    }
  }
`;

// Set all todos in the cache as completed
cache.updateQuery({ query }, (data) => ({
  todos: data.todos.map((todo) => ({ ...todo, completed: true }))
}));
```
