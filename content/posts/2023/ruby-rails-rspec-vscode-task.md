---
title: "Creating tasks in Vscode for running Rspec tests for Rails projects"
date: 2023-02-19T19:21:46Z
tags: ['ruby', 'rails', 'vscode']
icon: test-pipe
aliases:
  - /posts/ruby-rails-rspec-vscode-task/
summary: >-
    A guide to setting up tasks in vscode to run rspec tests for Rails projects.
---

## Using Rspec in larger projects

As a project grows, there is a pretty valid use case for running only a subset of the rspec tests at a time when you are actively working on a particular piece of code, for a new feature or working on a bug. When doing this, you often don't want to run the full sweet of rspec tests, just the ones which are affected by the code that you're looking at.

There is tooling in the rspec CLI to do this nicely, you can use `rspec <filename>` to run all of the tests for a given file, or `rspec <filename>:<linenumber>` if you want to run all of the tests which come under a certain line of code (for example if you file has 10 tests but you only want to run one specific test). While this is incredibly useful, it's a bit of a hassle to be writing out the file name and line number over and over, so it would be nice if your editor (in this case vscode) could handle part of this for you.

## Basic setup

As a starting point, it would be nice to be able to run these two different test setups nicely, one for a file, and one for a given file and line number.

Here, we have these two skeleton tasks:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Run rspec for the current file",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "command": "rspec ${file} --format documentation"
        }, 
        {
            "label": "Run rspec for the current line",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "command": "rspec ${file}:${lineNumber} --format documentation"
        } 
    ]
}
```

With these two tasks, we can open up a given rspec file and run a task, vscode will then use the file which is currently open (and the line number currently selected for that task) and pass these through automatically to the rspec CLI. This is where the `${file}` and `${lineNumber}` parts come in with these tasks.

Another small addition is to add the `--format documentation` argument to rspec. This is so that we don't only see the failures when running these tasks, we see both the successes and the failtures. This will make it easier to notice if rspec isn't running the tests we expect, for example if we've set up the test incorrectly.

This setup works great as a starting point, but it would be nice if any test failures were flagged up by vscode, but at the minute, we need to interrogate the output of these tasks ourselves.

## Adding a problem matcher

Problem matchers in vscode tasks let us tell vscode how to process the output of a tasks and raise problems in the Problems tab. This does, *unfortunately*, require us to use some regular expressions, but fortunately they aren't overly complicated in this situation.

The problem matcher is broken up into three parts:

- The file with the problem
- The line number with the problem
- The message that accompanies the problem

In this situation, all of these parts can be extracted from a single line from the output from rspec, so that should be pretty nice for us to get. We can do this by adding the following block to each of our tasks:

```json
"problemMatcher": [
    {
        "owner": "ruby",
        "pattern": [
            {
                "regexp": "rspec (.*):(\\d+) # (.*)",
                "file": 1,
                "line": 2,
                "message": 3,
            }
        ]
    },
    {
        "owner": "ruby",
        "pattern": [
            {
                "regexp": "rspec (.*)\\[((:|\\d)+)\\] # (.*)",
                "file": 1,
                "location": 2,
                "message": 4,
            }
        ]
    }
]
```

In this setup our `regexp` breaks up every line of output which matches this pattern into three parts:

- Group 1 - file name, including path
- Group 2 - the line number, or range if it's across a block of code
- Group 3/4 - the message from rspec (note: this is group 3 in the first pattern, and group 4 in the second)

We need two different versions of this in case your tests are written such that rspec soemtimes reports a line of code (the first pattern), and sometimes reports a block of code (the second pattern). This is based on tests which run multiple times for different parameters or similar situations.

We also have an added part, `owner`. This is so that when we re-run the task again, it will clear any problems which no longer appear, so long as they were *owned* by the same thing, in our case, I've called that `ruby`.

## Putting this all together

We can then bundle this all together with the problem matcher setup and the tasks themselves. This would look something like this:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Run rspec for the current file",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "command": "rspec ${file} --format documentation",
            "problemMatcher": [
                {
                    "owner": "ruby",
                    "pattern": [
                        {
                            "regexp": "rspec (.*):(\\d+) # (.*)",
                            "file": 1,
                            "line": 2,
                            "message": 3,
                        }
                    ]
                },
                {
                    "owner": "ruby",
                    "pattern": [
                        {
                            "regexp": "rspec (.*)\\[((:|\\d)+)\\] # (.*)",
                            "file": 1,
                            "location": 2,
                            "message": 4,
                        }
                    ]
                }
            ]
        }, 
        {
            "label": "Run rspec for the current line",
            "type": "shell",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "command": "rspec ${file}:${lineNumber} --format documentation",
            "problemMatcher": [
                {
                    "owner": "ruby",
                    "pattern": [
                        {
                            "regexp": "rspec (.*):(\\d+) # (.*)",
                            "file": 1,
                            "line": 2,
                            "message": 3,
                        }
                    ]
                },
                {
                    "owner": "ruby",
                    "pattern": [
                        {
                            "regexp": "rspec (.*)\\[((:|\\d)+)\\] # (.*)",
                            "file": 1,
                            "location": 2,
                            "message": 4,
                        }
                    ]
                }

            ]
        } 
    ]
}
```
