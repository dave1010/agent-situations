# Agent Situations
Dynamic, ephemeral context injection for AI Coding Agents.

Your shell prompt (PS1) knows when you switch git branches. It knows when your build is failing. It knows your Node version. **Why doesn't your AI Agent?**

Read the original blog post: [Giving coding agents situational awareness (from shell prompts to agent prompts)](https://dave.engineer/blog/2026/01/agent-situations/).

## The Problem

Most AI coding agents rely on static context:

- User Prompts: "Here is my file..." (Manual)
- README.md / AGENTS.md: "This project uses Node 14." (Often outdated/drifted)
- RAG: "Let me search your codebase." (Passive)
- Skills / Tools: "I have a tool to check the git status." (Requires the agent to decide to look)

## The Solution: Situations

Situations are lightweight, declarative definitions that allow an agent to automatically detect the state of the environment and inject relevant context into the system prompt only when applicable.

They bridge the gap between Prompt Engineering and Shell/Environment Context.

## How it works

1. **Discovery**: The agent scans a .situations/ directory (global or local).
2. **Evaluation**: The agent evaluates the check condition for every Situation (e.g., file existence, regex match, or exit code).
3. **Injection**: If (and only if) the check passes, the Situation's context is appended to the System Prompt.

## The Specification

A Situation is defined by a `SITUATION.yaml` file.

### Example: Git Status

Imagine an agent that always knows the state of your working tree without you having to paste `git status` output.

```yaml
# .situations/git-context/SITUATION.yaml
name: git-status
description: Injects the current branch and status if inside a git repo.

# The Check: Determining if this situation is active
run: ./git.sh # Output is appended to system prompt
```

## Example: Framework Detection (Static Check)

Situations don't always need to execute code. They can be safe, static checks.

```yaml
# .situations/nextjs/SITUATION.yaml
name: nextjs-router
description: Reminds the agent to use App Router syntax if detected.

check:
  type: file-exists
  path: "app/page.tsx" # Triggers only if the App Router structure exists

context:
  type: static
  text: |
    CRITICAL: This project uses Next.js App Router. 
    - Do not use `getStaticProps` or `getServerSideProps`.
    - Use async Server Components for data fetching.
```

## Situations vs. AGENTS.md

AGENTS.md is a static document written for agents. It's great for stable guidance like coding conventions or architecture decisions, but it drifts when the environment changes (node versions, failing builds, active branches).

Situations are dynamic and ephemeral. They only appear when the check passes and disappear when it doesn't, so the agent always sees context that is current and relevant.

**Use both**: keep enduring human guidance in AGENTS.md, and let Situations supply volatile, environment-specific context.

## Situations vs. Skills

Skills (like Claude Code's SKILL.md) are a discoverability mechanism: a table of contents that the agent can choose to read. They're great for optional knowledge and tool usage but still require manual enablement or selection.

Situations are self-selecting. They run checks automatically and push context into the system prompt only when applicable, reducing agent guesswork and avoiding unnecessary prompt bloat.

**Use both**: Skills help agents discover capabilities; Situations keep the prompt accurate to the current environment.

## Situations vs. MCP (Model Context Protocol)

MCP is great for connecting agents to external servers (Postgres, Slack, GitHub APIs). It provides Tools and Resources (and Prompts).

Situations are for local, immediate environment state. They are the "System Prompt" equivalent of PS1.

Situations are "Push" (context is forced on the agent). Tools/MCP are "Pull" (agent requests information).

## FAQ

### Where do Situations live?

In a `.situations/` directory. Agents can support a global path (shared across projects) and a local project path, then merge results.

### What types of checks are supported?

Checks can be file existence, regex matches, environment variables, or running an executable and checking its exit code.

### How does a Situation provide context?

Context can be static text, file content, or the output of an executable.

### Are Situations safe to run?

Executable Situations (`run`) can run arbitrary code, so they should only be used from trusted sources. Agents should prompt before running project-level scripts.

### How are Situations different from shell prompts?

Shell prompts render a view of state for humans; Situations render a view of state for agents. Both are fast, contextual, and refresh whenever you run a command.

## Schema Reference

### check triggers

- type: `file-exists` - true if file is found relative to root.
- type: `regex` - true if file content matches pattern.
- type: `env` - true if environment variable is set/matches.
- type: `run` - true if executable returns exit code 0.

### context sources

type: `static` - Raw markdown string.
type: `file` - Content of a specific file.
type: `run` - stdout of an executable.

### `run`

`run` takes an executable path and is a shortcut to use the same executable as the check and context source.

### Reference Implementation

This specification is currently implemented in [Jorin](https://github.com/dave1010/jorin).

## Security

⚠️ Executable Situations: Using type: `run` allows arbitrary code execution.

⚠️ Prompt injection: untrusted prompts could cause an agent to perform unwanted commands.

Agents should:

- Only run scripts from trusted locations.
- Prompt for permission before running project-level scripts

## Contributing

We welcome pull requests for the [Standard Library of Situations](./catalog), including checks for common languages, frameworks, and tools.

## Licence

Agent Situations is licensed CC0 and may be used or modified for any purpose.
