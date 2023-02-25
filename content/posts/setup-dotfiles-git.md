---
title: "How To Setup a Git Repository for Your Dotfiles"
date: 2023-02-25T11:33:26Z
---

## The Problem

Like many people, I use more than one computer for my work and free time, and I often want to share settings between my different Linux machines. This is can be accomplished using a git repository for your .files (e.g. your `.bashrc`, `.gitconfig`, etc.).

If you use a git repository for this, you can then quite easily push this repository up to a cloud based repository (e.g. GitHub) which will make it easy to pull those settings down onto new machines and push new changes across your machines.

## Setting Up Your Local Repository

There are only a few steps for this, but the first question is where we want to store the `.git` for our dotfiles, personally, I didn't want this to be in my home directory directly as that felt a little messy, so instead I created another directory for these files to be stored in. For me, I chose to use a directory called `.dotfilesgit` in my home directory. We'll put this into an environment variable to make this easier to reference with the following command:

```bash
export DOTFILES_PATH="$HOME/.dotfilesgit/"
```

This will mean that we can use `echo $DOTFiLES_PATH` to check this while we're putting this together, if we run that command now, you should receive an output like this:

```bash
/home/<YOUR USERNAME>/.dotfilesgit/
```

Now that we've decided where to store our `.git` repository for our dotfiles, we can create the git repository. We can do this by navigating into your home directly (either by typing `cd ~` or `cd /home/<YOUR USERNAME>`) and then running the following command:

```bash
git init --bare $DOTFILES_PATH
```

This command will create the git repository, but rather than storing the git files in the default location, it will store them in the path we've called `$DOTFILES_PATH`, i.e. `/home/<YOUR USERNAME>/.dotfilesgit/`.

## Setting up an alias

Next, we'll create an alias to make commands using this repo a bit easier to use. This will mean that rather than having to specify the work tree and git directory each time we want to do anything, we can use a command which will automatically do this. We can create this with the following command:

```bash
alias dotgitfiles='git --git-dir=$DOTFILES_PATH/ --work-tree=$HOME'
```

Let's break down what this command is going to do:

1. `alias dotgitfiles=` - create an alias in our shell called `dotgitfiles` which will run the command to the right of the `=`
2. `git` - this will start a git command
3. `--git-dir=$DOTFILES_PATH` - this tells git to ignore the default directory for storing files for git, and instead your the path we've defined in `$DOTFILES_PATH`
4. `--work-tree=$HOME` - this tell git that the repository starts in the `$HOME` directory, i.e. `/home/<YOUR USERNAME>`

## Telling git to ignore files by default

By default, git will try to track changes on all files within the directory it's monitoring. That perfect for most repositories, but because this is in our home directory, we only want git to monitor files we specify, otherwise there will be far too many files changes to manage without really slowing things down.

We can tell git to ignore all files by defaut with the following command:

```bash
dotgitfiles config --local status.showUntrackedFiles no
```

This utilises the alias which we created above, so the actual command which is executed would look like this:

```bash
git --git-dir=$DOTFILES_PATH/ --work-tree=$HOME config --local status.showUntrackedFiles no
```

This will set a config setting in your new git repository, `status.showUntrackedFiles` and give it the setting `no`, this will mean that untracked files will not be displayed when checking for the status of the repository.

## Commiting your first file

Now that our repository is set up and configured, we can commit our first file. As we aren't showing untracked files by default, we need to specifically add any file to git which we want to track. We'll start by adding our `.bashrc` file to git with the following command:

```bash
dotgitfiles add .bashrc
```

This will specifically add the `.bashrc` to our staged changes in git, and then we can commit these as normal, but with our alias for this repository:

```bash
dotgitfiles commit -m "<YOUR COMMIT MESSAGE>"
```

## Pushing your changes to a remote repository

First, we need to create a repository in the cloud like we normally do, we can do this easily enough in GitHub by navigating to your account and going through the "New Repository" process.

Next, we can add a remote like we would usually do for a repository, only we need to ensure that we use our new alias rather than the default `git` command. To do this, we can go back to our home directory and run the following commands:

```bash
dotgitfiles remote add origin <YOUR GIT URL>
dotgitfiles branch -M main
dotgitfiles push -u origin main
```

*Note: this assumes your default branch is `main`, you may need to change this if you have a different default branch on your machine.*

We should then be able to reload the repository on GitHub and view the `.bashrc` file that was commited.

## Downloading your repository onto a new machine

When we want to clone this repo onto a new computer, we need to re-do some of the steps we have gone through above in order to ensure that the repository on the new machine works correctly. We can do this with the following commands:

```bash
export DOTFILES_PATH="$HOME/.dotfilesgit/"
alias dotfilesgit='/usr/bin/git --git-dir=$DOTFILES_PATH/ --work-tree=$HOME'
git clone --bare <YOUR GIT URL> $DOTFILES_PATH
dotgitfiles config --local status.showUntrackedFiles no
```

This performs the following actions:

1. Create our environment variable for the path we want to store the git files in
2. Create an alias for git for this directory
3. Clone your repo into this path
4. Configure this git repository to not show untracked files by default

After doing this, you should be all set on a second machine with the configuration that you setup on your first machine and you're good to go!
