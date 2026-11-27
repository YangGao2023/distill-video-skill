---
name: distill-video
description: Break down someone else's video into a reproducible recipe. Studies timing and structure first (what happens at 0-1s, 0-3s, 0-12s), then visuals (colors, fonts, captions, effects, transitions, pacing, sound) using ffmpeg contact sheets and transcription, then finds existing open-source implementations. Use when asked to analyze, reverse-engineer, or learn from a viral video or copy a style.
---

# distill-video: turn a video into a reproducible spec

Goal: understand how a video works well enough that someone else could recreate the approach. If you only need to know what the video says, use `watch-video` if you also installed it, or simply transcribe it and summarize.

Tools needed: ffmpeg, yt-dlp (for downloads), and any speech-to-text tool (a Whisper-family tool works).

## Two rules first

1. **No sound, no player.** Download and analyze in the background. If a page must be opened, mute and pause all media first and close the tab afterwards.
2. **Report only what you observed.** Sample colors from real pixels. Label font guesses "looks like X" unless verified. Cite the source of any number you did not measure. Write "not verified" where you could not check.

## Step 0: timing and structure before pixels

Study when things happen before what they look like. Many people copy visuals and miss the pacing that made the video work.

- What is on screen at 0-1 s, 0-3 s, 0-12 s? A result, a stake, an action, or a slow intro?
- How is the setup or rules section delivered, and how long until the first payoff?
- How far apart are the small hooks and the big reversals?
- How does it end: a question, a cliffhanger, a call to action?

Write each lesson as a card. A card missing a field is not finished:
- **Use for**: which kind of video (tutorial, drama, product demo, vlog...).
- **Use at**: which step (topic, title/cover, 0-1 s, 0-3 s, first 12 s, middle, climax, ending, posting).
- **Evidence**: how many videos, which numbers, which source.
- **Do not use for**: where it does not apply, and why.

Keep the cards in your write-up; if you keep a checklist per video type, add them there so you can read them before starting that type of video.

## Step 1: get the file

- Local file: use it.
- YouTube and many other sites: `yt-dlp -o v.%(ext)s <url>` (https://github.com/yt-dlp/yt-dlp).
- If a site refuses anonymous downloads, say so; do not work around login walls.
- Use a new, empty directory per video.

## Step 2: look at the whole video, not a sample

Check at least one frame per second. Sampling every 2 seconds misses effects that flash by in under a second.

- Contact sheets, even seconds (24 s per sheet, 4x3 grid):
  `ffmpeg -i v.mp4 -vf "fps=1/2,scale=480:-1,tile=4x3:padding=4:color=red" sheets/sh_%02d.jpg`
  Sheet n, cell k is about (n-1)*24 + (k-1)*2 seconds.
- Contact sheets, odd seconds: same command with `-ss 1` before `-i`, output into `odd/`.
- Scene-change frames, for transitions:
  `ffmpeg -i v.mp4 -vf "select='gt(scene,0.04)',scale=384:-1" -fps_mode vfr dense/d_%04d.jpg`
  then tile them: `ffmpeg -i dense/d_%04d.jpg -vf tile=5x4:padding=4 dense/sheet_%02d.jpg`. Slow fades will not trigger this; rely on the sheets for those.
- For an interesting effect, sample that moment at 5 fps: `ffmpeg -ss <sec> -t 2 -i v.mp4 -vf fps=5,scale=480:-1,tile=5x2 move.jpg`
- Full-size frame for fonts and fine detail: `ffmpeg -ss <sec> -i v.mp4 -frames:v 1 f.png`
- Do not stamp time codes with `drawtext` (font configuration can crash on some systems); compute them from grid position.

Read every sheet. Then ask: what did I not list?

## Step 3: content

- Transcribe locally (`ffmpeg -i v.mp4 -ar 16000 -ac 1 a.wav`, then a Whisper-family tool). Save to a file and read the file; some consoles garble non-ASCII output.
- Copy meaningful on-screen text (prompts, steps, numbers) exactly.
- Record how many steps it teaches, the exact words per step, quoted numbers (time, cost, versions), and the tools or models used.
- Does the picture follow the speech? Using transcript timestamps, give at least five examples of what appears relative to the sentence (earlier, same time, later), and state the pattern: one visual per sentence, or visuals only on keywords?
- Real voice or synthetic? Look for breathing, slips, constant background noise, lip sync. Write the evidence; mark the conclusion as inference.

## Step 4: visual spec (fill every row)

| Item | How precise |
|---|---|
| Background, accent, subtitle colors | Sample real pixels at original size, give hex values |
| Fonts | Title, subtitle, annotation: sans/serif/handwritten, closest open-source look-alike, letter spacing |
| Type scale | Estimated px at 1080 high: big numbers, headline, subtitle, notes |
| Subtitle style | Vertical position (% of height), outline or box, keyword emphasis, one or two languages |
| Pop-up text | Per type: look, entrance, hold time, exit |
| Effects around text | Glow, rings, particles; rotation speed |
| Pop-up images | Framed or not, where from, scale, where they rest |
| Layout | Corner badges, progress bar, title strip, safe areas |
| Transitions | Fade, slide, blur, zoom; speed |
| Pacing | Seconds per screen, cuts on the beat or not, share of static time |
| Sound | Voice or none, music genre and tempo, effect types and where they land |

**Count everything.** One row per font, per text effect, per pop-up type, per transition, and state "N used in this video". Do not list five fonts if the video uses forty, unless those five really repeat that often. Other people's assets (a wall of thumbnails, screenshots) go in a separate list.

## Step 5: look for existing implementations

Before building, search GitHub and the web for the style name plus words like skill, template, Remotion, or editor-project export. Many viral styles already have open-source reproductions. Record each license and whether it can ship in a product. Some libraries need a paid license above a company size, and many font and effect packs forbid redistribution.

## Step 6: write it up

Save a page that states when you checked, the source video, and: (1) what it teaches, (2) how it looks (the table), (3) how it is built, (4) what it means for your project (mark as your judgment). List what you already can do and what is missing.

**Finished means used.** A write-up alone is not learning. For each technique choose one: wire it into your tooling or a checklist and test it, or decide not to use it and write why. Pure knowledge that cannot become a tool may stay as a page.

## Report format

Short headings and a table: what you saw, where it is applied, what is still missing.

## Additional extraction (every video)

- Track B for tutorial or tool videos: verify every tool name, command, quota, price and tip against official docs or the source repository (never trust the creator blindly; look for mixed-up tiers, outdated info, exaggeration). Then, if you keep a list of the tools you already use, note which parts you already use and which are new, and what they would save you. Skip for pure entertainment and say so.
- Body motion: each useful action (which side moves first, weight shift, order, amplitude, duration, hair and clothing response) with a phrase usable in a prompt.
- Facial expression: how the face changes (eyes, brows, mouth corners, cheeks, head and shoulders, hands, breathing, order) and how an emotion escalates in layers.
- Editing: average and median shot length from scene-change detection, how each situation is cut, where reaction shots sit, how transitions work. If one of these has nothing to learn (a single continuous take has no editing), write why.
- Color sampling: on full-size frames, for bright colors take pixels with channel sum above 350 and the mode after quantizing to 8 levels per channel.

## Recipes from interface tutorials

For screen-recorded tool tutorials: first look for the author's original (transcript, chapter summaries, pinned comments, author profile or collection, platform template gallery; note which places you checked). Only if none exists, read frames where the presenter pauses (sample every half second, keep the middle of any hold of 1 s or more), crop one node at a time and enlarge it; write "unclear" rather than guess. Make a recipe table, one row per canvas node (the tiles or nodes on the tool's canvas that the presenter fills in): purpose (first frame, character reference, outfit, scene, texture, voice, prompt text, parameters, connection, repair, upscale), verbatim text, platform parameters, how you will reproduce it, and implemented or not (with reason).

## Open-source references

Reported as of 2026-10, verify before relying: the HyperFrames effect catalog (reported Apache-2.0, https://hyperframes.heygen.com/catalog) and community lists of video-related agent skills on GitHub. Reported, verify: Remotion needs a paid license for companies with four or more people; effect assets bundled in consumer editing apps typically cannot be redistributed.

## Whole-creator benchmark

For account data, hooks, covers, lighting, cut points, motion graphics, audio and replication, use the 40-indicator checklist in `references/creator-teardown-40-indicators.md`.
