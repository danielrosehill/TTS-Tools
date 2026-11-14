---
name: listening-page
description: Build a local HTML page to listen to TTS/audio clips side by side and pick: voices, original vs processed, EQ mixes, checkpoints, blind A/B.
---

# Listening page

Whenever the user has to **listen** to more than two clips and choose (voice candidates, raw vs
restored, EQ chains, model checkpoints), give them a listening page, not a playlist or a stitched
file. Every clip gets its own player, the original sits next to its versions, starting one clip
stops the others, and the user's picks autosave to `choices.json` for you to read back.

CLI: `${PLUGIN_ROOT}/bin/listen-page` (stdlib Python 3). If `listen-page` is not on PATH, run it by
that path or `ln -sf ${PLUGIN_ROOT}/bin/listen-page ~/.local/bin/`.

## Pick the template

`listen-page templates` lists them:

| Template | Layout | Use for |
|---|---|---|
| `voice-pick` | grid | Many candidates of one line (voice design seeds, speakers): rate each, star a winner |
| `ab-original` | compare | Original vs processed versions of the same clips; the original is the pinned reference |
| `eq-mix` | compare | The same clips under several EQ chains or mixes; the tally names the chain to keep |
| `checkpoint-compare` | compare | Systems or checkpoints reading the same lines |
| `term-check` | compare | Per-word pronunciation: Right / Close / Wrong for each highlighted word under every clip (code-switching, names) |
| `blind-ab` | compare | As above, columns shuffled per row and labelled "Version n" until the user reveals |

**Grid** = flat candidates, grouped by filename prefix (`A0..A4`, `B0..` -> groups A, B).
**Compare** = rows are clips/lines, columns are versions; one subfolder per version, files matched by stem.

## Steps

1. Lay out the clips. Grid: all clips in one folder. Compare: `DIR/<version>/<item>.<ext>`, e.g.
   `raw/`, `sidon/`, `warm-eq/`, each with `l01.mp3, l02.mp3...`. Prefer mp3 (small, plays everywhere).
   Level-match versions (e.g. `loudnorm=I=-19`) so louder doesn't win.
2. Generate the spec and page:
   ```bash
   listen-page scan DIR --template <name> [--ref raw] --title "..." --question "..."
   ```
   This writes `DIR/listen.json` and `DIR/index.html`.
3. Enrich `listen.json` when it helps the listener, then `listen-page build DIR/listen.json`:
   - group `note` (e.g. the voice-design attributes behind set A),
   - row `text` (the sentence being read),
   - column `label` (`"warm: lowshelf +2, de-ess"` beats `warm`),
   - `highlight` on rows/clips: the words the listener should judge (code-switched terms, names).
4. Serve it in the background and open the browser:
   ```bash
   listen-page serve DIR        # prints the 127.0.0.1 URL; run in background
   ```
   Tell the user in one line what's on the page and to press **Submit**.
5. Wait: `listen-page wait DIR` (blocks until Submit; run in the background), or read when the
   user says they're done.
6. Read: `listen-page result DIR` (or `--json`). Fields: `winner`, `ratings{clip: choice}`,
   `picks{row: column}`, `terms{"row|column": {word: verdict}}` and `term_notes` (same keys, free text per word; term-check), `tally{column: n}`, `notes{id: text}`, `comment`, `status`. Notes are
   instructions. Record the verdict where the project keeps listening results, then stop the server
   (`pkill -f "listen-page serve DIR"`).

## Spec format (`listen.json`)

```json
{"template": "voice-pick", "title": "...", "question": "...", "intro": "...",
 "groups": [{"name": "Set A", "note": "male, young adult", "clips": [{"id": "A0", "label": "A0", "src": "A0.mp3", "text": "optional"}]}]}
{"template": "ab-original", "reference": "raw",
 "columns": [{"id": "raw", "label": "Original"}, {"id": "sidon", "label": "Sidon"}],
 "rows": [{"id": "l01", "label": "l01", "text": "the sentence", "clips": {"raw": "raw/l01.mp3", "sidon": "sidon/l01.mp3"}}]}
```

For term-check, a clip or row may also carry `syllables` ({word: [[syl, ...] per space-separated word]}) and
`asked` ({word: [stressed syllable index per word]}). When the listener marks a word Close or Wrong, its
syllables appear as "Heard stress" / "Should be" chips (the asked syllable underlined); the answers land in
`stress{cell: {word: {heard: ["w.s"], should: ["w.s"]}}}`. Use it for stress errors in code-switched words.

Any template key can be overridden in the spec: `layout`, `ratings` (grid buttons), `pick_label`,
`blind`, `seed` (blind shuffle), `ref_pickable` (whether the reference can be preferred),
`highlight` (list of words to mark in the clip/row `text`: top level, per clip or per row; use it for
the words under test, e.g. code-switched terms or names whose pronunciation the listener should judge).

## Data storage

Writes only inside the clip folder: `listen.json`, `index.html`, `choices.json`, `.listen-url`.
Opened as `file://` (not served), choices stay in the browser and the page shows JSON to copy back.

## Notes

- The server binds 127.0.0.1 only and rejects cross-origin POSTs. Nothing leaves the machine.
- Paths in the spec are relative to the page, so keep `index.html` in the clip folder.
- For a page to share remotely, upload the folder (page + clips) to static hosting as is; choices
  then fall back to copy-paste JSON.
