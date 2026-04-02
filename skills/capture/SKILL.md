---
name: capture
description: Extracts the last user input and Claude's response, saving them to .claude/history/ with a timestamped filename. Used to preserve important conversation exchanges for future reference.
version: 1.0.0
---

# Capture — Conversation Save Skill

A skill that saves the last input-output pair as a separate document in the **`temp-history/`** folder.
Use it to preserve important decisions, useful explanations, and noteworthy outputs separately from the conversation flow.

---

## Activation Conditions

### Manual Only: `/capture`

Runs only when explicitly invoked by the user. No automatic activation.

---

## Save Procedure

### Step 1: Identify Save Target

From the current conversation, the **immediately preceding exchange** (last user input + Claude's response) is the save target.

The `/capture` invocation itself is not included in the save target.

### Step 2: Generate Filename

Generate the filename based on the current time.

**Format**: `YYYY-MM-DD_HH-MM-SS.md`

Example: `2026-04-01_14-30-05.md`

### Step 3: Verify Save Path and .gitignore

Create the `.claude/history/` folder if it doesn't exist.

When creating the folder for the first time, check if the project has a version control system (.git, .svn, etc.).
If a VCS is detected, **always** add the `.claude/history/` path to the ignore file.

| VCS | Ignore file |
|-----|-------------|
| Git | `.gitignore` |
| SVN | `.svnignore` |
| Mercurial | `.hgignore` |

Do not add duplicate entries if already registered.
Create the ignore file if it doesn't exist.

### Step 4: Write and Save File

Write the file in the format below.

---

## Save File Format

```markdown
> {YYYY-MM-DD HH:MM:SS}

## Input

{Last user input exactly as-is}

## Output

{Claude's response exactly as-is}
```

---

## Completion Message

```
Saved: .claude/history/YYYY-MM-DD_HH-MM-SS.md
```

One line only. No additional explanation.

---

## Key Principles

- **Previous exchange only**: Save only the last input-output pair before the `/capture` call. Do not include earlier content.
- **Verbatim content**: Do not summarize or edit. Save the original text as-is.
- **Timestamp filenames**: Never base filenames on content. Always use the current time.
- **Save quietly**: Output only the save path in one line. Do not add content summaries or extra explanations.
- **Never deploy history**: Always register `.claude/history/` in the ignore file if a VCS exists.
