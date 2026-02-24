# Codex Partner — Second AI in the Team

This repository provides workflows and a Claude Code skill for using OpenAI Codex CLI as an independent verification AI alongside your primary AI agent. The pattern treats Codex not as a replacement but as a peer — a second model with different training, different reasoning, and different blind spots — that you consult at key decision points: before committing, when evaluating competing approaches, or when you want implementation delegated to an isolated environment.

## Why Multi-Model Verification

- Different models have different reasoning strengths and blind spots. What one model overlooks, another often catches.
- Independent review finds issues that single-model workflows miss — especially in security and logic, where model-specific reasoning patterns create systematic gaps.
- Style disagreements between models are noise, not signal. Two models trained differently will have different formatting preferences. Filter those out; only bugs, security vulnerabilities, and logic errors matter.
- Two perspectives structured into a clear format — with explicit divergence analysis — give you better information than asking one model to review its own work.

## Team Model

```
Human (domain knowledge, final decisions)
  ↕
Primary AI (orchestrator, deep codebase understanding)
  ↕
Codex CLI (independent verification, second opinion)
```

The human stays in control. The primary AI orchestrates and understands context. Codex CLI provides an independent signal. Nothing gets applied automatically — every finding goes through the human.

## Requirements

- OpenAI Codex CLI installed:
  ```bash
  npm install -g @openai/codex
  ```
- OpenAI API key configured (`codex auth` or `OPENAI_API_KEY` env var)
- Git repository (for implement mode and review mode)
- Optional: Claude Code (for plug-and-play skill integration)

## Install

**For Claude Code users (one-liner):**

```bash
git clone https://github.com/Real-AI-Engineering/codex-partner.git ~/.claude/skills/codex-partner && ln -s ~/.claude/skills/codex-partner/skills/codex-partner ~/.claude/skills/codex-partner-skill
```

**Verify:** restart Claude Code, then type `/codex` — the skill should activate and ask which mode you want (review, ask, or implement).

**Update:**

```bash
cd ~/.claude/skills/codex-partner && git pull
```

**Uninstall:**

```bash
rm ~/.claude/skills/codex-partner-skill && rm -rf ~/.claude/skills/codex-partner
```

**For standalone use:** no install needed — read the `docs/` directory for workflow documentation you can adapt to any AI assistant.

## Three Modes

| Mode | Purpose | Command |
|------|---------|---------|
| Review | Independent code review of current changes | `/codex review` |
| Ask | Get second opinion for debate | `/codex ask "question"` |
| Implement | Delegate implementation to isolated worktree | `/codex implement "spec"` |

Detailed documentation:
- [Review mode](docs/review-mode.md)
- [Ask mode](docs/ask-mode.md)
- [Implement mode](docs/implement-mode.md)

## Rules

Seven guardrails that make this pattern safe and useful:

1. **Never auto-apply Codex findings.** Always present to user first.
2. **Filter style disagreements.** Different models have different aesthetics — only flag bugs, logic errors, security, and performance.
3. **Max 2 Codex calls per invocation.** Each call has ~8K+ token overhead. If more are needed, invoke `/codex` again as a fresh call.
4. **Cost awareness.** Mention token cost if the task is trivial for Codex — not every question warrants a full model call.
5. **Error handling.** Report Codex CLI failures clearly (auth expired, network error, timeout) and suggest next steps.
6. **Worktree cleanup.** Always clean up after implement mode — on success, failure, or discard. Never leave orphaned worktrees.
7. **No Codex for Codex.** Do not use Codex to review Codex output. One round only.

## Related

For agent teams patterns and multi-agent orchestration, see [teams-field-guide](https://github.com/Real-AI-Engineering/teams-field-guide).

## Sources

- [OpenAI Codex CLI](https://github.com/openai/codex) — Official CLI repository
- [Codex CLI Documentation](https://platform.openai.com/docs/guides/codex) — OpenAI platform docs
- [Multi-model verification patterns](https://github.com/Real-AI-Engineering/teams-field-guide) — Agent Teams field guide with orchestration patterns

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
