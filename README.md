# provision-with-micromamba

[![test](https://github.com/mamba-org/provision-with-micromamba/workflows/test/badge.svg)](https://github.com/mamba-org/provision-with-micromamba/actions?query=workflow%3Atest)

GitHub Action to provision a CI instance using micromamba.

## Dependencies

`provision-with-micromamba` requires the `curl` and `tar` programs (with `bzip2` support).
They are preinstalled in the default GitHub Actions environments.

## Inputs

<!-- generated by generate-inputs-docs.js -->

### `environment-file`

Required. The 'environment.yml' or '.lock' file for the Conda environment. If 'false', only `extra-specs` will be considered and you should provide 'channels'. If both 'environment-file' and 'extra-specs' are empty, no enviroment will be created (only Micromamba will be installed). See the [Conda documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file) for more information.

Default value: "environment.yml"

### `environment-name`

The name of the Conda environment. Defaults to name from the environment.yml file. Required if 'environment-file' is a '.lock' file or 'false'.

### `micromamba-version`

Version of micromamba to use, eg. '0.20'. See https://github.com/mamba-org/mamba/releases/ for a list of releases.

Default value: "latest"

### `extra-specs`

Additional specifications (packages) to install.
Pretty useful when using matrix builds to pin versions of a test/run dependency.
For multiple packages, use multiline syntax:
```yaml
extra-specs: |
  python=3.10
  xtensor
```
Note that selectors
(e.g. `sel(linux): my-linux-package`, `sel(osx): my-osx-package`, `sel(win): my-win-package`)
are available.


### `channels`

Comma separated list of channels to use in order of priority (eg., `conda-forge,my-private-channel`)

### `condarc-file`

Path to a `.condarc` file to use. See the [Conda documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/configuration/) for more information.

### `channel-priority`

Channel priority to use. One of "strict", "flexible", and "disabled". See https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-channels.html#strict-channel-priority for more information.

Default value: "strict"

### `cache-downloads`

If 'true', cache downloaded packages across calls to the provision-with-micromamba action. Cache invalidation can be controlled using the 'cache-downloads-key' option.

### `cache-downloads-key`

Custom download cache key used with 'cache-downloads: true'. The default download cache key will invalidate the cache once per day.

### `cache-env`

If 'true', cache installed environments across calls to the provision-with-micromamba action. Cache invalidation can be controlled using the 'cache-env-key' option.

### `cache-env-key`

Custom environment cache key used with 'cache-env: true'. With the default environment cache key, separate caches will be created for each operating system (eg., Linux) and platform (eg., x64) and day (eg., 2022-01-31), and the cache will be invalidated whenever the contents of 'environment-file' or 'extra-specs' change.

### `log-level`

(Optional, default "info") Micromamba log level to use. One of "trace", "debug", "info", "warning", "error", "critical", "off".

### `installer-url`

Base URL to fetch Micromamba from. Files will be downloaded from `<base url>/<platform>/<version>`, eg. https://micro.mamba.pm/api/micromamba/linux-64/latest.

Default value: "https://micro.mamba.pm/api/micromamba"

### `condarc-options`

More options to append to `.condarc`. Must be a string of valid YAML:

```yaml
condarc-options: |
  proxy_servers:
    http: ...
```


<!-- end generated -->

## Example usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Install Conda environment from environment.yml
        uses: mamba-org/provision-with-micromamba@main

      # Linux and macOS
      - name: Run Python
        shell: bash -l {0}
        run: |
          python -c "import numpy"

      # Windows
      # With Powershell:
      - name: Run Python
        shell: powershell
        run: |
          python -c "import numpy"
      # Or with cmd:
      - name: Run cmd.exe
        shell: cmd /C CALL {0}
        run: >-
          micromamba info && micromamba list
```

> **Please** see the **[IMPORTANT](#IMPORTANT)** notes on additional information
> on environment activation.

## Example with customization

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        pytest: ["6.1", "6.2"]
    steps:
      - uses: actions/checkout@v2

      - name: Install Conda environment with Micromamba
        uses: mamba-org/provision-with-micromamba@main
        with:
          environment-file: myenv.yaml
          environment-name: myenvname
          extra-specs: |
            python=3.7
            pytest=${{ matrix.pytest }}
```

## Example with download caching

Use `cache-downloads` to enable download caching across action runs (`.tar.bz2` files).

By default the cache is invalidated once per day. See the `cache-downloads-key` option for custom cache invalidation.

```yaml
- name: Install Conda environment with Micromamba
  uses: mamba-org/provision-with-micromamba@main
  with:
    cache-downloads: true
```

## Example with environment caching

Use `cache-env` to cache the entire Conda environment (`envs/myenv` directory) across action runs.

By default the cache is invalidated whenever the contents of the `environment-file`
or `extra-specs` change, plus once per day. See the `cache-env-key` option for custom cache invalidation.

```yaml
- name: Install Conda environment with Micromamba
  uses: mamba-org/provision-with-micromamba@main
  with:
    cache-env: true
```

## Notes on caching

### Branches have separate caches

Due to a [limitation of GitHub Actions](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#restrictions-for-accessing-a-cache)
any download or environment caches created on a branch will not be available on the main/parent branch
after merging. This also applies to PRs.

In contrast, branches *can* use a cache created on the main/parent branch. 

See also [this thread](https://github.com/mamba-org/provision-with-micromamba/issues/42#issuecomment-1062007161).

### When to use download caching

Please see [this comment for now](https://github.com/mamba-org/provision-with-micromamba/pull/38#discussion_r808837618).

### When to use environment caching

Please see [this comment for now](https://github.com/mamba-org/provision-with-micromamba/pull/38#discussion_r808837618).

## More examples

More examples may be found in this repository's [tests](.github/workflows).

## Reference

See [action.yml](./action.yml).

## IMPORTANT

Some shells require special syntax (e.g. `bash -l {0}`). You can set this up with the `default` option:

```yaml
jobs:
  myjob:
    defaults:
      run:
        shell: bash -l {0}

# Or top-level:
defaults:
  run:
    shell: bash -l {0}
jobs:
  ...
```

Find the reasons below (taken from [setup-miniconda](https://github.com/conda-incubator/setup-miniconda/blob/master/README.md#important)):

- Bash shells do not use `~/.profile` or `~/.bashrc` so these shells need to be
  explicitely declared as `shell: bash -l {0}` on steps that need to be properly
  activated (or use a default shell). This is because bash shells are executed
  with `bash --noprofile --norc -eo pipefail {0}` thus ignoring updated on bash
  profile files made by `conda init bash`. See
  [Github Actions Documentation](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#using-a-specific-shell)
  and
  [thread](https://github.community/t5/GitHub-Actions/How-to-share-shell-profile-between-steps-or-how-to-use-nvm-rvm/td-p/33185).
- Cmd shells do not run `Autorun` commands so these shells need to be
  explicitely declared as `shell: cmd /C call {0}` on steps that need to be
  properly activated (or use a default shell). This is because cmd shells are
  executed with `%ComSpec% /D /E:ON /V:OFF /S /C "CALL "{0}""` and the `/D` flag
  disabled execution of `Command Processor/Autorun` Windows registry keys, which
  is what `conda init cmd.exe` sets. See
  [Github Actions Documentation](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#using-a-specific-shell).
- `sh` is not supported. Please use `bash`.

## Development

When developing, you need to

1. install `nodejs`
2. clone the repo
3. run `npm install -y`
4. run `npm run build` after making changes
