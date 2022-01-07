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

const [addTodo, { data, loading, error }] = useMutation(ADD_TODO);

  return (
      <button
        onSubmit={e => {
          e.preventDefault();
          addTodo({ variables: { text: input.value } });
          input.value = '';
        }}
      >
        Add todo
      </button>
  );
```

On peut également passer en deuxieme paramètre les variables
```TS
const [addCount, { data, loading, error }] = useMutation(INCREMENT_COUNTER, {
  variables: {
    text: input.value
  },
});
```
