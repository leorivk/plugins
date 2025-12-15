---
description: Autonomous iterative development with git worktree isolation and comprehensive review
argument-hint: optional task description
---

# Auto-Dev Command

Launches the autonomous development workflow that handles tasks through iterative improvement until completion.

## What This Does

When you run `/auto-dev`, it will:

1. **Create Isolated Environment**: Set up a git worktree for your task
2. **Autonomous Development**: An AI agent will:
   - Clarify requirements
   - Explore your codebase
   - Design architecture
   - Implement solution
   - Review comprehensively (6 specialized agents)
   - Fix issues iteratively
   - Prepare PR
3. **Deliver Results**: Ready-to-review branch with detailed PR description

## Usage

```bash
# With task description
/auto-dev "Add user authentication with OAuth 2.0"

# Interactive mode (will prompt for task)
/auto-dev
```

## Initial Request

**Task**: $ARGUMENTS

## Launch Orchestrator

Use the Task tool to launch the `auto-dev:orchestrator` agent now.

The orchestrator will guide you through the complete workflow and coordinate all subagents.

**Important**: Pass the initial task description to the orchestrator.
