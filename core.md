# Core

This document outlines the programmatic API for node-migrate.

## API

### `new Migrate(engine: EngineConstructor, configuration: object)`

Creates a new `Migrate` instance.

### `.initializeEngine() => Promise<void>`

Initialize the engine, typically consisting of creating the migration table.

### `.allUp() => Promise<void>`

Apply all migrations that have yet to be applied.

### `.oneDown() => Promise<void>`

Revert the latest migration.
