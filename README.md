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
