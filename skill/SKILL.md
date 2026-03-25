---
name: skill
description: Generate a complete, working Treage decision tree HTML file from a plain-language description of decision logic, or traverse an existing Treage TREE object as an agent to walk decision logic and produce a structured outcome. Use when asked to build, create, generate, or walk a Treage tree, decision tree, or routing flowchart using the Treage framework.
metadata:
  treage-engine: "1.4.0"
---

# Treage Skill

Treage is a lightweight, single-file HTML/JS framework for interactive decision trees. It renders from two plain JS objects — `CONFIG` (visual config) and `TREE` (decision logic) — using D3.js. No build step, no backend, no login required.

This skill covers two distinct capabilities:

- **Generation** — build a Treage HTML file from a description of decision logic
- **Traversal** — walk an existing TREE object as an agent, evaluate questions against input context, and produce a structured path and outcome

## When to Use This Skill

**Generation:**
- User asks to create or generate a Treage decision tree
- User asks to convert a flowchart, runbook, or decision process into Treage format
- User asks to add branches, nodes, or outcomes to an existing Treage tree
- User asks to update CONFIG styling or node types in an existing Treage file

**Traversal:**
- User provides a TREE object and asks the agent to walk or evaluate it
- User provides a support case, ticket, or log and asks the agent to route it through a tree
- User asks what outcome a given input would reach in a Treage tree

---

## Generation Workflow

Follow these steps in order. Do not skip steps or reorder them.

### Step 1 — Understand the decision logic

Before writing any code, identify:
- What is the top-level decision or routing question?
- What are the terminal outcomes (leaf nodes)?
- How many distinct outcome types are needed?
- Are branches binary (YES/NO) or multi-path (3–4 options)?

If the user's description is ambiguous or the branching logic is unclear, ask clarifying questions before proceeding. A bad tree structure is harder to fix than a delayed start.

### Step 2 — Plan the node structure

Sketch the tree mentally (or show the user a plain-text outline if the tree is complex) before writing CONFIG/TREE. Verify:
- Root → Start node → Q1 → branches → outcomes
- Every non-root node has an edgeLabel
- Every leaf node is a valid outcome type (not `question`)
- No question node has zero children
- IDs are unique across the entire TREE

For complex trees (5+ questions), show the user the proposed outline and confirm before generating the full file.

### Step 3 — Choose or define outcome types

Default outcome types in the boilerplate are: `action`, `review`, `combined`, `flag`, `stop`.

If the user's use case needs different types (e.g., `escalate`, `resolve`, `defer`), define new types in `CONFIG.nodeTypes` using the correct shape. See `references/grammar.md` for the full node type schema.

Remove unused default types from CONFIG to keep the legend clean.

### Step 4 — Generate the file

Produce a complete, self-contained HTML file using the template in `assets/starter-template.html` as the base. Always use:
- D3.js from `https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js`
- Treage engine from `https://cdn.jsdelivr.net/gh/rseldner/treage@1.4.0/treage.js`
- Script load order: D3 → CONFIG+TREE script block → treage.js

### Step 5 — Validate before delivering

Run through this checklist mentally before presenting output:

**Structure**
- [ ] Root node has `type: "start"` and exactly one child
- [ ] All branching nodes have `type: "question"`
- [ ] All leaf nodes have a type matching a key in `CONFIG.nodeTypes`
- [ ] No leaf node has a non-empty `children` array
- [ ] No question node has an empty or missing `children` array
- [ ] Every node has a unique `id` (no duplicates)
- [ ] Every non-root node has an `edgeLabel`
- [ ] The first question after `start` has `edgeLabel: ""`

**CONFIG**
- [ ] Both `palette` and `paletteLight` are present with all required keys
- [ ] Every `type` value used in TREE has a matching key in `CONFIG.nodeTypes`
- [ ] `start` and `question` types are present in `CONFIG.nodeTypes` (reserved, do not remove)

**Content**
- [ ] Question nodes are phrased as clear, answerable questions
- [ ] Outcome titles describe a concrete action or decision, not just a label
- [ ] `hint` is used where a gate condition or scope boundary would help users
- [ ] `eyebrow` text is consistent (e.g., "Question 1", "Question 2" or "Step 1", "Step 2")

## Generation Output Format

Deliver a complete HTML file. Do not deliver just the CONFIG or TREE blocks in isolation unless the user specifically asks for a partial update to an existing file.

Include a brief comment at the top of the `<script>` block noting the Treage version and date generated.

## Common Generation Failure Modes

- **Orphaned types**: using a type key in TREE that isn't defined in CONFIG.nodeTypes
- **Missing edgeLabel on non-root nodes**: every node except `start` needs one
- **Leaf nodes with children**: outcome nodes must have `children: []` or no children key
- **Duplicate IDs**: each `id` must be unique across the entire tree
- **Question nodes as leaves**: a node with no children must not have `type: "question"`
- **Wrong script order**: treage.js must load after D3 and after the CONFIG/TREE script block
- **Missing paletteLight**: both `palette` and `paletteLight` are required — omitting one breaks the theme toggle
- **isNewUntil dates in the past**: if using `isNewUntil`, use a future date or omit the field

---

## Traversal

For agent traversal of an existing TREE, follow the workflow and output format defined in `AGENT.md`.

Summary:
1. Read the TREE and input context
2. Walk from root, evaluating each question node against the context
3. Flag low-confidence decisions inline — do not halt unless `CONFIG.agentMode.onLowConfidence` is `"halt"`
4. Surface the node `hint` as the override instruction on flagged steps
5. Output the full path taken and final outcome