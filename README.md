# Minimal - run <task> action

A Github Action that executes a `minimal.toml`-defined task on a Linux runner.

## Features

 - Runs on any Linux runner that supports unprivileged user namespaces.
 - Supports both x86_64 and arm64 architectures.
 - Selectable release channel

## Usage

The following example steps run the `hello-world` task on the Github runner, using a version
of Minimal fetched from the `unstable` channel.

```yaml
 - uses: actions/checkout@v7 # Checkout the repo to access its `minimal.toml`
 - name: Run hello-world task
   uses: gominimal/run-task@v1
   with:
     channel: unstable
     task: hello-world
```

### Input parameters

 - `task` _(required)_: Name of the task to run
 - `channel`: Release channel to fetch Minimal from; the channel is resolved to an exact version at run time. Defaults to `stable`.

### Outputs

 - `version`: Exact version string of Minimal which was used

## Known-supported runners

We expect most distributions with modern (2024+) kernels to be able to run Minimal.

### Github-hosted

 * Ubuntu: `ubuntu-slim`, `ubuntu-latest`, `ubuntu-26.04`, `ubuntu-24.04`, `ubuntu-26.04-arm`, `ubuntu-24.04-arm`

### Self-hosted

 * Ubuntu: Version 24 or greater. Unprivileged user namespaces must be enabled.
   
   If unprivileged user namespaces are not enabled, the Github action will attempt to enable them using passwordless `sudo`.
   For runners which are not permitted to use sudo, you will need to set the appropriate sysctl yourself:
   
   ```shell
   sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0
   ```

 * Debian: Bullseye (v11), Bookworm (v12), Trixie (v13) or greater.
