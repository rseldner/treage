---
name: treage-build
description: >
  Generate a complete, working Treage decision tree HTML file from a plain-language
  description of decision logic. Use when asked to build, create, or generate a Treage
  tree, decision tree, or routing flowchart using the Treage framework.
metadata:
  author: rseldner
  version: 1.3.0
  treage-engine: "1.8.0"
---

# Treage Tree Builder

Treage is a lightweight, single-file HTML/JS framework for interactive decision trees. It renders from two plain JS objects — `CONFIG` (visual config) and `TREE` (decision logic) — using D3.js. No build step, no backend, no login required.

For detailed CONFIG and TREE schema, see [references/GRAMMAR.md](references/GRAMMAR.md).
For agent traversal of existing trees, see the **treage-agent** skill.

## When to Use This Skill

- User asks to create or generate a Treage decision tree
- User asks to convert a flowchart, runbook, or decision process into Treage format
- User asks to add branches, nodes, or outcomes to an existing Treage tree
- User asks to update CONFIG styling or node types in an existing Treage file

## Workflow

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

### Step 2a — Identify the workflow shape

Before choosing node types, classify the workflow structure. This determines whether a straightforward tree will work or whether compromises are needed.

**Binary/multi-path branching (standard)** — each decision gate leads to meaningfully different downstream steps. Model as question nodes with distinct child branches.

**Linear with pre-flight instructions** — some early steps are setup instructions, not real decision gates. If both branches of a question lead to identical downstream steps, collapse it into a single `action` node with the instruction in the `hint`. This keeps the path connected without duplication.

**Convergent paths (graph problem)** — if multiple branches need to rejoin at a shared downstream step (e.g. "check existing case → if yes, skip investigation and go straight to acknowledge"), Treage cannot express this natively. The TREE is a JSON object tree, not a graph — there is no ID-based node reference system, so a shared node would have to be duplicated in full on each branch. Options:
- Duplicate the shared subtree on each branch (explicit but bloated — maintenance burden doubles)
- Collapse the pre-convergence branch into an `action` node and note the skip in the `hint` (loses the branch but keeps paths connected)
- Note the limitation and reference issue [#28](https://github.com/rseldner/treage/issues/28) for the planned `jumpTo` enhancement

For most runbook-style workflows, the pre-flight collapse is the right call. Only duplicate if the two branches have genuinely different outcomes downstream.

### Step 3 — Choose or define outcome types

Default outcome types in the boilerplate are: `action`, `review`, `combined`, `flag`, `stop`.

If the user's use case needs different types (e.g., `escalate`, `resolve`, `defer`), define new types in `CONFIG.nodeTypes` using the correct shape. See [references/GRAMMAR.md](references/GRAMMAR.md) for the full node type schema.

Remove unused default types from CONFIG to keep the legend clean.

### Step 4 — Generate the file

Produce a complete, self-contained HTML file modelled on [assets/boilerplate.html](assets/boilerplate.html), which documents every feature with inline comments. Always use:
- D3.js from `https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js`
- Treage engine from `https://cdn.jsdelivr.net/gh/rseldner/treage@1.8.1/treage.js`
- Script load order: D3 → CONFIG+TREE script block → treage.js (strict — do not reorder)

**Schema source — do not work from memory.** The CONFIG and TREE schemas are defined by the engine and key names are easy to get wrong. If `references/GRAMMAR.md` is unavailable, fetch the authoritative defaults directly from the playground:

```bash
curl -s https://raw.githubusercontent.com/rseldner/treage/main/playground/playground.html | grep -A 100 "const DEFAULT_CONFIG"
```

Key schema facts to verify before writing CONFIG:
- `title`, `subtitle`, `fonts` are top-level CONFIG keys — not nested under a `meta` wrapper
- Palette keys are `bg`, `muted`, `dim`, `headerBg`, `gridLine`, `edgeYes`, `edgeNo` — not `background`, `textMuted`, `surfaceAlt`, etc.
- nodeType appearance uses `dark`/`light` sub-objects with `bg`, `border`, `borderLeft`, `titleColor`, `hintColor`, `eyebrowColor` — not `palette`/`paletteLight` with `fill`/`stroke`/`text`
- TREE nodes use `title` not `label`

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
- [ ] Any `jumpTo` values reference IDs that actually exist in the tree, and are only set on leaf nodes
- [ ] Any `links` entries have both `label` and `url` populated
- [ ] Any `code` values are plain strings; multiline content uses `\n` not literal newlines inside the JS object

## Output Format

Deliver a complete HTML file. Do not deliver just the CONFIG or TREE blocks in isolation unless the user specifically asks for a partial update to an existing file.

Include a brief comment at the top of the `<script>` block noting the Treage version and date generated.

## Common Failure Modes

- **Orphaned types** — using a type key in TREE that isn't defined in CONFIG.nodeTypes
- **Missing edgeLabel on non-root nodes** — every node except `start` needs one
- **Leaf nodes with children** — outcome nodes must have `children: []` or no children key
- **Duplicate IDs** — each `id` must be unique across the entire tree
- **Question nodes as leaves** — a node with no children must not have `type: "question"`
- **Wrong script order** — treage.js must load after D3 and after the CONFIG/TREE script block
- **Missing paletteLight** — both `palette` and `paletteLight` are required; omitting one breaks the theme toggle
- **isNewUntil dates in the past** — if using `isNewUntil`, use a future date or omit the field
- **Wrong CONFIG schema keys** — palette and nodeType key names are specific to the engine version. Do not invent keys like `background`, `textMuted`, `fill`, `stroke`. Fetch `DEFAULT_CONFIG` from the playground to verify (see Step 4).
- **`label` instead of `title` on TREE nodes** — TREE nodes use `title` for display text. Using `label` silently produces nodes with no visible title.
- **Convergent path treated as a tree** — if you find yourself writing identical subtrees on multiple branches, stop. Either the workflow has a convergent path (graph problem — see Step 2a) or the branching question is not a real gate and should be collapsed into a pre-flight `action` node.

## Open issues

The following engine enhancements are tracked and relevant when building trees:

- [#30](https://github.com/rseldner/treage/issues/30) — `jumpTo` navigation directly from diagram view click is not yet supported; Continue button only appears in the expanded outcome card
- [#31](https://github.com/rseldner/treage/issues/31) — Builder UI has no field for `jumpTo`; add it manually in the TREE source editor
- [#32](https://github.com/rseldner/treage/issues/32) — Builder UI has no fields for `links`; add them manually in the TREE source editor
- Builder UI has no field for `code`; add it manually in the TREE source editor