---
name: lookdev
description: Human-in-the-loop web studio to tune AI-generated output by eye. Stand
  up a local interactive studio (sliders, pickers, drag handles) or an inline edit/highlight/comment
  annotation studio for prose & media, instead of guessing values or shipping a static
  comparison grid.
license: MIT
metadata:
  risk: safe
  source: community
  source_type: community
  source_repo: connerkward/lookdev-studio-skill
  date_added: '2026-06-16'
  author: Conner K Ward
  tags: '[''lookdev'', ''design'', ''ui'', ''tuning'', ''studio'', ''visual-eval'',
    ''annotation'']'
  tools: '[''claude-code'', ''antigravity'', ''cursor'', ''gemini-cli'', ''codex-cli'']'
---
## When to Use

Use when the user says "lookdev", or asks to tune / dial in / iterate on the look of something, compare variations by feel, or review / edit / annotate a blog post, doc, copy, or media set. Use whenever "show me, I'll pick" beats asking the user to specify a number, and whenever you'd otherwise hand back a static grid or a wall of prose for review.

_Source: [connerkward/lookdev-studio-skill](https://github.com/connerkward/lookdev-studio-skill) (MIT)._

# Lookdev

When the user says **"lookdev"** — or any of: *tune*, *dial in*, *iterate on the look of*, *compare variations of*, *let me adjust*, *let me edit/annotate/mark up*, *review this post/doc/copy* — they mean **build an interactive in-browser tool the user directly manipulates**. Not a static grid of N variations. Not a Q&A where they specify numbers. Not a wall of prose they're asked to read and reply to in chat. A real-time studio where they act on the artifact and the change is captured.

**Two studio shapes — pick by what's being tuned:**

- **Visual-parameter lookdev** — the artifact's *look* is set by numbers/choices (color, type, layout, image treatment, animation, 3D). Controls = sliders, pickers, drag handles. This is the bulk of this skill (below).
- **Text & media lookdev** — the artifact is a *document, blog post, copy, or media set* and the user is editing/curating it: rewriting sentences, cutting boring paragraphs, highlighting, leaving margin comments, flagging "diagram goes here" / "wrong image, replace." Controls = **direct inline editing + selection highlight + anchored comments + media annotation**. See the dedicated section below. **A blog post / doc / script review IS this mode — never hand back a long markdown file and ask the user to react in chat. Stand up the annotation studio.**

## What it covers

Any visual decision the user picks by feel, not by spec. Expand this list as needed:

- **Image processing** — dither, halftone, posterize, ASCII, blur, edge, quantize, mosaic, color-grade
- **Color** — palette extraction (show coverage %), per-band pickers, saturation / contrast / gamma curves, harmony presets, theme tokens
- **Typography** — font selector, size / weight / leading / tracking / measure, live sample text, fallback stack
- **Layout, positioning, framing, spacing** — draggable & selectable elements; resize handles; margin / padding rulers; alignment guides; snap-to-grid; aspect-lock toggles
- **Crop & framing** — draggable crop rectangle with aspect lock; live cropped preview at production size
- **Animation / transitions** — easing curve editor, duration sliders, scrubber, replay
- **Component variants** — render hover / focus / disabled / loading / dark side by side on one page
- **Iconography** — stroke weight, corner radius, glyph on canvas
- **AI-generated content** — prompt input + param sliders + side-by-side regeneration grid
- **Anything else where "show me, I'll pick"** beats "ask me to specify a number"

## Controls must stay reachable while inspecting

If the studio shows a list, grid, or scroll-long set of variations, **controls must be visible from every scroll position**. The user has to be able to drag a slider while looking at row 14, not scroll back to the top each time.

Two approaches, pick by layout:

- **Sticky bar** (`position: sticky; top: 0`) at the top of the scroll container. Keep the bar visually distinct — paper background + blur backdrop + bottom border — so it doesn't muddy the specimens scrolling behind it. Sticky pins relative to the *nearest scrolling ancestor with a defined boundary*; if you nest it inside a sized parent (a `<header>` with `margin-bottom`, a `<div>` with a fixed height), it stops sticking at that parent's bottom edge. Lift it to be a direct child of `<body>` (or the page-wrap) so stickiness spans the whole page.
- **Floating overlay** (`position: fixed`) for hotkey-toggled controls — e.g. press `d` to reveal. The portfolio's `.debug-ctl` pattern is this: pinned top-left, transparent until summoned. Use when the controls shouldn't occupy permanent screen real estate (final viewers shouldn't see them; the author can summon on demand).

Anti-pattern: a top-of-page control panel that the user scrolls past and never sees again. They will tune blindly, give up, or guess. Either keep the controls in view *or* duplicate a compact control bar next to each variation row.

## Text & media lookdev — direct edit, highlight, comment, annotate

When the artifact is a **blog post, doc, copy deck, script, or media set**, the user is not turning knobs — they're *marking up the work the way an editor marks a manuscript*. The studio renders the **real artifact WYSIWYG** (the actual rendered blog with its real components/media, not a raw-markdown textarea) and lets the user act on it directly. Building this for a doc review is mandatory: **do not paste a long file into chat and ask "what do you think?" — that's the boring wall of text the user is rejecting.** Stand up the annotation studio and let them edit in place.

### The four affordances (build all that apply)

1. **Direct inline editing.** Every text block is editable in place — click a paragraph/heading and type. Use `contentEditable` per block (or click-to-swap-to-`<textarea>`), each block carrying a stable `data-block-id` that maps back to a source location (markdown/MDX line range, JSX node, or content key). Capture the *edited* text per block; the agent applies the diff to source. Don't make them retype in a separate field — they edit the rendered sentence.
2. **Selection highlight.** Select text → toolbar (or hotkey) applies a colored highlight (`<mark>`). Multiple colors = a legend the user defines (e.g. yellow "cut this", green "love it", red "wrong/fact-check"). Each highlight stores `{blockId, startOffset, endOffset, color, optional note}`.
3. **Anchored comments / margin notes.** Select text or click a media region → attach a comment shown in a **margin rail** (pin in the gutter, expand on hover/click) or as a numbered superscript. Comment = `{anchor, text}` where anchor is a block+range or a media region. This is how the user says "diagram goes here", "too long, cut to two sentences", "needs a real screenshot".
4. **Media annotation.** For images/figures: draw a box / drop a pin / arrow on the image and attach a note (`{mediaId, x, y, w, h, note}`); plus a per-media **flag menu** — "replace", "wrong model", "regenerate", "missing — generate one here". Placeholders ("DIAGRAM HERE", "MEDIA?") render as visible drop-zones the user clicks to specify what they want, directly addressing "where are the diagrams / where is the media."

### Round-trip is MANDATORY (same rule as the settings JSON)

The studio is worthless if the agent can't read the markup back out. Every edit, highlight, comment, and media-flag must export as **one machine-readable patch** with a single **Copy** button (and persist to `localStorage`/URL so a refresh doesn't lose work — this is human-labeled data; see `human-labeled-data-rule`). Shape:

```json
{
  "edits":      [{ "blockId": "p-12", "text": "new rewritten text" }],
  "highlights": [{ "blockId": "p-3", "range": [40, 88], "color": "cut", "note": "boring, drop" }],
  "comments":   [{ "anchor": "p-7", "text": "diagram goes here — flow of the save loop" }],
  "media":      [{ "mediaId": "fig-2", "flag": "replace", "note": "use a real screenshot, not ASCII" }]
}
```

The agent ingests this and bakes: applies the inline edits to the source file, acts on every comment/flag, swaps/generates the flagged media, resolves the highlights (cut the "cut" spans, etc.). Then re-serve the updated artifact for another pass. **No markup may exist that isn't in the export blob** — otherwise you're back to the user narrating changes by hand.

### Mechanics

- **Render the real thing.** MDX/React blog → mount the actual components; static page → render the real HTML/CSS. WYSIWYG per Architecture #5. An annotation layer over a fake-looking preview lies about the result.
- **Selection → offsets.** Use the `Selection`/`Range` API; store character offsets relative to the block's text content (not DOM node paths, which break on re-render). Re-apply highlights/comments on load by walking each block's text to the stored offsets.
- **Editing toolbar floats with the selection** (a small popover at the selection rect) or a sticky top bar — controls stay reachable (see section above). Hotkeys: highlight on a key (e.g. `h`), comment on `c`.
- **Keep edit/annotate modes distinct** so a stray click doesn't garble text while they meant to highlight — a mode toggle (Edit · Highlight · Comment) or modifier key.
- Everything else — serve locally on a free port, verify headless, tear down after baking — is identical to the visual-parameter workflow below.

## Control patterns

Pick controls by what the decision actually is.

| Decision type | Control |
|---|---|
| Continuous value (intensity, size, opacity, k) | `<input type=range>` **paired with an editable `<input type=number>`** (not a static label) — drag OR click-and-type; they two-way sync |
| Discrete choice (mode, blend, easing kind) | segmented buttons or radio chips |
| Color | `<input type=color>` swatches; pre-extract dominant palette with coverage % when relevant |
| Position / size on a canvas | **drag the element itself** — handles, not numeric inputs |
| Crop region | draggable rectangle + aspect-lock toggle |
| Multiple discrete states | render each in a labeled card on one page |
| Font choice | searchable picker + editable sample-text input |

**Spatial rule:** if the user could point at the thing and drag it, that *is* the control. Don't add an `x:` slider when a drag handle is the obvious affordance.

**Gesture capture — never make the gesturing hand leave the gesture.** When a control toggles a *live mouse action* the user is performing — recording a cursor path, scrubbing, freehand-drawing, demonstrating a motion — the start/stop must **not** be a button they have to click. Clicking it drags the mouse off the path, pollutes the start/end of the very motion being captured, and forces a round-trip back to where they were. Bind start/stop to the **keyboard (spacebar by default)** — `keydown` on `Space`, `e.preventDefault()` to kill page scroll, toggle the same handler the button would. Keep the button too (discoverability), but the hotkey is the real control. Generalize: any modal capture where one hand is committed to the primary input gets the *other* modality for mode-switching — gesture→key, and conversely a keyboard-heavy capture gets a foot/mouse toggle. The test: if triggering the control would move the thing you're capturing, it's the wrong modality.

## Coherent control ranges — bounds must propagate

When one control sets a **bound** on another (a min, a max, a threshold, an allowed set), the bounded control's UI must reflect the new bound the instant you change it. A "scrub" slider whose `min`/`max` attributes drift out of sync with its declared bounds is the most common silent bug — the user moves the bounding slider, nothing visible changes downstream, they assume both are broken.

Rules:

- **Single source of truth.** Hold the bound in state once. Every input that *displays* it (its own slider, the dependent control's `min`/`max`, anything else) reads from that state on every update.
- **Re-render `min`/`max` on every state change.** Don't rely on browser-cached attribute values; rewrite them via JS each render. `dependent.min = state.lo; dependent.max = state.hi`.
- **Clamp the dependent value into the new range immediately.** If the user shrinks the upper bound below the current dependent value, the dependent must snap into range, NOT silently stay outside while the slider shows it pinned to the rail.
- **No-op regions are slider bugs.** If dragging a slider past some value has zero downstream effect (because some other control's bound caps it), that's a coherence bug — either narrow this slider's range to where it actually does something, OR change behavior so it does. Sliders with dead zones train the user to think the studio is broken.
- **Test by visualisation, not numeric snapshots.** Take a screenshot, change the bounding slider, take another. The two must look meaningfully different — or the slider is decorative. A numeric `snapshot()` showing state changed doesn't prove the pixels did.

Pattern: every time `applyState()` runs (or its split equivalents), call a `syncBounds()` helper that walks the dependent-input registry and pushes the live bounds into every `min`/`max`/`disabled` attribute. Clamp values into the new bounds in the same pass.

### Paired controls must not cross

A common shape is **two sliders that together define an interval** — `min ⟷ max`, `near ⟷ far`, `tightEnd ⟷ wideEnd`, `start ⟷ end`. If the user can drag one past the other, the interval inverts or collapses. Downstream math typically does `(x - lo) / (hi - lo)` which **divides by zero or returns negative `t`** — producing `NaN` coordinates, collapsed views, or inverted lerps. The user sees the studio "break" but no error fires.

Both ends of the defense:

- **UI invariant.** Keep the two sliders from crossing. On every `syncBounds()` pass: `lower.max = upper.value - MIN_SPAN` and `upper.min = lower.value + MIN_SPAN` (small epsilon, e.g. 2 units, so they can't even touch). The user can't physically drag past the other anchor.
- **Math invariant.** The consuming code (lerp, normalisation, ratio) must guard `denominator > 0` and pick a sane fallback for the degenerate case (e.g. clamp `t = 1` or `t = 0`). UI can race the math — always assume the math could be hit with crossed bounds anyway (URL hash, JSON paste-back, programmatic state mutation).
- **Test the boundary explicitly.** When the lookdev exposes both ends of an interval, write a quick check: drag `tightEnd` to the same value as `wideEnd`, verify the scene doesn't break. Drag `tightEnd` past `wideEnd`, verify same. If you can crash the studio with two slider drags, that's a release blocker.

## Architecture

1. **Single-page HTML** — `<canvas>` and/or DOM, vanilla JS, a sidebar of controls. No build step, no framework, no deps unless one is genuinely required. Lives in a project-local scratch dir (e.g. `scripts/.lookdev-<name>/` or `scripts/.preview-<name>/`), **gitignored**.
2. **Live re-render** on every `input` event. Debounce heavy work via `requestAnimationFrame`. Keep the loop tight enough to feel like a real slider, not a survey.
   - **Every numeric control is dual-input (MANDATORY): a range slider AND an editable `<input type=number>`, two-way synced.** The drag is for exploring; the typed number is for hitting an exact value (and reading the current one). A static `<span>` readout is not enough — the user must be able to click it and type. Sync rule: on slider `input`, write the number field; on number `input`/`change`, update state and re-render — but **do not overwrite a field while it has focus** (guard with `document.activeElement`), or typing gets clobbered mid-keystroke. Clamp to [min,max] on commit (`change`), not on every keystroke, so intermediate values like "1" before "12" aren't snapped.
   - **Always include a Reset control** that restores every control to its defaults in one click (keep a `DEFAULTS` object; `Object.assign(state, DEFAULTS)` then re-render). Cheap to add, and essential once the user has wandered far from baseline.
   - **Always build undo/redo history (MANDATORY).** Dialing-in is iterative and lossy — the user *will* overshoot a good look and need to step back. Bind **Ctrl/Cmd-Z** (undo) and **Ctrl/Cmd-Shift-Z** / **Ctrl-Y** (redo), and surface visible **↶ Undo / ↷ Redo** buttons. Snapshot the *full* serialized state — every control **plus any drawn/spatial state** (polygons, crop rects, dragged handles, palettes), i.e. the same blob as the settings round-trip (#3), not just slider scalars. Debounce so a continuous drag collapses into **one** history step (snapshot ~350 ms after the last `input`, not per event), keep a bounded stack (~100–120 entries), and on a new edit after undo, truncate the redo branch. Restore by re-applying a snapshot through the same `applyState` path the loader uses (so it can't drift). Guard the key handler when focus is in an `<input>`/`<textarea>` so native text-undo still works. A lookdev without undo punishes exploration — the whole point of the tool.
3. **Structured settings round-trip (MANDATORY).** Every lookdev MUST expose its full current state as machine-readable, copy-pasteable text — a settings JSON (or equivalent) covering *every* control, with a one-click **Copy** button and a visible live readout. This is non-negotiable: the agent cannot bake by eyeballing a screenshot, and the user shouldn't have to describe what they dialed in. The round-trip is: user drags → studio serializes the exact state → user pastes the blob back (or it persists to URL/localStorage) → agent bakes from those literal values with identical math. No control may be tweakable without appearing in the export blob. Mirror the state into the URL query so a look is shareable by link, too.
4. **Reproducible export.** Beyond the settings blob, pick by what gets committed:
   - **Copy settings JSON** — user pastes back, agent bakes with identical math (port the renderer to Python / build script / etc. and verify the bake matches).
   - **Download asset** — page renders the final artifact at full resolution and triggers a download (PNG / SVG / WebP / JSON).
   - **Make exported artifacts re-loadable — sidecar + embedded metadata.** When the download is a *non-JSON* artifact (STL, PNG, GLB, SVG, WebP, video…), the look that produced it shouldn't be strandable. Do BOTH, where the format allows:
     - **Sidecar:** download a **zip** containing the artifact *and* its `settings.json`, so the exact state ships next to the result.
     - **Embed the settings inside the file itself**, so the bare artifact alo

<!-- Truncated for OpenGAP token limits -->
