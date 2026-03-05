# Treage - Interactive Decision Tree Framework

**[→ Live Demo](https://rseldner.github.io/treage/)**

A self-contained HTML file for building interactive decision trees.  no build tools, no server, no dependencies beyond a D3.js CDN script. Drop it in a browser and it works.

---

## What It Does

The framework renders the same tree in two modes, switchable from the header:

- **Full Tree** - a zoomable, pannable D3 diagram of the entire tree at once. Good for reviewing the full logic or sharing a screenshot.
- **Interactive** - a one-question-at-a-time wizard with breadcrumb navigation, progress dots, and a styled outcome card at the end. Good for day-to-day use by people making routing decisions.

Both modes are driven by the same data. You define the tree once; both views update automatically.

## Preview
![](tree.gif)



---

## How to Build a New Tree

Open the HTML file. There are two clearly marked sections to edit. Everything else is the engine - leave it alone!

---

### Section ① - `CONFIG`

Sets the tree's identity, color theme, and outcome types.

```js
const CONFIG = {
  title:    "My Decision Tree",
  subtitle: "Answer each question to reach a recommendation.",
  icon:     `<svg .../>`,   // SVG string, or "" to omit

  palette: { ... },         // all colors in one place
  fonts:   { ... },         // body and mono font stacks
  nodeTypes: { ... },       // outcome category definitions
};
```

#### `palette` - all color values are plain CSS strings

| Key | What it controls |
|---|---|
| `bg` | Canvas / page background |
| `surface` | Card and control backgrounds |
| `border` | Default border color |
| `text` | Primary text |
| `muted` | Secondary / subdued text |
| `dim` | Eyebrow labels, placeholders |
| `accent` | Brand color - hover states, active toggle |
| `edgeYes` | Color of YES edges and YES buttons |
| `edgeNo` | Color of NO edges and NO buttons |
| `edgeDefault` | Color of unlabelled edges |

Swap the entire palette to rebrand the tool in seconds.

#### `fonts`

```js
fonts: {
  body: "'IBM Plex Sans', sans-serif",
  mono: "'IBM Plex Mono', monospace",
}
```

Any Google Fonts import in the `<style>` block, plus a font-family string here. The CSS uses variables so both views pick up the change.

#### `nodeTypes` - define your outcome categories

Two keys are reserved and must stay:

- **`start`** - the root node (teal gradient pill at the top).
- **`question`** - internal branching nodes. Used automatically; you never set `type: "question"` for an outcome.

Add, rename, or remove any other key freely. Each outcome type needs:

| Property | Purpose |
|---|---|
| `label` | Short string used in the eyebrow and legend |
| `legendLabel` | Text in the legend pill. Set `null` to hide from legend |
| `accent` | Dot color in the legend |
| `bg` | Card background (use `rgba()` for transparency over the dark canvas) |
| `border` | Card border shorthand, e.g. `"1px solid rgba(0,191,179,0.3)"` |
| `borderLeft` | Left accent stripe, e.g. `"3px solid rgba(0,191,179,0.6)"` |
| `titleColor` | Main title text color |
| `hintColor` | Italic hint text color |
| `eyebrowColor` | Eyebrow label color |

The template ships with five generic outcome types as a starting point:

| Key | Color | Intended use |
|---|---|---|
| `action` | Teal | Positive "do this" paths |
| `review` | Pink | Investigate / review paths |
| `combined` | Yellow | Mixed / dual-output paths |
| `flag` | Purple | Known issues, flagged items |
| `stop` | Grey | No action / dead-end paths |

---

### Section ② - `TREE`

The actual content. This is the only section that controls what the tree says and how it branches.

#### Node object reference

```js
{
  id:         "q1",           // unique string, no spaces
  type:       "question",     // "question" or any key from CONFIG.nodeTypes
  eyebrow:    "Question 1",   // small label above the title
  isNew:      true,           // optional - shows a yellow NEW badge
  title:      "Your question or outcome text goes here.",
  hint:       "Optional italic subtext or gate condition.",
  edgeLabel:  "YES",          // label on the edge from parent → this node
  children:   [ ... ]         // nested child nodes; omit or [] for leaves
}
```

#### Node types

| `type` value | When to use |
|---|---|
| `"question"` | Any branching node - has children |
| `"action"`, `"review"`, etc. | Any terminal / outcome node - no children |

#### `edgeLabel` values

| Value | Rendered as |
|---|---|
| `"YES"` | Green edge + green YES button |
| `"NO"` | Red edge + red NO button |
| Any other string | Default grey edge + plain button with that label |
| `""` or omitted | No label on edge (use for the root question) |

Non-binary branching works fine - give a question three or four children with labels like `"LOW"`, `"MED"`, `"HIGH"` and the engine handles it in both views.

#### Tree shape rules

- The root object must have `type: "start"`. There is exactly one.
- The first child of `start` is the first question shown in Interactive mode.
- Any node with no `children` (or an empty `[]`) is treated as a terminal outcome.
- Nesting depth is unlimited.

#### Minimal example

```js
const TREE = {
  id: "start", type: "start", title: "Start",
  children: [
    {
      id: "q1", type: "question",
      eyebrow: "Question 1",
      title: "Is the issue reproducible in a clean environment?",
      hint: "Isolated from customer config, plugins, and custom code.",
      edgeLabel: "",
      children: [
        {
          id: "out-escalate", type: "flag",
          eyebrow: "Escalate",
          title: "Reproduce in a clean environment and open an engineering ticket.",
          hint: "Attach HAR, logs, and minimal repro steps.",
          edgeLabel: "YES", children: []
        },
        {
          id: "out-env", type: "review",
          eyebrow: "Environment Issue",
          title: "Likely a customer environment or config issue. Work the case.",
          hint: "Check plugins, custom mappings, and index settings first.",
          edgeLabel: "NO", children: []
        }
      ]
    }
  ]
};
```

---

## Adding a New Outcome Type

1. Add a key to `CONFIG.nodeTypes` with the visual properties above.
2. Use that key as `type` on any leaf node in `TREE`.
3. Done - the legend, full-tree renderer, and interactive outcome card all pick it up automatically.

---

## Adding or Changing Questions

- **Add a branch** - add an object to any question's `children` array.
- **Add a depth level** - set `type: "question"` on a new child node and give it its own `children`.
- **Add a new question between two existing ones** - insert a question node as the child of the current parent, and move the former children under the new question.
- **Rename a question** - edit `title` and/or `hint`. Node sizing adjusts automatically.

There is no maximum depth or branching factor enforced by the engine.

---

## Controls

### Full Tree view
| Action | How |
|---|---|
| Zoom in/out | Scroll wheel, or `+` / `−` buttons |
| Pan | Click and drag |
| Fit everything on screen | `⊙` button (bottom-right) |

### Interactive view
| Action | How |
|---|---|
| Answer a question | Click YES or NO (or any edge-label button) |
| Go back one step | ← Back button |
| Jump to an earlier step | Click a breadcrumb link |
| Start over from the outcome | ↺ Start over button |

---

## File Structure

The file is intentionally a single HTML file with no external dependencies other than D3 (loaded from cdnjs). There is no build step.

```
decision-tree-framework.html
│
├── <style>           CSS - layout, card styles, interactive view styles
│                     Uses CSS variables set at runtime from CONFIG.palette
│
├── CONFIG            ① Edit this - meta, colors, fonts, outcome types
├── TREE              ② Edit this - questions and outcomes
│
└── ENGINE            Do not edit
    ├── applyPalette()      Writes CONFIG.palette → CSS variables
    ├── autoSize()          Computes node dimensions from text length
    ├── buildLayout()       D3 tree layout → pixel positions
    ├── renderFullTree()    Draws SVG nodes + edges with D3
    ├── iRender()           Renders the interactive wizard step
    ├── iChoose/iGoTo()     Interactive state management
    └── init()              Wires everything together
```

---

## Requirements

- Any modern browser (Chrome, Firefox, Safari, Edge).
- Internet access to load fonts from Google Fonts and D3 from cdnjs. For air-gapped use, download both and update the `<link>` and `<script>` tags to point to local copies.
