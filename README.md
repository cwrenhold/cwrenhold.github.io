<!-- markdownlint-disable-next-line MD041 -->
[![Deploy Hugo site to Pages](https://github.com/cwrenhold/cwrenhold.github.io/actions/workflows/hugo.yml/badge.svg)](https://github.com/cwrenhold/cwrenhold.github.io/actions/workflows/hugo.yml)

# Summary

This is the code for my website, generated by the static site generator, [Hugo](https://gohugo.io/), and using the [PaperMod](https://github.com/nanxiaobei/hugo-paper) theme.

The deployed website can be found here: <https://www.wrenhold.com>

## Setting up this site

handle the Hugo installation and its dependencies, I've used a dev container setup here to handle the Hugo installation and its dependencies, so you'll need to have Docker installed and running on your machine. You can then open the repo in Vscode or start it using the [Dev Container CLI](https://github.com/devcontainers/cli).

After cloning the repo the first time, it is best to update the theme submodule by running this command in the root directory:

```bash
git submodule update --init themes/PaperMod
```

Alternatively, you can update all of the submodules (should any more be added in the future) by running:

```bash
git submodule update --init --recursive
```

## Running the site locally

To run the site locally, you can either use the vscode default test task, or run the following command:

```bash
hugo server -D
```

Note: the `-D` flag is used to include draft posts in the local build.

## Updating the website

The website itself is setup to be deployed to Github Pages using the [hugo.yml](./.github/workflows/hugo.yml) script, and the workflow is setup to run on every push to the main branch, so nothing special is required to do this so long as that is configured.
