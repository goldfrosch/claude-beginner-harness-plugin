---
name: input
description: When /input is invoked, reads INPUT.md from the project root and responds to its contents. INPUT.md is automatically added to the VCS ignore file to exclude it from version control.
version: 1.0.0
---

# Input — INPUT.md Reading Skill

A skill that reads the **`INPUT.md`** file from the project root and responds to its contents when `/input` is invoked.
Use it to pass long content that's difficult to type directly into the chat, reusable prompts, or sensitive instructions via file.

---

## Activation Conditions

### Manual Only: `/input`

---

## Execution Procedure

### Step 1: Check INPUT.md Existence

Check if `INPUT.md` exists at the project root.

**If the file doesn't exist**:

```
INPUT.md file not found.
Please create INPUT.md at the project root and invoke again.
```

End after guidance. Do not auto-create the file.

### Step 2: .gitignore Handling

If INPUT.md exists and the project has a version control system (.git, .svn, etc.), **always** register `INPUT.md` in the ignore file.

| VCS | Ignore file |
|-----|-------------|
| Git | `.gitignore` |
| SVN | `.svnignore` |
| Mercurial | `.hgignore` |

Do not add duplicate entries if already registered.
Create the ignore file if it doesn't exist.

### Step 3: Read INPUT.md and Respond

Treat the contents of INPUT.md as if they were direct user input and respond accordingly.

- If INPUT.md contains a task request → Perform the task.
- If INPUT.md contains a question → Answer the question.
- If INPUT.md contains instructions → Follow the instructions.

Depending on the nature of INPUT.md's content, other skills (scope, clarify, preplan, etc.) may be triggered as well.

---

## Key Principles

- **Never auto-create the file**: If INPUT.md doesn't exist, just guide and exit.
- **Treat contents as direct input**: Don't summarize or interpret INPUT.md. Process the written content exactly as user input.
- **Never deploy INPUT.md**: Always register it in the ignore file if a VCS exists.
