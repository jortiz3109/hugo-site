---
title: "Taskfile: installation and first steps"
date: 2026-03-09
draft: false
tags: ["taskfile", "task", "automation", "devtools"]
categories: ["DevOps", "Productivity"]
showTableOfContents: true
---

If you run repetitive development tasks (installing dependencies, running tests, building, deploying), **Taskfile** helps you automate them with a clean and readable approach.

<!--more-->

This guide covers the essentials: installation, first steps, and how to structure your tasks.

## What is Taskfile?

[Task](https://taskfile.dev/) is a modern task runner. Its configuration lives in a `Taskfile.yml` file, where you define reusable project commands.

It is similar to `Make`, but with a more approachable YAML syntax and strong developer ergonomics.

## Installation

### macOS

Using Homebrew:

```bash
brew install go-task/tap/go-task
```

### Linux

Using the official install script:

```bash
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d
```

You can also install it via package managers (`snap`, `apt`, etc.) or by downloading a binary release.

### Windows

Using Scoop:

```powershell
scoop install task
```

Using Chocolatey:

```powershell
choco install go-task
```

### Verify installation

```bash
task --version
```

## First steps

1. Go to your project root directory.
2. Create a `Taskfile.yml` file.
3. Define your common tasks (`install`, `dev`, `test`, `build`).
4. Run tasks with `task <name>`.

Basic example:

```yaml
version: "3"

tasks:
  install:
    desc: Install dependencies
    cmds:
      - npm install

  dev:
    desc: Start development environment
    deps: [install]
    cmds:
      - npm run dev

  test:
    desc: Run tests
    cmds:
      - npm run test
```

Useful commands:

```bash
task           # runs the default task (if defined)
task --list    # lists available tasks
task test      # runs the "test" task
```

## Recommended Taskfile structure

A `Taskfile.yml` usually contains these sections:

- `version`: schema version.
- `tasks`: map of all tasks.
- `desc`: task description for help output.
- `cmds`: commands to execute.
- `deps`: task dependencies.
- `vars`: reusable variables.
- `env`: environment variables.
- `sources` / `generates`: up-to-date checks (skip unnecessary runs).

More complete example:

```yaml
version: "3"

vars:
  APP_NAME: loan-calculator

env:
  NODE_ENV: development

tasks:
  default:
    desc: Show available tasks
    cmds:
      - task --list

  install:
    desc: Install dependencies
    cmds:
      - npm ci

  lint:
    desc: Check code quality
    cmds:
      - npm run lint

  test:
    desc: Run unit tests
    deps: [install]
    cmds:
      - npm run test:run

  build:
    desc: Build project
    deps: [install]
    cmds:
      - npm run build

  ci:
    desc: Local continuous integration flow
    deps: [lint, test, build]
```

## Task design best practices

- Use short, clear names (`build`, `test`, `deploy`).
- Add `desc` to every task.
- Create a `ci` task to run your full pipeline.
- Avoid duplicating long commands: wrap them in tasks.
- Reuse variables for paths, names, and flags.

## Conclusion

Taskfile is a practical way to standardize how project routines are executed.

For teams, it ensures everyone uses the same commands and reduces environment drift.

If you are just getting started, define four core tasks first: `install`, `dev`, `test`, and `build`. Then evolve toward advanced flows like `release` or `deploy`.

## Resources

- Official website: https://taskfile.dev/
- Documentation: https://taskfile.dev/docs/guide
- Repository: https://github.com/go-task/task
