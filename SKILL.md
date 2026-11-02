---
name: video-edit
description: Edit talking-head, tutorial, explainer and Short videos with Claude Code. Word-accurate cuts from raw takes, motion graphics built as HTML + GSAP (optionally three.js) pages locked to the spoken words, sound effects synthesized in Python, honest loudness handling and frame-exact rendering. Use when the user wants to cut raw footage, remove retakes and pauses, add animated cards or chapters, add or design sound effects, fix loudness, or render a final edit.
---

# Video edit

This skill turns raw footage into a finished edit together with a human editor. Read this whole file before touching a project. Then read `reference/audio.md` before any cut or level change, and `reference/style.md` before any graphics.

## Read first: what you can and cannot do

- **You cannot hear.** You never listen to audio. Everything you know about sound comes from numbers: Whisper word timestamps, a 10 ms energy envelope, loudness meters, spectrograms. Say this to the user plainly and never claim that something "sounds good".
- **The human ear is the final check.** Whether a cut clips a word, whether an effect is too loud, whether a voice sounds muffled: the user decides. Your job is to make every problem measurable and every fix fast.
- **The first pass is never the final edit.** Work stage by stage, hand over a preview after each stage, and expect corrections. Most good results come from short review loops, not from one long autonomous run.
- **Sync by construction, not by ear.** Animations and sound effects are triggered from the same timeline clock, and that clock is derived from the measured word times. That is how effects land on the picture without you hearing them.

## Paths

Commands run from the user's project folder. `$SKILL` is the folder this file is in (for example `~/.claude/skills/video-edit`); set it once in the shell: `SKILL=~/.claude/skills/video-edit`.

## Before you start: ask

Ask these once, keep the answers in `work/NOTES.md`:

1. **Video type:** educational / explainer, talk (talking head), tutorial with screen recording, or Short (vertical, under 60 s). This sets the motion level (see the table below).
2. **Raw material:** which clips, in which order, is there a script or outline to follow?
3. **On-screen language(s):** one language, or two (for example native + English)?
4. **Brand:** one accent colour, a font if they have one. If they have none, use the neutral palette in `reference/style.md`.
5. **Sound effects:** yes, no, or "only on chapter transitions". Default is very quiet transitions only.
6. **Deliverables:** resolution (1080p / 1440p / 4K), captions, YouTube chapter list, deadline.

## Video types

| Type | Motion | Sound effects | Cuts |
|---|---|---|---|
| Educational / explainer | Calm and serious. Clean diagrams, readable panels, slow camera moves. three.js only when it explains something (a 3D model of the thing being taught). | None, or very quiet transitions | Natural pace |
| Talk (talking head) | Few cards that stay on screen long, placed in empty areas, never over the face. | Quiet chapter transitions only | Natural or tight |
| Tutorial / screen recording | Zoom into the part of the screen being discussed, highlight boxes, step cards. | Optional soft clicks on highlights | Natural |
| Short | More energetic: faster reveals, bigger type, a hook in the first 2 s, captions on. Still eased, never jittery. | Allowed, still under the voice | Tight |

If the user does not say, infer the type from the transcript and confirm it. Do not over-animate an educational video, and do not make a Short feel slow.

## The workflow

Work in this order. After each numbered stage, stop and hand over a preview URL or file plus a short list of what you decided and why.

### 0. Project layout

```
project/
  src/            raw clips (or symlinks to them)
  work/           everything generated: wav, transcripts, program.json, proxy.mp4, sfx/, NOTES.md, backups
  scene/          the motion-graphics page (copied from this skill's template/)
  out/            renders and final files
```

Keep the project outside cloud-synced folders (iCloud Desktop, Dropbox) if possible. Synced files can be evicted from disk mid-render and reads then hang.

### 1. Transcribe

```bash
python3 $SKILL/scripts/transcribe.py --project . src/*.mov --lang en
```

This splits each clip at pauses, transcribes each short chunk separately (padded with silence), and writes `work/transcripts/<clip>.json` + `.txt` with numbered utterances and word times. Transcribing whole files hides retakes: Whisper merges repeated sentences and drops repeated words. See `reference/audio.md`.

Then **read the .txt files yourself**. Mark retakes, false starts, half sentences and filler. Speakers often repeat a sentence two or three times; keep the last fluent take unless the user says otherwise.

### 2. Rough cut

Write `work/edl.json` (format in `reference/edl.md`): the utterances to keep, in the script's order, with chapter markers. Then:

```bash
python3 $SKILL/scripts/cuts.py --project .          # edl.json -> work/program.json (cut points snapped to real word edges)
python3 $SKILL/scripts/build.py --project . audio video   # -> work/prog.wav + work/proxy.mp4 (1080p preview)
python3 $SKILL/scripts/verify.py --project .        # re-transcribes every join, flags clipped words and leftover repeats
```

Fix what `verify.py` flags, rebuild, and only then hand over. Tell the user the new length and how many cuts were made.

### 3. Review loop

- The user gives timestamps. **Map them by content, not by clock.** Their tab may show an older build. Quote the sentence you think they mean and confirm when unsure. If a time does not match, search the transcripts for the quoted words.
- Before every new version, copy `work/edl.json` and `work/program.json` to `work/backup/vN/`. If the user says "undo everything", restore exactly, do not reinterpret.
- Never replace `work/proxy.mp4` while the user is watching it; write the new one and say when it is ready.
- Change one thing at a time when the user is unhappy. A broken preview makes people distrust the whole revision.

### 4. Chapters

Propose chapters from the transcript (topic changes, the script's sections). Suggest short titles. After approval, add `{"chapter": "..."}` items to the EDL. `cuts.py` prints a YouTube chapter list (`0:00 Intro …`) and writes `work/chapters.txt`. The first chapter must start at 0:00, and YouTube needs at least three chapters, each at least 10 s long.

A chapter transition on screen is a full-frame veil (see `reference/style.md`): the chapter number, the title, and a progress row. Sub-topics inside a chapter do not get their own veil, only a small chip change.

### 5. Graphics, range by range

Copy the template into the project (`cp -R $SKILL/template scene`), serve the project and open the scene. `?demo` shows the template's own demo; without it the scene loads `work/program.json` and plays `work/proxy.mp4`:

```bash
python3 $SKILL/scripts/serve.py --project . --port 5600     # http://127.0.0.1:5600/scene/
```

Work on one chapter or time range at a time, never the whole video at once. For each range:

1. List the beats: the few things worth showing (numbers, steps, a comparison, a key idea, a source). **Do not put the narration on screen.** Cards are not subtitles; they carry what the voice cannot (structure, data, a picture).
2. Lock every beat to a spoken word with `at('phrase')`. An element rises when its word is said (about 0.1 s before) and leaves only after its sentence ends.
3. Check `?t=SECONDS` freezes at the beats, then hand over the URL.

Rules that apply to every frame are in `reference/style.md`. Run the layout audit before each handover:

```bash
node $SKILL/scripts/audit.mjs http://127.0.0.1:5600/scene/ --every 0.5
```

### 6. Sound effects (only if wanted)

Generate the library, then cue effects from the scene's timeline with `cue(t, 'whoosh')`:

```bash
python3 $SKILL/scripts/sfx.py lib --out scene/sfx            # --gain -3 makes every effect 3 dB quieter
```

Levels and rules are in `reference/audio.md`. The user's first reaction to the level decides: if they say "can't hear it" or "too much", change the level in small steps and remember the answer for the rest of the video.

### 7. Final render and mix

```bash
python3 $SKILL/scripts/build.py --project . video --final           # source-resolution program video (the render background)
node $SKILL/scripts/render.mjs "http://127.0.0.1:5600/scene/?src=../work/prog-final.mp4" --out out/frames --workers 4 --dpr 2
python3 $SKILL/scripts/mix.py --project . --video out/frames --out out/FINAL.mp4
python3 $SKILL/scripts/loudness.py out/FINAL.mp4
```

Only start a long final render after the user approves the preview. Ask before deleting anything to free disk space.

Deliver the file plus a short report: length, resolution, loudness (integrated LUFS, true peak), the YouTube chapter list, and what was not checked.

## Scripts

| Script | What it does |
|---|---|
| `transcribe.py` | Chunked transcription with word times (faster-whisper or whisperkit-cli) |
| `cuts.py` | EDL to program: cut points snapped to measured word edges, pause removal, chapters, words in program time |
| `build.py` | Sample-exact program audio (gain only, no filters by default) and frame-exact video |
| `verify.py` | Re-transcribes the program around every join to catch clipped words and repeats |
| `loudness.py` | Integrated LUFS, true peak, loudness range, with a plain verdict |
| `sfx.py` | Sound effects synthesized from math: whoosh, tick, click, key, typing, pop, chime |
| `serve.py` | Static server with HTTP range support (needed for video seeking) |
| `render.mjs` | Frame-exact render of the scene in headless Chrome, piped into ffmpeg, parallel workers |
| `audit.mjs` | Finds overlapping cards, text under the face, and elements touching the frame edge |
| `mix.py` | Joins the rendered parts with the voice and the effects track, final encode |

Requirements: Python 3.10+, numpy, ffmpeg + ffprobe, Node 20+, Google Chrome or Chromium, and one Whisper backend (`pip install faster-whisper`, or `whisperkit-cli` on Apple Silicon).

## Working notes

Keep `work/NOTES.md` up to date: the answers from the intake, every rule the user gives ("no sound effects in this video", "effects one step quieter"), and the current version. Rules the user gives in one video usually apply to the next one; ask before assuming so.
