# spreadfire-skills

Claude Code skills for publishing short-form video to social platforms via Playwright browser automation — no API keys required.

## Skills

- **post-on-instagram** — Upload a video as a reel to Instagram. Handles login (browser or 1Password), video upload, caption, and publish.
- **post-on-tiktok** — Upload a video to TikTok. Handles login (browser or 1Password), video upload, caption, and publish.

## Installing

Copy a skill folder into your Claude Code skills directory:

```bash
cp -R post-on-instagram ~/.claude/skills/
cp -R post-on-tiktok ~/.claude/skills/
```

Each skill is a single `SKILL.md`. Claude Code picks it up automatically the next session.

## Login & credentials

Both skills default to **you logging in yourself** in the browser — Claude never handles your credentials. Optionally, you can use the [1Password CLI](https://developer.1password.com/docs/cli/) (`op`) with an item titled `Instagram` / `TikTok` holding `username` + `password` fields.
