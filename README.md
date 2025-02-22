# Anka Buildkite Plugin

A [Buildkite plugin](https://buildkite.com/docs/agent/v3/plugins) for running pipeline steps in [Anka](https://ankadoc.bitbucket.io/#introduction) virtual machines.

- By default, cloned VMs will be deleted on pipeline cancellation, failure, or success.
- At this time, this plugin does not automatically mount the `buildkite-agent` or inject any `BUILDKITE_` environment variables.
- A lock file (`/tmp/anka-buildkite-plugin-lock`) is created around pull and cloning. This prevents collision/ram state corruption when you're running two different jobs and pulling two different tags on the same anka node. The error you'd see otherwise is `state_lib/b026f71c-7675-11e9-8883-f01898ec0a5d.ank: failed to open image, error 2`

## Example

```yml
steps:
  - command: make test
    plugins:
      - chef/anka#v0.5.2:
          vm-name: macos-base-10.14
```

## Configuration

### `vm-name` (required)

The name of the Anka virtual machine to use as the base. The plugin will create a step-specific clone prior to execution.

Example: `macos-base-10.14`

### `vm-registry-tag` (optional)

A tag associated with the VM you wish to pull from the Anka Registry.

Example: `latest`

### `vm-registry-version` (optional)

A version associated with the VM you wish to pull from the Anka Registry.

Example: `1`

### `always-pull` (optional)

By default, the `anka-buildkite-plugin` will only pull the VM from the Anka Registry is missing. Set this value to `true` if you wish to pull the VM for every step.

- Should your registry be down and the pull fail, we will not fail the buildkite run. This prevents your registry from being a single point of failure for pipelines. We suggest monitoring for registry availability or failures.
- You can also use `"shrink"` to remove other local tags for the vm-name, optimizing the footprint.

Example: `true`

### `inherit-environment-vars` (optional)

Set this to `true` to inject the environment variables set on your host into the Anka VM.

Example: `true`

### `environment-file` (optional)

The path to a file containing environment variables you wish to inject into you Anka VM.

Example: `./my-env.txt`

### `no-volume` (optional)

Set this to `true` if you do not wish to mount the current directory into the Anka VM.

Example: `true`

### `volume` (optional)

The path to a directory, other than the current directory, you wish to mount into the Anka VM.

Example: `/some/directory`

### `wait-network` (optional)

Set this to `true` if you wish to delay the execution of your `command` until networking has been established in the Anka VM.

Example: `true`

### `workdir` (optional)

The fully-qualified path of the working directory inside the Anka VM.

Example: `/some/directory`

### `debug` (optional)

Set this to `true` to enable debug output within the plugin.

Example: `true`

### `anka-debug` (optional)

Set this to `true` to enable anka --debug output when running anka commands.

Example: `true`

### `cleanup` (optional)

Set this to `false` to leave the cloned images in a failed or complete build for investigation.
- You will need to run your buildkite agent with `cancel-grace-period=60`, as the [default 10 seconds is not enough time](https://forum.buildkite.community/t/problems-with-anka-plugin-and-pre-exit/365/7).

Example: `false`

### `bash-interactive` (optional)

This allows you to execute commands through anka run with an interactive shell (`anka run` does not support tty/interactive shell by default).

Example: `true`

### `pre-commands` (optional) (DANGEROUS)

Commands to run on the HOST machine BEFORE any guest/anka run commands. Useful if you need to download buildkite artifacts into the current working directory from a previous step. This can destroy your host. Be very careful what you do with it.

Example: `buildkite-agent artifact download "build.tar.gz" . --step ":aws: Amazon Linux 1 Build"`

### `post-commands` (optional) (DANGEROUS)

Commands to run on the HOST machine AFTER any guest/anka run commands. Useful if you need to upload artifacts created in the build/test process. This can destroy your host. Be very careful what you do with it.

Example: `buildkite-agent artifact upload "build.tar.gz"`

## Anka Modify ---

### `modify-cpu` (optional)

Will stop the VM, set CPU cores, and then execute commands you've specified.

Example: `6`

### `modify-ram` (optional)

Will stop the VM, set memory size, and then execute commands you've specified.

- Input is interpreted as G; if you input 32, it will use 32G in the anka modify command.

Example: `32`

## Anka Start ---

### `start-devices` (optional)

Will stop the VM, then start it with the USB device(s).

- Input should be the USB device ID/Location and should already be claimed.

Example: `341835776`

## License

|                      |                                          |
|:---------------------|:-----------------------------------------|
| **Author:**          | Tom Duffield (<tom@chef.io>)
| **Copyright:**       | Copyright 2018, Chef Software, Inc.
| **License:**         | Apache License, Version 2.0

```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
