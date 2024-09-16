# ComposableQuery

**Create custom SQL building blocks**<br />
A lightweight library for building parameterized SQL queries from modular parts

- [Installation](#installation)
- [Example](#example)

## Installation

```sh
$ npm install @anderjason/composablequery
```

## Example

```typescript
import { ComposableQuery } from "@anderjason/composablequery";

const conditionPart = new ComposableQuery({
  sql: `state = $1 AND type = $2`
  params: ['California', 'Post Office']
});

const selectQuery = new ComposableQuery({
  sql: `SELECT * FROM locations WHERE $1 AND is_deleted = $2`
  params: [conditionPart, false]
});

const flatQuery = selectQuery.toFlatQuery();
```

The `flatQuery` value in the example above is set to:

```typescript
{
  sql: "SELECT * FROM locations WHERE state = $1 AND type = $2 AND is_deleted = $3",
  params: ['California', 'Post Office', false]
}
```

### Tokens

Tokens in your SQL query are placeholders for parameters. For instance, `$1` is replaced by the first value in the parameters array, `$2` by the second value, and so on.

This separation of SQL text and parameters helps prevent SQL injection attacks.

```typescript
import { ComposableQuery } from "@anderjason/composablequery";

const query = new ComposableQuery({
  sql: `SELECT * FROM people WHERE status = $1 AND company_id = $2`
  params: ['active', 'company123']
});
```

You can reuse the same token multiple times in the same query. The following query uses the token `$1` twice:

```typescript
const query = new ComposableQuery({
  sql: `SELECT * FROM locations WHERE city = $1 OR display_name = $1`
  params: ['San Francisco']
});
```

The number of unique tokens in a query must match the number of parameters. The following query is invalid because it has two tokens `$1` and `$2` but only one parameter:

```typescript
const query = new ComposableQuery({
  sql: `SELECT * FROM locations WHERE city = $1 OR display_name = $2`
  params: ['San Francisco']
});
```

### Composition

You can combine ComposableQuery objects to build more complex queries.

For example:

```typescript
import { ComposableQuery } from "@anderjason/composablequery";

// conditionPart is only a portion of a SQL query,
// intended to be embedded in another part
const conditionPart = new ComposableQuery({
  sql: `state = $1 AND type = $2`
  params: ['California', 'Post Office']
});

// selectQuery is the root query
const selectQuery = new ComposableQuery({
  sql: `SELECT * FROM locations WHERE $1 AND is_deleted = $2`
  params: [conditionPart, false]
});
```

In this example, conditionPart is a subquery that is included in the main selectQuery. The selectQuery embeds conditionPart using the token $1. Queries can be nested as needed.

To get a single SQL string and parameter list, use the toFlatQuery method:

```typescript
const flatQuery = selectQuery.toFlatQuery();
```

This method returns an object like this:

```typescript
{
  sql: "SELECT * FROM locations WHERE state = $1 AND type = $2 AND is_deleted = $3",
  params: ['California', 'Post Office', false]
}
```

### Supported SQL

ComposableQuery doesn't validate SQL syntax, so you can use any SQL that is compatible with your database engine. Ensure your SQL is valid for the engine you are using.


### Executing queries

ComposableQuery does not handle query execution. You need to use a database client library, such as [pg](https://www.npmjs.com/package/pg), to run your query:

```typescript
import { ComposableQuery } from "@anderjason/composablequery";
import { Pool } from "pg";

const pool = new Pool();

const condition = new ComposableQuery({
  sql: `state = $1 AND type = $2`
  params: ['California', 'Post Office']
});

const selectQuery = new ComposableQuery({
  sql: `SELECT * FROM locations WHERE $1 AND is_deleted = $2`
  params: [condition, false]
});

const flatQuery = selectQuery.toFlatObject();

pool.query(flatQuery.sql, flatQuery.params, (err, res) => {
  if (err) {
    // error
    console.log(err.stack);
  } else {
    // success
    console.log(res.rows);
  }
});
```

### Supported parameter types

The following parameter types are supported:

- `string`
- `string[]`
- `number`
- `number[]`
- `boolean`
- `Buffer`
- `null`
- `undefined`
- `ComposableQuery` (see [Composition](#composition))

## License

[MIT License](https://tldrlegal.com/license/mit-license)
