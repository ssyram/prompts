# Evolution Timeline Graph Generator

Given a set of research entries with names, years, categories, and connections, generate a 2D interactive evolution timeline graph (D3.js + SVG, single self-contained HTML file).

## Input

The user provides either:
1. A directory of survey entries (markdown files with Mentioned Sources sections)
2. A structured data array (JSON/JS) with entries having: id, name, year, category, connections

## Output

A single `.html` file with a D3.js v7 interactive 2D timeline graph.

## Technical Specification

### Layout Algorithm (Adaptive Grid)
- X axis = year (chronological)
- Y axis = categories (grouped horizontal bands)
- Node safe zone: `safeW=130px` (horizontal: circle + label), `safeH=32px` (vertical)
- **Adaptive expansion**: First expand column width (more horizontal space per year), then expand band height if needed
  - `maxRowsPerCol = 4`: prefer max 4 rows before adding columns
  - Column width = `ceil(maxNodesInAnyCat / maxRowsPerCol) * safeW`
  - Band height = `max(maxRowsActual * safeH + 20, 100)`
- Post-layout collision check: 3 passes of pairwise push-apart, clamped to band bounds

### Node Sizing
- Base radius: 10px
- Subtle variation by refCount: `radius = 10 + Math.sqrt(refCount) * 2` (max ~16px)
- **Never extreme differences** — just slightly bigger for more-cited nodes

### Connection Curves
- Simple Bezier arcs: `M sx,sy C cp1x,sy cp2x,ty tx,ty`
- Direction-aware: `cp` offsets flip based on whether target is left or right of source
- Default opacity: **0.15** (visible but not distracting)
- Highlighted opacity: **0.8**, stroke-width: **3px**
- Dimmed opacity: **0.02**

### Lineage Highlighting (TIME-BASED)
```javascript
// Direction determined by YEAR, not edge direction
traceAllPredecessors(id):
  for each connected neighbor where neighbor.year < this.year:
    highlight neighbor
    recurse traceAllPredecessors(neighbor)

traceAllSuccessors(id):
  for each connected neighbor where neighbor.year > this.year:
    highlight neighbor  
    recurse traceAllSuccessors(neighbor)
```
- "Connected" = either direction of edge (source→target or target→source)
- Same-year nodes are NOT recursively expanded (prevents loops)
- This gives clean ancestor/descendant chains without "cousin" contamination

### Interaction Model
| Action | Effect |
|--------|--------|
| Hover node | Temporary lineage highlight + tooltip (name, year, category, connection counts) |
| Single click node | Lock lineage highlight (persists during pan/zoom) |
| Click same node again | Clear highlight |
| Double-click node | Lock highlight + open sidebar with full connection details |
| Drag/pan | Preserves highlight (mousedown/mouseup distance check, threshold 5px) |
| Click empty space | Clear highlight (only if no drag) |
| Escape | Clear highlight |
| Close sidebar | Preserves highlight |

### Visual Style (Dark Theme)
```css
body { background: #0d1117; color: #eee; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; }
/* Year column backgrounds: alternating dark tones */
/* Category band backgrounds: very subtle category color at 0.04 opacity */
/* Nodes: colored circles with white 2px stroke */
/* Tooltip: rgba(13,17,23,0.98) with #30363d border */
/* Sidebar: #161b22 background, #58a6ff accent, #30363d borders */
```

### D3.js Zoom + Background Click
```javascript
const zoom = d3.zoom()
    .scaleExtent([0.2, 3])
    .on("zoom", (event) => g.attr("transform", event.transform));
svg.call(zoom);
svg.on("dblclick.zoom", null); // Disable zoom's dblclick (we use it for sidebar)

// IMPORTANT: Use native addEventListener, NOT d3's svg.on() — d3.zoom overwrites svg.on handlers
let _downPos = null;
svg.node().addEventListener("pointerdown", (event) => {
    _downPos = { x: event.clientX, y: event.clientY };
});
svg.node().addEventListener("pointerup", (event) => {
    if (!_downPos) return;
    const dx = event.clientX - _downPos.x;
    const dy = event.clientY - _downPos.y;
    const dist = Math.sqrt(dx * dx + dy * dy);
    _downPos = null;
    // dist < 5 = click (not drag); .closest('.node') = didn't hit a node
    if (dist < 5 && !event.target.closest('.node')) {
        clearHighlight();
    }
});
```

### Adaptive Layout Algorithm
```javascript
// Node safe zone
const safeW = 130;  // horizontal: circle + truncated label
const safeH = 32;   // vertical: circle + text + padding
const maxRowsPerCol = 4;  // prefer max 4 rows before adding horizontal columns

// Per year-category cell: count density
// Column width = ceil(maxNodesInAnyCat / maxRowsPerCol) * safeW + padding
// Band height = max(actualMaxRows * safeH + 20, 100)
// → First expand WIDTH (more columns), then HEIGHT (taller bands) if needed

// Post-layout collision safety net: 3 pairwise push-apart passes
// Clamp all nodes within their category band bounds
```

### File Structure
```html
<!DOCTYPE html>
<html><head>
  <script src="https://d3js.org/d3.v7.min.js"></script>
  <style>/* dark theme CSS */</style>
</head><body>
  <div id="header"><!-- title + subtitle --></div>
  <div id="graph"><!-- SVG renders here --></div>
  <div id="legend"><!-- category color legend --></div>
  <div class="tooltip"><!-- hover tooltip --></div>
  <div id="sidebar"><!-- detail panel on dblclick --></div>
  <script>
    // 1. DATA: const nodes = [...]; const links = [...];
    // 2. LAYOUT: adaptive grid with collision avoidance
    // 3. RENDER: year backgrounds, category bands, links, nodes
    // 4. INTERACTION: hover, click, dblclick, zoom
  </script>
</body></html>
```

## Checklist Before Delivery
- [ ] No node+label overlaps (verify visually at 1:1 zoom)
- [ ] Lines are simple gentle arcs, no wild bends
- [ ] Click-to-highlight shows clean ancestor/descendant chains (time-based)
- [ ] Drag to pan does NOT clear highlight
- [ ] Double-click opens sidebar, closing sidebar keeps highlight
- [ ] Escape clears everything
- [ ] Zoom in/out works smoothly
- [ ] All nodes visible and labeled (truncated with "..." if needed)
