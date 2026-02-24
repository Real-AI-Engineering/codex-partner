# Implement Mode

## Purpose

Delegate implementation to Codex in an isolated git worktree.

## When to Use

- Well-defined implementation tasks
- When you want Codex to write code independently
- For tasks where worktree isolation prevents conflicts with ongoing work
- When the primary AI is better used for review than generation

## Flow

1. **Extract spec** from user input.

2. **Verify git repository.** Stop if not in one.

3. **Create isolated worktree:**
   ```bash
   BRANCH="codex/$(date +%s)"
   git worktree add "/tmp/$BRANCH" -b "$BRANCH" HEAD
   ```

4. **Run Codex in full-auto mode:**
   ```bash
   codex exec --full-auto -C "/tmp/$BRANCH" "<spec>" 2>&1
   ```

5. **Capture output** and check exit code.

6. **Show diff:**
   ```bash
   git -C "/tmp/$BRANCH" diff HEAD
   ```

7. **Present results:**
   ```
   ## Codex Implementation

   ### Files Changed
   [list with +/- lines]

   ### Diff
   [key changes summarized — full diff on request]

   ### Primary AI Assessment
   [Review of Codex's implementation: correctness, style, completeness]

   ### Options
   1. Merge into current branch
   2. Cherry-pick specific files
   3. Discard
   ```

8. **If merge:**
   ```bash
   git -C "/tmp/$BRANCH" add -A && git -C "/tmp/$BRANCH" commit -m "codex: <spec summary>"
   git merge "$BRANCH"
   git worktree remove "/tmp/$BRANCH"
   ```

9. **If discard:**
   ```bash
   git worktree remove --force "/tmp/$BRANCH"
   git branch -D "$BRANCH"
   ```

## Worktree Lifecycle

- Created fresh for each implementation
- Isolated from main working tree
- ALWAYS cleaned up — on success, failure, or discard
- Never leave orphaned worktrees

## Key Principle

Worktree isolation means Codex can work freely without affecting your current state. Review the diff carefully before merging — Codex is the implementer, the primary AI and the human are the reviewers. If Codex's implementation is partially correct, cherry-pick rather than wholesale merge.
