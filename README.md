# claude-beginner-harness-plugin

<p align="center"><img src="Thumbnail.png" width="300"></p>
<p align="center"><font size="4"><strong><a href="README.md">English</a></strong> | <a href="README_KR.md">한국어</a></font></p>

A harness plugin that helps people who are new to or unfamiliar with Claude Code **build good habits**.

## Installation

```bash
/plugin marketplace add GoldFrosch/claude-beginner-harness-plugin
/plugin install claude-beginner-harness-plugin@beginner-harness
```

Built with reference to the [Vibe Coding Guide](https://drive.google.com/file/d/1x2x1T4lzTISnHGN8nd4KtMhL8iwi6QgU/view), this plugin lets Claude automatically catch habits that are hard to develop on your own when first starting vibe coding.

Even if token usage or work time increases somewhat, the priority is learning the right patterns for collaborating with Claude from the start.

## Core Philosophy

- **Habits over speed**: It's more important to internalize the right process through repetition than to get quick results.
- **Don't guess**: Starting work with unclear requirements leads to misguided results. Understand thoroughly before starting.
- **Beginner-friendly**: Guides users to achieve good results even if they're unfamiliar with Claude Code's features.

## How It Works

This plugin operates through **3 agents** coordinating **11 skills**. Each agent handles a specific role and can be activated automatically or invoked manually depending on the situation.

```
Coding task request
      |
      v
[setup-agent] Environment Preparation
  |-- init: Check and create CLAUDE.md
  |-- worktree: Assess need for isolated dev environment
  +-- compile-setup: Detect build command and register compile hook
      |
      v
[design-agent] Task Design
  |-- scope: Vibe coding vs collaborative design decision
  |-- clarify: Requirement clarification (collaborative design)
  |-- impact-check: Impact analysis and contract renewal (changes to existing code)
  +-- preplan: Suggest plan mode for complex tasks
      |
      v
Implementation
      |
      v
[session-agent] Session Management (continuous)
  |-- handoff: Write context handoff document
  |-- capture: Save conversation content
  +-- input: Read and process INPUT.md

Error occurs at any point
      |
      v
debug: Root cause analysis → fix agreement → save to debug log
```

## Agents

### Setup Agent — Environment Preparation Agent

Checks that the project environment is properly set up before starting work.

| Item              | Description                                                                              |
| ----------------- | ---------------------------------------------------------------------------------------- |
| Role              | Check CLAUDE.md existence, assess git worktree isolation needs, set up compile hook      |
| Auto-activation   | When CLAUDE.md is missing or large-scale changes are expected                            |
| Manual invocation | `/setup`                                                                                 |
| Managed skills    | `init`, `worktree`, `compile-setup`                                                      |

### Design Agent — Task Design Agent

Sets the right direction before starting coding work. Prevents the impulse to "just start building" and ensures alignment on what to build and how.

| Item              | Description                                                                 |
| ----------------- | --------------------------------------------------------------------------- |
| Role              | Vibe coding assessment, requirement clarification, impact analysis, plan mode recommendation |
| Auto-activation   | On all coding task requests                                                                  |
| Manual invocation | `/design`                                                                                    |
| Managed skills    | `scope`, `clarify`, `impact-check`, `preplan`                                                |

### Session Agent — Session Management Agent

Manages context and workflow continuity throughout the conversation. Prevents the issue of "Claude forgetting earlier context as the conversation grows longer."

| Item              | Description                                               |
| ----------------- | --------------------------------------------------------- |
| Role              | Context handoff, conversation saving, INPUT.md processing |
| Auto-activation   | When context usage is estimated at 50% or more            |
| Manual invocation | `/handoff`, `/capture`, `/input`                          |
| Managed skills    | `handoff`, `capture`, `input`                             |

## Skills

### `debug` — Error Analysis and Learning

When an error occurs, instead of immediately patching it, analyzes the root cause systematically and builds the habit of understanding why the error happened to prevent recurrence.

- **Auto-activation**: When the user shares an error message or stack trace, when an error occurs during code execution, or when the user expresses unexpected behavior ("why isn't this working", "it breaks when I...")
- **Manual invocation**: `/debug`

**6-step process**: Error classification → reproduction conditions → root cause analysis (symptom vs. actual cause) → fix direction agreement → fix and verify → recurrence prevention summary

After fixing, saves the case to `.claude/debug-log/` for future reference:

```
.claude/debug-log/
  team/       ← committed to git (project-wide patterns)
    INDEX.md
    compile/ / runtime/ / test-failure/ / logic/
  personal/   ← excluded from git (individual learning patterns)
    INDEX.md
    compile/ / runtime/ / test-failure/ / logic/
```

Claude auto-judges whether the error is a **team** pattern (project-specific module/API/business logic) or a **personal** pattern (language basics, tooling, general habits), presents the recommendation, and asks the user to confirm before saving. Filenames use `YYYY-MM-DD_HH-MM-SS_[error-keyword].md` to avoid duplicates. `INDEX.md` in each scope provides a one-line summary per case for quick browsing without opening individual files.

### `compile-setup` — Automatic Compile Verification Setup

Detects the project's build command and registers it as a `PostToolUse` hook in `.claude/settings.json`, so compilation is automatically verified after every file edit.

- **Auto-activation**: Runs as part of `setup-agent` during project initialization
- **Manual invocation**: `/compile-setup`

Supports Node.js/TS, Go, Rust, Java (Maven/Gradle), C#/.NET, Python, C/C++, and more. Confirms the detected command with the user before registering. If a fast type-check alternative exists (e.g., `tsc --noEmit` instead of full build), recommends it for faster feedback. Merges into existing hook configuration without overwriting.

### `init` — CLAUDE.md Initialization

When CLAUDE.md doesn't exist in the project root, creating it takes top priority before any other work.

- **Auto-activation**: When CLAUDE.md is missing at the project root during a coding task request
- **Manual invocation**: `/init`

CLAUDE.md contains a brief project overview, with detailed rules referencing documents in the `.claude` folder.

### `scope` — Vibe Coding Scope Assessment

Before starting a coding task, determines whether the AI can proceed autonomously or needs collaborative design with the user.

- **Auto-activation**: On all coding task requests
- **Manual invocation**: `/scope`

**Vibe coding zone** (AI autonomous): One-off pages, experiments/prototypes, demos, low-importance standalone code

**Collaborative design zone** (discuss with user): Production code, security-related, performance optimization, core abstractions, data model changes

### `clarify` — Requirement Clarification

When a task request is ambiguous, instead of jumping into code immediately, clarifies requirements first. Only applies to tasks identified as collaborative design zone.

- **Auto-activation**: When scope assessment indicates collaborative design and purpose/scope/I-O are unclear
- **Manual invocation**: `/clarify`

Before presenting any questionnaire, runs **Reference Exploration** first. AI identifies the request domain and presents well-known services, games, or tools as candidate references. Once a reference is selected, a single delta question — "Is there anything you'd like to handle differently from this?" — establishes a shared implicit contract. Only remaining unclear points are covered by follow-up questions.

**Reference-based alignment**: Instead of abstract descriptions, aligning on a shared reference lets both parties share the same behavioral assumptions without needing to enumerate them explicitly.

### `impact-check` — Change Impact Analysis and Contract Renewal

Before modifying existing code or requirements, identifies what is affected and explicitly renews the implicit contracts around those changes. Rather than warning about risks after the fact, surfaces the before/after contract for each impacted area and gets agreement before proceeding.

- **Auto-activation**: When changes to existing code or requirements are requested (interface, behavior, data model, or logic changes)
- **Manual invocation**: `/impact-check`
- **Clarify integration**: If `/clarify` was run previously, uses its Requirements Summary as the contract baseline

**Contract renewal over warning**: For each impacted area, the old assumption and new assumption are presented side by side. The user explicitly confirms the new contract before any code is written. When a conflict is detected, the user decides whether to expand scope, narrow the change, or proceed with acknowledged risk.

### `preplan` — Pre-work Preparation

Before starting complex work without context, establishes domain terminology and recommends entering plan mode when necessary.

- **Auto-activation**: When complex work is requested without initial context
- **Manual invocation**: `/preplan`

Plan mode recommendation criteria: Changes spanning multiple files/modules, new features integrating with existing modules, refactoring, architecture changes, tasks with high recovery costs.

### `input` — Read INPUT.md

Reads `INPUT.md` from the project root and responds to its contents. Used to pass long content that's difficult to type directly in the chat.

- **Manual invocation**: `/input`

INPUT.md is automatically registered in the version control system's ignore file.

### `capture` — Save Conversation Content

Saves the last user input and Claude's response to `.claude/history/` with a timestamped filename.

- **Manual invocation**: `/capture`

### `worktree` — Git Worktree Isolated Development

Recommends creating a git worktree when starting large features or work completely different from previous tasks.

- **Auto-activation**: When many files are being changed, modifying core shared modules, or starting a new feature unrelated to previous work
- **Manual invocation**: `/worktree`
- **MCP integration**: If the `EnterWorktree` tool is available, creates a worktree immediately upon user approval

### `handoff` — Context Management and Handoff

When the conversation grows long and context reaches 50% or more capacity, writes a HANDOFF.md handoff document to allow context reset without breaking the workflow.

- **Auto-activation**: When context usage is estimated at 50% or more
- **Manual invocation**: `/handoff`

## Project Structure

```
claude-beginner-harness-plugin/
|-- .claude-plugin/
|   +-- plugin.json          # Plugin configuration (v1.1.0)
|-- agents/
|   |-- setup/AGENT.md       # Environment preparation agent
|   |-- design/AGENT.md      # Task design agent
|   +-- session/AGENT.md     # Session management agent
|-- skills/
|   |-- debug/SKILL.md       # Error analysis and learning
|   |-- compile-setup/SKILL.md # Automatic compile verification setup
|   |-- init/SKILL.md        # CLAUDE.md initialization
|   |-- scope/SKILL.md       # Vibe coding scope assessment
|   |-- clarify/SKILL.md     # Requirement clarification
|   |-- impact-check/SKILL.md # Change impact analysis and contract renewal
|   |-- preplan/SKILL.md     # Pre-work preparation
|   |-- input/SKILL.md       # Read INPUT.md
|   |-- capture/SKILL.md     # Save conversation content
|   |-- worktree/SKILL.md    # Git worktree isolated development
|   +-- handoff/SKILL.md     # Context management and handoff
|-- CLAUDE.md                # Plugin instruction document
+-- README.md
```

## Development Principles

When adding new skills or features to this plugin, follow these criteria:

1. Does it solve a mistake or inefficient pattern that beginners frequently encounter?
2. Does it contribute to building good habits, not just providing convenience?
3. Does it improve the work process rather than the work output?
