# Treage Skills

This directory contains two Agent Skills for working with Treage decision trees.

| Skill | Folder | Purpose |
|---|---|---|
| `treage-build` | `treage-build/` | Generate a Treage HTML file from a description of decision logic |
| `treage-agent` | `treage-agent/` | Walk an existing Treage tree against input context and produce a structured outcome |

Skills follow the [Agent Skills](https://agentskills.io/) open standard — each is a self-contained folder with a `SKILL.md` file containing metadata and instructions an LLM agent follows.

---

## treage-build

Use when you want an LLM to **create** a Treage decision tree from a plain-language description.

The skill handles the full generation workflow: understanding the decision logic, planning the node structure, choosing outcome types, generating the HTML file, and validating the output before delivery.

**Example prompt:**
> "Build a Treage tree that routes support tickets based on severity and whether a workaround exists."

---

## treage-agent

Use when you want an LLM to **traverse** an existing Treage tree against a real input — a support case, log snippet, ticket description, or similar.

The agent walks the tree from root to outcome, records confidence at each decision, flags ambiguous steps with override instructions, and outputs the full path taken and final outcome.

**Example prompt:**
> "Route this ticket through the tree: [paste TREE object and case description]"

---

## How to use

### Claude Code
Both skills are auto-discovered when the `skill/` directory is present in your project. No setup needed — ask Claude to build or walk a Treage tree and the right skill loads automatically.

### Claude.ai Projects
Upload the skill folder contents as project knowledge. Both skills will be available in every conversation within that project.

### Any other LLM (Gemini, ChatGPT, etc.)
Each skill includes a `PROMPT.md` — a paste-anywhere bundle of the full instructions with no skill-loader frontmatter. Paste it into the system instructions field of any LLM interface:

- **Building trees** → paste `treage-build/PROMPT.md`
- **Traversing trees** → paste `treage-agent/PROMPT.md`, then provide your TREE object and input context as the user message

### Your own pipeline
Read the `SKILL.md` files programmatically and inject them as system prompts when the relevant task is detected.

---

## Files

```
skill/
├── README.md                               ← you are here
├── treage-build/
│   ├── SKILL.md                            ← generation skill (auto-loaded by skill runtimes)
│   ├── PROMPT.md                           ← paste-anywhere bundle for building trees
│   ├── MAINTENANCE.md                      ← how to keep the skill in sync with engine updates
│   ├── references/
│   │   └── GRAMMAR.md                      ← full CONFIG and TREE schema reference
│   ├── assets/
│   │   └── starter-template.html           ← base HTML template for generation
│   └── examples/
│       └── es-alert-treage.md              ← annotated example: input description → output tree
└── treage-agent/
    ├── SKILL.md                            ← traversal skill (auto-loaded by skill runtimes)
    ├── PROMPT.md                           ← paste-anywhere bundle for traversing trees
    └── references/
        └── GRAMMAR.md                      ← full CONFIG and TREE schema reference
```