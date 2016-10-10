# Migrations

Your migration should expose the following api

## API

### `.up(engine) => Promise<void>`

Apply this migration

### `.down(engine) => Promise<void>`

Revert this migration

## `engine`

The engine passed will expose to methods, one for viewing data and one for
modifying.

### `.view(...args) => Promise<T>`

Use this function to view current data.

### `.modify(...args) => Promise<void>`

Use this function make modifications. When running in dry-run mode, this will
just print the query.

## Example

A simple example using Postgresql.

```js
exports.up = function (pg) {
  return pg.mofidy(`
    CREATE TABLE "user" (
      "id" SERIAL PRIMARY KEY,
      "firstName" text NOT NULL,
      "lastName" text NOT NULL
    )
  `)
}

exports.down = function (pg) {
  return pg.modify(`
    DROP TABLE "user"
  `)
}
```
