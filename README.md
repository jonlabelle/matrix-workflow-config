# Matrix config

> A reusable workflow for loading a job's [matrix strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs) from a JSON configuration file.

## Example usage

Here's a basic example of how to load a matrix strategy from a config file.

### Configuration file

Define your matrix strategy in a config file.

```json
{
  "include": [
    {
      "label": "My first deploy target",
      "server-name": "server.name1",
      "root-unc-path": "\\\\server.name\\E$\\www\\public",
      "local-root-path": "E:\\www\\public",
      "iis-path": "IIS:\\Sites\\Default Web Site",
      // Use this setting to enable/disable a job step
      "enable": true
    },
    {
      // This label can be used a step heading
      "label": "My second deploy target",
      "server-name": "server.name2",
      "root-unc-path": "\\\\server.name2\\Z$\\www\\public\\sub-directory",
      "local-root-path": "Z:\\www\\public\\sub-directory",
      "iis-path": "IIS:\\Sites\\Mysite\\MyApp",
      "enable": true
    },
    {
      "label": "I am configured, but not enabled. So I will not be run.",
      "server-name": "server.name3",
      "root-unc-path": "\\\\server.name3\\Q$\\www\\public\\broken",
      "local-root-path": "Q:\\www\\public\\broken",
      "iis-path": "IIS:\\Sites\\Broken site",
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
name: Example using matrix config

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  load-matrix:
    uses: jonlabelle/matrix-workflow-config/.github/workflows/matrix-config.yml@main
    with:
      config: ./.github/matrix-config/example.json

  use-matrix:
    name: Use matrix config
    needs: [load-matrix]
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJson(needs.load-matrix.outputs.matrix) }}
      fail-fast: false
    steps:
      - run: echo ${{ matrix.label }}
        if: ${{ matrix.enable }}
      - run: echo ${{ matrix.server-name }}
        if: ${{ matrix.enable }}
      - run: echo ${{ matrix.root-unc-path }}
        if: ${{ matrix.enable }}
      - run: echo ${{ matrix.local-root-path }}
        if: ${{ matrix.enable }}
      - run: echo ${{ matrix.iis-path }}
        if: ${{ matrix.enable }}
      - run: echo ${{ matrix.enable }}
        if: ${{ matrix.enable }}
```

## License

[MIT](LICENSE)
