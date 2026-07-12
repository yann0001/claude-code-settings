---
name: codex-skill
description: 'Leverage OpenAI Codex/GPT models for autonomous code implementation. Triggers: "codex", "use gpt", "gpt-5", "let openai", "full-auto", "用codex", "让gpt实现". Use this skill whenever the user wants to delegate coding tasks to OpenAI models, run code reviews via codex, or execute tasks in a sandboxed environment.'
allowed-tools: Read, Write, Glob, Grep, Task, Bash(cat:*), Bash(ls:*), Bash(tree:*), Bash(codex:*), Bash(which:*), Bash(npm:*), Bash(brew:*)
---

# Codex

You are operating in **codex exec** - a non-interactive automation mode for hands-off task execution.

## Prerequisites

Before using this skill, ensure Codex CLI is installed and configured:

1. **Installation verification**:

   ```bash
   codex --version
   ```

2. **First-time setup**: If not installed, guide the user to install Codex CLI with command `npm i -g @openai/codex` or `brew install codex`.

## Core Principles

### Autonomous Execution

- Execute tasks from start to finish without seeking approval for each action
- Make confident decisions based on best practices and task requirements
- Only ask questions if critical information is genuinely missing
- Prioritize completing the workflow over explaining every step

### Output Behavior

- Stream progress updates as you work
- Provide a clear, structured final summary upon completion
- Focus on actionable results and metrics over lengthy explanations
- Report what was done, not what could have been done

### Operating Modes

Codex uses sandbox policies to control what operations are permitted:

**Read-Only Mode (Default)**

- Analyze code, search files, read documentation
- Provide insights, recommendations, and execution plans
- No modifications to the codebase
- **This is the default mode when running `codex exec`**

**Workspace-Write Mode (Recommended for Programming)**

- Read and write files within the workspace
- Implement features, fix bugs, refactor code
- Execute build commands and tests
- **Use `--full-auto` or `-s workspace-write` to enable file editing**
- **This is the recommended mode for most programming tasks**

**Danger-Full-Access Mode**

- All workspace-write capabilities, plus network access and system-level operations outside the workspace
- **Use only when explicitly requested and necessary**, with flag `-s danger-full-access`

## Common Commands

```bash
# Most programming tasks: full-auto enables file editing (workspace-write)
codex exec --full-auto "implement the user authentication feature"

# Analysis without modifications (default read-only)
codex exec "analyze the codebase structure and suggest improvements"

# Code review of uncommitted changes or against a base branch
codex exec review --uncommitted
codex exec review --base main

# Image-driven implementation
codex exec -i mockup.png --full-auto "implement the UI matching this design"
```

Codex uses the model from `~/.codex/config.toml` by default. Do NOT pass `-m`/`--model` unless the user explicitly asks for a specific model.

## Reference Files

- **[references/cli-reference.md](references/cli-reference.md)** — complete flag reference: sandbox modes, config overrides, feature toggles, profiles, JSON output, session resume, local models, and combined examples. Read this when the task needs a flag not covered above.
- **[references/examples.md](references/examples.md)** — worked scenarios mapping user requests to commands. Read this when unsure which mode fits the request.

## Execution Workflow

1. **Parse the Request**: Understand the complete objective and scope
2. **Plan Efficiently**: Create a minimal, focused execution plan
3. **Execute Autonomously**: Implement the solution with confidence
4. **Verify Results**: Run tests, checks, or validations as appropriate
5. **Report Clearly**: Provide a structured summary of accomplishments

## Best Practices

### Speed and Efficiency

- Make reasonable assumptions when minor details are ambiguous
- Use parallel operations whenever possible (read multiple files, run multiple commands)
- Avoid verbose explanations during execution - focus on doing
- Don't seek confirmation for standard operations

### Scope Management

- Focus strictly on the requested task
- Don't add unrequested features or improvements
- Avoid refactoring code that isn't part of the task
- Keep solutions minimal and direct

### Quality Standards

- Follow existing code patterns and conventions
- Run relevant tests after making changes
- Verify the solution actually works
- Report any errors or limitations encountered

## When to Interrupt Execution

Only pause for user input when encountering:

- **Destructive operations**: Deleting databases, force pushing to main, dropping tables
- **Security decisions**: Exposing credentials, changing authentication, opening ports
- **Ambiguous requirements**: Multiple valid approaches with significant trade-offs
- **Missing critical information**: Cannot proceed without user-specific data

For all other decisions, proceed autonomously using best judgment.

## Final Output Format

Always conclude with a structured summary:

```
✓ Task completed successfully

Changes made:
- [List of files modified/created]
- [Key code changes]

Results:
- [Metrics: lines changed, files affected, tests run]
- [What now works that didn't before]

Verification:
- [Tests run, checks performed]

Next steps (if applicable):
- [Suggestions for follow-up tasks]
```

## Error Handling

When errors occur:

1. Attempt automatic recovery if possible
2. Log the error clearly in the output
3. Continue with remaining tasks if error is non-blocking
4. Report all errors in the final summary
5. Only stop if the error makes continuation impossible

## Resumable Execution

If execution is interrupted:

- Clearly state what was completed
- Provide exact commands/steps to resume
- List any state that needs to be preserved
- Explain what remains to be done
