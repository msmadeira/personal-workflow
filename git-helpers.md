# Git Helpers

## Git Aliases

```
git config --global alias.branch-name '!git rev-parse --abbrev-ref HEAD'
git config --global alias.publish '!git push -u origin $(git branch-name)'
git config --global alias.ac '!git add -A && git commit -m'
git config --global alias.refresh '!git fetch origin master:master && git merge master'
```

## Terminal Functions
### Bash
```
function git-clear-local-branches() {
    git fetch -p && git branch -vv | awk '/: gone]/{print $1}' | xargs git branch -d
}
```
### PowerShell
```
Function git-clear-local-branches() {
    git fetch -p;
    git branch -vv | where {$_ -match '\[origin/.*: gone\]'} | foreach { git branch -d $_.split(" ", [StringSplitOptions]'RemoveEmptyEntries')[0]};
}
```
