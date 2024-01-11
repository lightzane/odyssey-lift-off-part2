# Catstronauts - server

The starting point of the `server` code for Odyssey Lift-off II course.

## REST API Server

- https://odyssey-lift-off-rest-api.herokuapp.com/

## Implementing RESTDataSource

```bash
npm install @apollo/datasource-rest
```

## GraphQL Codegen

```bash
npm install -D @graphql-codegen/cli @graphql-codegen/typescript @graphql-codegen/typescript-resolvers
```

**package.json**

```diff
{
  "scripts": {
    "compile": "tsc",
    "dev": "ts-node-dev --respawn ./src/index.ts",
    "start": "npm run compile && nodemon ./dist/index.js",
+   "generate": "graphql-codegen"
  }
}
```

Create `codegen.ts`

```ts
import type { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: './src/schema.ts',
  generates: {
    './src/types.ts': {
      plugins: ['typescript', 'typescript-resolvers'],
    },
  },
};

export default config;
```

```bash
lightzane@JPs-MacBook-Air server % npm run generate

> catstronauts-server@1.0.0 generate
> graphql-codegen

✔ Parse Configuration
✔ Generate outputs
```

Create `context.ts`

```ts
import { TrackAPI } from './datasources/track-api';

export type DataSourceContext = {
  dataSources: {
    trackAPI: TrackAPI;
  };
};
```

```diff
import type { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: './src/schema.ts',
  generates: {
    './src/types.ts': {
      plugins: ['typescript', 'typescript-resolvers'],
+     config: {
+       contextType: './context#DataSourceContext',
+     },
    },
  },
};

export default config;
```

## Adding a Model type

First, let's talk about what's going wrong with the `authorId`. If you hover over it, you'll see the following error.

```
Property authorId does not exist on type Track.
```

And if we reference the `Track` type in our `schema.ts` file... we'll see that this error is, in fact, correct! Our `Track` type doesn't actually have an `authorId` field. Instead, it has an `author` field.

```gql
type Track {
  id: ID!
  "The track's title"
  title: String!
  "The track's main author"
  author: Author!
  "The track's main illustration to display in track card or track page detail"
  thumbnail: String
  "The track's approximate length to complete, in minutes"
  length: Int
  "The number of modules this track contains"
  modulesCount: Int
}
```

Create new `models.ts` file

```ts
export type TrackModel = {
  id: string;
  title: string;
  authorId: string;
  thumbnail: string;
  length: number;
  modulesCount: number;
};
```

```diff
import type { CodegenConfig } from '@graphql-codegen/cli';

const config: CodegenConfig = {
  schema: './src/schema.ts',
  generates: {
    './src/types.ts': {
      plugins: ['typescript', 'typescript-resolvers'],
      config: {
        contextType: './context#DataSourceContext',
+       mappers: {
+         Track: './models#TrackModel',
+       },
      },
    },
  },
};
```

Execute `npm run generate`

> Also update the TrackAPI
