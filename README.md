[English](README.md) | [简体中文](README.zh-CN.md)

# distill-video: break down any viral video into a recipe you can reuse

A free skill for Claude Code and Codex. Give your agent a video (a link or a local file) and it turns it into a reproducible spec: what happens in the first 1, 3 and 12 seconds, how the story is structured, and how it looks and sounds: colors, fonts, captions, effects, transitions, pacing, music and sound effects. Then it searches for existing open-source tools that already do each effect, so you can make your own video in the same style instead of guessing.

It is plain text. There are no scripts to run and nothing is sent anywhere. Your agent uses tools you already have, such as ffmpeg and a transcriber.

## Example output

To be added with the video for this skill.

## Video

Coming soon.

## Install

Claude Code:

```
git clone https://github.com/YangGao2023/distill-video-skill
cp -r distill-video-skill/distill-video ~/.claude/skills/
```

Codex: copy the same folder into `~/.codex/skills/`. On Windows, copy it into the `.claude\skills` or `.codex\skills` folder in your user directory. Start a new agent session, then ask in plain words, for example "break down this video for me: <link>".

## More skills

This skill is one of a full set for making videos with AI: script, shot prompts, acting, editing, covers, publishing and review. See [ai-video-skills](https://github.com/YangGao2023/ai-video-skills).

## Run this skill from your phone: YAAH

YAAH is an iPhone app that lets you direct the Claude Code and Codex agents already running on your own Mac or Windows PC, by voice or by typing. When an agent needs permission, the request appears on your phone and you tap Allow or Deny. You can follow what it is doing step by step while you are away from your desk, as long as your computer is on and online.

- App Store: https://apps.apple.com/us/app/yaah-ai-agents/id6780028804
- Website: https://yaah.ai

<p>
<img src="assets/yaah-01-home.jpg" alt="YAAH home screen" width="220">
<img src="assets/yaah-02-assistant-chat.jpg" alt="Chatting with an assistant" width="220">
<img src="assets/yaah-04-agent-task.jpg" alt="An agent working on a task" width="220">
</p>

## License

MIT, see [LICENSE](LICENSE). You may modify and use this skill commercially. The names are not licensed; see [TRADEMARK.md](TRADEMARK.md).
