# Advanced Git Reference Notes
> GitHub-style reference notes for "Advanced Git" by Jawwad Ahmad & Chris Belanger

## Table of Contents
- [Git Fundamentals](#git-fundamentals)
- [Git Object Model](#git-object-model)
- [References and Refspecs](#references-and-refspecs)
- [The Index](#the-index)
- [Stashing](#stashing)
- [Rebasing](#rebasing)
- [Cherry-picking](#cherry-picking)
- [Reset and Checkout](#reset-and-checkout)
- [Merge Strategies](#merge-strategies)
- [Hooks](#hooks)
- [Rewriting History](#rewriting-history)
- [Submodules and Subtrees](#submodules-and-subtrees)
- [Worktrees](#worktrees)
- [Troubleshooting](#troubleshooting)
- [Advanced Workflows](#advanced-workflows)

## Git Fundamentals

### Git as a Distributed VCS
- Each clone is a full backup of the repository
- Commit locally without network access
- Push/pull changes when ready to share

### Configuration Levels
```bash
# Local (repository-specific)
git config --local user.name "Your Name"

# Global (user-specific)
git config --global user.name "Your Name"

# System (machine-wide)
git config --system user.name "Your Name"
```

### Essential Configuration
```bash
# Set identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch
git config --global init.defaultBranch main

# Configure editor  
git config --global core.editor "code --wait"

# Set up diff and merge tools
git config --global diff.tool vscode
git config --global merge.tool vscode
```

### Useful Aliases
```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
```

## Git Object Model

### Core Object Types
- **Blob**: Content of a file (no metadata)
- **Tree**: Directory listing, pointers to blobs and other trees
- **Commit**: Snapshot of the repo (points to tree and parent commits)
- **Tag**: Reference to a specific commit (for versioning)

### Hash IDs
- SHA-1 hash of object contents and headers
- Guarantees data integrity
- First few characters often used as shorthand (7+ characters)

### Viewing Objects
```bash
# Examine object content
git cat-file -p <hash>

# Determine object type
git cat-file -t <hash>

# List objects in packfile
git verify-pack -v .git/objects/pack/*.pack
```

### Object Storage
- Loose objects: Individual files in `.git/objects/`
- Packed objects: Compressed in `.git/objects/pack/`
- Git garbage collection: `git gc` compresses loose objects

## References and Refspecs

### References
- Branches: Pointers to commits (stored in `.git/refs/heads/`)
- Tags: Named pointers (stored in `.git/refs/tags/`)
- Remote branches: Bookmarks of remote state (`.git/refs/remotes/`)
- HEAD: Reference to current commit (`.git/HEAD`)

### Special References
- `HEAD`: Current commit/branch
- `HEAD~n`: nth ancestor of HEAD
- `HEAD^n`: nth parent of HEAD (for merge commits)
- `@{upstream}` or `@{u}`: Upstream branch
- `@{-n}`: nth previously checked out branch

### Refspecs
- Mapping between remote and local refs
- Format: `<src>:<dst>`

```bash
# Push local main to remote feature branch
git push origin main:feature

# Delete remote branch
git push origin :feature

# Track specific remote branch
git checkout -b feature origin/feature
# or
git branch --track feature origin/feature
```

### Reference Lists
```bash
# List all references
git show-ref

# List symbolic references
git symbolic-ref HEAD
```

## The Index

### Understanding the Staging Area
- Sits between working directory and repository
- Contains planned snapshot for next commit
- Stored in `.git/index`

### Index Operations
```bash
# Add files to index
git add <file>
git add -p  # Interactive staging

# See what's in the index vs HEAD
git diff --cached

# See what's not staged yet
git diff

# Remove from index but keep in working copy
git rm --cached <file>

# Update index with file modification time without checking content
git update-index --assume-unchanged <file>

# Reset index but not working directory
git reset
```

### Advanced Index Features
```bash
# Stage hunks interactively
git add -i

# Split hunks during staging
git add -p  # then use 's' option

# Apply stashed index
git stash apply --index
```

## Stashing

### Basic Stashing
```bash
# Stash changes
git stash

# Apply and drop most recent stash
git stash pop

# Apply but keep stash entry
git stash apply

# List all stashes
git stash list
```

### Advanced Stashing
```bash
# Create a named stash
git stash save "WIP: feature X implementation"

# Stash untracked files too
git stash -u

# Stash specific files
git stash push <path>

# Apply specific stash
git stash apply stash@{2}

# Show stash contents
git stash show -p stash@{1}

# Create a branch from stash
git stash branch feature-branch stash@{1}

# Interactive stashing
git stash -p

# Clear all stashes
git stash clear
```

## Rebasing

### Basic Rebasing
```bash
# Rebase current branch onto main
git rebase main

# Continue after resolving conflicts
git rebase --continue

# Skip current commit during rebase
git rebase --skip

# Abort rebase
git rebase --abort
```

### Interactive Rebasing
```bash
# Last 3 commits
git rebase -i HEAD~3

# Actions:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# d, drop = remove commit
# x, exec = run command (the rest of the line) using shell
```

### Auto-squashing
```bash
# Mark commit for squashing in a future rebase
git commit --fixup=<commit>
git commit --squash=<commit>

# Rebase with autosquash
git rebase -i --autosquash <base>
```

### Changing Base
```bash
# Move entire feature branch to new base
git rebase --onto new-base old-base feature-branch
```

## Cherry-picking

### Basic Cherry-picking
```bash
# Apply a single commit to current branch
git cherry-pick <commit-hash>

# Cherry-pick a range of commits
git cherry-pick A..B  # applies commits after A up to and including B
```

### Advanced Cherry-picking
```bash
# Cherry-pick but don't commit
git cherry-pick --no-commit <commit-hash>

# Keep original authorship information
git cherry-pick -x <commit-hash>

# Sign cherry-picked commit
git cherry-pick -S <commit-hash>

# Continue after resolving conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort
```

## Reset and Checkout

### Reset Types
```bash
# Soft reset - move HEAD but leave index and working directory
git reset --soft HEAD~1

# Mixed reset (default) - move HEAD and update index
git reset HEAD~1
git reset --mixed HEAD~1  # same as above

# Hard reset - move HEAD, update index and working directory
git reset --hard HEAD~1
```

### Checkout vs Reset
- Checkout is focused on updating working directory
- Reset is focused on moving branch pointers

```bash
# Reset moves the branch that HEAD points to
git reset <commit>

# Checkout moves HEAD itself
git checkout <commit>
```

### Path-specific Operations
```bash
# Reset specific file to HEAD
git reset -- <file>

# Reset specific file to specific commit
git reset <commit> -- <file>

# Checkout specific file from HEAD
git checkout -- <file>

# Checkout specific file from specific commit
git checkout <commit> -- <file>
```

## Merge Strategies

### Basic Merge
```bash
# Regular merge (creates merge commit)
git merge feature-branch
```

### Fast-forward Merge
```bash
# Fast-forward when possible
git merge --ff feature-branch  # default behavior

# Prevent fast-forward (always create merge commit)
git merge --no-ff feature-branch

# Only merge if fast-forward is possible
git merge --ff-only feature-branch
```

### Advanced Merge Strategies
```bash
# Resolve strategy
git merge -s resolve branch

# Recursive strategy (default)
git merge -s recursive branch

# Recursive with preference for ours/theirs
git merge -X ours branch
git merge -X theirs branch

# Octopus strategy (for more than two branches)
git merge branch1 branch2 branch3

# Ours strategy (discard all changes from other branches)
git merge -s ours branch

# Subtree merge
git merge -s subtree --no-commit project-branch
```

### Merge Tools
```bash
# Invoke merge tool for conflicts
git mergetool

# Configure specific tool
git config --global merge.tool <tool>
```

## Hooks

### Common Hooks
- **pre-commit**: Run before commit is created
- **prepare-commit-msg**: Edit auto-generated commit message
- **commit-msg**: Validate commit message format
- **post-commit**: Notification after commit
- **pre-push**: Run before pushing to remote
- **post-checkout**: Run after checking out branch
- **pre-rebase**: Run before rebase

### Setting Up Hooks
```bash
# Hooks are stored in .git/hooks
# Make hooks executable
chmod +x .git/hooks/pre-commit

# Use repository-tracked hooks
git config core.hooksPath .githooks/
```

### Sample pre-commit Hook
```bash
#!/bin/sh

# Check for trailing whitespace
if git diff --cached --check | grep -E '\s+$'; then
  echo "Error: Trailing whitespace found in staged changes"
  exit 1
fi

# Run tests
npm test
if [ $? -ne 0 ]; then
  echo "Error: Tests failed, commit aborted"
  exit 1
fi

exit 0
```

## Rewriting History

### Amending Commits
```bash
# Change last commit message
git commit --amend

# Add files to last commit without changing message
git commit --amend --no-edit

# Change author of last commit
git commit --amend --author="New Name <new.email@example.com>"
```

### Filter-branch
```bash
# Remove file from entire history
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch path/to/file' --prune-empty --tag-name-filter cat -- --all

# Change email in all commits
git filter-branch --commit-filter 'if [ "$GIT_AUTHOR_EMAIL" = "old@example.com" ]; then export GIT_AUTHOR_NAME="New Name"; export GIT_AUTHOR_EMAIL="new@example.com"; fi; git commit-tree "$@"' HEAD
```

### BFG Repo Cleaner
```bash
# Remove big files (requires java)
java -jar bfg.jar --strip-blobs-bigger-than 10M repo.git

# Remove sensitive data
java -jar bfg.jar --replace-text passwords.txt repo.git
```

### Rewriting Published History
```bash
# Push rewritten history (use with caution!)
git push --force origin main

# Safer force push
git push --force-with-lease origin main
```

## Submodules and Subtrees

### Submodules
```bash
# Add submodule
git submodule add https://github.com/user/repo.git path/to/submodule

# Initialize and update submodules after cloning
git submodule update --init --recursive

# Update all submodules to latest
git submodule update --remote

# Execute command in each submodule
git submodule foreach 'git checkout main && git pull'
```

### Subtrees
```bash
# Add subtree
git subtree add --prefix=path/to/subtree https://github.com/user/repo.git main --squash

# Update subtree
git subtree pull --prefix=path/to/subtree https://github.com/user/repo.git main --squash

# Push changes to subtree upstream
git subtree push --prefix=path/to/subtree https://github.com/user/repo.git main
```

### Comparison
- **Submodules**: Reference specific commit in external repo
- **Subtrees**: Copy content into main repo

## Worktrees

### Multiple Working Directories
```bash
# Create new worktree
git worktree add ../path/to/worktree branch-name

# Create worktree with new branch
git worktree add -b new-branch ../path/to/worktree

# List worktrees
git worktree list

# Remove worktree
git worktree remove ../path/to/worktree

# Prune worktree information
git worktree prune
```

### Use Cases
- Working on multiple branches simultaneously
- Running builds/tests on different branches
- Making hotfixes while working on features

## Troubleshooting

### Debugging Tools
```bash
# Show objects and references
git show-ref

# Check reflog history
git reflog

# Examine file history
git log -p -- path/to/file

# Find commit that introduced a bug
git bisect start
git bisect bad  # current version is bad
git bisect good v1.0  # v1.0 is known to be good
# Git checks out middle commit
# Test and mark as good or bad
git bisect good  # or git bisect bad
# Continue until git identifies the first bad commit
git bisect reset  # return to original HEAD
```

### Recovering Lost Work
```bash
# Recover lost commits (after reset --hard)
git reflog
git checkout <hash>

# Recover uncommitted changes
git fsck --lost-found

# Recover stashes
git fsck --no-reflog | grep commit | cut -d ' ' -f 3 | xargs git show
```

### Repository Maintenance
```bash
# Verify repository integrity
git fsck

# Clean up and optimize repository
git gc

# Aggressively optimize repository
git gc --aggressive

# Prune unreachable objects
git prune

# Count objects in repository
git count-objects -v
```

## Advanced Workflows

### Feature Branch Workflow
1. Create feature branch from main
2. Make changes and commit
3. Push branch to remote
4. Create Pull Request
5. Review, discuss, and refine
6. Merge or rebase

### Git Flow
- **main**: Production code
- **develop**: Integration branch
- **feature/***: New features
- **release/***: Preparing for release
- **hotfix/***: Emergency fixes for production

```bash
# With git-flow extension
git flow init
git flow feature start feature-name
git flow feature finish feature-name
```

### GitHub Flow
1. Branch from main
2. Make changes and commit
3. Open Pull Request
4. Discuss and review
5. Deploy and test
6. Merge to main

### Trunk-Based Development
- Short-lived feature branches
- Frequent integration to main
- Feature flags for incomplete features
- Emphasis on CI/CD

### Monorepo Strategies
```bash
# Use sparse checkout
git clone --no-checkout repo-url
cd repo
git sparse-checkout set dir1 dir2
git checkout

# Use partial clone
git clone --filter=blob:none repo-url
```
