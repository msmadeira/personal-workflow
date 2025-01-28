# Git Helpers

## Configurations
```
# Save a conflict resolution and use it for next conflicts
git config --global rerere.enabled true
# git branch improvements
git config --global branch.sort -committerdate
git config --global column.ui auto
# Improves speed of things in the background
git maintenance start
```

## Git Aliases

```
git config --global alias.branch-name '!git rev-parse --abbrev-ref HEAD'
git config --global alias.publish '!git push -u origin $(git branch-name)'
git config --global alias.refresh '!git fetch origin main:main && git merge main'

# Add + Commit
## Default
git config --global alias.ac '!git add -A && git commit -m'
## With conventional commit
git config --global alias.acm '!f() { \git add .; \branch_name=$(git symbolic-ref --short HEAD); \type=$(echo "$branch_name" | cut -d "/" -f 1); \id=$(echo "$branch_name" | cut -d "/" -f 2 | cut -d "-" -f 1); \commit_message="$type: #$id $1"; \git commit -m "$commit_message" "${@:2}"; \}; f'
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
