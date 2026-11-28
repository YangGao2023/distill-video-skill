---
name: distill-video
description: Break down any viral video into a reproducible recipe and turn what you learn into your own reusable AI skill. Checks and installs missing local tools automatically for beginners, studies hooks and pacing first (0-1s, 0-3s, 0-12s), then extracts visuals, captions, effects, transitions, and sound, finds open-source implementations, and packages the reusable style into a new skill folder.
---

# distill-video: turn any viral video into a reusable recipe (and your own skill)

Goal: take any video a user loves and break down *why* it works and *how* it was made, then turn those reusable lessons into a clear recipe — or a brand-new skill the user can keep combining with their other video skills.

If the user only wants a quick text summary of what the video says, use `watch-video` (if installed) or simply transcribe and summarize.

## Two rules first

1. **No sound, no player.** Download and analyze silently in the background. Never blast audio from a browser tab. If a webpage must be opened, mute and pause all media first and close the tab when done.
2. **Report only what you observed.** Sample colors from real full-size pixels. Label font guesses "looks like X" unless verified. Cite the source of any external number. Write "not verified" where you could not check.

## Step -1: Beginner-proof preflight (NEVER dump raw errors on the user)

Assume the user does not code and has never installed command-line tools.

1. **If the user asked you to install this skill**, put the entire `distill-video/` folder (including `SKILL.md` and `references/`) into the active agent's local skills directory — do **not** leave it sitting on the Desktop:
   - Claude Code: `~/.claude/skills/distill-video/`
   - Codex: `~/.codex/skills/distill-video/`
   - Antigravity / Gemini: `~/.gemini/config/skills/distill-video/`
   - Doubao Desktop Local PC Mode (豆包电脑版「本地电脑」) & cross-agent standard: `~/.agents/skills/distill-video/` (only fall back to manual zip upload via **技能 → 我的技能 → 上传技能** if network blocks downloading).
2. **Silently check what is installed** before running analysis commands: test `ffmpeg -version`, `yt-dlp --version`, and a speech-to-text option (Python `faster-whisper` / `whisper`, or `yt-dlp` subtitle download).
3. **Auto-install what is missing (ask permission once in plain words)**:
   - On **Windows**: run `winget install -e --id Gyan.FFmpeg` and `winget install -e --id yt-dlp.yt-dlp` (then refresh `PATH` or locate the installed `.exe` directly). For speech-to-text, also run `pip install faster-whisper` (if Python is available).
   - On **macOS**: run `brew install ffmpeg yt-dlp`, and for speech-to-text also run `pip3 install faster-whisper` (if Python is available).
   - On **Linux / cloud sandbox** (such as a cloud agent desktop): check `apt-get install -y ffmpeg` and `pip install yt-dlp faster-whisper imageio-ffmpeg`.
4. **Graceful fallbacks when tools or downloads are blocked**:
   - **No local Whisper / slow model download**: grab subtitles first with `yt-dlp --write-auto-sub --skip-download`, or pass the extracted audio/keyframes to your built-in multimodal model.
   - **Site blocks anonymous download (very common on Douyin, Xiaohongshu, Bilibili, or login-walled sites)**: if `yt-dlp` returns HTTP 403, login-required, or cookie errors, **never** bypass login/CAPTCHA and never dump a raw stack trace. Reply in one warm sentence: *"This platform blocks direct downloads — just save the video to your computer or drag the video file right into this chat, and I'll analyze it immediately."*
5. **Speak plain language to the user**: lead with clear, everyday takeaways ("the screen changes every 3 seconds so viewers never get bored") instead of burying beginners in raw command logs or dense math tables. Put exact specs in the saved recipe file.

## Step 0: timing and structure before pixels

Study *when* things happen before *what* they look like. Most beginners copy colors and miss the pacing that kept viewers watching.

- What happens on screen at **0-1 s**, **0-3 s**, and **0-12 s**? An instant result/contrast, a curiosity hook, or a slow intro?
- How quickly does the video deliver its first visual proof or payoff?
- How often does the screen meaningfully change (every 3-6 seconds, or long static holds)?
- How does it end: a question, a payoff callback, or a clean call to action?

Capture each structural lesson as a short card:
- **Use for**: video type (tutorial, demo, commentary, short drama...).
- **Use at**: moment (0-3 s hook, first 12 s, middle demo, comparison, ending).
- **Evidence**: what you directly saw in this video.
- **Do not use for**: where this trick would backfire.

## Step 1: get the video cleanly

- Local file dropped by the user: use it directly.
- YouTube and other supported links: `yt-dlp -f "bv*[height<=1080]+ba/b[height<=1080]/b" --merge-output-format mp4 -o v.mp4 "<url>"` (see https://github.com/yt-dlp/yt-dlp).
- Create a fresh empty working folder per video so frames never mix.

## Step 2: inspect the whole video (never lazy-sample)

Inspect at least one frame per second so half-second pop-ups and transitions are never missed.

- **Even-second contact sheets** (24 s per sheet, 4x3 grid):
  `ffmpeg -y -i v.mp4 -vf "fps=1/2,scale=480:-1,tile=4x3:padding=4:color=red" sheets/sh_%02d.jpg`
  (Sheet `n`, cell `k` is roughly `(n-1)*24 + (k-1)*2` seconds.)
- **Odd-second contact sheets** (staggered by 1 second):
  `ffmpeg -y -ss 1 -i v.mp4 -vf "fps=1/2,scale=480:-1,tile=4x3:padding=4:color=blue" odd/sh_%02d.jpg`
- **Scene-change frames** (catches fast cuts and card pop-ins):
  `ffmpeg -y -i v.mp4 -vf "select='gt(scene,0.04)',scale=384:-1" -fps_mode vfr dense/d_%04d.jpg`
  then tile them: `ffmpeg -y -i dense/d_%04d.jpg -vf tile=5x4:padding=4 dense/sheet_%02d.jpg`.
- **Fast motion / animation zoom-in** (5 fps for 2 seconds):
  `ffmpeg -y -ss <sec> -t 2 -i v.mp4 -vf "fps=5,scale=480:-1,tile=5x2" move.jpg`
- **Full-resolution frame** (for reading UI, fonts, and exact pixel colors):
  `ffmpeg -y -ss <sec> -i v.mp4 -frames:v 1 f.png`
- **Never use `ffmpeg drawtext` for timestamps** (missing system fonts crash it across Windows/Linux); calculate timestamps from grid index instead.

Read every contact sheet. Then ask: *what visual trick did I miss?*

## Step 3: transcript and speech-to-visual sync

- Extract mono 16 kHz audio (`ffmpeg -y -i v.mp4 -ar 16000 -ac 1 a.wav`) and transcribe to a UTF-8 text/JSON file (do not rely on raw console output, which garbles non-ASCII text on Windows).
- Copy exact on-screen text (prompts, headlines, callouts, numbers).
- Measure **speech-to-visual timing**: give at least 5 concrete examples of when a graphic/card/highlight enters relative to the spoken keyword (earlier, exact beat, or ~0.1 s after).
- Note voice delivery: pacing, pauses, warmth/energy, and whether an on-screen presenter uses side pop-ups or a talking corner bubble during demos.

## Step 4: visual spec table (measured, not guessed)

Fill every row from real full-size frames:

| Item | What to record |
|---|---|
| Palette | Real sampled hex colors (background, card surface, primary accent, highlight) — strip brand skin before reusing |
| Typography & scale | Estimated px at 1080p for headlines, body, subtitles; check mobile readability |
| Subtitles | Position, single-line vs multi-line, outline/shadow, keyword highlight |
| Pop-ups & motion | Entrance/exit style, hold duration, cursor zoom/highlights, side cards next to presenter |
| Cuts & pacing | Typical seconds between fresh visuals, longest static hold, share of real motion video vs static frames |
| Sound | Voice energy, music changes by act, subtle click/whoosh SFX placement |

**Count everything.** Do not list 3 effects if the video uses 15. Keep other creators' embedded screenshots separate from the host's own design system.

## Step 5: check existing open-source tools & track B fact-check

- Search GitHub and the web for open-source implementations (HTML/Canvas motion kits, Playwright + FFmpeg pipelines, Remotion, FFmpeg filters) before building from scratch. Always note license limits (for example, verify whether a framework requires a paid commercial license for larger teams, and avoid redistributing proprietary app assets).
- **For tutorial or tool videos (Track B)**: verify every tool name, command, price, and claim against official docs or the source repository so you never copy outdated or exaggerated advice. Skip for pure entertainment and say so.

## Step 6: interface-tutorial recipes (when breaking down UI / workflow tutorials)

First check the video description, chapters, or pinned comment for official templates. If none exist, sample moments where the presenter holds a screen for >= 1 second, crop and enlarge the exact button/node/prompt area, and record a step-by-step recipe table (what button is clicked, exact prompt text, parameters, and how to reproduce it). Write "unclear" rather than guessing.

## Step 7: turn the teardown into YOUR OWN reusable skill (finished means reusable!)

A report sitting in a folder does not change the user's next video. Always finish by turning what you extracted into a real skill folder the user's AI can reuse immediately:

1. **Give the user a plain-language summary first** (3-5 crisp takeaways anyone can understand — no dense jargon dump).
2. **Strip the original creator's brand skin** (never copy their exact logo, signature color theme, or watermark); keep the *reusable mechanics* (hook timing, 4-8 s visual refresh cadence, button zoom + highlight, side pop-up timing, before/after video compare).
3. **Create a brand-new reusable skill folder** inside the user's active skills directory (e.g. `~/.claude/skills/<my-style-name>/SKILL.md`, `~/.codex/skills/<my-style-name>/SKILL.md`, `~/.gemini/config/skills/<my-style-name>/SKILL.md`, or a ready-to-upload zip folder for Doubao Desktop) using this exact starter structure:

```markdown
---
name: <short-kebab-name>
description: Reusable video style extracted from reference breakdowns. Use when making <video type> with <key mechanics>.
---

# <Style Name>

## Hook & Pacing Rules
- 0-3s opening move: ...
- Visual refresh cadence: every X-Y seconds (never static >10s)
- Speech-to-visual sync: ...

## Reusable Visual Mechanics (Brand-Neutral)
- Layout & framing (mobile-readable sizes): ...
- Motion & highlights (e.g., 3x button zoom, side pop-ups, real-video compare): ...
- Subtitles & audio pacing: ...

## Do Not Copy / Watch-Outs
- Swap colors/fonts to the user's own channel identity.
- Avoid using this pattern for: ...
```

4. **Show how to combine it & give the exact one-line trigger prompt**: explain that the user can combine this new skill with their other video skills (such as `faceless-tech-explainer`, `script-writing`, `video-packaging`, and `post-production`), and give them one ready-to-copy line:
   - *"Use my `<short-kebab-name>` skill together with `faceless-tech-explainer` to make a 3-minute video about [topic]."*

## Whole-creator benchmark (optional deep dive)

When analyzing an entire channel or multiple videos from one creator, use the 40-indicator checklist in `references/creator-teardown-40-indicators.md`.
