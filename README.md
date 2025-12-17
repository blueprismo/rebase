# Git Rebase Interactive Demo

## Why Conflicts Appear with Outdated Main

When working on a feature branch for an extended period, the main branch often receives new commits from other developers. This creates a situation where:

- Your feature branch is based on an older version of main
- Main has new commits that your branch doesn't know about
- When you try to merge, Git finds conflicting changes in the same files

## Why Rebase Instead of Reset

**Rebase** preserves your commit history while moving your changes on top of the latest main:
- Maintains a clean, linear history
- Shows your commits as if they were made on the current main
- Easier to review and understand the changes

**Reset** would discard your work entirely:
- Loses all your commits and changes
- Forces you to start over from scratch
- Not suitable when you want to keep your work

## Demo Time: Interactive Rebase

**This demo requires `git rebase -i origin/main` to resolve conflicts.** Follow the steps below to create a conflict scenario with a remote repository (origin).

### Visual Overview: How Conflicts Happen and Get Resolved

```
┌─────────────────────────────────────────────────────────────────┐
│                        INITIAL STATE                            │
└─────────────────────────────────────────────────────────────────┘

main: A───B───C
              ↑
         Both branches
         start here

┌─────────────────────────────────────────────────────────────────┐
│                   AFTER CREATING BRANCHES                       │
└─────────────────────────────────────────────────────────────────┘

main: A───B───C
              ├── D (navbar)     ← feature/add-navbar
              └── E (sidebar)    ← feature/add-sidebar

┌─────────────────────────────────────────────────────────────────┐
│            FIRST BRANCH MERGES TO ORIGIN/MAIN                   │
└─────────────────────────────────────────────────────────────────┘

origin/main: A───B───C───D (navbar merged!)
                      
feature/add-sidebar: A───B───C───E (sidebar - based on old main)
                                 ↑
                    🚨 OUTDATED! Will conflict during rebase

┌─────────────────────────────────────────────────────────────────┐
│                       REBASE IN ACTION                          │
└─────────────────────────────────────────────────────────────────┘

BEFORE REBASE:
origin/main:         A───B───C───D (navbar)
feature/add-sidebar: A───B───C───E (sidebar) 

DURING INTERACTIVE REBASE (git rebase -i origin/main):
origin/main:         A───B───C───D (navbar)
                                 \
feature/add-sidebar:              E' ❌ CONFLICT! 
                                      Git stops here and asks you to resolve

CONFLICT RESOLUTION:
origin/main:         A───B───C───D (navbar)
                                 \
feature/add-sidebar:              E' ⚡ You resolve conflicts manually
                                      git add + git rebase --continue

AFTER SUCCESSFUL REBASE:
origin/main:         A───B───C───D (navbar)
feature/add-sidebar: A───B───C───D───E' ✅ Clean history, ready to merge!
```

### Step 1: Create Two Feature Branches

```bash
# Start from main and ensure it's up to date
git checkout main
git pull origin main

# Create a base file that both branches will modify
echo "Base component" > components.txt
git add components.txt
git commit -m "Add base components file"
git push origin main

# Create first feature branch
git checkout -b feature/add-navbar
# Modify the first line - this will conflict with sidebar branch
echo "NavBar Component v1" > components.txt
git add components.txt
git commit -m "Add navbar component"

# Create second feature branch from main (based on old main)
# IMPORTANT: Checkout main BEFORE the navbar was merged
git checkout main
git checkout -b feature/add-sidebar
# Modify the SAME first line differently - this will cause a conflict!
echo "Sidebar Component v1" > components.txt
git add components.txt
git commit -m "Add sidebar component"
```

**Note:** Both branches modify the same line in `components.txt` with different content. This ensures a conflict when rebasing, since Git cannot auto-merge conflicting content on the same line.

### Step 2: First Developer Merges Their Branch

```bash
# Switch to main and merge first feature (simulating another developer)
git checkout main
git merge feature/add-navbar
git push origin main
```

Now `origin/main` has moved forward and contains the navbar changes in `components.txt`, but your `feature/add-sidebar` branch doesn't know about this yet.

### Step 3: Attempt Interactive Rebase (Conflicts Occur!)

Your `feature/add-sidebar` branch is now outdated. **Use interactive rebase** to fix it:

```bash
# Switch to your outdated branch
git checkout feature/add-sidebar

# Fetch latest changes from remote
git fetch origin

# Start interactive rebase onto origin/main
# This will open an editor and then hit conflicts!
git rebase -i origin/main
```

**🚨 When you save and close the editor, Git will attempt to apply commits and immediately stop with conflicts:**

```
Auto-merging components.txt
CONFLICT (content): Merge conflict in components.txt
error: could not apply 7a8b9c0... Add sidebar component
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: git rebase --skip
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply 7a8b9c0... Add sidebar component
```

**Note:** The interactive rebase editor will show your commits. You can leave them as `pick` (default) or modify them. After saving, Git will attempt to apply them and hit the conflict.

**Check the conflicted file:**

```bash
cat components.txt
# Shows conflict markers:
# <<<<<<< HEAD
# NavBar Component v1
# =======
# Sidebar Component v1
# >>>>>>> 7a8b9c0... Add sidebar component
```

**Check git status:**

```bash
git status
# On branch feature/add-sidebar
# You are currently rebasing branch 'feature/add-sidebar' on 'abc123d'.
#   (fix conflicts and run "git rebase --continue")
#   (use "git rebase --skip" to skip this patch)
#   (use "git rebase --abort" to check out the original branch)
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#       both modified:   components.txt
```

### Step 4: Resolve the Rebase Conflicts

The interactive rebase stopped because of conflicts. Now you need to resolve them:

```bash
# Check the status - you're in the middle of an interactive rebase
git status
# On branch feature/add-sidebar
# interactive rebase in progress; onto abc123d
# You are currently rebasing branch 'feature/add-sidebar' on 'origin/main'.
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#       both modified:   components.txt

# Look at the conflicted file
cat components.txt
# You'll see conflict markers:
# <<<<<<< HEAD
# NavBar Component v1
# =======
# Sidebar Component v1
# >>>>>>> Add sidebar component

# Edit components.txt to resolve the conflict
# Remove conflict markers and combine both:
echo "NavBar Component v1
Sidebar Component v1" > components.txt

# Mark the conflict as resolved and continue the rebase
git add components.txt
git rebase --continue
```

**Important:** This is an interactive rebase (`rebase -i`), so you can:
- Edit commits during the rebase
- Squash multiple commits together
- Reorder commits
- Modify commit messages

The conflict resolution process is the same, but you have more control over how commits are applied.

### Step 5: After Successful Rebase

```bash
# Push your rebased branch (may need force push since history changed)
git push origin feature/add-sidebar --force-with-lease

# Now your branch can merge cleanly into main
git checkout main
git merge feature/add-sidebar
git push origin main
# This should merge cleanly!
```

### Visual Result: Clean Linear History

```
┌─────────────────────────────────────────────────────────────────┐
│                         FINAL RESULT                            │
└─────────────────────────────────────────────────────────────────┘

BEFORE REBASE (outdated branch):
origin/main:         A───B───C───D (navbar)
feature/add-sidebar: A───B───C───E (sidebar) 💥 CONFLICTS DURING REBASE!

AFTER REBASE (clean timeline):
origin/main:         A───B───C───D (navbar)
feature/add-sidebar: A───B───C───D───E' (sidebar applied after navbar)
                                     ↑
                              🎉 Ready for clean merge!

┌─────────────────────────────────────────────────────────────────┐
│                   FINAL components.txt                          │
├─────────────────────────────────────────────────────────────────┤
│  NavBar Component v1      ← From commit D (navbar)              │
│  Sidebar Component v1     ← From commit E' (sidebar rebased)    │
│                             (conflict resolved by combining)     │
└─────────────────────────────────────────────────────────────────┘

🎯 BENEFITS ACHIEVED:
├── ✅ Both features preserved and working
├── ✅ Clean linear git history  
├── ✅ No messy merge commits
└── ✅ Easy to review and rollback
```

## Key Commands

- `git fetch origin` - Fetch latest changes from remote (optional for local demo)
- `git rebase -i origin/main` - **Start interactive rebase** against remote main (required for this demo)
- `git rebase --continue` - Continue after resolving conflicts
- `git rebase --abort` - Cancel the rebase and return to original state
- `git rebase --edit-todo` - Modify remaining rebase steps during interactive rebase
- `git status` - Check current rebase status
- `git push --force-with-lease` - Safely force push after rebase (if using remote)

## Pro Tips

- Always fetch latest changes first: `git fetch origin`
- Make small, focused commits for easier rebasing
- Test your code after rebasing before merging
- Use `git log --oneline --graph` to visualize your branch history
