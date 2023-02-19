---
title: "Setting up Hugo for GitHub Pages using a Dev Container"
date: 2023-02-19T20:01:40Z
tags: ['hugo', 'vscode', 'dev containers', 'static websites']
---

Hugo is a tool for generating static websites for you using MarkDown files, and it's what I use to create this website. This makes the process of setting up a static website for things like this quite simple and easy to maintain and extend. However, in order to use Hugo, you need to install it on your machine along with Go. I like to keep my system relatively clean of this kind of thing just because otherwise I end up with a lot of different tools installed all over the shop. To make this easier, I decided I would use a dev container setup instead.

## Dev Containers

Dev containers act as a nice way to isolate your development environment, keeping it separate from other things on your system, and making it easier to configure this in a similar way to other, non-development environments when it comes to bigger projects. I'm not too worried about replicating a production environment here, but I only want Hugo (and Go) to be installed when I'm working on this project and not cluttering up things when I'm working on other projects.

Dev containers depend on Docker, which is an incredibly useful tool for creating and running containers. I won't spend too much time on this, but if you're unfamiliar with Docker, I would recommend reading up a bit about it on their [website](https://www.docker.com/).

Dev containers are designed to work well with vscode with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers), although you can also use them using the [Dev Container CLI](https://github.com/devcontainers/cli). In this setup, I used vscode, but either would have worked.

## Setting up the Dev Container

To start up, I created a new folder to put my code into and opened up that directory in vscode. After that, you can open the command palette and select "Add Dev Container Configuration Files...". This opens a prompt for creating a dev container setup, and for this we only need to use the Hugo template. This creates a `.devcontainer` folder in the folder, along with a `devcontainer.json` file and a `Dockerfile` file. In this situation, nothing needs to be modified in these files, but we can now open up the command palette again and select "Reopen in Container". This reopens the current window but from inside of a Docker container that is configured for Hugo, along with all of its dependencies.

## Creating your Hugo project

The first step once you're in your project is to create your blank project. You can do this by opening a terminal inside of vscode inside of your container (note: this is a terminal running in the container itself, so your local tools aren't available here, and vice versa if you're in the same directory in your local terminal). In the terminal, you can use the command `hugo new site .`, this creates a new Hugo project in the current directory. This creates a lot of files, but we don't need to worry about those just yet.

## Moving over to a yml/yaml configuration

Although this isn't required, I would recommend switching over to a yml/yaml configuration file. This works just as nicely as a toml configuration file, but is a little easier to work with, plus it seems that a lot of other people also use this format which makes looking up information about the configuration easier.

You can do this by opening the generated toml file and running this through a converter. I used [this tool](https://www.convertsimple.com/convert-toml-to-yaml/) however, there are many available. You can then replace the yaml version of your initial config file into your `config.toml` and then rename it to `config.yml`.

## Setting up your theme

There are many themes available for Hugo, however, I am currenlty using [PaperMod](https://github.com/adityatelange/hugo-PaperMod). You can see a larger list of themes which are available [here](https://themes.gohugo.io/). To install a theme, you can run the following command (note: this is using the PaperMod theme as an example):

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive
```

This will clone the theme as a submodule which will make it easier to update later on. You can then add the theme to your `config.yml` file by adding the following line:

```yaml
theme: PaperMod
```

## Setting up your first post

Now that we have a theme installed, we can create our first post. To do this, we can run the following command:

```bash
hugo new posts/hello-world.md
```

This will create a new file in the `content/posts` directory with the name `hello-world.md`. You can then open this file and add some content to it.

## Testing out your site

To test our the site we've created, we can either run use the test task in vscode which was created by the dev container setup (via Tasks: Run Test Task), or we can run the following command in the terminal:

```bash
hugo server -D
```

Note: the `-D` flag is used to include draft posts in the server, which any newly created posts will have by default.

Once we're happy with our site and our first post, we can remove the `draft: true` line from the post and move onto getting this into GitHub.

## Setting up your local repository

To begin with, we want to create a `.gitignore` in our project, this is so that we don't include any unnecessary files in our repository. Our default one should look something like this (which I generated from [gitignore.io](https://www.toptal.com/developers/gitignore/api/hugo):

```gitignore
# Created by https://www.toptal.com/developers/gitignore/api/hugo
# Edit at https://www.toptal.com/developers/gitignore?templates=hugo

### Hugo ###
# Generated files by hugo
/public/
/resources/_gen/
/assets/jsconfig.json
hugo_stats.json

# Executable may be added to repository
hugo.exe
hugo.darwin
hugo.linux

# Temporary lock file while building
/.hugo_build.lock

# End of https://www.toptal.com/developers/gitignore/api/hugo
```

We can then create our local repository and committing our initial setup by running the following commands:

```bash
git init
git add .
git commit -m "Initial commit"
```

Note: I would recommend running this from outside of your dev container in case you have any signing keys or similar configured which may not be available inside the container

## Setting up your GitHub repository

For this, we need to create our GitHub Pages repository on GitHub itself. This can be done by creating a new repository on your profile on GitHub and calling it `<username>.github.io`. This is the defualt repository name for GitHub Pages, so it makes things a little simpler. Once you've created this repository, you can then push your local repository to the commands recommended by GitHub for existing repositories, which will look something like this:

```bash
git remote add origin <GitHub URL>
git branch -M main
git push -u origin main
```

After your code is on GitHub, you can then go to the settings for your repository on GitHub and select "Pages", from here, there is an option labelled "Source" which you can change to "GitHub Actions". This allows us to use a GitHub Action to process our site and publish it to GitHub Pages. There is one called "Hugo" here which we can use, and covers everything which we need to do. After you select this, you should be able to commit the GitHub Action which has been templated for you and it will automatically run and publish your site. This will also re-run every time your main branch is updated with any new changes.

## Viewing your site

If you now navigate to your code base on GitHub, you should see a link under "Environments" which is called "github-pages". If you click on this, you can see a list of your deployments, and also a "View deployment" on the most recent one. This will take you to your GitHub Pages site, which should be live and ready to go. You should also be able to see your site at `<username>.github.io`.

## Linking this up with your domain

If you have a domain to host this content on, you can do this by returning to the Settings page for your repository on GitHub and selecting "Pages" again. From here, you can select "Custom domain" and enter your domain name. You will then need to update your domain's DNS settings to points to GitHub's hosting which you can read more about [here](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages), however, this configuration is different for each domain provider, so you will need to check with yours.
