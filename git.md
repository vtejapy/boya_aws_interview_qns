
## Foundation Level Questions (0-2 Years Experience)

### Q1: What is Git and how is it different from other version control systems?

**Answer:**
Git is a **distributed version control system** that tracks changes in source code during software development. 

**Key Differences:**
- **Distributed**: Every developer has a complete copy of the project history locally
- **Speed**: Most operations are local, making them very fast
- **Branching**: Extremely efficient branching and merging
- **Data Integrity**: Uses SHA-1 checksums to ensure data integrity
- **Staging Area**: Has a staging area for preparing commits

**vs Centralized VCS (like SVN):**
- No single point of failure
- Can work offline
- Better collaboration capabilities
- More complex but more powerful

---

### Q2: Explain the three states of files in Git and the Git workflow.

**Answer:**
**Three States:**
1. **Modified**: File has been changed but not yet committed
2. **Staged**: Modified file marked to go into next commit
3. **Committed**: Data is safely stored in local Git directory

**Git Workflow:**
1. **Working Directory**: Where you modify files
2. **Staging Area (Index)**: Where you prepare files for commit
3. **Git Repository**: Where Git stores metadata and object database

**Basic Commands:**
- `git add` - moves files from working directory to staging area
- `git commit` - moves files from staging area to repository
- `git status` - shows the state of files

---

### Q3: What's the difference between `git fetch` and `git pull`?

**Answer:**
**git fetch:**
- Downloads data from remote repository
- Does NOT automatically merge changes
- Updates remote tracking branches
- Safe operation - won't change your working files

**git pull:**
- Downloads data from remote repository
- Automatically merges changes into current branch
- Equivalent to `git fetch` + `git merge`
- Can cause merge conflicts

**When to use:**
- Use `git fetch` when you want to see what others have done before merging
- Use `git pull` when you're sure you want to merge remote changes immediately

**Scenario:** "You're working on a feature and want to see if anyone pushed changes to main branch"
**Solution:** Use `git fetch` first to see changes, then decide whether to merge

---

## Intermediate Level Questions (2-4 Years Experience)

### Q4: Explain different types of Git merges and when to use each.

**Answer:**
**Types of Merges:**

**1. Fast-Forward Merge:**
- Happens when target branch hasn't diverged
- Simply moves the branch pointer forward
- No merge commit created
- Clean, linear history

**2. Three-Way Merge:**
- When both branches have diverged
- Creates a new merge commit
- Preserves branch history
- Shows where branches merged

**3. Squash Merge:**
- Combines all commits from feature branch into single commit
- Clean history but loses individual commit information
- Good for feature branches with many small commits

**When to use:**
- **Fast-forward**: Simple features, hotfixes
- **Three-way**: Complex features where history matters
- **Squash**: Cleaning up messy feature branch history

**Problem:** "Your feature branch has 20 commits with unclear messages"
**Solution:** Use squash merge to combine into one meaningful commit

---

### Q5: How do you resolve merge conflicts? Walk through the process.

**Answer:**
**Merge Conflict Resolution Process:**

**Step 1: Identify Conflicts**
- Git stops merge process
- `git status` shows conflicted files
- Files contain conflict markers

**Step 2: Open Conflicted Files**
```
<<<<<<< HEAD
Your current changes
=======
Incoming changes
>>>>>>> feature-branch
```

**Step 3: Resolve Conflicts**
- Choose which changes to keep
- Edit file to desired state
- Remove conflict markers

**Step 4: Complete Merge**
- `git add <resolved-files>`
- `git commit` (will open editor with merge message)

**Best Practices:**
- Use `git config --global merge.conflictstyle diff3` for better conflict view
- Consider using merge tools like VS Code, Sublime Merge
- Communicate with team members about conflicts
- Test thoroughly after resolution

**Scenario:** "Two developers modified the same function in different ways"
**Solution:** Review both changes, understand the intent, merge manually, and test

---

### Q6: What's the difference between `git rebase` and `git merge`?

**Answer:**
**Git Merge:**
- Creates a new merge commit
- Preserves complete history
- Shows branching structure
- Non-destructive operation
- Better for public/shared branches

**Git Rebase:**
- Replays commits on top of another branch
- Creates linear history
- No merge commit
- Rewrites commit history
- Better for private/feature branches

**When to Use:**
- **Merge**: Public branches, preserving context, team collaboration
- **Rebase**: Private branches, cleaning history, before pushing

**Golden Rule of Rebase:** Never rebase public branches that others are working on

**Scenario:** "Your feature branch is behind main by 10 commits"
**Solution:** 
- If private branch: `git rebase main` for clean history
- If shared branch: `git merge main` to avoid rewriting shared history

---

## Advanced Level Questions (4+ Years Experience)

### Q7: Explain Git workflow strategies and their use cases.

**Answer:**
**1. GitFlow Workflow:**
- **Branches**: main, develop, feature/, release/, hotfix/
- **Use Case**: Large teams, scheduled releases, complex projects
- **Pros**: Structured, clear release process, good for enterprise
- **Cons**: Complex, overhead for simple projects

**2. GitHub Flow:**
- **Branches**: main + feature branches
- **Use Case**: Continuous deployment, web applications, small teams
- **Pros**: Simple, fast iteration, good for CI/CD
- **Cons**: Less control over releases, can be chaotic for large teams

**3. GitLab Flow:**
- **Branches**: main + environment branches (staging, production)
- **Use Case**: Need for multiple environments, controlled deployments
- **Pros**: Balances simplicity and control
- **Cons**: More complex than GitHub Flow

**4. Trunk-Based Development:**
- **Branches**: Mostly main branch, very short-lived feature branches
- **Use Case**: Continuous integration, feature flags
- **Pros**: Simple, fast integration, reduces merge conflicts
- **Cons**: Requires discipline, good testing, feature flags

**Choosing Strategy:**
- Team size and experience
- Release cycle requirements
- Project complexity
- Deployment frequency

---

### Q8: How do you handle a situation where you accidentally committed to the wrong branch?

**Answer:**
**Scenario Analysis:**
- Committed to main instead of feature branch
- Need to move commits without losing work
- Keep history clean

**Solution Steps:**

**Step 1: Identify Commits to Move**
```bash
git log --oneline -n 5  # See recent commits
```

**Step 2: Create/Switch to Correct Branch**
```bash
git checkout -b feature-branch  # Create new branch from current position
# OR
git checkout existing-feature-branch
```

**Step 3: Cherry-pick Commits (if needed)**
```bash
git cherry-pick <commit-hash>  # Apply specific commits
```

**Step 4: Reset Original Branch**
```bash
git checkout main
git reset --hard HEAD~n  # Remove n commits from main
```

**Alternative Method (if commits not yet pushed):**
```bash
git checkout main
git reset --soft HEAD~n  # Keep changes but remove commits
git stash  # Save changes
git checkout feature-branch
git stash pop  # Apply changes to correct branch
git commit -m "Correct message"
```

**Prevention:**
- Always check current branch before committing
- Use Git hooks for validation
- Configure shell prompt to show current branch

---

### Q9: Your team lead asks you to remove sensitive data from Git history. How do you handle this?

**Answer:**
**Immediate Assessment:**
- Has the commit been pushed to remote?
- How far back is the sensitive data?
- How many people have access to the repository?

**Solution Approaches:**

**Method 1: BFG Repo-Cleaner (Recommended)**
```bash
# Download BFG tool
java -jar bfg.jar --delete-files secret-file.txt repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```

**Method 2: Git Filter-Branch**
```bash
git filter-branch --force --index-filter \
'git rm --cached --ignore-unmatch secret-file.txt' \
--prune-empty --tag-name-filter cat -- --all
```

**Method 3: Interactive Rebase (for recent commits)**
```bash
git rebase -i HEAD~n  # Edit history
# Change 'pick' to 'edit' for problematic commit
# Remove sensitive data
git add .
git rebase --continue
```

**Post-Cleanup Actions:**
1. **Force push** (coordinate with team first)
2. **Rotate compromised secrets** immediately
3. **Notify team** about force push
4. **Review security practices** to prevent recurrence

**Important Notes:**
- Force push affects everyone's local repositories
- All team members need to re-clone or reset their branches
- Consider the data as potentially compromised

---

### Q10: Explain how you would use Git hooks in a real project.

**Answer:**
**Git Hooks Overview:**
Hooks are scripts that run automatically at certain points in Git workflow.

**Common Hooks and Use Cases:**

**1. Pre-commit Hook:**
- **Purpose**: Validate code before committing
- **Use Cases**: 
  - Code linting and formatting
  - Running unit tests
  - Checking for secrets/sensitive data
  - Validating commit message format

**Example Implementation:**
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run linter
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Please fix errors before committing."
    exit 1
fi

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Please fix tests before committing."
    exit 1
fi
```

**2. Pre-push Hook:**
- **Purpose**: Validate before pushing to remote
- **Use Cases**:
  - Run full test suite
  - Security scans
  - Prevent direct pushes to main branch

**3. Post-receive Hook (Server-side):**
- **Purpose**: Actions after receiving push
- **Use Cases**:
  - Trigger CI/CD pipelines
  - Send notifications
  - Deploy to staging environments

**4. Commit-msg Hook:**
- **Purpose**: Validate commit messages
- **Use Cases**:
  - Enforce conventional commit format
  - Check for issue references
  - Validate message length

**Implementation Strategy:**
1. **Shared Hooks**: Store in repository and symlink
2. **Tool Integration**: Use tools like Husky for JavaScript projects
3. **Team Alignment**: Ensure all team members use same hooks
4. **Bypass Options**: Allow emergency bypasses with `--no-verify`

---

## Expert Level Questions (5+ Years Experience)

### Q11: You discover that a commit from 6 months ago introduced a subtle bug that's now causing production issues. How do you handle this?

**Answer:**
**Problem Analysis:**
- Bug is old and many commits have been built on top
- Direct revert might cause conflicts
- Need to fix without breaking existing functionality

**Investigation Steps:**

**Step 1: Identify the Problematic Commit**
```bash
git log --oneline --since="6 months ago" | grep "keyword"
git bisect start
git bisect bad HEAD
git bisect good <known-good-commit>
# Git will help you find the exact commit
```

**Step 2: Assess Impact**
```bash
git show <problematic-commit>  # See what changed
git log --oneline <commit>..HEAD  # See subsequent commits
git blame <file>  # See if code was modified since
```

**Solution Approaches:**

**Option 1: Revert (Safest)**
```bash
git revert <problematic-commit>
# This creates a new commit that undoes the changes
# Handle any conflicts that arise
```

**Option 2: Cherry-pick Fix from Another Branch**
```bash
# If fix exists elsewhere
git cherry-pick <fix-commit>
```

**Option 3: Create Targeted Fix**
```bash
# Create minimal fix for the specific issue
git checkout -b hotfix/production-bug
# Make minimal changes to fix the bug
git commit -m "Fix: Resolve production bug from commit abc123"
```

**Deployment Strategy:**
1. **Test thoroughly** in staging environment
2. **Create hotfix branch** for urgent deployment
3. **Merge to main** after validation
4. **Deploy to production** during maintenance window
5. **Monitor** for any side effects

**Prevention Measures:**
- Improve testing coverage for similar scenarios
- Add regression tests
- Consider implementing feature flags for risky changes

---

### Q12: Your Git repository has become very large and slow. How do you optimize it?

**Answer:**
**Diagnosis Steps:**

**Step 1: Analyze Repository Size**
```bash
git count-objects -vH  # See object counts and sizes
git gc --aggressive  # Clean up and compact
du -sh .git  # Check .git directory size
```

**Step 2: Identify Large Files**
```bash
git rev-list --objects --all | \
grep "$(git verify-pack -v .git/objects/pack/*.idx | \
sort -k 3 -nr | head -10 | awk '{print $1}')"
```

**Optimization Strategies:**

**1. Large File Management:**
- **Git LFS (Large File Storage)**: For binary files, media assets
- **Remove large files** from history using BFG or filter-branch
- **Gitignore patterns** for build artifacts, dependencies

**2. Repository Cleanup:**
```bash
# Aggressive garbage collection
git gc --aggressive --prune=now

# Remove unreachable objects
git fsck --unreachable
git prune

# Clean reflog
git reflog expire --expire=now --all
```

**3. History Optimization:**
- **Shallow clones** for CI/CD: `git clone --depth 1`
- **Partial clones**: `git clone --filter=blob:none`
- **Archive old branches** instead of keeping them

**4. Workflow Improvements:**
- **Branch cleanup policies**: Delete merged branches
- **Squash small commits**: Reduce commit count
- **Separate repositories**: Split monoliths into smaller repos

**5. Advanced Techniques:**
- **Git worktrees**: Multiple working directories
- **Sparse checkout**: Only checkout needed files
- **Split repositories**: Use submodules or subtrees

**Monitoring and Maintenance:**
- **Regular cleanup scripts**: Automated repository maintenance
- **Size monitoring**: Track repository growth over time
- **Team education**: Best practices for large repositories

---

### Q13: Design a Git strategy for a team of 50 developers working on a microservices architecture.

**Answer:**
**Architecture Considerations:**
- Multiple services, each potentially deployable independently
- Different teams working on different services
- Need for code sharing between services
- Different release cycles for different services

**Repository Strategy:**

**Option 1: Monorepo Approach**
- **Pros**: Shared libraries, unified versioning, atomic commits across services
- **Cons**: Large repository size, complex CI/CD, potential conflicts
- **Tools**: Nx, Lerna, Bazel for build optimization

**Option 2: Multi-Repo Approach (Recommended)**
- **Pros**: Service isolation, independent deployments, team autonomy
- **Cons**: Shared library management, cross-service changes complexity
- **Structure**: One repo per service + shared libraries repo

**Branching Strategy: GitFlow Modified for Microservices**

**Main Branches per Repository:**
- `main`: Production-ready code
- `develop`: Integration branch for next release
- `staging`: Pre-production testing

**Feature Branch Strategy:**
```
feature/service-name/feature-description
hotfix/service-name/issue-description
release/service-name/version-number
```

**Shared Libraries Management:**
- **Separate repository** for shared components
- **Semantic versioning** for shared libraries
- **Dependency management** through package managers
- **API contracts** versioning for service communication

**CI/CD Integration:**
- **Monorepo**: Build only changed services
- **Multi-repo**: Independent pipelines per service
- **Cross-service testing**: Integration test pipelines
- **Deployment coordination**: Service dependency management

**Code Review Process:**
- **Service ownership**: Teams own their service repositories
- **Cross-team reviews**: For shared libraries and contracts
- **Security reviews**: For sensitive services
- **Architecture reviews**: For major changes

**Coordination Mechanisms:**
- **API-first development**: Define contracts before implementation
- **Regular integration**: Automated testing of service interactions
- **Communication protocols**: Teams notify of breaking changes
- **Documentation**: Centralized service catalog and API docs

---

## Troubleshooting & Problem-Solving Scenarios

### Q14: A developer on your team ran `git reset --hard HEAD~5` and lost important work. How do you recover it?

**Answer:**
**Assessment:**
- Understanding what `git reset --hard HEAD~5` does
- Checking if work was committed or just staged/unstaged
- Determining recovery options

**Recovery Process:**

**Step 1: Check Reflog**
```bash
git reflog  # Shows history of HEAD movements
# Look for the commit before the reset
```

**Step 2: Identify Lost Commits**
```bash
git log --oneline --all --graph  # See all branches and commits
git fsck --lost-found  # Find dangling commits
```

**Step 3: Recovery Options**

**Option 1: Reset to Previous State**
```bash
git reset --hard HEAD@{1}  # Reset to state before the reset
# OR
git reset --hard <commit-hash>  # Reset to specific commit from reflog
```

**Option 2: Cherry-pick Lost Commits**
```bash
git cherry-pick <lost-commit-hash>  # Apply specific commits
git cherry-pick <commit1>..<commit5>  # Apply range of commits
```

**Option 3: Create Branch from Lost Commit**
```bash
git branch recovery-branch <lost-commit-hash>
git checkout recovery-branch
```

**Prevention Measures:**
- **Regular commits**: Commit work frequently
- **Use git stash**: For temporary changes
- **Backup important branches**: Push to remote frequently
- **Team education**: Understanding destructive commands
- **Git hooks**: Warn before destructive operations

**If Recovery Fails:**
- Check if code exists in IDE history
- Look for automatic backups from development tools
- Review if code was shared via other means (email, slack)

---

### Q15: Your team has been working on a feature branch for 3 months. Now you need to merge it to main, but there are hundreds of conflicts. How do you approach this?

**Answer:**
**Problem Analysis:**
- Long-lived feature branch has diverged significantly
- High conflict volume indicates substantial changes in both branches
- Need systematic approach to avoid mistakes

**Strategic Approach:**

**Step 1: Preparation**
```bash
# Update both branches
git checkout main
git pull origin main
git checkout feature-branch
git pull origin feature-branch

# Create backup branch
git checkout -b feature-branch-backup
git checkout feature-branch
```

**Step 2: Incremental Integration**
```bash
# Strategy: Rebase in smaller chunks
git rebase -i main~20  # Start with recent commits from main
# Resolve conflicts in smaller batches
```

**Step 3: Systematic Conflict Resolution**

**Categorize Conflicts:**
1. **Auto-resolvable**: Different files, non-overlapping changes
2. **Simple conflicts**: One-line changes, formatting differences
3. **Complex conflicts**: Logic changes, refactoring overlaps
4. **Architectural conflicts**: Major structural differences

**Resolution Process:**
```bash
# Use merge tools for better visualization
git config --global merge.tool vimdiff  # or your preferred tool
git mergetool  # Opens tool for each conflict

# For complex conflicts, involve original authors
git log --oneline <file>  # See who made changes
git blame <file>  # Understand change context
```

**Step 4: Validation**
- **Compile and test** after each conflict resolution batch
- **Run full test suite** before final merge
- **Code review** of conflict resolutions
- **Functional testing** of integrated features

**Alternative Strategies:**

**Option 1: Squash and Rewrite**
- Create new branch from main
- Manually apply feature changes
- Cleaner but loses detailed history

**Option 2: Break into Smaller Merges**
- Identify logical chunks in feature branch
- Merge incrementally with testing between

**Prevention for Future:**
- **Regular integration**: Merge/rebase main into feature branches weekly
- **Feature flags**: Deploy incomplete features safely
- **Shorter feature cycles**: Break large features into smaller pieces
- **Communication**: Coordinate with teams making conflicting changes

---

## Latest Git Features & Best Practices (2024-2025)

### Q16: What are the latest Git features you've used, and how do they improve your workflow?

**Answer:**
**Recent Git Features (2022-2025):**

**1. Scalar (Git 2.38+)**
- **Purpose**: Optimizes Git for very large repositories
- **Benefits**: Faster operations, reduced disk usage
- **Use Case**: Monorepos, large codebases with many files

**2. Git Maintenance (Git 2.30+)**
- **Purpose**: Background repository optimization
- **Features**: Automatic garbage collection, pack optimization
- **Benefits**: Keeps repository performant without manual intervention

**3. Improved Sparse Checkout (Git 2.25+)**
- **Purpose**: Work with subset of files in large repositories
- **Benefits**: Faster clones, reduced disk usage
- **Use Case**: Microservices in monorepos, large codebases

**4. Multi-pack Index (Git 2.24+)**
- **Purpose**: Better performance with many pack files
- **Benefits**: Faster object lookups, improved gc performance

**5. Enhanced Git Switch/Restore (Git 2.23+)**
- **Purpose**: Clearer separation of branch switching and file restoration
- **Benefits**: Less confusing than `git checkout` for new users

**Modern Workflow Improvements:**

**Configuration:**
```bash
# Modern Git configuration
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global fetch.prune true
git config --global rebase.autostash true
```

**Security Enhancements:**
- **Signed commits**: Using GPG/SSH signing for verification
- **Credential helpers**: Better security for authentication
- **Protected branches**: Advanced branch protection rules

**Integration with Modern Tools:**
- **GitHub Codespaces**: Cloud development environments
- **VS Code Git integration**: Enhanced IDE features
- **GitHub Actions**: CI/CD integration with Git events
- **Dependabot**: Automated dependency updates

---

### Q17: How do you handle secrets and sensitive data in Git repositories?

**Answer:**
**Prevention Strategies:**

**1. Gitignore Patterns**
```bash
# .gitignore
*.env
*.key
*.pem
config/secrets.yml
.aws/credentials
```

**2. Environment Variables**
- Store secrets in environment variables
- Use .env files (but gitignore them)
- Container orchestration secrets management

**3. External Secret Management**
- **HashiCorp Vault**: Enterprise secret management
- **AWS Secrets Manager**: Cloud-native solution
- **Azure Key Vault**: Microsoft ecosystem
- **Kubernetes Secrets**: Container orchestration

**Detection and Prevention Tools:**

**1. Git Hooks**
```bash
# Pre-commit hook to detect secrets
#!/bin/sh
# Check for potential secrets
if git diff --cached --name-only | xargs grep -l "password\|secret\|key" > /dev/null; then
    echo "Potential secret detected. Please review."
    exit 1
fi
```

**2. Automated Scanning**
- **GitLeaks**: Detect secrets in Git repos
- **TruffleHog**: Search for high-entropy strings
- **Detect-secrets**: Python tool for secret detection
- **GitHub Secret Scanning**: Built-in GitHub feature

**If Secrets Are Committed:**

**Immediate Actions:**
1. **Rotate the secret** immediately
2. **Remove from Git history** using BFG or filter-branch
3. **Force push** (coordinate with team)
4. **Audit access logs** to see who might have seen the secret

**Long-term Prevention:**
- **Team training** on secret management
- **Code review processes** to catch secrets
- **Automated scanning** in CI/CD pipelines
- **Secret management policies** and procedures

---

## Quick Fire Questions & Answers

### Q18: What's the difference between HEAD, head, and heads in Git?

**Answer:**
- **HEAD**: Pointer to current branch's latest commit (symbolic reference)
- **head**: Common term for the tip of a branch
- **heads**: Directory containing branch references (.git/refs/heads/)

### Q19: How do you undo the last commit without losing changes?

**Answer:**
```bash
git reset --soft HEAD~1  # Keeps changes staged
git reset HEAD~1         # Keeps changes unstaged (default --mixed)
git reset --hard HEAD~1  # Discards changes completely
```

### Q20: What's a bare repository and when would you use it?

**Answer:**
**Bare Repository**: Repository without a working directory
**Use Cases:**
- Central repositories for team sharing
- Git servers (like GitHub, GitLab)
- Deployment targets
**Create**: `git init --bare` or `git clone --bare`

### Q21: How do you rename a branch locally and remotely?

**Answer:**
```bash
# Rename local branch
git branch -m old-name new-name

# Delete old remote branch and push new one
git push origin :old-name
git push -u origin new-name
```

### Q22: What's the difference between git stash pop and git stash apply?

**Answer:**
- **git stash pop**: Applies stashed changes and removes from stash stack
- **git stash apply**: Applies stashed changes but keeps in stash stack
- **Use pop**: When you're sure you want to remove from stash
- **Use apply**: When you might need the stash again

### Q23: How do you find which commit introduced a bug?

**Answer:**
**Git Bisect Method:**
```bash
git bisect start
git bisect bad           # Current version has bug
git bisect good <hash>   # Known good version
# Git will checkout middle commit
# Test and mark as good/bad
git bisect good/bad
# Repeat until Git finds the problematic commit
git bisect reset        # Return to original branch
```

### Q24: What's the difference between git clone, git fork, and git branch?

**Answer:**
- **git clone**: Creates local copy of remote repository
- **git fork**: GitHub/GitLab feature to create personal copy of someone's repository
- **git branch**: Creates new branch within existing repository

### Q25: How do you recover a deleted branch?

**Answer:**
```bash
# Find the branch in reflog
git reflog

# Recreate branch from commit hash
git branch <branch-name> <commit-hash>

# Or use reflog references
git branch <branch-name> HEAD@{n}
```

---

## Interview Success Tips

### What Interviewers Look For:
1. **Practical experience** with real Git scenarios
2. **Problem-solving approach** when things go wrong
3. **Understanding of team workflows** and collaboration
4. **Knowledge of best practices** and security considerations
5. **Ability to explain concepts** clearly to different audiences

### Red Flags to Avoid:
- Only knowing basic add/commit/push commands
- Not understanding the difference between local and remote
- Never having dealt with merge conflicts
- Not knowing how to recover from mistakes
- Being unfamiliar with different workflow strategies

### Questions to Ask Them:
1. "What Git workflow does your team currently use?"
2. "How do you handle code reviews and pull requests?"
3. "What's your branching strategy for releases?"
4. "How do you handle hotfixes and emergency deployments?"
5. "What tools do you use for Git repository management?"

### Practice Scenarios:
1. **Set up a local repository** and practice all basic operations
2. **Create merge conflicts** intentionally and resolve them
3. **Practice rebasing** and understand when not to use it
4. **Simulate team workflows** with multiple branches
5. **Try different Git hosting platforms** (GitHub, GitLab, Bitbucket)

### Key Commands to Master:
```bash
# Essential commands every developer should know
git status
git add
git commit
git push/pull
git branch
git checkout/switch
git merge
git rebase
git stash
git log
git diff
git reset
git revert
git cherry-pick
git reflog
```
