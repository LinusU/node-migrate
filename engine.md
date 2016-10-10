# Engines

Engines are used for interfacing with the target a user wants to migrate, and
to store information on which migrations has been run.

When running a migration, e.g. `1-create-user-table`, the following functions
will be called in this order:

- `.migrationHasApplied('1-create-user-table')`
  - *if the promise returned resolves to true, no more action is taken*
- `.startMigration('1-create-user-table')`
- `.modify('CREATE TABLE user (...)')`
  - *This call is made by the actual migration. It could be any number of calls to `.view` and `.modify`.*
- `.recordMigration('1-create-user-table')`
- `.finishMigration('1-create-user-table')`

Any of these functions are free to reject the promise they return, and thus the
whole migration will be considered failed.

## API

An engine should expose the following api:

### `Constructor(...args)`

This is the main entry to the engine and is typically used for setting up the
connection.

### `.setupMigrationStore()`

Create any necessary prerequisites for storing applied migrations. This is
typically used for creating the migration table.

### `.migrationHasApplied(key: string) => Promise<boolean>`

Check whether a migration has been applied or not.

### `.recordMigration(key: string) => Promise<void>`

Record that a migration has been applied.

### `.startMigration(key: string) => Promise<void>`

Initiate a migration. This function is typically used to start a transaction.

### `.finishMigration(key: string) => Promise<void>`

Finish the migration. This function is typically used to commit the transaction.

### `.view(...args) => Promise<T>`

Run a query that shouldn't have side effects, and that are free to return data.

### `.modify(...args) => Promise<void>`

Run a query that will have side effects, but that cannot return data.

### `.formatQuery(...args)`

Return a string representation of the query, suitable for printing to stdout.

## Dry runs

When the main migrate binary is invoked with the `--dry-run` flag, calls to
`.modify` will instead be send to `.formatQuery` and logged to stdout.

They way `.view` and `.modify` is set up makes even migrations that looks at the
current data possible to run in dry-run mode.

## Example implementation

Below is a sample implementation for the Postgresql database.

```js
class PostgresEngine {
  constructor (client) {
    this.client = client
  }

  setupMigrationStore () {
    return this.client.query(`
      CREATE TABLE IF NOT EXISTS "migrations" (
        "key" text PRIMARY KEY,
        "applied_at" timestamptz DEFAULT transaction_timestamp()
      )
    `)
  }

  migrationHasApplied (key) {
    return this.client.query('SELECT COUNT(*) AS "count" FROM "migrations" WHERE "key" = $1', [key])
      .then(res => res.rows[0].count > 0)
  }

  recordMigration (key) {
    return this.client.query('INSERT INTO "migrations" ("key") VALUES ($1)', [key])
  }

  startMigration () {
    return this.client.query('START TRANSACTION')
  }

  finishMigration () {
    return this.client.query('COMMIT TRANSACTION')
  }

  view (sql, values) {
    return this.client.query(sql, values).then(res => res.rows)
  }

  modify (sql, values) {
    return this.client.query(sql, values)
  }

  formatQuery (sql, values) {
    return `${sql}\n${values.map((val, idx) => `$${idx + 1}: ${val}`)}`
  }
}
```
