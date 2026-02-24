# Ask Mode

## Purpose

Get Codex's independent opinion for debate or second perspective.

## When to Use

- Architecture decisions with multiple valid approaches
- When you suspect your primary AI might have a blind spot
- For "sanity check" on complex logic
- When you want competing analysis before deciding

## Flow

1. **Extract question** from user input.

2. **Run Codex in ephemeral mode:**
   ```bash
   codex exec --ephemeral -C "$PWD" "<question>" 2>&1
   ```

3. **Capture full response.**

4. **Present BOTH perspectives:**
   ```
   ## Two Perspectives

   ### Codex
   [Codex's answer, summarized]

   ### Primary AI
   [Your own independent analysis of the same question]

   ### Key Differences
   [Where the two perspectives diverge]

   ### Recommendation
   [Synthesis — what you'd recommend and why]
   ```

5. **Let user decide.** Do not auto-implement either perspective.

## Key Principle

Present both perspectives fairly. The value is in the DIVERGENCE — where two different models disagree is where the most interesting insights live. A recommendation that both models agree on is more trustworthy; a recommendation where they diverge deserves explicit discussion before the human decides.
