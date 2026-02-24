# Review Mode

## Purpose

Send current changes to Codex for independent code review.

## When to Use

- Before committing significant changes
- When you want a second pair of eyes from a different model
- After implementing a complex feature

## Flow

1. **Verify changes exist** — check `git diff --stat` and `git diff --cached --stat`. If no changes, stop.

2. **Show scope** — display file list and line count to user.

3. **Run Codex review:**
   ```bash
   codex review 2>&1
   ```

4. **Capture output** — full Codex response.

5. **Parse findings** into categories:
   - **Bugs / logic errors** — always report
   - **Security issues** — always report
   - **Performance concerns** — report if non-trivial
   - **Style / formatting** — IGNORE (different model aesthetics, not actionable)

6. **Present structured findings:**
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

7. **Ask user** which findings to address. Never auto-apply.

## Key Principle

Filter style disagreements ruthlessly. Different models have different formatting preferences — these are noise, not signal. Only bugs, logic errors, security vulnerabilities, and meaningful performance concerns matter.
