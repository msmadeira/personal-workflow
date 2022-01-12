# Git Helpers

## Git Aliases

```
branch-name = !git rev-parse --abbrev-ref HEAD
publish = !git push -u origin $(git branch-name)
ac = !git add -A && git commit -m
refresh = git config --global alias.refresh '!git fetch origin master:master && git merge master'
```

## Terminal Functions

```
function git-clear-local-branches() {
    git fetch -p && git branch -vv | awk '/: gone]/{print $1}' | xargs git branch -d
}
```
