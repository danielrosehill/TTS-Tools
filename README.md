# TTS-Tools

A harness for text-to-speech and voice work. The first tool is **`listen-page`**: a local web page
for listening to clips side by side and choosing between them, with the choices saved as JSON for
the agent to act on. Use it instead of playlists or stitched comparison files when you need to
hear the original, then one version, then another.

| Template | Layout | For |
|---|---|---|
| `voice-pick` | grid | Many voice candidates: rate each (Shortlist / Maybe / No), star a winner |
| `ab-original` | compare | Original vs processed versions of the same clips |
| `eq-mix` | compare | One set of clips under several EQ chains or mixes, with a per-chain tally |
| `checkpoint-compare` | compare | Model systems or checkpoints reading the same lines |
| `term-check` | compare | Per-word Right / Close / Wrong under every clip, for code-switched terms or names |
| `pairwise` | pairwise | Blind A/B pairs against a target, with how-much and why; Bradley-Terry ranking and play-off rounds |
| `blind-ab` | compare | Columns shuffled per row, labels hidden until "Reveal labels" |

```bash
bin/listen-page scan runs/voice-design --template voice-pick --question "Most neutral narrator?"
bin/listen-page scan runs/polish --template eq-mix --ref raw   # one subfolder per chain, files matched by stem
bin/listen-page serve runs/polish        # http://127.0.0.1:<port>/index.html, autosaves choices.json
bin/listen-page wait runs/polish         # returns when the listener presses Submit
bin/listen-page result runs/polish --json
```

- Stdlib Python 3, no dependencies; the page is one self-contained HTML file next to the clips.
- Highlighting: mark the words under test (code-switched terms, names) in each line's text.
- One clip plays at a time; loop toggle; per-clip and per-row notes; overall comment.
- The server binds 127.0.0.1 and rejects cross-origin writes. Opened from disk, the page keeps
  choices in the browser and shows JSON to copy back.

## Skills

- `listening-page`: when and how an agent builds, serves and reads a listening page.
- `voice-seeds`: keep a git library of reference voice clips (designed voices, anchors) with full provenance; add, denoise, review. Library path in `~/.claude-user-data/tts-tools/config.json`.

## Installation

```bash
claude plugins install tts-tools@danielrosehill
```

## Licence

MIT.
