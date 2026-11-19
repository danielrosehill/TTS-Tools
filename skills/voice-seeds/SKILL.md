---
name: voice-seeds
description: Keep and browse a library of reference voice clips ("voice seeds"), meaning designed or generated voices kept as clone anchors for future TTS work. Use on "add these to voice seeds", "save this voice for later", "review the voice library", "do we have a voice for X".
---

# Voice seeds

A voice seed is a clip of a voice worth keeping: a voice-design preview, a chosen anchor, a generated
character voice. It is kept with enough provenance to regenerate or reuse it: provider, prompt, seed,
voice ID, the text read, processing and terms. The library is a git repo the user owns.

## Config

Read `~/.claude-user-data/tts-tools/config.json`:

```json
{"voice_seeds_repo": "~/path/to/Voice-Seeds", "denoise_rnnoise_model": "~/path/to/cb.rnnn"}
```

If it is missing, ask where the library should live (suggest a private git repo), create it with the
layout below, and write the config.

## Layout

```
index.json               {"updated": "YYYY-MM-DD", "seeds": [{slug, created, status, persona, provider, prompt_id, seed, generated_voice_id, path}]}
seeds/<slug>/seed.json   full provenance (fields below)
seeds/<slug>/raw.<ext>   exactly as generated, never edited
seeds/<slug>/clean.flac  optional processed copy; the chain goes in seed.json "files"
```

`seed.json` fields: `slug`, `created` (absolute date), `status` (`candidate` | `anchor` | `retired`),
`persona` (one line: who the voice is for), `provider`, `endpoint` or model ID, `prompt`, `seed`,
`generated_voice_id` (with a note on whether it persists), `text`, `seconds`, `files` (one line per file
saying what it is), `used_by` (repos), `terms` (provider licence and usage terms), `notes`.

## Add seeds

1. Collect the clips plus their generation log: prompt, seed, voice IDs. Never invent a field. Write
   `null` and say so instead.
2. Slug: `<persona-short>-<variant>`, lowercase, hyphens, unique in `index.json`.
3. Copy the original to `raw.<ext>`. If it is noisy, measure first: the noise floor in the gaps and the
   bandwidth (a spectrogram plus per-band noise vs speech). Then make `clean.flac`: a light chain is
   `arnndn=m=<rnnoise model>:mix=1,afftdn=nr=20:nf=-60:tn=1,highpass=f=70,loudnorm=I=-19:TP=-1.5` at
   48 kHz. Record the before and after gap level.
4. Write `seed.json`, add the entry to `index.json`, update the README's seed list, then commit and push.
5. When a project picks one, set `status: anchor` and add the project to `used_by`.

## Review the library

- Answer from `index.json`: group by persona and status, and show provider, prompt and date.
- To listen, build a `voice-pick` listening page over the chosen seeds' `clean.flac`, or `raw.*` if there
  is no clean copy (see the `listening-page` skill), with each seed's `persona` and `prompt` as the group note.
- Flag gaps: seeds without `terms`, voice IDs not known to persist, and anchors missing `used_by`.
