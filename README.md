# Matrix config

[![example](https://github.com/jonlabelle/matrix-workflow-config/actions/workflows/example.yml/badge.svg)](https://github.com/jonlabelle/matrix-workflow-config/actions/workflows/example.yml)

> A reusable workflow for loading a job's [matrix strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs) from a JSON configuration file.

## Usage

See [matrix-config.yml](https://github.com/jonlabelle/matrix-workflow-config/blob/main/.github/workflows/matrix-config.yml).

## Examples

### Using the matrix config (reusable) workflow

Here's a basic example of how to load a matrix strategy from a config file.

#### Matrix config file

Define your matrix strategy in a config file.

```json
{
  "include": [
    {
      "label": "My first item",
      "server-name": "host01",
      "enable": true
    },
    {
      "label": "My second item",
      "server-name": "host02",
      "enable": true
    },
    {
      "label": "I am configured, but not enabled. I will not be run.",
      "server-name": "host03",
      "enable": false
    }
  ]
}
```

JSON comments (or JSONC) are fully supported in config files.All comments are
stripped before parsing.

> [!IMPORTANT]  
> GitHub only allows matrix configurations to be two-levels deep.
> Meaning, any item within the `include` array cannot have a value that is an
> object literal, or array.

For more information on configuring matrix strategies, see [GitHub's documentation](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs).

#### Job setup

Here's how you'll setup your job to call the matrix config reusable workflow,
and load the configuration into your matrix strategy.

```yaml
jobs:
  load-matrix:
    name: Load matrix
    uses: jonlabelle/matrix-workflow-config/.github/workflows/matrix-config.yml@main
    with:
      config: ./.github/matrix-config/example.json

  use-matrix:
    name: Use matrix on ${{ matrix.server-name }}
    needs: [load-matrix]
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJson(needs.load-matrix.outputs.matrix) }}
    steps:
      - name: ${{ matrix.label }}
        if: ${{ matrix.enable }}
        run: |
          echo "Server: ${{ matrix.server-name }}"
```

### Extend your config at runtime

You can extend your matrix config (at runtime) by passing in additional
key-value pairs that will bind to each item in the `include` array.

> [!NOTE]  
> You can run this example from the [actions](https://github.com/jonlabelle/matrix-workflow-config/actions/workflows/example.yml) section of this repo.

#### Job setup

> Here, we'll extend our config using the `extend` key, `'{ "existing-key": "overridden by extend input option", "prop2": "value2", "prop3": 3 }'`.

```yaml
jobs:
  load-matrix:
    name: Load matrix
    uses: ./.github/workflows/matrix-config.yml
    # Use the following path when calling from your own repo/workflow
    # uses: jonlabelle/matrix-workflow-config/.github/workflows/matrix-config.yml@main
    needs: [show-matrix-config-file]
    with:
      config: ./.github/matrix-config/example.json

      # Additional key/value pairs to bind to each item in matrix config's `include` array.
      # If a key already exists, its value will be overwritten (for each item in the include array).
      extend: '{ "existing-key": "overridden by extend input option", "prop2": "value2", "prop3": 3 }'

      # If the provided value for extend is a flat array (e.g. '["one", "two", "three"]'),
      # then the matrix is also assumed to be a flat array, and the new values will simply
      # be appended to the existed matrix array.
      # extend: '["one", "two", "three"]'

  use-matrix:
    name: Use matrix on ${{ matrix.server-name }}
    needs: [show-matrix-config-file, load-matrix]
    runs-on: [self-hosted, Linux, X64, onprem, enterprise]
    strategy:
      matrix: ${{ fromJson(needs.load-matrix.outputs.matrix) }}
    steps:
      - name: Show extended matrix config json
        if: ${{ matrix.enable }}
        run: echo '${{ needs.load-matrix.outputs.matrix }}' | jq '.' --color-output
```

#### Matrix config file (before extending)

Original matrix json config file contents.

```json
{
  "include": [
    {
      "label": "My first item",
      "server-name": "host01",
      "enable": true,
      "existing-key": "this value will be overridden by a value defined in the extend input option."
    },
    {
      "label": "My second item",
      "server-name": "host02",
      "enable": true,
      "existing-key": "this value will be overridden by a value defined in the extend input option."
    },
    {
      "label": "I am configured, but not enabled. I will not be run.",
      "server-name": "host03",
      "enable": false,
      "existing-key": "this value will be overridden by a value defined in the extend input option."
    }
  ]
}
```

#### Matrix config output (after extending)

Matrix json config after extending.

```json
{
  "include": [
    {
      "label": "My first item",
      "server-name": "host01",
      "enable": true,
      "existing-key": "overridden by extend input option",
      "prop2": "value2",
      "prop3": 3
    },
    {
      "label": "My second item",
      "server-name": "host02",
      "enable": true,
      "existing-key": "overridden by extend input option",
      "prop2": "value2",
      "prop3": 3
    },
    {
      "label": "I am configured, but not enabled. I will not be run.",
      "server-name": "host03",
      "enable": false,
      "existing-key": "overridden by extend input option",
      "prop2": "value2",
      "prop3": 3
    }
  ]
}
```
