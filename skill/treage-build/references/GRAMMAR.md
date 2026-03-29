# Treage Grammar Reference
> Engine: v1.8.0 | Last updated: 2026-03-29

This document is the authoritative reference for the CONFIG and TREE object schemas.
It is loaded into context when the Treage skill is active.

---

## File Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Tree Title</title>

  <!-- 1. D3 — must load first -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js"></script>

  <!-- 2. Your CONFIG and TREE -->
  <script>
    const CONFIG = { ... };
    const TREE   = { ... };
  </script>

  <!-- 3. Treage engine — must load last -->
  <script src="https://cdn.jsdelivr.net/gh/rseldner/treage@1.7.0/treage.js"></script>
</head>
<body></body>
</html>
```

Script load order is strict. Treage will throw if D3 or CONFIG/TREE are missing when it initializes.

---

## CONFIG

### Top-level fields

| Field | Type | Required | Description |
|---|---|---|---|
| `title` | string | yes | Browser tab title and header h1 |
| `subtitle` | string | yes | Shown below title in header |
| `icon` | string | no | Raw SVG string injected into header. Use `""` to omit. |
| `palette` | object | yes | Dark theme color map |
| `paletteLight` | object | yes | Light theme color map (same keys as palette) |
| `fonts` | object | no | `{ body: "...", mono: "..." }` font stacks |
| `nodeTypes` | object | yes | One entry per node type (see below) |
| `edgeLabels` | object | no | Override colors for specific edge label strings (v1.2.3+) |

### Palette keys

Both `palette` and `paletteLight` require the same set of keys:

| Key | Controls |
|---|---|
| `bg` | Page / canvas background |
| `surface` | Card and control backgrounds |
| `border` | Default border color |
| `text` | Primary text |
| `muted` | Secondary / subdued text |
| `dim` | Eyebrow labels, placeholders |
| `accent` | Brand color — hover states, active toggle, progress dots |
| `edgeYes` | YES edges and YES answer buttons |
| `edgeNo` | NO edges and NO answer buttons |
| `edgeDefault` | Unlabelled or custom-label edges |
| `gridLine` | Background grid lines (use `rgba()`) |
| `headerBg` | Header and footer background (use semi-transparent `rgba()`) |

### edgeLabels (optional, v1.2.3+)

Override the color for any specific edge label string:

```js
edgeLabels: {
  "CRITICAL": "#ff4444",
  "LOW":      "#3dd68c",
}
```

If omitted, YES uses `edgeYes`, NO uses `edgeNo`, everything else uses `edgeDefault`.

### nodeTypes

Two keys are **reserved** and must always be present. Do not rename or remove them:

- `start` — the root node. Exactly one per tree.
- `question` — all internal branching nodes.

All other keys are outcome types. You can add, remove, or rename them freely.

#### nodeType shape

```js
myType: {
  label:       "Short Label",   // shown in eyebrow and outcome badge
  legendLabel: "Legend Text",   // shown in header legend pill. null = hidden
  accent:      "#hexcolor",     // dot color in legend pill

  dark: {
    bg:          "...",   // card background (CSS color or gradient)
    border:      "...",   // CSS border shorthand
    borderLeft:  "...",   // left accent stripe (slightly more opaque than border)
    titleColor:  "...",   // main title text color
    hintColor:   "...",   // italic hint text color
    eyebrowColor:"...",   // small label above title
  },

  light: {
    // same keys as dark
  },
}
```

For `start` and `question`, `label` and `legendLabel` are optional (both are typically `null`).

#### Default outcome types (boilerplate)

| Key | Color | Use for |
|---|---|---|
| `action` | Teal `#00bfb3` | Positive "go do this" paths |
| `review` | Pink `#f04e98` | Needs human investigation |
| `combined` | Yellow `#fec514` | Two things both apply |
| `flag` | Purple `#a87ff0` | Flag for follow-up / known issue |
| `stop` | Grey `#7a8099` | No action needed |

---

## TREE

The TREE object is a single root node with nested `children` arrays.

### Node fields

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Unique identifier. No spaces or slashes. Convention: `"q1"`, `"q2a"`, `"out-approve"` |
| `type` | string | yes | A key from `CONFIG.nodeTypes`. Use `"question"` for all branching nodes. |
| `eyebrow` | string | yes | Small label above the title. Questions: "Question 1". Outcomes: category label. |
| `title` | string | yes | Main node text. Questions: phrase as a clear answerable question. Outcomes: the recommended action. |
| `hint` | string or null | no | Italic subtext below title. Use for gate conditions, scope boundaries, clarifying notes. `null` omits. |
| `edgeLabel` | string | yes* | Label on the edge from parent to this node. *Omit or use `""` on the root `start` node's first child only. |
| `children` | array | yes** | Nested child nodes. **Must be `[]` or omitted on outcome (leaf) nodes. Must be non-empty on question nodes. |
| `isNew` | boolean | no | Renders a yellow NEW badge next to the eyebrow. |
| `isNewUntil` | string (ISO date) | no | Like `isNew` but auto-expires. Format: `"2025-12-31"`. If date is past, badge does not show. (v1.2.3+) |
| `links` | array | no | Array of `{ label, url }` objects. Renders as clickable buttons. Available on any node type. (v1.7.0+) |
| `jumpTo` | string | no | ID of another node. Renders a "Continue →" button on the outcome card that teleports the walk to the target node. **Intended for leaf nodes only.** Setting `jumpTo` on a question node is not enforced by the engine but will produce a misleading Continue button alongside active branch choices — do not do this. (v1.6.0+) |
| `code` | string | no | A monospace code block rendered below the hint. Multiline strings supported — use `\n` for line breaks. In diagram view, blocks of ≤2 lines are always visible; longer blocks show the first 2 lines with a Show/Hide toggle. In walk/interactive view, the full block is always visible with a Copy button. (v1.8.0+) |

### edgeLabel values

| Value | Renders as |
|---|---|
| `"YES"` | Green edge + green answer button |
| `"NO"` | Red edge + red answer button |
| Any other string | Grey edge (or custom color if defined in `CONFIG.edgeLabels`) |
| `""` or omitted | No label — use only on the first question after `start` |

Non-binary branching is supported. A question can have 2, 3, or 4 children with labels like `"LOW"`, `"MED"`, `"HIGH"` or `"PATH A"`, `"PATH B"`, `"PATH C"`.

### code (optional, v1.8.0+)

A multiline string rendered as a styled monospace block below the hint. Use for query syntax, CLI commands, API calls, or any content that benefits from fixed-width rendering.

```js
{
  id: "out-run-query",
  type: "action",
  eyebrow: "Action Required",
  title: "Run process tree query.",
  hint: "Use alert @timestamp ± 1hr as the time boundary.",
  code: "FROM logs-endpoint.events.process-*\n| WHERE agent.id == '<id>'\n| KEEP @timestamp, process.name, process.parent.name\n| LIMIT 80",
  edgeLabel: "YES",
  children: []
}
```

**Diagram view:** Blocks of ≤2 lines render inline with no toggle. Blocks of >2 lines show the first 2 lines with an ellipsis and a Show/Hide toggle that expands the full block in place. The toolbar "show links" button also expands/collapses all code blocks.

**Walk/Interactive view:** The full block is always visible with a Copy button.

**Search:** `code` content is included in node search alongside `title` and `hint`.

### links (optional, v1.7.0+)

An array of `{ label, url }` objects. Renders as clickable buttons on the node card. Available on any node type.

```js
links: [
  { label: "KB: Red cluster recovery",  url: "https://www.elastic.co/guide/..." },
  { label: "Runbook: shard allocation", url: "https://wiki.example.com/..." },
]
```

In diagram view, a node with `links` shows a bottom strip indicator ("1 action" / "N actions"). First click activates the node and expands the buttons in place; click outside deactivates. The toolbar "Show links" toggle expands all link/jumpTo nodes simultaneously for scanning or screenshots.

In walk/interactive view, links render inline on the active node card.

### jumpTo (optional, v1.6.0+)

A string referencing another node's `id`. When the walk reaches a leaf node with `jumpTo`, a "Continue →" button appears. Clicking it teleports the walk to the target node. The jump is recorded in the audit path as a distinct segment (`,>targetId`) and is included in copy path output and shareable URLs.

```js
{
  id: "out-check-snapshots",
  type: "action",
  eyebrow: "Action Required",
  title: "Check snapshot status before proceeding.",
  hint: null,
  edgeLabel: "PARTIAL RED",
  jumpTo: "q-snapshot-health",
  children: []
}
```

If the `jumpTo` target ID does not exist in the tree, the engine logs a console warning and the button is not rendered.

**Only use `jumpTo` on leaf nodes.** Setting it on a question node is not enforced but produces a misleading Continue button alongside active branch choices.



1. Root node must be `type: "start"` with `id: "start"` and exactly one child.
2. The `start` node's single child is the first question. Its `edgeLabel` is `""` or omitted.
3. Every other node must have an `edgeLabel`.
4. Question nodes must have at least 2 children.
5. Outcome (leaf) nodes must have `children: []` or no `children` key.
6. All `id` values must be unique across the entire tree.
7. Every `type` value must match a key in `CONFIG.nodeTypes`.

### Text length guidelines

For best rendering in the full tree view:
- Titles: under ~120 characters
- Hints: under ~160 characters
- Eyebrows: 1–4 words

### Minimal valid tree example

```js
const TREE = {
  id: "start",
  type: "start",
  title: "Start",
  children: [
    {
      id: "q1",
      type: "question",
      eyebrow: "Question 1",
      title: "Is this a critical severity issue?",
      hint: "Critical = customer-facing outage or data loss.",
      edgeLabel: "",
      children: [
        {
          id: "out-escalate",
          type: "action",
          eyebrow: "Action Required",
          title: "Escalate to on-call engineer immediately.",
          hint: "⊘ Gate: Only if no workaround is available.",
          edgeLabel: "YES",
          children: []
        },
        {
          id: "out-queue",
          type: "stop",
          eyebrow: "No Action Needed",
          title: "Add to standard queue. Follow normal SLA.",
          hint: null,
          edgeLabel: "NO",
          children: []
        }
      ]
    }
  ]
};
```

---

## Features by version

| Feature | Version | Field / mechanism |
|---|---|---|
| Basic tree | 1.0.0 | `CONFIG`, `TREE` |
| Interactive walk mode | 1.2.0 | (automatic) |
| SVG export | 1.2.2 | (automatic toolbar button) |
| `isNewUntil` badge | 1.2.3 | `isNewUntil: "YYYY-MM-DD"` |
| Custom edge label colors | 1.2.3 | `CONFIG.edgeLabels` |
| Copy node link | 1.3.0 | (automatic) |
| Node search | 1.3.0 | (automatic, "/" key) |
| Copy path summary | 1.3.2 | (automatic toolbar button) |
| Keyboard navigation | 1.4.0 | (automatic, arrow keys) |
| Builder focus sync | 1.5.0 | (automatic, postMessage highlight) |
| `jumpTo` — walk teleport with audit trail | 1.6.0 | `jumpTo: "nodeId"` on leaf nodes |
| `links` — clickable action buttons on nodes | 1.7.0 | `links: [{ label, url }]` |
| `code` — monospace code block in nodes | 1.8.0 | `code: "..."` (multiline via `\n`) |

All features listed are automatic — no TREE/CONFIG changes needed to enable them.

---

## Maintenance notes

Update this file when:
- A new node field is added to TREE
- A new CONFIG key is introduced
- A new built-in feature changes what valid CONFIG/TREE looks like
- The CDN version tag changes

Do not update this file for rendering fixes, UI tweaks, or changes that don't affect the CONFIG/TREE surface.

---

## agentMode (optional, agent traversal)

An optional CONFIG block that controls LLM agent behavior when traversing the tree. Has no effect on the visual rendering — it is read only by traversal agents using `AGENT.md`.

```js
const CONFIG = {
  // ... existing CONFIG fields ...

  agentMode: {
    onLowConfidence: "flag-and-continue",  // or "halt"
    outputFormat:    "path-and-outcome",   // or "outcome-only"
  },
};
```

### agentMode fields

| Key | Values | Default | Effect |
|---|---|---|---|
| `onLowConfidence` | `"flag-and-continue"` | yes | Flag low-confidence decisions inline, continue traversal |
| `onLowConfidence` | `"halt"` | — | Stop traversal, return partial path, request clarification |
| `outputFormat` | `"path-and-outcome"` | yes | Full path + outcome (assisted / audit mode) |
| `outputFormat` | `"outcome-only"` | — | Outcome + flags only (automated pipeline mode) |

If `agentMode` is absent, the agent uses both defaults.

### When to add agentMode

Add `agentMode` to CONFIG when:
- You intend the tree to be consumed by an LLM traversal agent
- You want to control confidence handling or output verbosity for your deployment
- You are building an automated pipeline and need outcome-only output

Leave it out if the tree is only used as a visual/interactive artifact.

### Maintenance notes

Update this section when:
- New `agentMode` keys are added
- Confidence levels or output format options change
- The traversal behavior described in `AGENT.md` changes in a way that affects CONFIG