# Git Rebase Continue Demo

This demo shows how `git rebase --continue` works when conflicts occur during an interactive rebase.

## Setup: Create a Conflicting Scenario

### Step 1: Create Initial File and Branch

```bash
# Start fresh
git checkout main

# Create a shared file
echo "Line 1: Original content" > shared.txt
echo "Line 2: Original content" >> shared.txt
echo "Line 3: Original content" >> shared.txt
git add shared.txt
git commit -m "Initial shared file"

# Create feature branch
git checkout -b feature/update-content
```

### Step 2: Make Changes on Feature Branch

```bash
# Modify the file on feature branch
echo "Line 1: Feature branch change" > shared.txt
echo "Line 2: Original content" >> shared.txt
echo "Line 3: Feature branch addition" >> shared.txt
git add shared.txt
git commit -m "Update content in feature branch"

# Add another commit
echo "Line 4: Another feature change" >> shared.txt
git add shared.txt
git commit -m "Add more feature content"
```

### Step 3: Simulate Main Branch Progress

```bash
# Switch to main and add conflicting changes
git checkout main
echo "Line 1: Main branch change" > shared.txt
echo "Line 2: Main branch modification" >> shared.txt
echo "Line 3: Original content" >> shared.txt
git add shared.txt
git commit -m "Update content from main branch"
```

## The Rebase Process with Conflicts

### Step 4: Start Interactive Rebase

```bash
# Switch back to feature branch
git checkout feature/update-content

# Start interactive rebase
git rebase -i main
```

Git will open an editor showing:
```
pick abc1234 Update content in feature branch
pick def5678 Add more feature content

# Rebase main..feature/update-content onto main
```

### Step 5: Conflict Occurs

After you save and close the editor, Git will attempt to apply the first commit and encounter a conflict:

```
Auto-merging shared.txt
CONFLICT (content): Merge conflict in shared.txt
error: could not apply abc1234... Update content in feature branch

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: git rebase --skip
Or abort the rebase: git rebase --abort
```

### Step 6: Check Rebase Status

```bash
# See what's happening
git status
```

Output shows:
```
interactive rebase in progress; onto xyz9012
Last command done (1 command done):
   pick abc1234 Update content in feature branch
Next command to do (1 remaining command):
   pick def5678 Add more feature content
  (use "git rebase --edit-todo" to view and edit)
You are currently rebasing branch 'feature/update-content' on 'xyz9012'.
  (fix conflicts and run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   shared.txt
```

### Step 7: Examine the Conflict

```bash
# Look at the conflicted file
cat shared.txt
```

You'll see conflict markers:
```
<<<<<<< HEAD
Line 1: Main branch change
Line 2: Main branch modification
Line 3: Original content
=======
Line 1: Feature branch change
Line 2: Original content
Line 3: Feature branch addition
>>>>>>> abc1234 (Update content in feature branch)
```

### Step 8: Resolve the Conflict

Edit the file to resolve conflicts:
```bash
# Choose how to merge the changes
echo "Line 1: Combined main and feature changes" > shared.txt
echo "Line 2: Main branch modification" >> shared.txt
echo "Line 3: Feature branch addition" >> shared.txt
```

### Step 9: Mark Conflict as Resolved

```bash
# Stage the resolved file
git add shared.txt

# Check status again
git status
```

Status now shows:
```
interactive rebase in progress; onto xyz9012
Last command done (1 command done):
   pick abc1234 Update content in feature branch
Next command to do (1 remaining command):
   pick def5678 Add more feature content
You are currently rebasing branch 'feature/update-content' on 'xyz9012'.
  (all conflicts fixed: run "git rebase --continue")

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
        modified:   shared.txt
```

### Step 10: Continue the Rebase

```bash
# Continue with the rebase
git rebase --continue
```

Git will:
1. Apply the resolved first commit
2. Attempt to apply the second commit
3. If there are more conflicts, repeat the process
4. If no conflicts, complete the rebase

### Step 11: Handle Potential Second Conflict

If the second commit also conflicts, you'll see:
```
Auto-merging shared.txt
CONFLICT (content): Merge conflict in shared.txt
error: could not apply def5678... Add more feature content
```

Repeat the resolution process:
```bash
# Resolve conflicts in shared.txt
# Add resolved file
git add shared.txt
# Continue rebase
git rebase --continue
```

### Step 12: Rebase Complete

When all commits are successfully applied:
```
Successfully rebased and updated refs/heads/feature/update-content.
```

## Key Commands During Rebase

- `git status` - Check current rebase state
- `git add <file>` - Mark conflicts as resolved
- `git rebase --continue` - Continue after resolving conflicts
- `git rebase --skip` - Skip current commit (use carefully)
- `git rebase --abort` - Cancel rebase and return to original state
- `git rebase --edit-todo` - Modify remaining rebase steps

## Pro Tips for Rebase Continue

1. **Always check `git status`** before continuing to ensure all conflicts are resolved
2. **Test your resolution** - make sure the merged content works correctly
3. **One conflict at a time** - focus on resolving the current commit's conflicts
4. **Don't rush** - carefully review what each side of the conflict is trying to do
5. **Use `git diff --cached`** to review staged changes before continuing

## What Happens Behind the Scenes

During `git rebase --continue`:
1. Git creates a new commit with your resolved changes
2. Moves to the next commit in the rebase sequence
3. Attempts to apply it on top of the previous result
4. Repeats until all commits are processed or another conflict occurs
5. Updates your branch pointer to the final rebased commit