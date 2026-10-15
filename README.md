# Minimal - run <task> action

A Github Action that executes a `minimal.toml`-defined task on a Linux runner
with `min task run`: it installs `min` and `minimald` from the release
channel, and runs the task in an ephemeral session that is destroyed when the
task exits.

## Features

 - Runs on any Linux runner that supports unprivileged user namespaces.
 - Supports both x86_64 and arm64 architectures.
 - Selectable release channel
 - Tasks that inherit environment variables, through the `allow-env` input

## Usage

The following example steps run the `hello-world` task on the Github runner.

```yaml
 - uses: actions/checkout@v7 # Checkout the repo to access its `minimal.toml`
 - name: Run hello-world task
   uses: gominimal/run-task@v1
   with:
     task: hello-world
```

### Input parameters

 - `task` _(required)_: Name of the task to run
 - `channel`: Release channel to install Minimal from (`stable`, `unstable`, `nightly`); the channel is resolved to an exact version at run time. Defaults to `stable`.
 - `allow-env`: Environment variable names (one per line, globs allowed) that the task's `env_vars` may inherit from the runner. `min task run` checks every declared name against the runner user's `~/.config/minimal/user_policy.toml`, and without a terminal it fails on any name that file does not allow, so a task declaring `env_vars.FOO = { inherit = true }` needs `FOO` here. The action writes that file and refuses to overwrite one that already exists (self-hosted runners manage their own).

   ```yaml
    - uses: gominimal/run-task@v2
      env:
        FOO: ${{ secrets.FOO }}
      with:
        task: deploy
        allow-env: |
          FOO
   ```

### Outputs

 - `version`: Exact version string of Minimal which was used

### What a task run can and cannot do

The task runs in a session composed from the project's `minimal.toml`: its
`packages`, `patches`, `vars` and `state_key` apply as documented in the
[tasks reference](https://minimal.dev/docs/reference/tasks). The session sees
an uploaded **copy** of the checkout, so:

 - Files the task writes stay in the session and are destroyed with it. A task
   is useful here for its exit code and its output, not for files it produces.
 - There is no host-side `materialize` or `dump`; those remain `mip` commands.
 - Each run starts with an empty `/state`, so a `state_key` cache does not
   carry across runs ([gominimal/minimal#1729](https://github.com/gominimal/minimal/issues/1729)).

The action resolves the channel pointer at `storage.googleapis.com/minimal-one/<channel>`
for its `version` output and installs through `go.minimal.dev/<channel>`, the
same installer users run.

## Known-supported runners

We expect most distributions with modern (2024+) kernels to be able to run Minimal.

### Github-hosted

 * Ubuntu: `ubuntu-slim`, `ubuntu-latest`, `ubuntu-26.04`, `ubuntu-24.04`, `ubuntu-26.04-arm`, `ubuntu-24.04-arm`

### Self-hosted

 * Ubuntu: Version 24 or greater. Unprivileged user namespaces must be enabled.
   
   If unprivileged user namespaces are not enabled, the Github action will attempt to enable them using passwordless `sudo` (the one host change `min finalize-install` makes that a task run needs).
   For runners which are not permitted to use sudo, you will need to set the appropriate sysctl yourself:
   
   ```shell
   sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0
   ```

 * Debian: Bullseye (v11), Bookworm (v12), Trixie (v13) or greater.
