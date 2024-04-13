---
title: Useful Global Git Configurations
date: 2024-03-06
tags: ["git"]
ShowToc: false
---

In this post, I'm going to share with you six global Git configurations that I use and find incredibly useful.

Global Git configurations are settings that apply to all your local repositories. They are stored in a file called `.gitconfig` in your home directory. You can view and edit this file, or you can use the `git config` command to modify it from the terminal.

To see the global configurations through the terminal, you can run:

```bash
git config --global --list
```

Let's get to it!

### 1. `git config --global init.defaultBranch main`

This sets the default branch name to `main` instead of `master` when creating a new repository using `git init`.

### 2. `git config --global help.autoCorrect prompt`

By default, Git checks for spelling errors and suggests corrections, but it doesn’t automatically apply them. After configuring this setting, Git will prompt you to run the suggested correction.

```git
$ git pusj
WARNING: You called a Git command named 'pusj', which does not exist.
Run 'push' instead [y/N]?
```

To allow git to run corrections automatically, you can use `help.autoCorrect 1` (runs after 0.1 seconds), `help.autoCorrect 10` (runs after 10 seconds), or `help.autoCorrect immediate` (runs immediately).

I prefer using `help.autoCorrect prompt` to ensure that I will run the command that I intend to.

### 3. `git config --global branch.sort -committerdate`

This makes `git branch` sort the branches by the most recently used branches instead of alphabetically. This helps you to quickly find the branches that you are working on.

### 4. `git config --global fetch.prune true`

When you run `git fetch`, any branches that have been deleted on the remote repository will automatically be deleted from your local repository.

### 5. `git config --global log.date iso`

When you run `git log`, dates will be displayed in ISO format for better readability.

For example, `Sun Mar 16 05:30:02 2024` instead of `2024-03-16 05:30:02`.

### 6. `git config --global push.autoSetupRemote true`

This makes `git push` automatically set up the remote branch. Otherwise, you'd need to do it manually like so:

```bash
git push --set-upstream origin <branch-name>
```

## Conclusion

I hope you found these configurations helpful. Please feel free to share yours!

You can find more on [Git documentation](https://git-scm.com/docs/git-config).