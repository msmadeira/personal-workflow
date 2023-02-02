# Git Helpers

## Git Aliases

```
git alias --global alias.branch-name !git rev-parse --abbrev-ref HEAD
git alias --global alias.publish !git push -u origin $(git branch-name)
git alias --global alias.ac !git add -A && git commit -m
git alias --global alias.refresh git config --global alias.refresh '!git fetch origin master:master && git merge master'
```

## Terminal Functions

```
function git-clear-local-branches() {
    git fetch -p && git branch -vv | awk '/: gone]/{print $1}' | xargs git branch -d
}
```
