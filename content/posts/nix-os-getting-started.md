---
title: "Getting Started with Nix OS"
date: 2024-03-09T10:40:39Z
tags: ['nixos', 'configuration']
icon: snowflake
summary: >-
    A simple guide to getting started with Nix OS, including setting up a basic configuration, and then extending this with flakes and home-manager.
---

## What is Nix OS?

I've been reading about Nix OS for a while now, and I've been wanting to try it out. While it might at first sound like it's *just another Linux distribution*, but it's actually quite different in some key ways. The main difference is that rather than using a package manager like `apt` or `yum`, Nix OS uses a declarative configuration language to describe what state you *want* the system to be in, and then it uses the Nix package manager to make that happen. This isn't just for things like installing software and updates, but it's also for configuring the system itself.

This means that, in an ideal world, you should be able to take a Nix OS configuration file, and apply it to a fresh install of Nix OS and end up in an identical state. This is a pretty powerful concept, but it comes with a bit of a learning curve. For example, you wouldn't create a bash script (or similar) to install and configure a piece of software, you would write a Nix file to do this.

Configurations with Nix have their own language, which is called Nix. This is a functional language, but when you're first starting off, you don't actually need to know that much about the language. You can get by with just knowing the basics of the language, and then you can start to learn more as you go.

## Getting started

After installing Nix OS, you are left with two Nix files which work as a nice starting point. These are both located in the `/etc/nixos` directory. The first file is `configuration.nix`, which is the main configuration file, and should be *relatively* portable to different machines (more on this later), and the other file is `hardware-configuration.nix`, which is specific to the hardware that Nix OS is running on.

This `configuration.nix` file includes a huge amount of configuration in a relatively small amount of code. This really helps from a learning perspective, but at the same time, it can be a bit overwhelming. An example of this is that in just three lines of code, it configures `grub` to handle the bootloader. It's a good idea to read through this initial file before making any changes as it provides a good introduction into what is possible with Nix OS.

One thing which I would highly recommend setting up as quickly as possible after you're familiar with the broad outline of this file is setting up support for flakes. Flakes are a relatively new feature in Nix OS, however, they provide a way to manage Nix projects in a more modern way. This can be done by adding the following line to the `configuration.nix` file:

```nix
nix.settings.experimental-features = [ "nix-command flakes" ];
```

Another setting which I would recommend setting up quickly is setting up your editor. In my case, I will be using neovim for this VM. This can be done in a few ways:

- A system-wide program via `programs`

    ```nix
    programs.neovim = {
        enable = true;
    };
    ```

- A system-wide program via `environment.systemPackages`

    ```nix
    environment.systemPackages = with pkgs; [
        neovim
    ];
    ```

- A user-specific program via `users.users.<username>.packages`

    ```nix
    users.users.<username>.packages = with pkgs; [
        neovim
    ];
    ```

You can then apply the changes to the system by running `sudo nixos-rebuild switch` from the `/etc/nixos` directory. This will load the new configuration and apply it to the system. I was initially surprised that I didn't need to restart my VM after running this command, but it seems that the changes are applied on the fly.

## Getting started with Flakes

Flakes are like an extended version of the main `/etc/nixos/configuration.nix` file. One advantage which they have is that they also create a `flake.lock` file, which is a lock file for the Nix packages which are being used. This is similar to a `Gemfile.lock` in Ruby, or a `package-lock.json` in Node. This means that when applying a configuration from a flake, not only does the configuration match the flake, but even the versions of the packages are locked in as well.

I started my main flake configuration by creating a `~/.dotfiles` directory, which I am using as a place to store all of my dotfiles, and the configuration of my system. This is a good place to start with flakes, as it's a relatively small project, and it's a good way to get to grips with the new features.

I followed two fantastic videos by LibrePhoenix on YouTube to get started with flakes. These videos are:

- [You Should User Flakes Right Away in NixOS!](https://www.youtube.com/watch?v=ACybVzRvDhs)
- [Manage Your Fotfiles with Home Manager](https://www.youtube.com/watch?v=IiyBeR-Guqw)

At the end of these videos, you'll end up with the following files:

- `flake.nix` - this is the starting point for your flake
- `flake.lock` - this is the lock file for the flake
- `configuration.nix` - this is the main configuration file
- `hardware-configuration.nix` - this is the hardware configuration file
- `home.nix` - this is the home manager configuration file

You can then apply the changes to your system with the following commands from your `~/.dotfiles` directory:

- `sudo nixos-rebuild switch --flake .` - this will apply the main system configuration
- `home-manager switch --flake .` - this will apply the home manager configuration
- `sudo nix flake update` - this will update the flake.lock file (but not apply the changes)

## Sharing application configuration

After getting familiar with these commands and the files, I started adding some additional configuration to get used to things. For example, I moved my `neovim` and `tmux` configurations into the `~/dotfiles` directory. This meant that I could use `home-manager` to manage these configurations. This was as simple as moving those directories into the `~/.dotfiles` directory, and then adding the following to the `home.nix` file:

```nix
  home.file = {
    ".config/nvim".source = ./nvim;
    ".tmux.conf".source = ./tmux/tmux.conf;
  };
```

This will then take the directory of `~/.dotfiles/nvim` and symlink it to `~/.config/nvim`, and the file `~/.dotfiles/tmux/tmux.conf` and symlink it to `~/.tmux.conf`. This meant that I could put all of my `.dotfiles` into git and have them managed by `home-manager` across systems.

## Extending configuration

After setting up the basics above, I decided to play around with the configuration a bit more by adding aliases for `bash` and `zsh` to my `home.nix` file. Leveraging the Nix language, these can even be shared between the two shells. I did this by adding a `let` block to the `home.nix` file, and then adding the aliases to the `bash` and `zsh` sections of the `home.file` block.

### Adding the `let` block

This is as simple as adding the following to the start of your `home.nix` file:

```nix
let
    shellAliases = {
        ll = "ls -lh";
        la = "ls -lha";
        gs = "git status";
        gf = "git fetch";
        ".." = "cd ..";
    };
in
{
    # The rest of your home.nix file
    ...
}
```

This provides a new variable `shellAliases` which can be used in the rest of the `home.nix` file.

### Adding the aliases

To add these aliases, we can add them using the `inherit` keyword in Nix and the variable we created in the `let` block. This is done by adding the following to the `home.file` block:

```nix
programs.bash = {
    enable = true;
    inherit shellAliases;
};

programs.zsh = {
    enable = true;
    inherit shellAliases;
};
```

This will then add the two sets of aliases to both `bash` and `zsh` on the system. This can be applied to the system with `home-manager switch --flake .`. and then the aliases will be available in both shells.
