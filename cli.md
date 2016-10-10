# CLI

The node-migrate cli.

## Usage

```text
migrate [configuration] [command]
```

### Configuration

The migrate command takes two parameters affecting the configuration.

- `--engine` - Identifier of the engine to use
- `--config` - Filename of the configuration file

The `engine` option specifies which engine to use, e.g. `pg` to use the
`migrate-engine-pg` package.

The `config` option specifies which file to read the config from. This file can
have be one of three things:

1. A JSON file with the config
1. A JavaScript file with the config as the `module.exports`
1. A JavaScript file with the `Promise` of the config as the `module.exports`

### Commands

The following commands can be run.

#### `up`

Apply all migrations that haven't yet been applied.

#### `down`

Revert the last applied migration.

#### `create <name>`

Create a new migration with the specified name.
