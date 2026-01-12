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
## Situations vs. MCP (Model Context Protocol)

MCP is great for connecting agents to external servers (Postgres, Slack, GitHub APIs). It provides Tools and Resources (and Prompts).

Situations are for local, immediate environment state. They are the "System Prompt" equivalent of PS1.

Situations are "Push" (context is forced on the agent). Tools/MCP are "Pull" (agent requests information).

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
