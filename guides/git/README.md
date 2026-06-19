# Git for DevOps

## 🎯 Overview

Git is the most widely used distributed version control system. For DevOps engineers, mastering Git is essential for source code management, collaboration, and implementing CI/CD pipelines.

**Duration:** 1 week (Week 11)
**Time Investment:** 3-4 hours/day
**Difficulty:** Beginner to Intermediate
**Prerequisites:** Basic command line knowledge

---

## 📚 What You'll Learn

### Day 1: Git Fundamentals
- Git installation and configuration
- Repository initialization
- Basic commands (add, commit, status)
- Understanding .git directory
- Git workflow basics

### Day 2: Branching and Merging
- Creating and switching branches
- Merging strategies
- Resolving merge conflicts
- Branch management
- Git flow workflow

### Day 3: Remote Repositories
- Working with remotes
- Push and pull operations
- Fetch vs pull
- GitHub/GitLab/Bitbucket
- SSH keys and authentication

### Day 4: Advanced Git Operations
- Rebasing
- Cherry-picking
- Stashing
- Tags and releases
- Git hooks

### Day 5: Collaboration and Best Practices
- Pull requests/Merge requests
- Code review workflows
- Commit message conventions
- .gitignore patterns
- Git aliases

### Day 6: Git for CI/CD
- Git in pipelines
- Monorepo vs multi-repo
- Git submodules
- Git LFS
- Automated versioning

### Day 7: Troubleshooting and Recovery
- Undoing changes
- Recovering lost commits
- Reflog
- Bisect for debugging
- Performance optimization

---

## 🎓 Important Concepts

### 1. Git Architecture

```
Git Architecture
├─ Working Directory
│  └─ Your local files
│
├─ Staging Area (Index)
│  └─ Files ready to commit
│
├─ Local Repository (.git)
│  ├─ Commits
│  ├─ Branches
│  └─ Tags
│
└─ Remote Repository
   ├─ origin (default remote)
   └─ upstream (fork parent)
```

**Three States of Files:**
```
Modified → Staged → Committed

Working Directory → Staging Area → Repository
     (git add)         (git commit)
```

**Git Objects:**
- **Blob:** File content
- **Tree:** Directory structure
- **Commit:** Snapshot with metadata
- **Tag:** Named reference to commit

---

### 2. Basic Git Workflow

```
┌─────────────────────────────────────────┐
│         Git Basic Workflow              │
├─────────────────────────────────────────┤
│  1. git init / git clone                │
│  2. Make changes to files               │
│  3. git add <files>                     │
│  4. git commit -m "message"             │
│  5. git push origin branch              │
└─────────────────────────────────────────┘
```

**Essential Commands:**
```bash
# Initialize repository
git init

# Clone repository
git clone <url>

# Check status
git status

# Add files to staging
git add <file>
git add .

# Commit changes
git commit -m "Commit message"

# View history
git log
git log --oneline --graph --all

# Push to remote
git push origin main

# Pull from remote
git pull origin main
```

---

### 3. Branching Strategies

```
Git Flow
├─ main (production)
├─ develop (integration)
├─ feature/* (new features)
├─ release/* (release preparation)
├─ hotfix/* (production fixes)
└─ bugfix/* (bug fixes)

GitHub Flow (Simplified)
├─ main (production)
└─ feature/* (all changes)

GitLab Flow
├─ main (production)
├─ pre-production
└─ feature/*
```

**Branch Operations:**
```bash
# Create branch
git branch feature-login

# Switch to branch
git checkout feature-login
# or
git switch feature-login

# Create and switch
git checkout -b feature-login

# List branches
git branch
git branch -a  # include remote

# Delete branch
git branch -d feature-login
git branch -D feature-login  # force delete

# Rename branch
git branch -m old-name new-name
```

---

### 4. Merging Strategies

```
Fast-Forward Merge
main:     A---B---C
                   \
feature:            D---E
                         \
result:   A---B---C---D---E

Three-Way Merge
main:     A---B---C-------F
                   \     /
feature:            D---E

Rebase
main:     A---B---C
                   \
feature:            D---E

After rebase:
main:     A---B---C
                   \
feature:            D'---E'
```

**Merge Commands:**
```bash
# Merge branch into current
git merge feature-branch

# Merge with no fast-forward
git merge --no-ff feature-branch

# Abort merge
git merge --abort

# Rebase
git rebase main

# Interactive rebase
git rebase -i HEAD~3

# Continue after resolving conflicts
git rebase --continue

# Abort rebase
git rebase --abort
```

---

### 5. Remote Operations

```
Remote Workflow
Local:    feature-branch
              ↓ (git push)
Remote:   origin/feature-branch
              ↓ (pull request)
Remote:   origin/main
              ↓ (git pull)
Local:    main
```

**Remote Commands:**
```bash
# Add remote
git remote add origin https://github.com/user/repo.git

# List remotes
git remote -v

# Fetch from remote
git fetch origin

# Pull from remote (fetch + merge)
git pull origin main

# Push to remote
git push origin main

# Push new branch
git push -u origin feature-branch

# Delete remote branch
git push origin --delete feature-branch

# Update remote tracking
git remote update origin --prune
```

---

### 6. Commit Best Practices

```
Good Commit Message Structure
┌─────────────────────────────────────┐
│ <type>(<scope>): <subject>          │
│                                     │
│ <body>                              │
│                                     │
│ <footer>                            │
└─────────────────────────────────────┘

Types:
├─ feat: New feature
├─ fix: Bug fix
├─ docs: Documentation
├─ style: Formatting
├─ refactor: Code restructuring
├─ test: Adding tests
├─ chore: Maintenance
└─ perf: Performance improvement
```

**Examples:**
```bash
# Good commit messages
git commit -m "feat(auth): add JWT authentication"
git commit -m "fix(api): resolve null pointer exception in user service"
git commit -m "docs(readme): update installation instructions"

# Detailed commit
git commit -m "feat(auth): add JWT authentication

- Implement JWT token generation
- Add token validation middleware
- Update user login endpoint
- Add refresh token support

Closes #123"
```

---

### 7. Git Hooks

```
Git Hooks Lifecycle
├─ Client-side Hooks
│  ├─ pre-commit (before commit)
│  ├─ prepare-commit-msg (edit message)
│  ├─ commit-msg (validate message)
│  ├─ post-commit (after commit)
│  ├─ pre-push (before push)
│  └─ post-checkout (after checkout)
│
└─ Server-side Hooks
   ├─ pre-receive (before push accepted)
   ├─ update (per branch)
   └─ post-receive (after push accepted)
```

**Example pre-commit Hook:**
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Run linter
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Commit aborted."
    exit 1
fi

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Commit aborted."
    exit 1
fi

exit 0
```

---

## 🔨 Hands-On Exercises

### Exercise 1: Git Setup and First Repository (30 minutes)

**Objective:** Install Git and create your first repository

```bash
# 1. Install Git
# Ubuntu/Debian
sudo apt update
sudo apt install git

# macOS
brew install git

# Verify installation
git --version

# 2. Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "vim"

# Set default branch name
git config --global init.defaultBranch main

# View configuration
git config --list
git config --global --list

# 3. Create repository
mkdir my-project
cd my-project
git init

# 4. Create files
echo "# My Project" > README.md
echo "node_modules/" > .gitignore
echo "*.log" >> .gitignore

# 5. Stage and commit
git add README.md .gitignore
git status
git commit -m "Initial commit: add README and gitignore"

# 6. View history
git log
git log --oneline
git log --graph --oneline --all

# 7. Make more changes
echo "## Installation" >> README.md
echo "npm install" >> README.md

git diff
git add README.md
git commit -m "docs: add installation instructions"

# 8. View commit details
git show HEAD
git show HEAD~1
```

**Challenge:**
Create a repository with:
1. Multiple files
2. Proper .gitignore
3. At least 5 meaningful commits
4. Good commit messages

---

### Exercise 2: Branching and Merging (45 minutes)

**Objective:** Master branch operations and merge strategies

```bash
# 1. Create and switch to new branch
git checkout -b feature/user-auth

# 2. Make changes
cat > auth.js << 'EOF'
function login(username, password) {
    // Login logic
    return true;
}

function logout() {
    // Logout logic
}

module.exports = { login, logout };
EOF

git add auth.js
git commit -m "feat(auth): add login and logout functions"

# 3. Create another branch from main
git checkout main
git checkout -b feature/user-profile

cat > profile.js << 'EOF'
function getProfile(userId) {
    // Get user profile
    return {};
}

module.exports = { getProfile };
EOF

git add profile.js
git commit -m "feat(profile): add user profile function"

# 4. Merge feature/user-auth into main
git checkout main
git merge feature/user-auth

# 5. Merge feature/user-profile (may have conflicts)
git merge feature/user-profile

# 6. View branch graph
git log --graph --oneline --all --decorate

# 7. Create a merge conflict scenario
git checkout -b feature/readme-update
echo "## Features" >> README.md
echo "- User authentication" >> README.md
git add README.md
git commit -m "docs: add features section"

git checkout main
echo "## About" >> README.md
echo "This is a demo project" >> README.md
git add README.md
git commit -m "docs: add about section"

# 8. Merge and resolve conflict
git merge feature/readme-update
# Conflict! Edit README.md to resolve

git add README.md
git commit -m "merge: resolve README conflict"

# 9. Delete merged branches
git branch -d feature/user-auth
git branch -d feature/user-profile
git branch -d feature/readme-update

# 10. View final state
git log --graph --oneline --all
```

**Challenge:**
Practice different merge strategies:
1. Fast-forward merge
2. Three-way merge
3. Resolve complex conflicts
4. Use merge --no-ff

---

### Exercise 3: Working with Remotes (45 minutes)

**Objective:** Connect to remote repositories and collaborate

```bash
# 1. Create repository on GitHub/GitLab
# (Use web interface)

# 2. Add remote
git remote add origin https://github.com/username/my-project.git

# 3. Verify remote
git remote -v

# 4. Push to remote
git push -u origin main

# 5. Create and push new branch
git checkout -b feature/api-integration
echo "API integration code" > api.js
git add api.js
git commit -m "feat(api): add API integration"
git push -u origin feature/api-integration

# 6. Simulate collaboration (clone in different directory)
cd ..
git clone https://github.com/username/my-project.git my-project-clone
cd my-project-clone

# 7. Make changes in clone
git checkout -b feature/database
echo "Database code" > db.js
git add db.js
git commit -m "feat(db): add database module"
git push -u origin feature/database

# 8. Back to original, fetch changes
cd ../my-project
git fetch origin

# 9. View remote branches
git branch -r
git branch -a

# 10. Pull changes
git pull origin main

# 11. Checkout remote branch
git checkout feature/database

# 12. Delete remote branch
git push origin --delete feature/api-integration

# 13. Update local tracking
git fetch --prune
```

**Challenge:**
Set up a complete workflow:
1. Fork a repository
2. Add upstream remote
3. Create feature branch
4. Push and create pull request
5. Sync with upstream

---

### Exercise 4: Advanced Git Operations (60 minutes)

**Objective:** Master rebasing, stashing, and cherry-picking

```bash
# 1. Interactive Rebase
git checkout -b feature/cleanup
echo "file1" > file1.txt
git add file1.txt
git commit -m "add file1"

echo "file2" > file2.txt
git add file2.txt
git commit -m "add file2"

echo "file3" > file3.txt
git add file3.txt
git commit -m "add file3"

# Rebase last 3 commits
git rebase -i HEAD~3
# In editor: squash, reword, or reorder commits

# 2. Stashing
git checkout main
echo "work in progress" > wip.txt
git add wip.txt

# Stash changes
git stash save "WIP: working on new feature"

# List stashes
git stash list

# Apply stash
git stash apply stash@{0}

# Pop stash (apply and remove)
git stash pop

# Drop stash
git stash drop stash@{0}

# 3. Cherry-picking
git checkout -b feature/cherry-pick
echo "important fix" > fix.txt
git add fix.txt
git commit -m "fix: critical bug fix"

# Get commit hash
COMMIT_HASH=$(git rev-parse HEAD)

# Switch to main and cherry-pick
git checkout main
git cherry-pick $COMMIT_HASH

# 4. Tagging
git tag v1.0.0
git tag -a v1.0.1 -m "Release version 1.0.1"

# List tags
git tag

# Push tags
git push origin v1.0.0
git push origin --tags

# Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0

# 5. Amending commits
echo "update" >> README.md
git add README.md
git commit -m "docs: update readme"

# Oops, forgot something
echo "more updates" >> README.md
git add README.md
git commit --amend --no-edit

# 6. Reset operations
# Soft reset (keep changes staged)
git reset --soft HEAD~1

# Mixed reset (keep changes unstaged)
git reset HEAD~1

# Hard reset (discard changes)
git reset --hard HEAD~1
```

**Challenge:**
Practice advanced scenarios:
1. Rebase feature branch onto updated main
2. Resolve rebase conflicts
3. Use interactive rebase to clean history
4. Cherry-pick multiple commits

---

### Exercise 5: Git Hooks and Automation (45 minutes)

**Objective:** Implement Git hooks for automation

```bash
# 1. Create pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

echo "Running pre-commit checks..."

# Check for console.log
if git diff --cached | grep -i "console.log"; then
    echo "Error: console.log found in staged files"
    exit 1
fi

# Check for TODO comments
if git diff --cached | grep -i "TODO"; then
    echo "Warning: TODO comments found"
fi

# Run linter (if available)
if command -v eslint &> /dev/null; then
    eslint .
    if [ $? -ne 0 ]; then
        echo "Linting failed"
        exit 1
    fi
fi

echo "Pre-commit checks passed!"
exit 0
EOF

chmod +x .git/hooks/pre-commit

# 2. Create commit-msg hook
cat > .git/hooks/commit-msg << 'EOF'
#!/bin/bash

commit_msg_file=$1
commit_msg=$(cat "$commit_msg_file")

# Check commit message format
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+"; then
    echo "Error: Commit message must follow conventional commits format"
    echo "Example: feat(auth): add login functionality"
    exit 1
fi

# Check message length
if [ ${#commit_msg} -lt 10 ]; then
    echo "Error: Commit message too short (minimum 10 characters)"
    exit 1
fi

exit 0
EOF

chmod +x .git/hooks/commit-msg

# 3. Create pre-push hook
cat > .git/hooks/pre-push << 'EOF'
#!/bin/bash

echo "Running pre-push checks..."

# Run tests
if [ -f "package.json" ]; then
    npm test
    if [ $? -ne 0 ]; then
        echo "Tests failed. Push aborted."
        exit 1
    fi
fi

echo "Pre-push checks passed!"
exit 0
EOF

chmod +x .git/hooks/pre-push

# 4. Test hooks
echo "console.log('test')" > test.js
git add test.js
git commit -m "test commit"  # Should fail

# Fix and try again
echo "// test" > test.js
git add test.js
git commit -m "feat(test): add test file"  # Should succeed

# 5. Share hooks with team (using templates)
mkdir -p .githooks
cp .git/hooks/pre-commit .githooks/
cp .git/hooks/commit-msg .githooks/

# Configure git to use custom hooks directory
git config core.hooksPath .githooks
```

**Challenge:**
Create comprehensive hooks:
1. Pre-commit: Run linter and formatter
2. Commit-msg: Enforce message format
3. Pre-push: Run full test suite
4. Post-commit: Update changelog

---

### Exercise 6: Git for CI/CD (45 minutes)

**Objective:** Integrate Git with CI/CD pipelines

```bash
# 1. Create .github/workflows/ci.yml
mkdir -p .github/workflows
cat > .github/workflows/ci.yml << 'EOF'
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      
      - name: Install dependencies
        run: npm install
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v2
      
      - name: Deploy to production
        run: echo "Deploying to production..."
EOF

# 2. Create version bump script
cat > bump-version.sh << 'EOF'
#!/bin/bash

# Get current version
current_version=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
echo "Current version: $current_version"

# Increment version
IFS='.' read -ra VERSION <<< "${current_version#v}"
major=${VERSION[0]}
minor=${VERSION[1]}
patch=${VERSION[2]}

# Determine bump type
case "$1" in
    major)
        major=$((major + 1))
        minor=0
        patch=0
        ;;
    minor)
        minor=$((minor + 1))
        patch=0
        ;;
    patch|*)
        patch=$((patch + 1))
        ;;
esac

new_version="v$major.$minor.$patch"
echo "New version: $new_version"

# Create tag
git tag -a "$new_version" -m "Release $new_version"
git push origin "$new_version"
EOF

chmod +x bump-version.sh

# 3. Create changelog generator
cat > generate-changelog.sh << 'EOF'
#!/bin/bash

# Get last tag
last_tag=$(git describe --tags --abbrev=0 2>/dev/null)

if [ -z "$last_tag" ]; then
    echo "No previous tag found"
    commits=$(git log --pretty=format:"- %s (%h)" --reverse)
else
    echo "Changes since $last_tag:"
    commits=$(git log $last_tag..HEAD --pretty=format:"- %s (%h)" --reverse)
fi

# Group by type
echo "## Features"
echo "$commits" | grep "^- feat"

echo ""
echo "## Bug Fixes"
echo "$commits" | grep "^- fix"

echo ""
echo "## Documentation"
echo "$commits" | grep "^- docs"

echo ""
echo "## Other Changes"
echo "$commits" | grep -v "^- \(feat\|fix\|docs\)"
EOF

chmod +x generate-changelog.sh

# 4. Create release script
cat > release.sh << 'EOF'
#!/bin/bash

# Ensure we're on main branch
current_branch=$(git branch --show-current)
if [ "$current_branch" != "main" ]; then
    echo "Error: Must be on main branch"
    exit 1
fi

# Ensure working directory is clean
if [ -n "$(git status --porcelain)" ]; then
    echo "Error: Working directory not clean"
    exit 1
fi

# Pull latest changes
git pull origin main

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed"
    exit 1
fi

# Bump version
./bump-version.sh $1

# Generate changelog
./generate-changelog.sh > CHANGELOG.md
git add CHANGELOG.md
git commit -m "docs: update changelog for release"
git push origin main

echo "Release complete!"
EOF

chmod +x release.sh
```

**Challenge:**
Set up complete CI/CD:
1. Automated testing on PR
2. Automated deployment on merge
3. Version bumping
4. Changelog generation
5. Release tagging

---

### Exercise 7: Git Troubleshooting (45 minutes)

**Objective:** Learn to recover from common Git problems

```bash
# 1. Undo last commit (keep changes)
git reset --soft HEAD~1

# 2. Undo last commit (discard changes)
git reset --hard HEAD~1

# 3. Recover deleted branch
git reflog
git checkout -b recovered-branch <commit-hash>

# 4. Find lost commits
git reflog
git cherry-pick <commit-hash>

# 5. Undo pushed commit
git revert HEAD
git push origin main

# 6. Fix wrong commit message
git commit --amend -m "Correct message"
git push --force origin branch-name

# 7. Remove file from history
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
# or use git-filter-repo (recommended)
git filter-repo --path passwords.txt --invert-paths

# 8. Find bug with bisect
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
# Test each commit
git bisect good  # or bad
# Repeat until found
git bisect reset

# 9. Clean untracked files
git clean -n  # dry run
git clean -f  # remove files
git clean -fd  # remove files and directories

# 10. Recover from detached HEAD
git checkout -b temp-branch
git checkout main
git merge temp-branch
```

---

## 🎯 Practice Projects

### Project 1: Open Source Contribution
Contribute to an open source project:
- Fork repository
- Create feature branch
- Make meaningful changes
- Submit pull request
- Respond to code review
- Follow project guidelines

### Project 2: Team Collaboration Workflow
Set up team workflow:
- Define branching strategy
- Create PR templates
- Set up code review process
- Implement Git hooks
- Configure CI/CD
- Document workflow

### Project 3: Monorepo Management
Manage monorepo:
- Multiple projects in one repo
- Shared dependencies
- Independent versioning
- Selective CI/CD
- Git submodules or subtrees

### Project 4: Git Migration
Migrate from another VCS:
- Preserve history
- Convert branches and tags
- Migrate issues/PRs
- Update documentation
- Train team

---

## 📝 Best Practices

### 1. Commit Practices
- Commit often, push regularly
- Write meaningful commit messages
- Keep commits atomic and focused
- Use conventional commits format
- Don't commit sensitive data
- Review changes before committing

### 2. Branch Management
- Use descriptive branch names
- Delete merged branches
- Keep branches short-lived
- Rebase feature branches regularly
- Protect main/production branches
- Use branch naming conventions

### 3. Collaboration
- Pull before push
- Resolve conflicts promptly
- Review code thoroughly
- Communicate in PRs
- Follow team conventions
- Document decisions

### 4. Security
- Never commit secrets
- Use .gitignore properly
- Sign commits with GPG
- Use SSH keys
- Review dependencies
- Enable branch protection

---

## ✅ Completion Checklist

### Day 1
- [ ] Install and configure Git
- [ ] Create first repository
- [ ] Make commits
- [ ] View history
- [ ] Understand Git workflow

### Day 2
- [ ] Create branches
- [ ] Merge branches
- [ ] Resolve conflicts
- [ ] Delete branches
- [ ] Understand Git flow

### Day 3
- [ ] Add remote repository
- [ ] Push and pull
- [ ] Work with forks
- [ ] Create pull requests
- [ ] Collaborate with team

### Day 4
- [ ] Use rebase
- [ ] Cherry-pick commits
- [ ] Stash changes
- [ ] Create tags
- [ ] Amend commits

### Day 5
- [ ] Write good commit messages
- [ ] Use .gitignore
- [ ] Create aliases
- [ ] Review code
- [ ] Follow conventions

### Day 6
- [ ] Set up Git hooks
- [ ] Integrate with CI/CD
- [ ] Automate versioning
- [ ] Generate changelogs
- [ ] Implement workflows

### Day 7
- [ ] Undo changes
- [ ] Recover commits
- [ ] Use reflog
- [ ] Debug with bisect
- [ ] Clean repository

---

**Next:** [Jenkins →](../jenkins/README.md)