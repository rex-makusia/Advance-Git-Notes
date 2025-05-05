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

**What it covers:**
This chapter establishes the foundation of Git as a distributed version control system. It explains how Git differs from centralized systems by giving each user a complete copy of the repository. This architecture enables offline work, local commits, and flexible collaboration.

**Key concepts:**
- Git's distributed architecture
- Configuration levels (local, global, system)
- Setting up your Git identity and preferences
- Creating useful aliases for common commands

**Most useful commands:**
```bash
# View your current configuration
git config --list

# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Create time-saving aliases
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset'"

# Set default branch name
git config --global init.defaultBranch main

# Configure diff and merge tools
git config --global core.editor "code --wait"
git config --global diff.tool vscode
```

## Git Object Model

**What it covers:**
This chapter delves into Git's internal data structure—how Git actually stores your project's history. Understanding the object model helps you grasp what's happening behind the scenes with every Git command and troubleshoot complex issues.

**Key concepts:**
- The four core object types: blobs, trees, commits, and tags
- How Git uses SHA-1 hashes to identify objects
- How objects are stored and compressed
- The relationship between these objects

**Most useful commands:**
```bash
# View the content of any Git object
git cat-file -p <hash>

# Determine the type of a Git object
git cat-file -t <hash>

# List contents of a tree object
git ls-tree <tree-hash>

# Show commit history with file changes
git log --stat

# Show a specific commit
git show <commit-hash>

# Explore packfiles
git verify-pack -v .git/objects/pack/*.pack

# Manually trigger garbage collection
git gc
```

## References and Refspecs

**What it covers:**
This chapter explains how Git uses references (refs) to provide human-readable names for SHA-1 hashes. It covers branches, tags, remote references, and special references like HEAD. It also explains refspecs, which determine how references are mapped between repositories during fetch/push operations.

**Key concepts:**
- How branches and tags work as pointers to commits
- Special reference notations (HEAD~n, HEAD^n)
- Remote tracking branches 
- Refspecs for customizing push/fetch behavior

**Most useful commands:**
```bash
# List all references in the repository
git show-ref

# Display where HEAD points
git symbolic-ref HEAD

# Push a local branch to a differently named remote branch
git push origin local-branch:remote-branch

# Delete a remote branch
git push origin :branch-to-delete
# or
git push origin --delete branch-to-delete

# Set up tracking relationship with remote branch
git branch --set-upstream-to=origin/main main

# Create a branch that tracks a remote branch
git checkout -b feature origin/feature

# Fetch a specific branch from remote
git fetch origin feature:refs/remotes/origin/feature
```

## The Index

**What it covers:**
This chapter focuses on Git's staging area (the index), which sits between your working directory and the repository. Understanding the index is crucial for controlling exactly what goes into each commit and for advanced operations.

**Key concepts:**
- How the index acts as a middle ground between working directory and repository
- Staging entire files vs. parts of files
- Manipulating the index directly
- Comparing differences between working directory, index, and HEAD

**Most useful commands:**
```bash
# Add specific parts of a file to the index
git add -p [file]

# Interactive staging for selecting files and hunks
git add -i

# Compare working directory to index
git diff

# Compare index to last commit (HEAD)
git diff --cached

# Remove file from index but keep in working directory
git rm --cached [file]

# Temporarily mark a file as unchanged even if it changes
git update-index --assume-unchanged [file]

# Reset index but preserve working directory changes
git reset
```

## Stashing

**What it covers:**
This chapter explains Git's stash feature, which allows you to temporarily store uncommitted changes when you need to switch contexts. Stashing is especially useful when you need to handle interruptions in your workflow without committing incomplete work.

**Key concepts:**
- Basic stashing and applying stashes
- Named stashes for organization
- Stashing specific files or parts of files
- Creating branches from stashes

**Most useful commands:**
```bash
# Save current changes to stash
git stash

# Save with descriptive message
git stash save "WIP: feature implementation"

# Include untracked files in stash
git stash -u

# Stash specific files only
git stash push path/to/file1 path/to/file2

# Apply most recent stash and remove it from stash list
git stash pop

# Apply specific stash without removing it
git stash apply stash@{2}

# View stash contents
git stash show -p stash@{1}

# Create a branch from a stash
git stash branch new-branch stash@{1}

# Interactively select parts to stash
git stash -p

# List all stashes
git stash list

# Clear all stashes
git stash clear
```

## Rebasing

**What it covers:**
This chapter covers rebasing, a powerful technique for integrating changes from one branch to another by replaying commits. Rebasing creates a cleaner project history compared to merging but requires careful use, especially with shared branches.

**Key concepts:**
- Basic rebasing workflow
- Interactive rebasing for history cleanup
- Resolving conflicts during rebasing
- The "golden rule" of rebasing (don't rebase published history)
- Advanced rebasing techniques like rebasing onto different branches

**Most useful commands:**
```bash
# Rebase current branch onto main
git rebase main

# Interactive rebase for editing, reordering, squashing commits
git rebase -i HEAD~5  # Last 5 commits

# During conflict resolution
git rebase --continue
git rebase --skip
git rebase --abort

# Auto-squash related commits
git commit --fixup=<commit-hash>
git rebase -i --autosquash <base>

# Move a branch to a new base
git rebase --onto new-base old-base branch-to-move
```

## Cherry-picking

**What it covers:**
This chapter explains cherry-picking, which allows you to apply specific commits from one branch to another. This is useful when you want to selectively apply changes rather than merging or rebasing entire branches.

**Key concepts:**
- Selecting and applying individual commits
- Handling conflicts during cherry-picking
- Tracking cherry-picked commits
- Cherry-picking ranges of commits

**Most useful commands:**
```bash
# Apply a single commit to current branch
git cherry-pick <commit-hash>

# Apply multiple commits
git cherry-pick <commit1> <commit2>

# Apply a range of commits
git cherry-pick <start-commit>..<end-commit>

# Cherry-pick without automatically committing
git cherry-pick --no-commit <commit-hash>

# Preserve original authorship information
git cherry-pick -x <commit-hash>

# Sign the cherry-picked commit
git cherry-pick -S <commit-hash>

# Handle cherry-pick conflicts
git cherry-pick --continue
git cherry-pick --abort
```

## Reset and Checkout

**What it covers:**
This chapter explains the differences between reset and checkout, two commonly confused Git commands. Understanding these commands is crucial for manipulating the working directory, the index, and branch pointers safely.

**Key concepts:**
- Three reset modes: soft, mixed, and hard
- How reset moves branch pointers
- How checkout moves HEAD itself
- Path-specific vs. commit-level operations
- Safety considerations for each operation

**Most useful commands:**
```bash
# Soft reset - move branch pointer but keep changes staged
git reset --soft HEAD~1

# Mixed reset (default) - move branch pointer and unstage changes
git reset HEAD~1

# Hard reset - discard all changes
git reset --hard HEAD~1

# Reset a specific file to HEAD version
git reset -- path/to/file

# Reset a specific file to a specific commit version
git reset <commit> -- path/to/file

# Checkout a file from HEAD (discard changes)
git checkout -- path/to/file

# Checkout a file from a specific commit
git checkout <commit> -- path/to/file

# Checkout a specific commit (detached HEAD)
git checkout <commit>
```

## Merge Strategies

**What it covers:**
This chapter explores Git's various merge strategies and options that control how changes from different branches are combined. Understanding these strategies helps you manage complex integrations and resolve conflicts effectively.

**Key concepts:**
- Fast-forward vs. non-fast-forward merges
- Different merge strategies (recursive, resolve, octopus, etc.)
- Merge strategy options (ours, theirs, patience)
- Handling and resolving merge conflicts
- Using merge tools

**Most useful commands:**
```bash
# Default merge (with merge commit)
git merge feature-branch

# Fast-forward only if possible, abort otherwise
git merge --ff-only feature-branch

# Always create a merge commit, even for fast-forward
git merge --no-ff feature-branch

# Favor our version in conflicts
git merge -X ours feature-branch

# Favor their version in conflicts
git merge -X theirs feature-branch

# Use custom merge strategy
git merge -s recursive feature-branch

# Merge multiple branches at once
git merge branch1 branch2 branch3

# Abort a merge in progress
git merge --abort

# Launch configured merge tool for conflict resolution
git mergetool
```

## Hooks

**What it covers:**
This chapter explains Git hooks, which are scripts that Git executes before or after events like commits, pushes, and merges. Hooks can automate tasks, enforce standards, and integrate with other tools in your workflow.

**Key concepts:**
- Client-side vs. server-side hooks
- Common hook types and their use cases
- Sample hook implementations
- Sharing hooks with your team
- Bypassing hooks when needed

**Most useful commands:**
```bash
# Make a hook executable
chmod +x .git/hooks/pre-commit

# Use team-shared hooks
git config core.hooksPath .githooks/

# Bypass pre-commit and pre-push hooks
git commit --no-verify
git push --no-verify

# List available sample hooks
ls -la .git/hooks/

# Run hook manually for testing
sh .git/hooks/pre-commit
```

## Rewriting History

**What it covers:**
This chapter covers techniques for modifying Git history, from simple commit amendments to complex history rewrites. These tools are powerful but require care, especially with shared repositories, as they change commit hashes.

**Key concepts:**
- When and why to rewrite history
- Simple vs. complex history edits
- Tools for history rewriting (amend, rebase, filter-branch)
- Force pushing and its risks
- Best practices for safe history modification

**Most useful commands:**
```bash
# Change the last commit message
git commit --amend

# Add files to the last commit without changing message
git commit --amend --no-edit

# Change author of last commit
git commit --amend --author="New Name <new.email@example.com>"

# Remove a file from entire history
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch path/to/file' --prune-empty -- --all

# Change email in all commits
git filter-branch --env-filter '
    if [ "$GIT_AUTHOR_EMAIL" = "old@example.com" ]
    then
        export GIT_AUTHOR_EMAIL="new@example.com"
    fi
'

# Force push changed history (use with extreme caution)
git push --force origin branch-name

# Safer force push (aborts if remote has changes you don't have)
git push --force-with-lease origin branch-name
```

## Submodules and Subtrees

**What it covers:**
This chapter explains Git's approaches to managing external dependencies: submodules and subtrees. These features allow you to include other repositories within your project while maintaining separation or integration as needed.

**Key concepts:**
- Submodules as references to specific commits in external repos
- Subtrees as copied content integrated into your repo
- When to use each approach
- Common workflows for both approaches
- Advantages and disadvantages of each

**Most useful commands:**
```bash
# Submodules
# Add a submodule
git submodule add https://github.com/user/repo.git path/to/submodule

# Clone a repo with submodules
git clone --recursive https://github.com/user/repo.git

# Initialize submodules after standard clone
git submodule update --init --recursive

# Update submodules to latest remote version
git submodule update --remote

# Execute command in all submodules
git submodule foreach 'git checkout main && git pull'

# Subtrees
# Add a subtree
git subtree add --prefix=path/to/subtree https://github.com/user/repo.git main --squash

# Update subtree from upstream
git subtree pull --prefix=path/to/subtree https://github.com/user/repo.git main --squash

# Push subtree changes back to its origin
git subtree push --prefix=path/to/subtree https://github.com/user/repo.git main
```

## Worktrees

**What it covers:**
This chapter explains Git worktrees, which allow you to check out multiple branches simultaneously in different directories. This feature is particularly useful for working on multiple tasks concurrently without stashing or committing incomplete work.

**Key concepts:**
- Creating and managing multiple working directories
- Linking worktrees to the main repository
- Common worktree workflows
- Benefits over traditional branch switching
- Limitations and best practices

**Most useful commands:**
```bash
# Create a new worktree with existing branch
git worktree add ../path/to/worktree branch-name

# Create a new worktree with new branch
git worktree add -b new-branch ../path/to/worktree

# List all linked worktrees
git worktree list

# Remove a worktree
git worktree remove ../path/to/worktree

# Prune information for removed worktrees
git worktree prune

# Move a worktree to a new location
git worktree move old/path new/path
```

## Troubleshooting

**What it covers:**
This chapter provides tools and techniques for diagnosing and solving problems in Git repositories. It covers strategies for recovering lost work, identifying bugs, and maintaining repository health.

**Key concepts:**
- Debugging tools (reflog, bisect, fsck)
- Recovering from common mistakes
- Finding when bugs were introduced
- Repository maintenance
- Data recovery techniques

**Most useful commands:**
```bash
# View history of HEAD movements (useful for recovery)
git reflog

# Find which commit introduced a bug
git bisect start
git bisect bad  # current version has the bug
git bisect good v1.0  # this version was known to work
# Git checks out a commit halfway between
# Test and mark as good or bad
git bisect good  # or git bisect bad
# Continue until Git identifies the first bad commit
git bisect reset  # return to original state

# Recover uncommitted changes after hard reset
git fsck --lost-found

# Verify integrity of Git database
git fsck

# Optimize repository storage
git gc

# Deep repository optimization (slow but thorough)
git gc --aggressive

# Find unreferenced objects
git prune --dry-run

# Check repository size and statistics
git count-objects -v
```

## Advanced Workflows

**What it covers:**
This chapter explores different Git branching strategies and workflows used by teams. Understanding these patterns helps you choose and implement the right workflow for your project's needs.

**Key concepts:**
- Feature branch workflow
- Git Flow (feature/develop/release/hotfix branches)
- GitHub Flow (simplified branch and PR workflow)
- Trunk-based development
- Monorepo strategies
- Choosing the right workflow for your team

**Most useful commands:**
```bash
# Feature Branch Workflow
git checkout -b feature/new-feature
git push -u origin feature/new-feature
# After PR approval:
git checkout main
git pull
git merge feature/new-feature
git push

# Git Flow (with extension)
git flow init
git flow feature start new-feature
git flow feature finish new-feature
git flow release start 1.0
git flow release finish 1.0
git flow hotfix start critical-fix
git flow hotfix finish critical-fix

# GitHub Flow
git checkout -b feature-branch
# Work, commit, push
# Create PR via GitHub UI
# After merging PR:
git checkout main
git pull

# Trunk-Based Development
git checkout -b small-feature
# Work quickly, get reviewed, merge to main frequently
git checkout main
git pull
git merge small-feature
git push

# Monorepo sparse checkout
git clone --no-checkout repo-url
cd repo
git sparse-checkout set dir1 dir2
git checkout main
```
