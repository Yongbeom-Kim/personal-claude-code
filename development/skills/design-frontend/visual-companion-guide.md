# Visual Companion Guide

Reference for the browser-based visual companion used during frontend design.

## How It Works

The server watches a directory for HTML files and serves the newest one. You write HTML content fragments to `screen_dir`, the user sees them in their browser and can click to select options. Selections are recorded to `state_dir/events`.

**Content fragments vs full documents:** If your HTML starts with `<!DOCTYPE` or `<html`, the server serves it as-is (injects the helper script). Otherwise, the server wraps your content in the frame template — adding header, CSS theme, selection indicator, and interactivity. **Write content fragments by default.**

## Starting a Session

```bash
skills/design-frontend/scripts/start-server.sh --project-dir /path/to/project
```

Returns JSON:
```json
{"type":"server-started","port":52341,"url":"http://localhost:52341",
 "screen_dir":"/path/to/project/.visual-companion/12345-1706000000/content",
 "state_dir":"/path/to/project/.visual-companion/12345-1706000000/state"}
```

Save `screen_dir` and `state_dir`. Tell the user to open the URL.

**Platform notes:**
- **Linux/macOS:** Default mode works — the script backgrounds the server.
- **Windows:** Use `run_in_background: true` on the Bash tool call.
- **Remote/containerized:** Use `--host 0.0.0.0 --url-host localhost` if the URL is unreachable.

## The Loop

1. **Write HTML** to a new file in `screen_dir`:
   - Before each write, check `$STATE_DIR/server-info` exists (server auto-exits after 30min idle)
   - Use semantic filenames: `layout.html`, `nav-options.html`
   - Never reuse filenames — each screen gets a fresh file
   - Use the Write tool, never cat/heredoc

2. **Tell user what's on screen and end your turn:**
   - Remind them of the URL
   - Brief text summary of what's showing
   - Ask them to respond in the terminal

3. **On your next turn** — read `$STATE_DIR/events` if it exists for browser interactions (JSON lines). Merge with terminal text for full picture.

4. **Iterate or advance** — if feedback changes current screen, write a new file (e.g., `layout-v2.html`). Only advance when current step is validated.

5. **Clear stale content** — when returning to terminal-only questions, push a waiting screen:
   ```html
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

## Writing Content Fragments

Write just the content. The server wraps it in the frame template automatically.

```html
<h2>Which layout works better?</h2>
<p class="subtitle">Consider readability and visual hierarchy</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single Column</h3>
      <p>Clean, focused reading experience</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Two Column</h3>
      <p>Sidebar navigation with main content</p>
    </div>
  </div>
</div>
```

## CSS Classes Available

### Options (A/B/C choices)
```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content"><h3>Title</h3><p>Description</p></div>
  </div>
</div>
```

**Multi-select:** Add `data-multiselect` to the container.

### Cards (visual designs)
```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup --></div>
    <div class="card-body"><h3>Name</h3><p>Description</p></div>
  </div>
</div>
```

### Mockup container
```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard</div>
  <div class="mockup-body"><!-- content --></div>
</div>
```

### Split view (side-by-side)
```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### Pros/Cons
```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>Benefit</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>Drawback</li></ul></div>
</div>
```

### Mock elements (wireframe building blocks)
```html
<div class="mock-nav">Logo | Home | About</div>
<div class="mock-sidebar">Navigation</div>
<div class="mock-content">Main content</div>
<button class="mock-button">Action</button>
<input class="mock-input" placeholder="Input">
<div class="placeholder">Placeholder area</div>
```

### Typography
- `h2` — page title
- `h3` — section heading
- `.subtitle` — secondary text
- `.section` — content block with margin
- `.label` — small uppercase label

## Browser Events Format

```jsonl
{"type":"click","choice":"a","text":"Option A","timestamp":1706000101}
{"type":"click","choice":"b","text":"Option B","timestamp":1706000108}
```

Last `choice` event is typically the final selection. If `$STATE_DIR/events` doesn't exist, the user didn't interact via browser.

## File Reference

- `scripts/frame-template.html` — CSS reference and frame structure
- `scripts/helper.js` — client-side WebSocket and selection tracking
- `scripts/server.cjs` — zero-dependency Node.js server
- `scripts/start-server.sh` — launcher script
