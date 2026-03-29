# Example: Elasticsearch Alert Triage

## Input (plain language)

> "Build a tree that helps support engineers decide what to do when an Elasticsearch
> cluster alert fires. First ask if the cluster is red or yellow. If red, ask if it's
> a partial or full red. Full red should escalate immediately. Partial red should
> check snapshots then escalate. Yellow just needs monitoring. There's also a no-cluster
> path for when the alert is a false positive."

---

## Reasoning trace

1. Top question: cluster status — not binary, three states (red, yellow, false positive)
   → use custom edge labels: "RED", "YELLOW", "FALSE POSITIVE"

2. Red branch splits into partial vs full → binary YES/NO sub-question

3. Outcomes needed:
   - `escalate` (urgent action) — use `action` type
   - `snapshot-check` (investigation before escalation) — use `review` type
   - `monitor` (watch and wait) — use `flag` type
   - `close` (no action) — use `stop` type

4. Remove unused boilerplate types: `combined` not needed here.

5. IDs: q1, q2-red, out-escalate-full, out-snapshot, out-monitor, out-close

---

## Output

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ES Alert Triage</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js"></script>
<script>
/* Generated with Treage skill v1.1.0 | Engine v1.5.0 */

const CONFIG = {
  title:    "ES Alert Triage",
  subtitle: "What to do when a cluster alert fires.",
  icon: "",

  palette: {
    bg: "#0d0f14", surface: "#13161e", border: "#252a38",
    text: "#e8eaf0", muted: "#7a8099", dim: "#4a5068",
    accent: "#00bfb3", edgeYes: "#3dd68c", edgeNo: "#ff6b6b",
    edgeDefault: "#3a4158", gridLine: "rgba(255,255,255,0.018)",
    headerBg: "rgba(13,15,20,0.95)",
  },

  paletteLight: {
    bg: "#f4f5f7", surface: "#ffffff", border: "#dde0e8",
    text: "#1a1d26", muted: "#6b7080", dim: "#9298aa",
    accent: "#009e99", edgeYes: "#1fa860", edgeNo: "#d94f4f",
    edgeDefault: "#b0b6c6", gridLine: "rgba(0,0,0,0.045)",
    headerBg: "rgba(244,245,247,0.97)",
  },

  fonts: {
    body: "'IBM Plex Sans', sans-serif",
    mono: "'IBM Plex Mono', monospace",
  },

  nodeTypes: {
    start: {
      legendLabel: null, accent: "#00bfb3",
      dark:  { bg: "linear-gradient(135deg,#00bfb3,#009e99)", border: "none", borderLeft: null, titleColor: "#0d0f14" },
      light: { bg: "linear-gradient(135deg,#00bfb3,#009e99)", border: "none", borderLeft: null, titleColor: "#ffffff" },
    },
    question: {
      legendLabel: null, accent: "#3a4158",
      dark:  { bg: "#13161e", border: "1px solid #252a38", borderLeft: "3px solid #252a38", titleColor: "#e8eaf0", hintColor: "#7a8099", eyebrowColor: "#4a5068" },
      light: { bg: "#ffffff", border: "1px solid #dde0e8", borderLeft: "3px solid #c4c9d8", titleColor: "#1a1d26", hintColor: "#6b7080", eyebrowColor: "#9298aa" },
    },
    action: {
      label: "Escalate", legendLabel: "Escalate", accent: "#00bfb3",
      dark:  { bg: "rgba(0,191,179,0.06)", border: "1px solid rgba(0,191,179,0.3)", borderLeft: "3px solid rgba(0,191,179,0.6)", titleColor: "rgba(0,191,179,0.9)", hintColor: "rgba(0,191,179,0.5)", eyebrowColor: "#00bfb3" },
      light: { bg: "rgba(0,158,153,0.05)", border: "1px solid rgba(0,158,153,0.35)", borderLeft: "3px solid rgba(0,158,153,0.65)", titleColor: "#007a76", hintColor: "#009e99", eyebrowColor: "#007a76" },
    },
    review: {
      label: "Investigate", legendLabel: "Investigate", accent: "#f04e98",
      dark:  { bg: "rgba(240,78,152,0.06)", border: "1px solid rgba(240,78,152,0.3)", borderLeft: "3px solid rgba(240,78,152,0.6)", titleColor: "rgba(240,78,152,0.9)", hintColor: "rgba(240,78,152,0.5)", eyebrowColor: "#f04e98" },
      light: { bg: "rgba(240,78,152,0.04)", border: "1px solid rgba(200,50,120,0.3)", borderLeft: "3px solid rgba(200,50,120,0.6)", titleColor: "#b5306e", hintColor: "#c8407e", eyebrowColor: "#b5306e" },
    },
    flag: {
      label: "Monitor", legendLabel: "Monitor", accent: "#a87ff0",
      dark:  { bg: "rgba(168,127,240,0.06)", border: "1px solid rgba(168,127,240,0.3)", borderLeft: "3px solid rgba(168,127,240,0.6)", titleColor: "rgba(168,127,240,0.9)", hintColor: "rgba(168,127,240,0.5)", eyebrowColor: "#a87ff0" },
      light: { bg: "rgba(130,90,210,0.05)", border: "1px solid rgba(130,90,210,0.3)", borderLeft: "3px solid rgba(130,90,210,0.6)", titleColor: "#6236b5", hintColor: "#7a50c8", eyebrowColor: "#6236b5" },
    },
    stop: {
      label: "Close", legendLabel: "Close", accent: "#7a8099",
      dark:  { bg: "rgba(122,128,153,0.05)", border: "1px solid rgba(122,128,153,0.25)", borderLeft: "3px solid rgba(122,128,153,0.4)", titleColor: "rgba(122,128,153,0.85)", hintColor: "rgba(122,128,153,0.5)", eyebrowColor: "#7a8099" },
      light: { bg: "rgba(100,110,140,0.04)", border: "1px solid rgba(100,110,140,0.25)", borderLeft: "3px solid rgba(100,110,140,0.4)", titleColor: "#505870", hintColor: "#7a8099", eyebrowColor: "#606880" },
    },
  },
};

const TREE = {
  id: "start",
  type: "start",
  title: "Start",
  children: [
    {
      id: "q1",
      type: "question",
      eyebrow: "Question 1",
      title: "What is the current cluster health status?",
      hint: "Check /_cluster/health or Kibana Stack Monitoring.",
      edgeLabel: "",
      children: [
        {
          id: "q2-red",
          type: "question",
          eyebrow: "Question 2",
          title: "Is the red status affecting all shards or only some?",
          hint: "Full red = no primary shards available. Partial = some primaries unassigned.",
          edgeLabel: "RED",
          children: [
            {
              id: "out-escalate-full",
              type: "action",
              eyebrow: "Escalate",
              title: "Page on-call engineer immediately. Full red is a cluster-wide outage.",
              hint: "⊘ Gate: Confirm no scheduled maintenance window is active.",
              edgeLabel: "FULL RED",
              children: []
            },
            {
              id: "out-snapshot",
              type: "review",
              eyebrow: "Investigate",
              title: "Check recent snapshot status and run /_cluster/allocation/explain before escalating.",
              hint: "⊘ Gate: If snapshot restore is in progress, wait for completion before escalating.",
              edgeLabel: "PARTIAL RED",
              children: []
            }
          ]
        },
        {
          id: "out-monitor",
          type: "flag",
          eyebrow: "Monitor",
          title: "Yellow is degraded but not critical. Monitor for 15 minutes and reassess.",
          hint: "Set a reminder. If status has not improved, re-enter this tree.",
          edgeLabel: "YELLOW",
          children: []
        },
        {
          id: "out-close",
          type: "stop",
          eyebrow: "Close",
          title: "Alert is a false positive. Dismiss and document in the ticket.",
          hint: "⊘ Gate: Verify cluster health is actually green before closing.",
          edgeLabel: "FALSE POSITIVE",
          children: []
        }
      ]
    }
  ]
};
</script>
<script src="https://cdn.jsdelivr.net/gh/rseldner/treage@1.8.0/treage.js"></script>
</head>
<body></body>
</html>
```

---

## What this example demonstrates

- Non-binary root question (3 children with custom edge labels)
- Relabeled outcome types (kept same colors, changed `label` values to match domain)
- Removed unused `combined` type from CONFIG
- `hint` used as gate conditions on outcome nodes (⊘ prefix convention)
- Consistent eyebrow pattern: "Question N" for questions, outcome label for leaves
- All IDs unique and meaningful (`q2-red`, `out-escalate-full`, not `q2`, `out1`)