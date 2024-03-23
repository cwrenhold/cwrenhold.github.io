---
title: "Setting up devc-nvim-commands"
date: 2024-03-17T10:33:21Z
tags: ["neovim", "dev containers", "bash"]
toc: true
icon: brand-docker
summary: >-
    A collection of commands I've created in bash to make it easier to manage neovim configurations and control dev containers.
---

## What is devc-nvim-commands?

[`devc-nvim-commands`](https://github.com/cwrenhold/devc-nvim-commands) is a collection of commands that I use in my daily development workflow. These are wrapped up into a bash script which can be run from the terminal to provide easy access to some common commands which are useful when using dev containers.

## What are dev containers?

Dev containers are a way to run your development environment in a container. This uses the [development container spec](https://containers.dev/implementors/spec/) to define a container (usually with docker), or a collection of containers (if using something like `docker compose`) to set up isolated development environments.

This is an approach to making the development environment which is used by developers more aligned with how the systems will run when deployed. For example, rather than having a shared database, redis cache, and the like across all projects, a developer can spin up isolated services for a particular project. This is then extended with dev containers to include the development environment as well, so that different developers can share a common local configuration, and this configuration has all of its dependencies isolated from the rest of the system.

These are very useful when using an editor such as vscode (along with the [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension), but vscode is not a requirement. Another approach to running these dev container configurations is to use the [Dev Container CLI](https://github.com/devcontainers/cli), and it is this which is the focus of this set of commands.

## Why use devc-nvim-commands?

One of the problems which I have been trying to resolve is how to manage my neovim configuration in a way which allows me to utilise dev containers. My initial approach had been to use neovim plugins such as [nvim-dev-container](https://codeberg.org/esensar/nvim-dev-container), which allow you to run a dev container through neovim and then connect to it, however, these approaches didn't really work with my setup. I wanted to be able to run neovim *inside* of the container rather than run neovim on my local machine and connect to the container.

To achieve this, I started off by installing neovim inside the container by hand, and then copying my configuration across, but this was a bit of a pain to do every time I started a new container, especially when I wanted to tweak the container's Dockerfile or my neovim configuration. This was when I moved to creating a collection of commands I could run more easily.

With this set of commands, I can use a single bash script to start up a dev container (through the dev container CLI), install neovim inside that container, and connect to it easily, all through that one script. This makes it much easier to manage my neovim configuration, and it also makes it easier to share this configuration with others.

## How to use devc-nvim-commands

To use this set of commands, you can clone the [repository](https://github.com/cwrenhold/devc-nvim-commands), and then run the `devc-nvim` command from that directory, however, this is a bit of a pain as you'd need to run the script from the directory your dev container configuration is in for this to work nicely. Instead, I would recommend adding the `devc-nvim` script to your path, and then you can run it from anywhere.

To do this, you can run the following commands:

```bash
git clone git@github.com:cwrenhold/devc-nvim-commands.git ~/devc-nvim-commands
ln -s ~/devc-nvim-commands/devc-nvim /usr/local/bin/devc-nvim
```

This will clone the repository into your home directory, and then create a symlink to the `devc-nvim` script in your `/usr/local/bin` directory. This means that you can run the `devc-nvim` command from anywhere on your system.

*An additional option if you use this a lot is to then alias this in your shell to something shorter, for example, I use `dn`*

After this is setup, you can navigate to any directory which contains as `.devcontainer` directory, and run the `devc-nvim` command with `help` to display the available commands.

### Examples

- `devc-nvim up` - This will start up your dev container through the dev container CLI
- `devc-nvim setup` or `devc-nvim setup-nightly` - This will install neovim inside of the dev container, and then copy your neovim configuration across (the `setup-nightly` command will install the nightly version of neovim rather than stable)
- `devc-nvim nvim` or `devc-nvim` - This will connect to the container and start up neovim
- `devc-nvim bash` - This will connect to the container and start up a bash shell

## Updating your neovim configuration

If you're working on your neovim configuration at the same time as working in a dev container, this isn't a problem either - there is a `devc-nvim config` command which will copy your current configuration for neovim back into your dev container. This means that you can make changes outside of the container and then update the container without having to rebuild the container or re-setup the container for neovim.

## Setting up a new dev container

For the time being, I haven't provided a command to set up a new dev container. For the time being, I would recommend using vscode to set up the initial dev container configuration, and then switching to this afterwards.

**Note: This setup does not automatically expose ports like vscode would do, so if you're running a container which needs to talk to the outside world, remember to expose the ports in the dev container's docker-compose configuration.**

## Conclusion

While this is only a small set of commands, they have made a huge difference to my workflow. It's now easier for me to use dev containers with neovim, it's easier to start up dev containers, and easier to try things out in a configuration of neovim inside of a container.

This is only a starting point for this project, though, and there are limitations. For example, my setup process for neovim inside of a container currently depends on the container having access to `apt`, which means that it isn't suitable for dev containers based on different distributions, such as alpine. This is something which I hope to look into more in the future, however, for now, this is a good starting point.

## Links

- [devc-nvim-commands](https://github.com/cwrenhold/devc-nvim-commands)
- [Dev Containers specification](https://containers.dev/implementors/spec/)
- [Dev Containers extension for vscode](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- [Dev Container CLI](https://github.com/devcontainers/cli)
