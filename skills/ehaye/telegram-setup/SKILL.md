---
name: telegram-setup
description: Set up Telegram for a newly downloaded ehAye Dojo so it can send idle summaries and receive replies, voice notes, and files from private groups.
version: 1.0.1
category: guides
tags: telegram, bot, setup, dojo, notifications, chatops
---

# Telegram Setup for a Fresh Dojo Install

Simple setup guide for connecting a newly downloaded **ehAye Dojo / ehAye Engine** to Telegram so Dojo can send idle summaries to your phone and you can reply back with text, voice notes, photos, documents, and other supported files.

## What You Are Setting Up

For **ehAye Engine / Dojo**, you need:

- **1 Telegram bot**
- **2 private Telegram groups**
  - **ehAye Dojo (P)** — Primary Dojo
  - **ehAye Dojo (S)** — Secondary Dojo
- the bot added to both groups
- the bot token and both group chat IDs entered into **ehAye Engine > Settings > Telegram**

This gives you **two-way Telegram for Dojo**:

- **Outbound** — when Dojo goes idle, it sends you a summary in Telegram
- **Inbound text** — you reply in Telegram and that message goes back into Dojo
- **Voice notes** — Telegram voice notes are transcribed and injected back into Dojo
- **Files** — photos, documents, video, audio, video notes, and animations can be forwarded into Dojo

## Before You Start

Have these ready:

- Telegram installed on your phone
- a newly downloaded and running copy of **ehAye Engine / Dojo**
- a safe place to store your bot token and chat IDs temporarily

> Store bot tokens securely. Do not commit them to git or paste them into public chat.

## Quick Start Checklist

- [ ] Create a Telegram bot with **@BotFather**
- [ ] Disable **privacy mode** for that bot
- [ ] Create **ehAye Dojo (P)** and **ehAye Dojo (S)** as private groups
- [ ] Add the bot to both groups
- [ ] Send a normal message like `hello` in both groups
- [ ] Use `getUpdates` to find both chat IDs
- [ ] Make the bot an admin in both groups
- [ ] Lock down both groups so only admins can post
- [ ] Open **ehAye Engine > Settings > Telegram**
- [ ] Enter the bot token and both chat IDs
- [ ] Press **Test** for each chat
- [ ] Send `hey dojo` in Telegram to confirm two-way routing

## Why Privacy Mode Must Be Disabled

This is the part people second-guess, so here is the important distinction:

- a **private group** controls who can join and read the group
- Telegram bot **privacy mode** controls which messages the bot can see *inside* the group

For Dojo, the bot must receive more than slash commands. It needs to see:

- normal typed replies
- voice notes
- photos
- documents
- video
- audio
- video notes
- animations

If privacy mode stays enabled, Telegram will hide most group messages from the bot and Dojo will not receive normal replies.

**For Dojo Telegram to work in groups: keep the groups private, and disable bot privacy mode.**

---

## Step-by-Step Setup

### Step 1: Create the Bot

1. Open Telegram on your phone
2. Search for **@BotFather**
3. Send `/newbot`
4. Enter a display name such as `ehAye Dojo`
5. Enter a username ending in `_bot`, such as `my_dojo_bot`
6. BotFather will return a **bot token**
7. Save that token somewhere secure for now

### Step 2: Disable Privacy Mode for the Bot

In Telegram with **@BotFather**:

1. Send `/setprivacy`
2. Select your bot
3. Choose **Disable**

#### Verify

```bash
curl -s "https://api.telegram.org/bot<TOKEN>/getMe" | python3 -m json.tool
```

Confirm this appears:

```json
"can_read_all_group_messages": true
```

> If you disable privacy mode **after** the bot was already added to a group, remove the bot from the group and add it again. Telegram often applies the new setting only after re-join.

### Step 3: Create the Two Private Groups

Create these two private groups in Telegram:

- **ehAye Dojo (P)**
- **ehAye Dojo (S)**

Recommended meaning:

| Group | Purpose |
| --- | --- |
| **ehAye Dojo (P)** | Notifications and replies for the Primary Dojo |
| **ehAye Dojo (S)** | Notifications and replies for the Secondary Dojo |

Keep both groups **private**.

### Step 4: Add the Bot to Both Groups

For each group:

1. Open the group
2. Tap the group name
3. Choose **Add Members**
4. Search for your bot username
5. Add the bot

### Step 5: Send One Normal Message in Each Group

In **both** groups, send a regular non-command message such as:

```text
hello
```

Use a normal message, not just `/start`, because you want to confirm the bot can see ordinary group messages too.

### Step 6: Get Both Chat IDs

Run:

```bash
curl -s "https://api.telegram.org/bot<TOKEN>/getUpdates" | python3 -m json.tool
```

Look for entries like this:

```json
"chat": {
  "id": -1001234567890,
  "title": "ehAye Dojo (P)"
}
```

You need two chat IDs:

- the ID for **ehAye Dojo (P)**
- the ID for **ehAye Dojo (S)**

Group chat IDs are negative numbers.

#### If `getUpdates` is empty

Check these in order:

1. Privacy mode is still enabled
2. The bot was added before privacy mode was disabled
3. You did not send a normal message yet
4. You are using the wrong bot token

Fix:

- disable privacy mode
- remove and re-add the bot
- send `hello` in each group again
- rerun `getUpdates`

### Step 7: Make the Bot an Admin in Both Groups

For each group:

1. Open the group
2. Tap the group name
3. Go to **Edit** or **Administrators**
4. Add the bot as an administrator

Enable at least:

- **Send Messages**
- **Change Group Info**

Optional:

- Pin Messages
- Manage Chat

### Step 8: Turn Off “All Members Are Administrators”

If Telegram shows this option in the group settings, turn it off.

If it remains on, Telegram may refuse to apply member restrictions.

### Step 9: Lock Down the Groups

To keep the groups private and quiet, restrict all non-admin members from posting. That way only **you** and the **bot** can talk.

Run this once for each group, replacing `<TOKEN>` and `<CHAT_ID>`:

```bash
curl -s "https://api.telegram.org/bot<TOKEN>/setChatPermissions" \
  -H "Content-Type: application/json" \
  -d '{
    "chat_id": "<CHAT_ID>",
    "permissions": {
      "can_send_messages": false,
      "can_send_audios": false,
      "can_send_documents": false,
      "can_send_photos": false,
      "can_send_videos": false,
      "can_send_video_notes": false,
      "can_send_voice_notes": false,
      "can_send_polls": false,
      "can_send_other_messages": false,
      "can_add_web_page_previews": false,
      "can_change_info": false,
      "can_invite_users": false,
      "can_pin_messages": false,
      "can_manage_topics": false
    }
  }'
```

Expected response:

```json
{"ok": true, ...}
```

#### Common errors

- `not enough rights` — the bot is not admin, or it lacks **Change Group Info**
- `chat not found` — wrong chat ID, wrong bot token, or bot is not in the group

### Step 10: Verify Lockdown

```bash
curl -s "https://api.telegram.org/bot<TOKEN>/getChat?chat_id=<CHAT_ID>" | python3 -m json.tool
```

The permissions section should show all member permissions as `false`.

### Step 11: Enter the Settings in ehAye Engine

Open:

**ehAye Engine > Settings > Telegram**

Fill in:

1. **Bot Token**
2. **Primary Chat ID** — the ID for **ehAye Dojo (P)**
3. **Secondary Chat ID** — the ID for **ehAye Dojo (S)**
4. **Message Detail Level** — Brief, Talkative, or Chatty
5. Turn **Enabled** on
6. Use the **Test** button for each chat

### Step 12: Test the Full Dojo Flow

After saving the settings:

1. Make sure Dojo is open and available
2. Send `hey dojo` in **ehAye Dojo (P)**
3. Dojo should wake and answer in the Primary group
4. Send plain text like `summarize what you are doing`
5. Send a voice note if you want to test transcription
6. Send a photo or document if you want to test file forwarding

If everything is working, your Telegram setup is complete.

---

## What Messages Does Dojo Understand in Telegram?

Send these as regular messages in the Dojo group.

| Message | What it does |
| --- | --- |
| `hey dojo` | Wakes Dojo |
| `status` | Shows Dojo status for that group |
| any normal text | Sends that text to Dojo |
| voice note | Transcribes and forwards it to Dojo |
| photo | Downloads and forwards it to Dojo |
| document | Downloads and forwards it to Dojo |
| video | Downloads and forwards it to Dojo |
| audio | Downloads and forwards it to Dojo |
| video note | Downloads and forwards it to Dojo |
| animation | Downloads and forwards it to Dojo |
| sticker / GIF | Ignored silently |

## Notes About Voice Messages

- typed text is forwarded directly
- voice notes are transcribed first
- if your Dojo Telegram settings include a wake word, voice notes may require that wake word before the transcribed text is forwarded
- if no wake word is configured, voice notes are forwarded directly after transcription

## How Routing Works

When you send a message in Telegram:

1. Messages in **ehAye Dojo (P)** go to Primary Dojo
2. Messages in **ehAye Dojo (S)** go to Secondary Dojo
3. If you reply to a specific Dojo message, Telegram uses that thread context to keep the conversation clear

Mapping:

| Group | Dojo Role |
| --- | --- |
| **ehAye Dojo (P)** | Primary |
| **ehAye Dojo (S)** | Secondary |

---

## Troubleshooting

| Problem | Cause | Fix |
| --- | --- | --- |
| Bot does not see normal group messages | Privacy mode still enabled | Disable privacy mode, then remove and re-add the bot |
| `getUpdates` returns empty | No non-command message has been seen yet | Send `hello` in the group, then retry |
| `getUpdates` still empty after disabling privacy | Bot was added before privacy mode was disabled | Remove and re-add the bot |
| `not enough rights` during lockdown | Bot is missing admin rights | Make bot admin and enable **Change Group Info** |
| Messages do not arrive on your phone | Wrong chat ID or bot is not in the group | Re-check `getUpdates` and group membership |
| `hey dojo` does nothing | Wrong role mapping, Telegram not enabled, or Dojo is not available | Re-check Primary/Secondary chat IDs, the Enabled toggle, and that Dojo is open |
| Voice note is received but not understood | Transcription failed or wake-word rules blocked it | Try a clearer recording and test with text first |
| File upload is rejected | File download failed or the file is too large | Retry with a smaller file; large Telegram files may be rejected |

## Security Checklist

- [ ] Group is **private**
- [ ] Bot privacy mode is **disabled**
- [ ] Bot is admin in both groups
- [ ] Only you and the bot are admins
- [ ] All member posting permissions are disabled
- [ ] No public username is set on the group
- [ ] No old invite links are left active
- [ ] Bot token is stored securely and never committed to git

---

## Optional: Telegram Setup for ehAye Portal

The Portal uses Telegram differently from Dojo.

### ehAye Portal (@ehaye_alerts_bot)

Three groups, one bot:

| Group | Purpose |
| --- | --- |
| **ehAye Biz** | Purchases, invoices, refunds, subscriptions, disputes |
| **ehAye Ops** | Signups, logins, downloads, trial activity, license activation |
| **ehAye Sys** | System errors, API warnings, DB warnings, security alerts |

Portal is usually **one-way alerts only**.

Environment variables:

```env
TELEGRAM_BOT_TOKEN=<token from BotFather>
TELEGRAM_CHAT_BIZ=<chat ID for business alerts>
TELEGRAM_CHAT_OPS=<chat ID for operations alerts>
TELEGRAM_CHAT_SYS=<chat ID for system alerts>
```

> For one-way alerts, disabling privacy mode is optional. It becomes required only if you want the bot to read replies from the group.

## Telegram Bot API Quick Reference

Base URL:

```text
https://api.telegram.org/bot<TOKEN>/
```

| Action | Method | Endpoint |
| --- | --- | --- |
| Verify the bot | GET | `/getMe` |
| Read incoming messages | GET | `/getUpdates` |
| Send a message | POST | `/sendMessage` |
| Read group info | GET | `/getChat?chat_id=X` |
| Lock down a group | POST | `/setChatPermissions` |
| Download file metadata | GET | `/getFile?file_id=X` |

## BotFather Commands

| Command | Purpose |
| --- | --- |
| `/newbot` | Create a bot |
| `/mybots` | Manage existing bots |
| `/setprivacy` | Change group privacy mode |
| `/setdescription` | Set bot description |
| `/setabouttext` | Set bot about text |
| `/setuserpic` | Set bot profile image |
| `/revoke` | Revoke and regenerate token |

> **Creator:** Ehaye
> **License:** MIT
> **Source Repo:** `neekware/ehaye-skills`
> **Source Bucket:** `ehaye`
> **Original Path:** `ehaye/telegram-setup`
