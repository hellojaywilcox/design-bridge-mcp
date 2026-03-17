# Design Bridge MCP — Developer Notes

**Plugin:** Design Bridge MCP ([Marketplace](https://framer.com/marketplace/plugins/design-bridge-mcp))
**Publisher:** Jay Wilcox at Orange Lamp Studio LLC
**Stack:** Framer Plugin API (`framer-plugin`) + Cloudflare Worker relay + MCP protocol
**Mode:** `"modes": ["canvas"]` (set in `framer.json`)

---

## What This Plugin Does

Design Bridge MCP connects AI assistants (Claude, Cursor, etc.) to Framer projects via the [Model Context Protocol](https://modelcontextprotocol.io). It exposes 63 tools across 9 categories: Nodes, Selection, Editor, Components, Project, Pages, Design Tokens, Code Files, CMS, and Localization.

The plugin runs in Framer's canvas mode and communicates with a Cloudflare Worker relay over WebSocket. The Worker exposes a standard MCP endpoint that AI clients connect to.

---

## Plugin API: What Works (54 of 63 tools)

These tools function correctly via the Framer Plugin API:

- **Nodes (14):** Create frames, add text, set text content, add SVG, add images from URL, set attributes (position/size/opacity/backgroundColor), get node details, remove, duplicate, get parent/children, get bounding rect, get nodes by type, get text content
- **Selection & Editor (4):** Get/set selection, zoom into view, show notifications
- **Components (2):** List components, add component instances
- **Project (4):** Get project info, publish info, locales, custom code
- **Pages (3):** List pages, create design pages, create web pages
- **Design Tokens (8):** Full CRUD on color styles and text styles (list, get, create, update, delete), list fonts, get font details
- **Code Files (4):** List, get, create, set content
- **CMS (13 of 20):** Collections, items, fields, redirects (read operations + basic writes)
- **Localization (2 of 4):** Get active/default locale

---

## Plugin API: Issues & Limitations

### Issue 1: Text styling properties are silently ignored (CRITICAL)

**Impact:** Users cannot set font size, color, line height, letter spacing, or text alignment on any TextNode. Only font family and weight work.

**What works:**
```ts
// Font family/weight — this works
const font = await framer.getFont("Inter", { weight: 700, style: "normal" });
await framer.setAttributes(nodeId, { font });
```

**What does NOT work (all silently fail — no error, no change):**

```ts
// Approach 1: inlineTextStyle as plain object
await framer.setAttributes(nodeId, {
  inlineTextStyle: { fontSize: "32px", color: "#FF0000" }
});
// Result: ok: true, but no visual change

// Approach 2: Direct attributes on node
await framer.setAttributes(nodeId, { fontSize: "32px", color: "#FF0000" });
// Result: ok: true, but no visual change

// Approach 3: Read inlineTextStyle, then call setAttributes on it
const node = await framer.getNode(nodeId);
console.log(node.inlineTextStyle); // Always null — even on nodes styled in the editor

// Approach 4: setHTML (exists in TypeScript types but blocked at runtime)
await node.setHTML('<span style="color: red">Hello</span>');
// Result: Error — "Invalid method: INTERNAL_setHTMLForNode"
```

**To reproduce:**
1. Open any Framer project with existing styled TextNodes
2. Run a plugin in canvas mode
3. Read any TextNode via `framer.getNode(id)` — `inlineTextStyle` is always `null`
4. Attempt any of the 4 approaches above — none produce a visual change
5. Verify that `framer.getFont()` + `font` attribute DOES work for font family/weight

**Questions for Framer team:**
- What is the correct Plugin API call to set `fontSize`, `color`, `lineHeight`, `letterSpacing`, and `textAlign` on a TextNode?
- Is `inlineTextStyle` readable/writable via the Plugin API, or is it internal-only?
- Is `setHTML`/`getHTML` intentionally blocked, or is it an alpha restriction that will be lifted?

---

### Issue 2: No layout or spacing control via Plugin API

The Plugin API cannot set any of these properties on FrameNodes:

| Property | Attempted | Result |
|----------|-----------|--------|
| `padding` | `setAttributes(id, { padding: "20px" })` | Silently ignored |
| `gap` | `setAttributes(id, { gap: "16px" })` | Silently ignored |
| `layout` | `setAttributes(id, { layout: "stack" })` | Silently ignored |
| `stackDirection` | `setAttributes(id, { stackDirection: "vertical" })` | Silently ignored |
| `stackDistribution` | `setAttributes(id, { stackDistribution: "center" })` | Silently ignored |
| `stackAlignment` | `setAttributes(id, { stackAlignment: "center" })` | Silently ignored |
| `borderRadius` | `setAttributes(id, { borderRadius: "12px" })` | Silently ignored |
| `overflow` | `setAttributes(id, { overflow: "hidden" })` | Silently ignored |
| `border` | `setAttributes(id, { border: "2px solid red" })` | Silently ignored |
| `zIndex` | `setAttributes(id, { zIndex: 10 })` | Silently ignored |
| `aspectRatio` | `setAttributes(id, { aspectRatio: 1.5 })` | Silently ignored |

**To reproduce:**
1. Create a FrameNode via `framer.createFrameNode()`
2. Call `framer.setAttributes(nodeId, { padding: "20px", layout: "stack" })`
3. Re-read the node — properties remain at their defaults

**What DOES work:** `width`, `height`, `left`, `top` (as CSS strings like `"200px"`), `backgroundColor`, `opacity`, `rotation`, `name`, `visible`.

---

### Issue 3: No responsive sizing via Plugin API

When reading node attributes, responsive frames return numeric values (e.g., `width: 1`) that appear to be fraction/fill values. But only fixed pixel values can be set:

```ts
// Reading a responsive frame
const node = await framer.getNode(frameId);
console.log(node.width);  // Returns: 1 (meaning 1fr or fill)
console.log(node.height); // Returns: 1

// Setting responsive sizing — does not work
await framer.setAttributes(frameId, { width: "1fr" });       // Ignored
await framer.setAttributes(frameId, { width: "fit-content" }); // Ignored
await framer.setAttributes(frameId, { width: "100%" });       // Ignored
```

**Question:** Is there a way to set fill, fit-content, or fractional sizing via the Plugin API?

---

### Issue 4: Canvas mode blocks 5 API methods

Our plugin runs in `"modes": ["canvas"]`. The following methods are blocked with: `"Method: <name>, is not allowed while in mode: canvas"`

| Method | Tool |
|--------|------|
| `getManagedCollections()` | `cms_getManagedCollections` |
| `getActiveManagedCollection()` | `cms_getActiveManagedCollection` |
| `getLocalizationGroups()` | `localization_getLocalizationGroups` |
| `setLocalizationData()` | `localization_setLocalizationData` |
| `setFields()` on managed collections | `cms_setFields` (calls `getManagedCollections` internally) |

**To reproduce:**
1. Create a plugin with `"modes": ["canvas"]` in `framer.json`
2. Call `await framer.getManagedCollections()`
3. Error: `"Method: getManagedCollections, is not allowed while in mode: canvas"`

**Questions:**
- Which mode(s) do these methods require?
- Can a plugin support multiple modes simultaneously?
- Is there a reference for which API methods are available in which modes?

---

### Issue 5: CMS enum fields require undocumented `id` parameter

When adding a CMS field with `type: "enum"`, the API returns a typia validation error:

```
Error on typia.createAssert(): invalid type on $input[1][0].id, expect to be string
```

Adding fields with `type: "string"` works fine without an `id`. The TypeScript types don't indicate that `id` is required for enum fields.

**To reproduce:**
1. Get a CMS collection via `framer.getCollections()`
2. Call `collection.addFields([{ type: "enum", name: "Status" }])`
3. Error: typia validation fails on missing `id`

**Question:** What is the required shape of an enum field definition? Does `id` need to be a specific format (e.g., UUID)?

---

### Issue 6: CMS redirects require `expandToAllLocales` boolean

The `addRedirects` method fails with a typia validation error when `expandToAllLocales` is omitted:

```
Error on typia.createAssert(): invalid type on $input[0][0].expandToAllLocales, expect to be boolean
```

The TypeScript types suggest this field is optional, but Framer's runtime validation requires it.

**Note:** Even with this fixed, redirect operations require a paid Framer plan that includes the Redirects feature (`"Current plan does not include Redirects"`).

---

### Issue 7: Background images on FrameNodes

We can add images to the canvas via `framer.addImage()`, which creates a new node. However, we've observed existing FrameNodes in projects that have background images applied (the node is still a FrameNode, not an ImageNode).

**Question:** Is there a Plugin API method to set a background image on an existing FrameNode?

---

## Server API Findings (framer-api npm package)

During development, we discovered the Framer Server API (`framer-api` v0.1.2) and tested it extensively to understand its capabilities relative to the Plugin API.

### Why we explored the Server API

The Plugin API limitations above (text styling, layout, responsive sizing) are critical gaps for an MCP plugin. Users expect AI assistants to be able to create properly-styled, properly-laid-out sections — not just unstyled frame hierarchies. The Server API was explored to determine whether a hybrid approach (Plugin API + Server API) could fill these gaps.

### Server API: What works

**Connection:**
```ts
import { connect } from "framer-api";
const framer = await connect(projectUrl, apiKey);
```

**Text styling** — works via TextStyle class instances:
```ts
const font = await framer.getFont("Inter", { weight: 500 });
const style = await framer.createTextStyle({ font, fontSize: "48px" });
await style.setAttributes({ color: "rgb(255, 0, 0)" });
await node.setAttributes({ inlineTextStyle: style });
// Result: Text is visually red and 48px ✓
```

**Important:** Plain objects do NOT work for `inlineTextStyle`. It must be a TextStyle class instance created via `framer.createTextStyle()`. Passing `{ fontSize: "48px", color: "red" }` as a plain object is silently ignored — identical behavior to the Plugin API.

**Layout** — all properties work:
```ts
await frame.setAttributes({
  layout: "stack",
  stackDirection: "vertical",
  stackDistribution: "center",
  stackAlignment: "center",
  gap: "16px",
  padding: "80px 20px 0px 20px",  // 4-value shorthand works
  overflow: "hidden",
  borderRadius: "12px",
});
// Result: All properties applied correctly ✓
```

**Responsive sizing** — works:
```ts
await frame.setAttributes({ width: "1fr", height: "fit-content" });
await child.setAttributes({ width: "100%" });
// Result: Responsive sizing applied correctly ✓
```

**Border** — works with object or CSS string:
```ts
await frame.setAttributes({ border: { color: "red", width: "2px", style: "solid" } });
// OR
await frame.setAttributes({ border: "2px solid red" });
// Both work — Framer normalizes to { color, width, style } object ✓
```

**Text node creation:**
```ts
// createTextNode takes an attrs object, NOT a text string
const node = await framer.createTextNode({ width: "1fr", height: "fit-content" });
await node.setText("Hello World");  // Set text content separately
```

**Deletion** — works via `.remove()` on instances:
```ts
await textStyle.remove();   // Deletes a text style
await colorStyle.remove();  // Deletes a color style
await node.remove();        // Deletes a node (and its children)
// Note: framer.deleteTextStyle() and framer.removeTextStyle(id) do NOT exist
```

### Server API: What does NOT work

| Feature | Result |
|---------|--------|
| `minWidth` / `maxWidth` / `minHeight` / `maxHeight` | Silently ignored; reads back as `null` |
| `setHTML()` / `getHTML()` | Blocked — same error as Plugin API |
| Reading `inlineTextStyle` on existing nodes | Always `null` (computed styles not readable) |
| Individual `paddingTop`/`paddingRight`/etc. | Can set, but reads back as `undefined`. Use shorthand `padding` instead |
| `borderWidth` / `borderColor` as separate props | Not real properties — use the `border` object |
| Removing a TextStyle after applying as `inlineTextStyle` | Removes the styling from the node — do not call `.remove()` on styles you've applied |

### Server API: Section recreation test

We successfully recreated a complete section (`skills-training-1`) using only the Server API. This included:

- Nested stack layouts (vertical and horizontal) with correct gap and alignment
- 4-value padding shorthand (`80px 20px 0px 20px`)
- Text nodes with correct `fontSize` (14px, 48px, 16px), `color`, `alignment` (center), and `letterSpacing`
- Buttons with `backgroundColor`, `borderRadius`, `padding`, and `border`
- Image placeholder with `overflow: hidden`

The recreated section was visually accurate compared to the original, confirming that the Server API can handle the full range of layout and styling operations needed for section-level design work.

### Server API requirements

| Aspect | Plugin API | Server API |
|--------|-----------|------------|
| Setup | Zero-config (auto-discovery) | Requires API key + project URL per project |
| Text styling | Font family/weight only | Full (color, fontSize, lineHeight, letterSpacing, textAlign) |
| Layout | Not supported | Full (padding, gap, stack, grid, distribution, alignment) |
| Responsive sizing | Not supported | Full (1fr, fit-content, 100%) |
| Border | Not supported | Full (object or CSS string) |
| UI operations | Selection, zoom, notifications | Not available |
| CMS | 13 of 20 tools working | Untested |
| Stability | Production | Beta (free during beta period) |

---

## Architecture: Hybrid Approach (Under Consideration)

Given the complementary strengths of each API, we're considering a hybrid architecture where the plugin facilitates both:

1. **Plugin API** — handles UI operations (selection, zoom, notifications), component instances, CMS, code files, and anything that requires canvas mode access
2. **Server API** — handles text styling, layout, responsive sizing, borders, and section-level creation

The plugin UI would collect the user's project URL and API key (stored locally in the browser, never sent to our servers). The plugin would then route tool calls to the appropriate API based on what each tool needs.

**Key question for Framer team:** Is this hybrid approach supported/allowed? Specifically:
- Can a plugin running in canvas mode also establish a Server API connection using `framer-api`?
- Are there any ToS or technical restrictions on plugins using the Server API internally?
- Would this pass marketplace review?

---

## Summary of Questions

1. **Text styling via Plugin API** — What is the correct way to set `fontSize`, `color`, `lineHeight`, `letterSpacing`, and `textAlign` on TextNodes? Is `inlineTextStyle` writable?
2. **Layout via Plugin API** — Is there a way to set `padding`, `gap`, `layout`, `stackDirection`, etc. on FrameNodes? Or are these Server API-only?
3. **Responsive sizing** — Can `fill`, `fit-content`, or `1fr` be set via the Plugin API?
4. **Canvas mode restrictions** — Which modes do `getManagedCollections`, `getLocalizationGroups`, and `setLocalizationData` require? Is there a mode reference?
5. **Enum fields** — What is the required shape of an enum field definition for `addFields()`?
6. **Background images** — Is there a Plugin API method to set a background image on a FrameNode?
7. **setHTML / getHTML** — Are these intentionally blocked, or alpha-only? Will they be available in the future?
8. **Hybrid approach** — Can a canvas-mode plugin also use the Server API (`framer-api`)? Any restrictions?
9. **min/max dimensions** — Are `minWidth`, `maxWidth`, `minHeight`, `maxHeight` supported in the Server API? They exist in the TypeScript types but writes are silently ignored.
10. **Breakpoints** — Do Plugin API changes cascade across breakpoints (Desktop → Tablet → Phone) the same way manual edits do?

---

*Marketplace: [framer.com/marketplace/plugins/design-bridge-mcp](https://framer.com/marketplace/plugins/design-bridge-mcp)*
