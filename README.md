# Recursive Self-Improving Software Engineering Agents

A framework for self-improving AI software engineering agents in Claude Code. Below are battle-tested principles and practices for accelerated software engineering.

<br>

**I.** **Automatic Training Hooks.** A hard-coded script automatically injects domain skills into every agent session.  
**II.** **Primary & Secondary Skills.** A primary skill teaches essential knowledge and introduces secondary skills.  
**III.** **Learning from Mistakes.** Following any errors, agents update their skills with lessons learned.

<br>

## I. Automatic Training Hooks

The fundamental problem with pre-trained AI agents is that they lack proprietary and domain knowledge.  
We must therefore automatically inject such knowledge into every AI agent session via hooks.

Place `SessionStart` and `SubagentStop` hooks at `project_root/.claude/settings.json`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "hooks": {
    "SessionStart": [
      {
        "matcher": "^(startup|compact|resume)$",
        "hooks": [
          {
            "type": "command",
            "command": "python3 scripts/startup_context.py || python scripts/startup_context.py"
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "^Plan$",
        "hooks": [
          {
            "type": "command",
            "command": "python3 scripts/startup_context.py skill || python scripts/startup_context.py skill"
          }
        ]
      }
    ]
  }
}
```

Place a script at `project_root/scripts/startup_context.py` for reading the primary skill:

```python
#!/usr/bin/env python3                                                                                                                              
"""                                                                                                                                               
Cross-platform context injector for Claude Code hooks.
Works on Windows, Mac, Linux.

Usage:
  python3 scripts/startup_context.py        # Full: CLAUDE.md + primary skill
  python3 scripts/startup_context.py skill  # Skill only (after Plan mode)                                                                          
"""
import os                                                                                                                                           
import sys                                                                                                                                        

base = os.getcwd()
mode = sys.argv[1] if len(sys.argv) > 1 else "all"
                                                                                                                                                    
files = ["CLAUDE.md"]
if mode == "skill":                                                                                                                                 
    files = []                                                                                                                                    
files.append(os.path.join("skills", "primary", "SKILL.md"))

for rel_path in files:                                                                                                                              
    path = os.path.join(base, rel_path)
    if os.path.exists(path):                                                                                                                        
        with open(path, "r", encoding="utf-8") as f:                                                                                              
            print(f.read())
    else:
        print(f"WARNING: {rel_path} not found at {path}", file=sys.stderr)  
```

<br>

## II. Primary & Secondary Skills

A skill is a markdown file, contained in a `skills` directory, like so:

```
project_root/
└── skills/
    ├── primary/SKILL.md      # Primary skill injected into every session
    ├── database/SKILL.md     # Secondary skill for database interactions
    ├── api_x/SKILL.md        # Secondary skill for API called X
    └── ui_dev/SKILL.md       # Secondary skill for UI development
```

The primary skill keeps a table, which details when to read specialized skills, by task:

```markdown
| Task | Specialized Skill |
|------|-------|
| "When making database queries..." | `database` |
| "When engaging with API X..." | `api_x` |
| "To execute UI changes..." | `ui_dev` |
```

Each secondary skill is structured as follows:

```markdown
---
name: [domain]
description: [Covers {scope}.]
---

# [Domain]

## Overview
[Most common high-level patterns]

## Workflows
[Details on primary use cases]

## Errors & Solutions
[Prescriptive rules]
```

In this manner, all secondary skills are introduced by the primary skill, and loaded as needed.

<br>

## III. Learning from Mistakes

The primary skill should specify the following:
- Every mistake becomes a learning opportunity.
- Every lesson must be documented in a skill as an error.
- An error log must be prescriptive to avoid repeating it.

Error entry format:  
```markdown
### [Rule Name]
**Rule:** [What to always/never do]
**Wrong:** `[pattern that fails]`
**Right:** `[pattern that works]`
**Why:** [Root cause]
```

Fatal errors and mandatory rules, however specialized, must be enforced through the primary skill.  
Never allow agents to make mistakes without learning lessons.

<br>

# Applications 

To apply this framework to an existing software system:

1. **Profile all systems** (code, databases, platforms, ...) through sub-agents
2. **Document skills** in essential (primary) and specialized (secondary) `.md` files
3. **Set automatic hooks** in `.claude/settings.json`
4. **Train and refine** skills following errors, ensuring progressive improvement 

Setup Checklist:

- [ ] Define `skills/.../SKILL.md` primary & secondary skills 
- [ ] Enforce `.claude/settings.json` primary skill loading hook
- [ ] Verify hook fires on new session
- [ ] Automatically update skills following agent errors

<br>

_**This [recursive self-improvement](https://en.wikipedia.org/wiki/Recursive_self-improvement) framework leads to increasingly fast, effective, and reliable software engineering.**_
