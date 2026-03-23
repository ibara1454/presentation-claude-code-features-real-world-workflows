---
theme: seriph
title: Claude Code Features for Real-World Workflows
background: /backgrounds/candidate-9.jpg
transition: slide-left
class: text-center
drawings:
  enabled: false
fonts:
  sans: Inter
  mono: Fira Code
themeConfig:
  primary: '#60a5fa'
---

# Claude Code Features for <br> Real-World Workflows

<div class="pt-4">
  <span class="text-xl text-gray-400">
    <!-- Speaker name / date -->
  </span>
</div>

<!--
Welcome everyone. Today I'll share how to use Claude Code's key features to build real-world workflows — not just toy demos, but the kind of automation you'd actually use in a team.
-->

---
transition: fade-out
---

# Agenda

<div class="accent-bar"></div>

- **Main Features** — MCPs, SubAgents, Skills, Rules, Hooks
- **Why Claude Code** — how it compares to alternatives
- **Effective Use** — reality check for team environments
- **Real World Examples** — live demos of actual skills
- **Learn More** — resources to get started

<!--
Main Features takes roughly half the talk. We'll cover five features, then zoom out to why Claude Code, practical advice, and finish with two live demos.
-->

---

# Main Features

<div class="accent-bar"></div>

<div class="card-grid-2">
  <div class="card card-muted">
    <strong>MCPs</strong> — plug into external tools (databases, APIs, Slack, Jira)
    <div class="muted mt-1">Caveat: consumes context even when the task doesn't need MCP tools</div>
  </div>
  <div class="card card-muted">
    <strong>SubAgents</strong> — specialized AI assistants in isolated context windows
  </div>
  <div class="card card-skill">
    <span class="text-skill font-semibold">Skills</span> — reusable instructions that extend capabilities
  </div>
  <div class="card card-rule">
    <span class="text-rule font-semibold">Rules</span> — deterministic context loading by file pattern
  </div>
  <div class="card card-hook">
    <span class="text-hook font-semibold">Hooks</span> — automation triggered at lifecycle points
  </div>
</div>

<!--
Quick overview of the five main features. MCPs are like a plugin system for external services, but they consume context even when unused — we tend to replace them with Skills where possible. SubAgents run in isolated context windows for focused tasks. We'll dive deep into Skills, Rules, and Hooks in the next few slides.
-->

---
---

# <span class="text-skill">Skills</span> — What Is a Skill? <span class="muted">(1/3)</span>

<div class="subtitle">Reusable instructions that extend what Claude Code can do</div>

<div class="layers-grid">
<div class="layers-col">
<div class="text-xs opacity-60 mb-1 font-mono">.claude/skills/drafting-pr/SKILL.md</div>
<div class="relative">

```text
---
name: drafting-pr
description: Create a pull request
  with consistent formatting
---

## Workflow
1. Read the git diff for changes
2. Draft a PR title and summary
3. Create the PR via gh cli

## Examples
See references/example.md
```

<div v-click="1" class="layer-box layer-box-1" style="top: 0; height: 37%;"><span class="layer-tag layer-tag-1">1</span></div>
<div v-click="3" class="layer-box layer-box-2" style="top: 39%; bottom: 0;"><span class="layer-tag layer-tag-2">2</span></div>
</div>
</div>
<div class="layers-col">

- A Skill = a **directory** in `.claude/skills/` with `SKILL.md` as entry point
- `SKILL.md` can **reference other files** in the directory
- **Loaded progressively** — 3-layer architecture

<div class="relative">

```text
.claude/skills/drafting-pr/
├── SKILL.md
├── scripts/
└── references/
    └── example.md
```

<div v-click="4" class="layer-box layer-box-3" style="top: 38%; bottom: 0;"><span class="layer-tag layer-tag-3">3</span></div>
</div>
</div>
</div>

<div v-click="2" class="terminal-prompt">
<span class="terminal-chevron">❯</span> <span class="terminal-cmd">/drafting-pr</span>
</div>

<div class="layers-legend">
<span v-click="1" class="legend-item"><span class="legend-dot" style="background: #60a5fa;"></span> Frontmatter indexed at discovery</span>
<span v-click="3" class="legend-item"><span class="legend-dot" style="background: #34d399;"></span> Body loaded on activation</span>
<span v-click="4" class="legend-item"><span class="legend-dot" style="background: #fbbf24;"></span> Files resolved on demand</span>
</div>

<!--
A skill is a directory under .claude/skills/ with SKILL.md as the entry point. The 3-layer architecture: (1) Only frontmatter is indexed at startup — name and description. (2) The SKILL.md body is loaded into context when the skill is activated. (3) Referenced files like scripts and templates are resolved on demand during execution. This progressive loading keeps context lean. Unlike MCPs which consume context even when unused, Skills only enter the context when needed.
-->

---
layout: two-cols-header
---

# <span class="text-skill">Skills</span> — Invocation <span class="muted">(2/3)</span>

::left::

**Model invocation** — Triggered automatically

`.claude/skills/api-conventions/SKILL.md`

```markdown
---
name: api-conventions
description: |
  API design patterns for this codebase.
  Use when writing API endpoints or reviewing API code.
user-invocable: false
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
```

<div class="muted mt-2">Claude decides relevance from <code>description</code> — explicit trigger phrases help</div>

::right::

**User invocation** — you trigger with `/skill-name`

`.claude/skills/drafting-pr/SKILL.md`

```markdown
---
name: drafting-pr
description: Create a pull request with consistent formatting
---

## Workflow
1. Read the git diff for changes
2. Draft a PR title and summary
3. Create the PR via gh cli

## Examples
See references/example.md for real examples.
```

<!--
User invocation = slash command you type. Model invocation = Claude reads the description and decides if it's relevant. The description is the key — write it like trigger phrases that tell Claude when to activate the skill. Set disable-model-invocation: true for workflows with side effects.
-->

---
layout: two-cols-header
---

# <span class="text-skill">Skills</span> — Content Types <span class="muted">(3/3)</span>

::left::

**Reference** content type — "Know this"

`.claude/skills/api-conventions/SKILL.md`

```markdown
---
name: api-conventions
description: |
  API design patterns for this codebase.
  Use when writing API endpoints or reviewing API code.
user-invocable: false
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
```

<div class="muted mt-2">Adds knowledge to Claude's context</div>

::right::

**Task** content type — "Do this"

`.claude/skills/pr-review/SKILL.md`

```markdown
---
name: drafting-pr
description: Create a pull request with consistent formatting
---

## Workflow
1. Read the git diff for changes
2. Draft a PR title and summary
3. Create the PR via gh cli

## Examples
See references/example.md for real examples.
```

<div class="muted mt-2">Step-by-step instructions Claude executes</div>

<!--
Reference makes Claude smarter about your project — coding standards, API conventions. Task gives Claude a playbook to follow — PR review, deployment steps. You can combine both in one skill.
-->

---
layout: two-cols-header
---

# <span class="text-rule">Rules</span> — Deterministic Context Loading

::left::

`.claude/rules/api-design.md`

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules

- All endpoints must include input validation
- Use the standard error response format
- Include OpenAPI documentation comments
```

::right::

- Markdown files in `.claude/rules/`
- Like `CLAUDE.md`, but **conditionally loaded** by file pattern
- **Always loaded** when conditions match — no AI decision-making
- `paths` field accepts **glob patterns** for path-specific rules
- Without `paths` — loads unconditionally at session start

<!--
Rules are deterministic — no model discretion. When Claude touches a file matching a glob pattern, the rule is injected automatically. Use rules for things that must always apply. Different parts of your codebase can have different rules.
-->

---

# <span class="text-hook">Hooks</span> — Deterministic Automation

<div class="subtitle">Automation triggered at lifecycle points — configured in <code>.claude/settings.json</code></div>

- Key events: `PreToolUse`, **`PostToolUse`**, `Stop`, `SessionStart`
- Three handler types: **Command** (shell) · **HTTP** (endpoint) · **Prompt** (Claude evaluates)
- Claude sees errors and **fixes them in the same session**

```jsonc {5,8}
// .claude/settings.json — run tsc & eslint after every edit
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "npx tsc --noEmit && npx eslint --fix"
      }]
    }]
  }
}
```

<!--
Hooks run automated checks whenever Claude takes an action. PostToolUse fires after Claude edits a file. The matcher "Edit|Write" targets file modifications. It runs tsc and eslint automatically — if there are errors, Claude sees them and fixes them right away. No human intervention needed.
-->

---

# <span class="text-skill">Skills</span> vs <span class="text-rule">Rules</span> vs <span class="text-hook">Hooks</span>

<div class="accent-bar"></div>

<div class="mt-2">

|               | <span class="text-skill">**Skills**</span> | <span class="text-rule">**Rules**</span> | <span class="text-hook">**Hooks**</span> |
|---------------|---------------------|------------------------------|------------------------|
| **Trigger**   | User or model       | File glob / always           | Lifecycle event        |
| **Deterministic?** | No <span class="muted">(LLM output varies)</span> | No <span class="muted">(LLM output varies)</span> | Yes / No <span class="muted">(depends on handler)</span> |
| **Purpose**   | Add knowledge / run tasks | Inject context          | Automate actions       |
| **Example**   | `/drafting-pr`      | "Use camelCase in `src/`"    | Lint after every edit  |

</div>

<div class="mt-6 px-4 py-3 rounded-lg" style="background: rgba(96, 165, 250, 0.08); border-left: 3px solid #60a5fa;">

**Use all three together:** Rules set standards · Hooks enforce them · Skills add capabilities

</div>


<!--
Key distinction: Skills and Rules feed instructions to the LLM, so you can't guarantee the same output every time. Hooks run external tools like linters — calling eslint always produces the same result for the same code. Hooks with Command/HTTP handlers are deterministic; Prompt handlers use the LLM, so they are not. That's why Hooks are your safety net.
-->

---

# Why Claude Code?

<div class="subtitle">Copilot now also has Skills, Rules, and Hooks — but Claude Code <strong>ships first</strong></div>

| Feature | Claude Code | GitHub Copilot |
|---------|-------------|----------------|
| **Skills** | **Oct 2025** | Dec 2025 |
| **Rules** | **Dec 2025** | Dec 2025 |
| **Hooks** | **Jul 2025** | Feb 2026 |

<div class="card-grid-2 mt-4">
  <div class="card">
    <strong>More efficient</strong>
    <div class="muted mt-1">Higher output quality = less rework. More expensive, but better ROI.</div>
  </div>
  <div class="card">
    <strong>Direct from the model maker</strong>
    <div class="muted mt-1">Tightest integration possible between model and tooling.</div>
  </div>
</div>

<!--
Claude Code consistently ships features first. Skills launched Oct 2025, two months before Copilot's Agent Skills in Dec 2025. Hooks came to Claude Code in Jan 2026, with Copilot following in Jan-Mar 2026. For Rules, Copilot's path-specific .instructions.md files (Jul 2025) predate Claude Code's .claude/rules/ (Dec 2025) — but Copilot's earlier Custom Instructions (Oct 2024) are more like CLAUDE.md, not Rules.
-->

---
layout: center
---

# Effective Use: Reality Check

<div class="text-2xl font-light mb-8 opacity-80">
"Build complex products in one day" — <strong>only for new projects</strong>
</div>

- In existing projects, most time = **planning, negotiation, coordination** — not coding
- AI code can mean **more review burden** — not less
- **You own every line** that gets merged
  <br><span class="muted">Function docs · PR descriptions · Screenshots · Test evidence</span>

<!--
The hype is real for brand-new projects. But most of us work in teams on existing products. Coding is maybe 30% of the work. When AI writes code, someone still reviews it. You are accountable for every line that gets merged.
-->

---

# Effective Use: Automate, Don't Generate

<div class="accent-bar"></div>

- AI generates code fast — but without guardrails, **review becomes the bottleneck**
- **Invest in infrastructure first** — linters, Rules, Hooks, validators
  <br><span class="muted">Generated code becomes controllable and predictable</span>
- Then use **Skills** to automate repetitive workflows — PR drafting, formatting, boilerplate
- With the right foundation, AI output is **fast and trustworthy**

<!--
Speed without control is an illusion. If AI generates code that doesn't follow conventions, you spend just as long reviewing it. The key: set up your infrastructure first — linters, Rules, Hooks, validators. Once those guardrails are in place, generated code is predictable and consistent. Then layer on Skills for repetitive tasks. That's how you actually go fast.
-->

---

# Example 1: /drafting-pr

<div class="subtitle">Automated PR Creation — consistent formatting, every time</div>

- Generates PRs with summary, change list, test plan, screenshots
- Invoke with `/drafting-pr` after completing your changes
- Source: `ibara1454/agent-marketplace`

<div class="mt-8 flex justify-center">
  <div class="badge badge-demo">Live Demo</div>
</div>

<!--
Show how typing /drafting-pr produces a complete, well-structured PR. Emphasize consistency — every PR follows the same template, every time.
-->

---

# Example 2: /uploading-attachments

<div class="subtitle">Upload Files to GitHub — no more manual drag-and-drop</div>

- Uploads local files (images, logs) to GitHub issues or PRs
- Handles GitHub API interaction automatically
- Source: `ibara1454/agent-marketplace`

<div class="mt-8 flex justify-center">
  <div class="badge badge-demo">Live Demo</div>
</div>

<!--
Show how the skill takes a local file and attaches it to a GitHub issue/PR without manual drag-and-drop.
-->

---
layout: end
---

# Learn More

<div class="text-left mx-auto max-w-lg">

- [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action) — interactive course
- [Best Practices](https://code.claude.com/docs/en/best-practices) — official guide
- [Extend Claude with Skills](https://code.claude.com/docs/en/skills) — skills docs
- [Complete Guide to Building Skills (PDF)](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf)
- [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- Follow [@bcherny on X](https://x.com/bcherny) for updates

</div>

<div class="mt-8 text-xl opacity-50">
  Thank you
</div>

<!--
The Claude Code in Action course on Skilljar is a great hands-on starting point. The Complete Guide to Building Skills PDF is the most thorough reference. Thank you!
-->
