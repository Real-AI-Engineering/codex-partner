---
name: codex-partner
description: Use when user says "/codex", wants a second opinion from a different AI, needs independent code review, or wants to delegate implementation to Codex CLI. Also trigger when user mentions "ask codex", "codex review", "second opinion", "different perspective", or "delegate to codex".
---

# Codex Partner — Second AI in the Team

## Team Model
- **Human**: domain knowledge, final decisions
- **Primary AI**: orchestrator, deep codebase understanding, planning
- **Codex CLI**: independent verification, second opinion

## Modes

Parse the user's input to determine mode:
- `/codex review` → Review mode
- `/codex ask "<question>"` → Ask mode
- `/codex implement "<spec>"` → Implement mode
- `/codex` alone → ask the user which mode they want

---

### /codex review

**Purpose:** Send current changes to Codex for independent code review.

**Flow:**
1. Verify there are uncommitted or staged changes (`git diff --stat` and `git diff --cached --stat`). If no changes, tell user and stop.
2. Show the user what will be reviewed (file list and line count).
3. Run Codex review:
   ```bash
   codex review 2>&1
   ```
4. Capture the full output.
5. Parse findings into categories:
   - **Bugs / logic errors** — always report
   - **Security issues** — always report
   - **Performance concerns** — report if non-trivial
   - **Style / formatting** — IGNORE (different model aesthetics, not actionable)
6. Present findings to user in a structured format:
   ```
   ## Codex Review Findings

   ### Bugs
   - [file:line] description

   ### Security
   - [file:line] description

   ### Performance
   - [file:line] description

   ### Summary
   X issues found (Y bugs, Z security, W performance). Style issues filtered out.
   ```
7. Ask user which findings to address. Never auto-apply fixes.

---

### /codex ask "<question>"

**Purpose:** Get Codex's independent opinion for debate or second perspective.

**Flow:**
1. Extract the question from user input.
2. Run Codex in ephemeral mode:
   ```bash
   codex exec --ephemeral -C "$PWD" "<question>" 2>&1
   ```
3. Capture the full response.
4. Present BOTH perspectives to user:
   ```
   ## Two Perspectives

   ### Codex
   [Codex's answer, summarized]

   ### Primary AI
   [Your own independent analysis of the same question]

   ### Key Differences
   [Where the two perspectives diverge]

   ### Recommendation
   [Your synthesis — what you'd recommend and why]
   ```
5. Let user decide. Do not auto-implement either perspective.

---

### /codex implement "<spec>"

**Purpose:** Delegate implementation to Codex in an isolated worktree.

**Flow:**
1. Extract the implementation spec from user input.
2. Verify we're in a git repository. If not, stop.
3. Create an isolated worktree:
   ```bash
   BRANCH="codex/$(date +%s)"
   git worktree add "/tmp/$BRANCH" -b "$BRANCH" HEAD
   ```
4. Run Codex in full-auto mode in the worktree:
   ```bash
   codex exec --full-auto -C "/tmp/$BRANCH" "<spec>" 2>&1
   ```
5. Capture output and check exit code.
6. Show the diff between HEAD and worktree:
   ```bash
   git -C "/tmp/$BRANCH" diff HEAD
   ```
7. Present results to user:
   ```
   ## Codex Implementation

   ### Files Changed
   [list of files with +/- lines]

   ### Diff
   [key changes, summarized — full diff available on request]

   ### Primary AI Assessment
   [Your review of Codex's implementation: correctness, style, completeness]

   ### Options
   1. Merge into current branch
   2. Cherry-pick specific files
   3. Discard
   ```
8. If user chooses to merge:
   ```bash
   git -C "/tmp/$BRANCH" add -A && git -C "/tmp/$BRANCH" commit -m "codex: <spec summary>"
   git merge "$BRANCH"
   git worktree remove "/tmp/$BRANCH"
   ```
9. If user discards:
   ```bash
   git worktree remove --force "/tmp/$BRANCH"
   git branch -D "$BRANCH"
   ```

---

## Rules

1. **Never auto-apply Codex findings.** Always present to user first.
2. **Filter style disagreements.** Different models have different aesthetics — only flag bugs, logic errors, security, and performance.
3. **Max 2 Codex calls per skill invocation.** Each call has ~8K+ token overhead. If user needs more, they invoke `/codex` again.
4. **Cost awareness.** Mention token cost if the task seems trivial for Codex.
5. **Error handling.** If Codex CLI fails (auth expired, network error, timeout), report the error clearly and suggest alternatives (e.g., "try `codex auth` to re-authenticate").
6. **Worktree cleanup.** Always clean up worktrees on error or discard. Never leave orphaned worktrees.
7. **No Codex for Codex.** Do not use Codex to review Codex output. One round only.
