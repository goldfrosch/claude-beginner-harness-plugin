---
name: scope
description: Determines before coding whether to proceed autonomously (vibe-coding) or collaboratively with the user. Runs before clarify and preplan to distinguish tasks that can proceed without questions from those requiring careful discussion.
version: 1.0.0
---

# Scope — Vibe-Coding Scope Assessment Skill

A skill that **determines "should AI handle this autonomously, or should it discuss with the user first"** before starting a coding task.

Presenting a questionnaire for every task, or conversely having AI do everything on its own, are both suboptimal.
The goal is to choose the right collaboration approach for the nature of the task.

---

## Two Categories

### Vibe-Coding — AI decides and proceeds autonomously

If two or more of the following conditions apply, classify as vibe-coding:

- **One-off**: Code that won't be reused or has low reuse potential
- **Low importance**: Code where the cost of reverting or rebuilding is low
- **Experiment/prototype**: Purpose is rapid idea validation or PoC (proof of concept)
- **Independence**: Standalone code not connected to production code or other modules
- **Non-security**: Code unrelated to authentication, authorization, or sensitive data handling

**Vibe-coding examples**:
- Simple marketing landing page
- Presentation demo/mockup
- One-off script for personal use
- Quick experimental code in an isolated environment
- Internal utility unrelated to production

**Vibe-coding behavior**:
- Do not present a clarify questionnaire
- AI chooses tech stack, structure, and abstractions on its own
- After completion, briefly explain the choices alongside the result

---

### Collaborative Design — Proceed after thorough discussion with the user

If any of the following conditions apply, collaborative design is required:

- **Production deployment**: Code that will serve real users or be deployed to production environments
- **Security-related**: Authentication (login, tokens, sessions), authorization, encryption, sensitive data handling
- **Performance optimization**: Algorithms, caching strategies, DB query optimization in performance-critical areas
- **Core abstractions**: Shared interfaces, base classes, utilities that other code depends on
- **Data model changes**: DB schema, migrations, data structure changes
- **External integrations**: External APIs, payment systems, third-party service integrations
- **High recovery cost**: Tasks where rollback or correction costs are high if something goes wrong

**Collaborative design examples**:
- Sign-up/login functionality
- Payment processing logic
- Shared API client wrapper
- DB table structure changes
- Query optimization for performance issues

**Collaborative design behavior**:
- First inform the user of the scope assessment result
- Clarify requirements through the clarify skill
- If needed, align on design direction through the preplan skill
- AI does not arbitrarily choose abstractions or structure

---

## Assessment Method

When a request is received, assess in this order:

1. **Check collaborative design conditions first**: If any collaborative design condition applies, immediately classify as collaborative design
2. **Check vibe-coding conditions**: If no collaborative design conditions apply and two or more vibe-coding conditions are met, classify as vibe-coding
3. **When unclear, confirm with the user**: If neither category is clear, ask briefly

---

## Confirmation Message for Ambiguous Cases

```
The approach differs depending on the purpose of this task.

Please let me know which applies:
- Is this for quick experimentation or a demo? (Proceed with vibe-coding)
- Is this code for a real service/production? (Proceed after clarifying requirements)
```

---

## Post Vibe-Coding Completion Format

When work is completed via vibe-coding, briefly explain the following alongside the result:

```
[Code/result]

---
**Approach chosen**: [Tech stack, structure, key decisions]
**Reasoning**: [Why this approach was chosen, in one or two lines]
**Note**: This code was written for [experiment/demo/one-off] purposes. For production use, [security review / error handling / performance review] is needed.
```

---

## Key Principles

- **No vibe-coding if any collaborative design condition exists**: The right direction matters more than fast progress.
- **Transparency even in vibe-coding**: Don't hide AI's choices — explain them.
- **Ask when ambiguous**: Don't proceed on guesses when the assessment is unclear.
- **State the limitations of vibe-coded results**: Always note what needs review before production deployment.
