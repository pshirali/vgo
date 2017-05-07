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

3. The `project-path` is stored into the file `$VGO_HOME/foo/.vgo.proj`. If the `project-path` doesn't exist, folders upto the basename of the `project-path` are recursively created inside the `$VGO_HOME/src` folder. In the above example, this would be `$VGO_HOME/src/github.com/johndoe`.

> When the `project-path` folder doesn't exist, vGo doesn't know whether you intend to create a new project or pull existing code from a remote repo. The recursive folder creation till the basename of the `project-path` allows you to perform the last mile change yourself. From this point, you can:
> 1. `mkdir foo` and start writing code
> 1. `git clone <repo-url>`: A leaf folder is created with the repo name
> 1. `go get <repo-url>`: This works because the leaf folder doesn't yet exist. It has the same effect as `git clone`

w.i.p
