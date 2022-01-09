## SCHEMA : DEFINIR LE FORMAT DE LA DATA

schema.js
```JS
import { gql } from "apollo-server"

export const typeDef = gql`
  type Query {
      tracksForHome: [Track!]!
    }

  type Track {
      id: Id!
      title: String!
      author: Author!
      thumbnail: String
      length: Int
      modulesCount: Int
  }

  type Author {
      id: ID!
      name: String!
      photo: String
    }
`
```

## SERVEUR : MISE EN PLACE DU SERVEUR APOLLO

index.js
```JS
const {ApolloServer} = require('apollo-server');
const typeDefs = require('./schema');

const server = new ApolloServer({typeDefs});

server.listen().then(() => {
    console.log(`
      🚀  Server is running!
      🔉  Listening on port 4000
      📭  Query at https://studio.apollographql.com/dev
    `);
  });
```
