---
name: post-on-tiktok
description: Post a video to TikTok using Playwright browser automation — no API needed. Use this skill whenever the user wants to upload a video to TikTok, post a TikTok, share a clip on TikTok, or publish content to their TikTok profile. Also use when the user says things like "post this on TikTok", "upload to TikTok", "share this on TikTok", or "put this on my TikTok". Credentials come from the 1Password CLI (`op`) or manual entry — the user is asked which to use.
---

# Post on TikTok

Automates posting a video to TikTok via TikTok Studio using Playwright MCP.

## Prerequisites

- Playwright MCP must be available
- A video file to upload
- A TikTok account — the user either logs in themselves in the browser, or uses 1Password (see step 1)

## Inputs

Two modes are supported:

**Mode A — Local file upload:**
- **Video file path** — absolute path to the video (mp4 recommended)
- **Caption / description** — the caption text (ask if missing, see step 0)
- **Tags** — hashtags to append to the caption (ask if missing, see step 0)

**Mode B — Copy from TikTok URL:**
- **TikTok URL** — a URL like `https://www.tiktok.com/@user/video/123...`
- Caption is automatically copied from the original video's description

## Steps

### 0. Gather the caption and tags (ask if missing — Mode A only)

Before doing anything else, check whether the user provided a caption/description and tags for the video. (In Mode B the caption is copied from the original video, so skip this step.)

- **If no caption/description was provided:** ask the user what caption or description they want for the video.
- **If no tags/hashtags were provided:** ask the user which hashtags they want (or confirm they want none).

Use the `AskUserQuestion` tool (or a plain question) to collect whatever is missing. Do not invent a caption or tags on your own — wait for the user's input. Once you have both, combine them into the final caption text (caption/description followed by the hashtags).

### 0. (Mode B only) Download video from TikTok URL

If the argument is a TikTok URL (starts with `https://www.tiktok.com`), download the video and its description using `yt-dlp`:

```bash
# Download the video
yt-dlp -o "/path/to/project/tiktok_download.mp4" --no-playlist "<TIKTOK_URL>"

# Fetch the description
yt-dlp --skip-download --print description "<TIKTOK_URL>" 2>/dev/null
```

Use the downloaded file path as the video file path and the fetched description as the caption. Then proceed with the normal upload steps below.

### 1. Choose how to log in (browser or 1Password)

Ask the user how they want to log in, using `AskUserQuestion` with two options:

- **Log in myself in the browser** (default) — you open the login page and the user types their own phone/email and password directly into the browser. **Never ask the user to hand you their credentials, and never type credentials into the login form yourself.**
- **Use 1Password** — requires the 1Password CLI (`op`) installed, the desktop app running and unlocked, and an item titled **"TikTok"** with `username` (phone number with country code e.g. `+31612345678`) + `password` fields.

**If the user chooses 1Password:**

```bash
op item get "TikTok" --fields username,password
```

Parse the output to extract `username` (phone number) and `password`. If `op` is not installed, not signed in, or the item is missing, tell the user and fall back to the browser login. Never log, echo, or write these credentials to disk — hold them only for the login step below.

**If the user chooses the browser login (or 1Password is unavailable):** there is nothing to collect here — the login happens in step 3.

### 2. Copy video to an accessible location

Playwright MCP can only access files within the current project directory. If the video is outside the project directory, copy it there first:

```bash
cp "/path/to/video.mp4" "./reel.mp4"
```

Use the project-local path for the upload step.

### 3. Log in to TikTok

Navigate to `https://www.tiktok.com/login/phone-or-email/phone-password`.

**If using 1Password credentials:** the country code selector defaults to NL +31 — if the username starts with `+31`, enter only the digits after `+31` in the phone field (e.g. `612345678`). Otherwise click the country code button and select the right country first. Fill in the phone number and password, then click **Log in**. If a verification screen (SMS/email code, CAPTCHA, 2FA) appears, pause and ask the user to handle it.

**If the user is logging in themselves (default):** **hand control to the user so they can type their own credentials directly into the browser — do not fill in the login form yourself.** Tell them the login page is open and ask them to log in and reply when they're done (e.g. "done" / "logged in"). Then **wait for their confirmation** before continuing — do not proceed while the login page is still showing. Any verification (SMS/email code, CAPTCHA, 2FA) is handled by the user in the browser too.

Either way, once login should be complete, take a `browser_snapshot` to verify you're logged in (you should no longer see the login form). If the login form is still there, ask the user to finish logging in and wait again.

### 4. Navigate to the upload page

Go to `https://www.tiktok.com/upload` (redirects to TikTok Studio).

### 5. Upload the video

Click the **Select video** button — a file chooser opens. Use `browser_file_upload` immediately with the project-local video path.

Wait a few seconds for the upload to complete. You'll see "Uploaded" with a green checkmark when done.

### 6. Dismiss popups

Two dialogs may appear after upload — dismiss both:
- **"Turn on automatic content checks?"** → click **Cancel**
- **"New editing features added"** → click **Got it**

### 7. Add the caption

Find the **Description** field (a combobox/rich text area under "Details"). Clear any existing text, then type the final caption (description + tags gathered in step 0, or the copied description in Mode B).

### 8. Post

Scroll down and click the **Post** button.

TikTok redirects to `tiktokstudio/content` and shows a **"Video published"** toast notification. The post is live (it may show "Content under review" briefly — this is normal).

Report success to the user.

## Notes

- The video file must be within the Playwright-allowed directory (the project directory)
- After posting you can clean up the copied file from the project directory if desired
- New posts may show as "Private" + "Content under review" initially — they go public once TikTok's review passes
- If a CAPTCHA or extra verification step appears, pause and ask the user to handle it manually
