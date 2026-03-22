<!--
Time Budget (15 minutes total):

| Section                              | Slides | Time |
|--------------------------------------|--------|------|
| Title + Agenda                       | 1-2    | 0:30 |
| MCPs + SubAgents (brief)             | 3      | 0:45 |
| Skills (most emphasis)               | 4-6    | 3:00 |
| Rules                                | 7      | 1:00 |
| Hooks                                | 8      | 1:15 |
| Comparison table                     | 9      | 0:45 |
| Why Claude Code                      | 10     | 1:15 |
| Effective Use                        | 11-12  | 2:30 |
| Example 1: /drafting-pr + Demo       | 13     | 2:00 |
| Example 2: /uploading-attachments    | 14     | 1:30 |
| Learn More                           | 15     | 0:30 |
-->

# Claude Code Features for Real-World Workflows

<!-- Speaker name / date -->

---

## Agenda

- Main Features: MCPs, SubAgents, Skills, Rules, Hooks
- Why Claude Code
- How to Effectively Use Claude Code
- Real World Examples
- Learn More

<!-- Speaker notes:
Main Features takes roughly half the talk. Live demos are included at the end.
-->

---

## Main Features

- **MCPs** — connect Claude Code to external tools/services (database, APIs, Slack, Jira)
  - Caveat: consumes context even when the task doesn't need MCP tools
- **SubAgents** — specialized AI assistants in their own context window
- **Skills** — reusable instructions that extend Claude Code's capabilities
- **Rules** — deterministic context loading based on file patterns
- **Hooks** — automation triggered at lifecycle points

<!-- Speaker notes:
Quick overview of the five main features. MCPs are like a plugin system for external services, but they consume context even when unused — we tend to replace them with Skills where possible. SubAgents run in isolated context windows for focused tasks. We'll dive deep into Skills, Rules, and Hooks in the next few slides.
-->

---

## Skills — What Is a Skill? (1/3)

- Reusable instructions that extend what Claude Code can do
- Custom commands have been merged into **Skills**
- A Skill = a markdown file in `.claude/skills/` with YAML frontmatter

```
.claude/skills/drafting-pr/
├── SKILL.md           ← main instructions (markdown with frontmatter)
├── scripts/           ← helper scripts
└── references/        ← additional context files
```

- Frontmatter controls: name, description, invocation behavior
- **Loaded progressively** — only activated when needed, not consuming context upfront
- Shareable via version control — entire team benefits

<!-- Speaker notes:
A skill is a set of reusable instructions you give Claude Code. It's just a markdown file with YAML frontmatter — commit it to the repo and every teammate using Claude Code gets the same skill. Unlike MCPs which consume context even when unused, Skills are loaded progressively — they only enter the context when invoked or when Claude determines they're relevant.
-->

---

## Skills — Invocation Types (2/3)

- **User invocation** — you trigger it explicitly with `/skill-name`
- **Model invocation** — Claude triggers automatically when relevant
- The `description` field is the trigger — Claude matches it against your conversation

```yaml
# User-invocable only (side effects like deploy, PR creation)
---
name: drafting-pr
description: Create a pull request with consistent formatting
disable-model-invocation: true
---
```

```yaml
# Model-invocable (Claude activates when description matches)
---
name: api-conventions
description: API design patterns for this codebase.
  Use when writing API endpoints or reviewing API code.
---
```

<!-- Speaker notes:
User invocation = slash command you type. Model invocation = Claude reads the description and decides if it's relevant. The description is the key — write it like trigger phrases that tell Claude when to activate the skill. Set disable-model-invocation: true for workflows with side effects.
-->

---

## Skills — Content Types (3/3)

- **Reference** — "Know this" (adds knowledge to Claude's context)
- **Task** — "Do this" (step-by-step instructions Claude executes)

```markdown
---
name: api-conventions
description: API design patterns for this codebase
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

```markdown
<!-- Task example: .claude/skills/pr-review/SKILL.md -->
---
name: pr-review
description: Review a pull request
disable-model-invocation: true
---
1. Check for breaking API changes
2. Verify test coverage
3. Review error handling
```

<!-- Speaker notes:
Reference makes Claude smarter about your project — coding standards, API conventions. Task gives Claude a playbook to follow — PR review, deployment steps. You can combine both in one skill.
-->

---

## Rules — Deterministic Context Loading

- Markdown files in `.claude/rules/`
- **Always loaded** when conditions match — no AI decision-making
- Support **glob patterns** for path-specific rules
  - Example: a rule scoped to `src/api/**/*.ts` only
- Loaded at session start (unconditional) or when matching files are referenced
- Committed to version control — shared across the team

<!-- Speaker notes:
Rules are deterministic — no model discretion. When Claude touches a file matching a glob pattern, the rule is injected automatically. Use rules for things that must always apply. Different parts of your codebase can have different rules.
-->

---

## Hooks — Deterministic Automation

- Automation triggered at lifecycle points — configured in `.claude/settings.json`
- Key events: `PreToolUse` (before), **`PostToolUse`** (after), `Stop`, `SessionStart`
- Three handler types: **Command** (shell script), HTTP (endpoint), Prompt (Claude evaluates)
- Claude sees the errors and **fixes them in the same session**

```json
// .claude/settings.json — run tsc & eslint after every edit
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx tsc --noEmit && npx eslint --fix",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

<!-- Speaker notes:
Hooks run automated checks whenever Claude takes an action. PostToolUse fires after Claude edits a file. The matcher "Edit|Write" targets file modifications. It runs tsc and eslint automatically — if there are errors, Claude sees them and fixes them right away. No human intervention needed. Other handler types include HTTP (call an endpoint) and Prompt (ask Claude to evaluate).
-->

---

## Skills vs. Rules vs. Hooks

|               | Skills              | Rules                        | Hooks                 |
|---------------|---------------------|------------------------------|-----------------------|
| **Trigger**   | User or model       | File glob / always           | Lifecycle event       |
| **Deterministic?** | No (LLM output varies) | No (LLM output varies)  | Yes / No (depends on handler type) |
| **Purpose**   | Add knowledge / run tasks | Inject context          | Automate actions      |
| **Example**   | `/drafting-pr`      | "Use camelCase for variables" | Lint after every edit  |

- Skills & Rules guide the LLM, but output may vary — **Hooks** guarantee consistent results
- Hooks: Command/HTTP handlers are deterministic; Prompt handlers use the LLM, so they are not
- Use all three together: **Rules** set standards, **Hooks** enforce them, **Skills** add capabilities

<!-- Speaker notes:
Key distinction: Skills and Rules feed instructions to the LLM, so you can't guarantee the same output every time. Hooks run external tools like linters — calling eslint always produces the same result for the same code. That's why Hooks are your safety net: even if the LLM drifts, the hook catches it deterministically.
-->

---

## Why Claude Code?

- Copilot now also has Skills, Rules, and Hooks — but Claude Code ships first

```
            2024        2025                          2026
              |     Oct    Dec    Jan    Feb    Mar
              |      |      |      |      |      |
Claude Code   |  Skills  Rules  Hooks
              |      |      |      |      |      |
Copilot       | Instructions  Skills  Hooks(VS)  Hooks(JB)
              |  (Oct '24)     |      |      |      |
```

- More expensive, but **more efficient** — higher output quality = less rework
- Direct from the model maker — tightest integration possible

<!--
Investigation result:

| Feature | Claude Code               | GitHub Copilot                          |
|---------|---------------------------|-----------------------------------------|
| Skills  | Oct 2025                  | Dec 2025 (Agent Skills)                 |
| Rules   | Dec 2025 (.claude/rules/) | Jul 2025 (path-specific .instructions)  |
| Hooks   | Jan 2026 (v2.1.0)        | Jan 2026 (VS Code), Mar 2026 (JetBrains)|

Speaker notes:
Claude Code consistently ships features first. Skills launched Oct 2025, two months before Copilot's Agent Skills in Dec 2025. Hooks came to Claude Code in Jan 2026, with Copilot following in Jan-Mar 2026. For Rules, Copilot's path-specific .instructions.md files (Jul 2025) predate Claude Code's .claude/rules/ (Dec 2025) — but note that Copilot's earlier Custom Instructions (Oct 2024) are more like CLAUDE.md, not Rules.
-->

---

## Reality Check: AI Coding in Teams

- "Build complex products in one day" — true for **greenfield**
- In a team with a pre-existing product:
  - Most time is **planning, negotiation, coordination** — not coding
  - AI-generated code can be **low quality** — more review burden
  - **You are responsible** for the output:
    - Function documentation
    - Comprehensive PR descriptions
    - Screenshots and test evidence

<!-- Speaker notes:
The hype is real for greenfield projects. But most of us work in teams on existing products. Coding is maybe 30% of the work. When AI writes code, someone still reviews it. You are accountable for every line that gets merged.
-->

---

## My Approach

- Use Claude Code for **development planning** — not just simple tasks
  - Architecture discussions, trade-off analysis, implementation plans
- Set up **Rules + Hooks + linters** to enforce conventions automatically
  - AI-written code follows the same standards as human-written code
- Create **Skills for repetitive, time-consuming tasks** — consistent results every time
- Let Claude handle the tedious parts; you handle the judgment calls

<!-- Speaker notes:
Invest upfront in rules and hooks so AI code follows team conventions automatically — this dramatically reduces review friction. Automate the automatable, spend human time on judgment — design decisions, review, stakeholder communication.
-->

---

## Example 1: `/drafting-pr` — Automated PR Creation

- Generates PR with consistent formatting and comprehensive descriptions
- Invoke with `/drafting-pr` after completing your changes
- Includes: summary, change list, test plan, screenshots
- Source: `ibara1454/agent-marketplace`

**Live demo**

<!-- Speaker notes:
Show how typing /drafting-pr produces a complete, well-structured PR. Emphasize consistency — every PR follows the same template, every time.
-->

---

## Example 2: `/uploading-attachments` — Upload to GitHub

- Uploads local files (images, logs) to GitHub issues or PRs
- Invoke the skill; it handles GitHub API interaction automatically
- No more manual drag-and-drop
- Source: `ibara1454/agent-marketplace`

**Live demo**

<!-- Speaker notes:
Show how the skill takes a local file and attaches it to a GitHub issue/PR without manual drag-and-drop.
-->

---

## Learn More

- [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action) — interactive course
- [Best Practices](https://code.claude.com/docs/en/best-practices) — official guide
- [Extend Claude with Skills](https://code.claude.com/docs/en/skills) — skills documentation
- [Complete Guide to Building Skills (PDF)](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf)
- [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- Follow [@bcherny on X](https://x.com/bcherny) for updates

<!-- Speaker notes:
The Claude Code in Action course on Skilljar is a great hands-on starting point. The Complete Guide to Building Skills PDF is the most thorough reference. Thank you!
-->
