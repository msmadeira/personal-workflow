# Git Helpers

## Git Aliases

```
ac = !git add -A && git commit -m
publish = "!git push -u origin $(git branch-name)"
```

## Terminal Functions

```
function git-clear-local-branches() {
    git fetch -p && git branch -vv | awk '/: gone]/{print $1}' | xargs git branch -d
}
```
