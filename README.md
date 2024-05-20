# Matrix config

[![example](https://github.com/jonlabelle/matrix-workflow-config/actions/workflows/example.yml/badge.svg)](https://github.com/jonlabelle/matrix-workflow-config/actions/workflows/example.yml)

> A reusable workflow for loading a job's [matrix strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs) from a JSON configuration file.

## Example usage

Here's a basic example of how to load a matrix strategy from a config file.

### Configuration file

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
> GitHub only allows matrix configuration to be two-levels deep.
> Meaning, any key under `include` cannot have a value that is an object or array.

For more information on configuring matrix strategies, see [GitHub's official documentation](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs).

### Calling the reusable workflow

Here's how you call the reusable workflow from your own workflow, and load the
configuration into your matrix strategy.

```yaml
name: example

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  load-matrix:
    name: Load matrix
    uses: jonlabelle/matrix-workflow-config/.github/workflows/matrix-config.yml@main
    with:
      config: ./.github/matrix-config/example.json

      # Uncomment `no-sparse-checkout` below to disable sparse checkout of
      # your config file only, and checkout the entire repository
      #no-sparse-checkout: true

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

## License

[MIT](LICENSE)
