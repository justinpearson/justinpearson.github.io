---
title: "show git branch and status info in zsh command prompt"
date: 2025-02-05
tags: [zsh, git, shell]
---

In `~/.zshrc`:

```bash
# show git branch & status of unstaged / staged-but-uncommitted files

git_branch() {
    # Check if we are inside a Git repository
    if ! git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
        return
    fi

    # Get the current branch name
    branch=$(git symbolic-ref --short HEAD 2>/dev/null)
    if [ -z "$branch" ]; then
        return
    fi

    # Initialize status indicators
    git_status=""

    # Check for staged changes (cached diff)
    if ! git diff --cached --quiet --ignore-submodules; then
        git_status="*"
    fi

    # Check for unstaged changes (working tree diff)
    if ! git diff --quiet --ignore-submodules; then
        git_status="${git_status}+"
    fi

    # Print the formatted Git branch with status
    echo "%F{blue}($branch$git_status)%f"
}


update_prompt() {
    PROMPT='%(?.%F{green}√.%F{red}!%?)%f 20%D %* %B%F{240}%1~%f%b '"$(git_branch)"' %# '
}

precmd() {
    update_prompt
}
```

Prompt breakdown:

- `%(?.%F{green}√.%F{red}!%?)` Shows √ (green) if last command succeeded, or !exit_code (red) if it failed
- `%f` Resets the foreground color
- `20%D` Displays date in YYYY-MM-DD format
- `%*` Displays time in HH:MM:SS format
- `%B` Starts bold text
- `%F{240}` Sets color to gray
- `%1~` Shows current directory, shortened to one level
- `$(git_branch)` Calls function to display Git branch and status
- `%#` Shows $ (normal user) or # (root user)
