---
name: post-on-instagram
description: Post a video as a reel on Instagram using Playwright browser automation — no API needed. Use this skill whenever the user wants to upload a video to Instagram, post a reel, share a clip on Instagram, or publish content to their Instagram profile. Also use when the user says things like "post this on Instagram", "upload to Instagram", "share this reel", or "put this on my Instagram". Credentials come from the 1Password CLI (`op`) or manual entry — the user is asked which to use.
---

# Post on Instagram

Automates posting a video as a reel on Instagram via the web UI using Playwright MCP.

## Prerequisites

- Playwright MCP must be available
- A video file to upload
- An Instagram account — the user logs in themselves in the browser (see step 1)

## Inputs

- **Video file path** — absolute path to the video (mp4 recommended)
- **Caption / description** — the caption text to use (ask if missing, see step 0)
- **Tags** — hashtags to append to the caption (ask if missing, see step 0)

## Steps

### 0. Gather the caption and tags (ask if missing)

Before doing anything else, check whether the user provided a caption/description and tags for the video.

- **If no caption/description was provided:** ask the user what caption or description they want for the video.
- **If no tags/hashtags were provided:** ask the user which hashtags they want (or confirm they want none).

Use the `AskUserQuestion` tool (or a plain question) to collect whatever is missing. Do not invent a caption or tags on your own — wait for the user's input. Once you have both, combine them into the final caption text (caption/description followed by the hashtags).

### 1. Choose how to log in (browser or 1Password)

Ask the user how they want to log in, using `AskUserQuestion` with two options:

- **Log in myself in the browser** (default) — you open the login page and the user types their own username and password directly into the browser. **Never ask the user to hand you their credentials, and never type credentials into the login form yourself.**
- **Use 1Password** — requires the 1Password CLI (`op`) installed, the desktop app running and unlocked, and an item titled **"Instagram"** with `username` + `password` fields.

**If the user chooses 1Password:**

```bash
op item get "Instagram" --fields username,password
```

Parse the output to extract `username` and `password`. If `op` is not installed, not signed in, or the item is missing, tell the user and fall back to the browser login. Never log, echo, or write these credentials to disk — hold them only for the login step below.

**If the user chooses the browser login (or 1Password is unavailable):** there is nothing to collect here — the login happens in step 3.

### 2. Copy video to an accessible location

Playwright MCP can only access files within the current project directory (or `.playwright-mcp/`). If the video is outside the project directory, copy it there first:

```bash
cp "/path/to/video.mp4" "./reel.mp4"
```

Use the project-local path for the upload step.

### 3. Log in to Instagram

Navigate to `https://www.instagram.com/accounts/login/`.

**If using 1Password credentials:** fill in the login form with the `username` and `password` from step 1 and click **Log in**. If a verification screen appears (WhatsApp/SMS/2FA code), pause and ask the user for the code, then enter it and click Continue.

**If the user is logging in themselves (default):** **hand control to the user so they can type their own username and password directly into the browser — do not fill in the login form yourself.** Tell them the login page is open and ask them to log in and reply when they're done (e.g. "done" / "logged in"). Then **wait for their confirmation** before continuing — do not proceed while the login page is still showing. Any verification (WhatsApp/SMS code, 2FA) is handled by the user in the browser too.

Either way, once login should be complete, take a `browser_snapshot` to verify you're logged in (you should see the home feed and their profile in the sidebar, not the login form). If the login form is still there, ask the user to finish logging in and wait again.

Ignore any "Save your login info?" prompt — it appears in the background and doesn't block the upload flow.

### 4. Open the post dialog

Click **New post** in the left sidebar, then click **Post** from the dropdown menu that appears.

### 5. Upload the video

Click **Select From Computer** — this opens a file chooser. Use `browser_file_upload` immediately with the project-local video path.

### 6. Dismiss the reels info dialog

After upload, Instagram may show a dialog: *"Video posts are now shared as reels"*. Click **OK** to dismiss it.

### 7. Navigate through the editor

- On the **Crop** screen → click **Next**
- On the **Edit** screen → click **Next**

### 8. Add the caption and share

On the caption screen (*"New reel"* heading):
- Click the **"Write a caption..."** textbox and type the final caption (description + tags gathered in step 0)
- Click **Share**

### 9. Wait for confirmation

Wait for the *"Reel shared"* / *"Your reel has been shared."* dialog to appear. Once it does, the post is live.

Report success to the user.

## Notes

- The "Save your login info?" prompt sits behind the upload dialog — it's safe to ignore throughout
- If the file chooser modal is already open from a previous click, call `browser_file_upload` directly without clicking again
- Instagram automatically treats video uploads as reels on the web — no special reel mode needed
- After the reel is shared you can clean up the copied file from the project directory if desired
