---
title: "Vscode Tasks With Prompts"
date: 2023-02-19T16:33:24Z
tags: ['vscode', 'automation']
icon: brand-vscode
aliases:
  - /posts/vscode-tasks-with-prompts/
summary: >-
    A guide to setting up tasks in vscode with prompts to make them more flexible and powerful.
---

Using tasks with inputs is a quick and powerful way to add a lot of additional functionality to Vscode, this can extend the normal task tooling to make it much more flexible and powerful. In this short article, we'll walk through creating a task with a simple input and how we can then leverage that.

## The Problem

For this, I wanted to create a task to simplify the process of creating new posts for this website. This is already simple enough, using the Hugo CLI, you can type in something like:

```bash
hugo new posts/my-new-post.md
```

This will create a new post in the `content/posts` directory called `my-new-post.md` and that post will have a title of "My New Post". However, I won't want to have to remember this command - I want vscode to remember it, and then all I want to need to do is type in the new filename, ideally without the file extension, as that is always going to be the same.

## The Solution

This setup is broken down into two parts:

1. The input - this is a way to handle user input in a task in vscode, and these can be re-used between different tasks
2. The task - this will be what runs the process, utilising the input that we've defined

### The input

This is a very simple input, we want a string from the user, and we can use a simple prompt for this. We'll go into our `tasks.json` file and add the following:

```json
"inputs": [
    {
        "id": "postName",
        "description": "Post filename:",
        "type": "promptString"
    }
]
```

Here, we are creating a new set of inputs to exist in our set of tasks for the project, and for this we only need one input. This input will be called `postName`, and the `description` will provide a prompt to the user running out task. The `type` then sets how vscode will handle this request, in our situation, `promptString` will prompt the user to enter a string of text - nice an obvious!

### The task

The task itself is very similar to one without a prompt, the only difference is that we want to use out `input` (which we called `postName`) so that when the task is run, the user received that prompt. This will look something like this:

```json
{
    "label": "Create new post",
    "command": "hugo",
    "type": "process",
    "options": {
        "cwd": "${workspaceFolder}"
    },
    "args": [
        "new",
        "posts/${input:postName}.md"
    ],
    "problemMatcher": []
}
```

Like most tasks, we're setting up a simple `label`, `command`, and so on, it's only when we get down to the `args` that our input comes in. In this, rather than putting a hardcoded string for the last argument (i.e. the path to the file to be created), we're going to put in the parts which will never change (the folder path and the file extension), and the put in an `input` in the middle. This ends up looking like this:

Note: There is an empty `problemMatcher` here, this is to tell vscode that we don't want to process the output of this task in anyway, which means we don't get any information for vscode from this task other than what we see in the task output ourselves.

```json
"posts/${input:postName}.md"
```

This will mean that if the user entered `my-new-post`, the argument that is actually sent to `hugo` would end up as `posts/my-new-post.md`.

## Finished tasks.json

After these additions, your `tasks.json` file should look something like this:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Create new post",
            "command": "hugo",
            "type": "process",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "args": [
                "new",
                "posts/${input:postName}.md"
            ],
            "problemMatcher": []
        }
    ],
    "inputs": [
        {
            "id": "postName",
            "description": "Post filename:",
            "type": "promptString"
        }
    ]
}
```
