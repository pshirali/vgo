# vGo


## Introduction


vGo is a simple, yet powerful virtual workspace manager for Golang, implemented as a single-file bash script. In a nutshell it provides the following features:

1. Isolation: All configuration is done in a bash subshell. Your invoking shell remains untouched.

2. Define a project-path per workspace (Example: `github.com/johndoe/foo`). Jump to `<workspace>/src/github.com/johndoe/foo` on workspace activation.

3. Convenience commands to chdir between important subfolders within the workspace. Command names are configurable. Sane defaults.

4. Option of sourcing two user-defined bashrc extension files: (1) applies across all workspaces, (2) per-workspace bashrc. Commands (implemented as bash functions) including vGo commands are overridable.

5. Minimal sub-commands. Single character aliases.


## Installation

vGo requires a POSIX platform with bash. It officially supports `Linux` and `OSX`. Place the `vgo` script in any folder which is part of your `PATH`.

```
sudo wget https://raw.githubusercontent.com/pshirali/vgo/0.0.1/vgo -O /usr/local/bin/vgo
sudo chmod +x /usr/local/bin/vgo
```

## Initial Setup

vGo expects you to place all your workspaces under a single parent folder. Before you use vGo, you need to set the environment variable `VGO_HOME` to point to this folder. When you start using vGo, your folder structure will look like this:

```
<VGO_HOME>
  \- <workspace-1>
      \- bin/
      \- pkg/
      \- src/
  \- <workspace-2>
      \- bin/
      \- pkg/
      \- src/
  ...
```

In this document, we'll consider this folder to be `$HOME/vgo`. Create this folder if it doesn't exist. Place the following line in your `.bashrc`

```
export VGO_HOME=$HOME/vgo
```

vGo also needs your `.bashrc` file when it attempts to creates a bash subshell. vGo creates a local copy of this file inside the workspace folder and adds additinal content to the copy. This copy is generated and sourced each time the workspace is activated.

To cater to both `OSX` and `Linux`, the `.bashrc` is sought in the following order. The last found file is chosen.

1. `$HOME/.bash_profile`
1. `$HOME/.bashrc`

You can also explicitly set the `bashrc` file you wish to use by setting the env variable `VGO_BASHRC_FILE`.

```
export VGO_BASHRC_FILE=<your-bash-rc-file>
```

## Command Reference

#### `vgo w [workspace-name] [project-path]`

The `vgo w | vgo workon` command is used to create as well as activate a workspace. Calling `vgo w` without arguments will list the current workspaces inside `VGO_HOME`.

*Usage Example:*
```
> vgo w foo github.com/johndoe/foo
```

1. With `foo` as the `workspace-name`, vGo creates/verifies the presence of the following folders:
```
$VGO_HOME/foo/bin
$VGO_HOME/foo/pkg
$VGO_HOME/foo/src
```

2. A copy of your `bashrc` is placed inside `$VGO_HOME/foo` as `.vgo.bashrc`. All vGo configuration is append to this file. This includes setting `PATH` to `$VGO_HOME/foo/bin:$PATH`, setting `GOPATH` to `$VGO_PATH/foo` and code to configure all other commands.

3. The `project-path` is stored into the file `$VGO_HOME/foo/.vgo.proj`. If the `project-path` doesn't exist, folders upto the dirname of the `project-path` are recursively created inside the `$VGO_HOME/src` folder. In the above example, this would be `$VGO_HOME/src/github.com/johndoe`.

> When the `project-path` folder doesn't exist, vGo doesn't know whether you intend to create a new project or pull existing code from a remote repo. The recursive folder creation till the dirname of the `project-path` allows you to perform the last mile change yourself. From this point, you can:
> 1. `mkdir foo` and start writing code
> 1. `git clone <repo-url>`: A leaf folder is created with the repo name
> 1. `go get <repo-url>`: This works because the leaf folder doesn't yet exist.


4. vGo allows you to source your own bash code in two different files. A `.vgo.baserc` can be placed in the `VGO_HOME` folder. The code from this file applies to all workspaces. A `.vgo.extendrc` can be placed in each workspace with code that applies only to that workspace. This allows you to override any functionality implemented in the preceeding `bashrc`. Here is an illustration of the order in which they are executed:

```
+-------------------------------------------+
|    original .bashrc or .bash_profile      |
+-------------------------------------------+
|   vGo workspace configuration + commands  |
+-------------------------------------------+
|               .vgo.baserc                 |
+-------------------------------------------+
|              .vgo.extendrc                |
+-------------------------------------------+
```

5. The command to deactivate your workspace is `d`. (command names can be renamed. More on that later)

6. To activate your workspace again call `vgo w foo`. Because your project-path is already configured, you'll either be taken to `$VGO_HOME/foo/src/github.com/johndoe` or `$VGO_HOME/foo/src/github.com/johndoe/foo` depending on whether the `foo` folder exists.

7. To reconfigure a different project-path on an existing workspace, call `vgo w [exiting-workspace] [new-project-path]`. Example: `vgo w foo bitbucket.org/johndoe/baz`.

#### `vgo u <workspace-name>`

`vgo u | vgo unset` removes the the file `$VGO_HOME/foo/.vgo.proj` file which holds the project-path. On a workspace where a project-path is not set, a `vgo w <workspace>` will land you at `$VGO_HOME/<workspace>/src`.

#### `vgo i`

`vgo i | vgo info` lists all workspaces with their project-paths. It'll also indicate if `bashrc` override files exist.

#### `vgo e`

`vgo e | vgo env` lists all environment variables starting with `VGO_`. This command can be invoked both outside and inside the workspace.

#### `vgo v`

`vgo v | vgo version` displays the vGO version.


## Build-in commands


vGo has the following built-in commands which work only within an active workspace. The default commands are single-character (for :zap: productivity & minimalist freaks). They can however be renamed by setting an environment variable. Include a line with the format `export <Env. Variable>=<new-command>` into your original `.bashrc`.

Command | Description | Env. Variable
--------|-------------| -----------------------
r | Switch to the workspace root folder `$VGO_HOME/<workspace>`  | `VGO_CMD_CHDIR_ROOT`
p | Switch to the project-path. This can either be `$VGO_HOME/<workspace>/src/<project-path>` or `$VGO_HOME/<workspace>/src/$(dirname <project-path>)` | `VGO_CMD_CHDIR_PROJ`
d | Deactivate workspace | `VGO_CMD_DEACTIVATE`

> The `exit` command has been overridden to NOT exit the subshell. This is the default behavior in vGo. This has been done to discourage the use of `exit` habitually for both deactivation as well as exiting shells. You are encouraged to use a different deactivate command (or even the default) instead of `exit`. However, if you still wish to use `exit`, you can `export VGO_CMD_DEACTIVATE=exit`.


## Setting `PS1`

When a workspace is activated, the prompt is prefixed with the name of the workspace. vGo does this by modifying the `PS1` environment variable within the workspace. It sets this to `[<workspace-name>] <original-PS1>` by default, where `[<workspace-name>]` appears in bright cyan. (ANSI esc: 96m)

You can override the `PS1` by setting the environment variables `VGO_LEFT_BRACE` and `VGO_RIGHT_BRACE` in your original `.bashrc`. The format for the `PS1` within the workspace is `PS1=$VGO_LEFT_BRACE$VGO_NAME$VGO_RIGHT_BRACE$PS1`, where `VGO_NAME` is the name of the workspace (injected automatically by vGo). You'd need to handle ANSI colors and blank spaces in the values of `VGO_LEFT_BRACE` and `VGO_RIGHT_BRACE`.

> The presence of `VGO_NAME` is used to detect whether a workspace is active or not. DO NOT set the environment variable `VGO_NAME` yourself either inside or outside the workspace.

