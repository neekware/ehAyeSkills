---
name: telegram-setup
description: A guide on how to set up a Telegram bot for notifications and two-way communication.
version: 1.0.0
category: guides
tags: telegram, bot, setup, notifications, chatops
---

# Telegram Bot Setup Guide

Complete guide for creating Telegram bots, groups, and securing them for private communication between your application and your phone.

## Overview

ehAye uses Telegram bots to send notifications and receive replies. Each bot talks to dedicated private groups where only the bot and the group owner can communicate.

### ehAye Portal (@ehaye_alerts_bot)

Three groups, one bot. Routes business events by category:

| Group         | Purpose                                                        |
| ------------- | -------------------------------------------------------------- |
| **ehAye Biz** | Purchases, invoices, refunds, subscriptions, disputes          |
| **ehAye Ops** | Signups, logins, downloads, trial activity, license activation |
| **ehAye Sys** | System errors, API warnings, DB warnings, security alerts      |

The Portal sends notifications fire-and-forget. No inbound replies — these are one-way alert channels.

### ehAye Engine (@ehaye_dojo_bot)

Two groups, one bot. Routes by Dojo voice role:

| Group              | Purpose                                          |
| ------------------ | ------------------------------------------------ |
| **ehAye Dojo (P)** | Primary Dojo session notifications and replies   |
| **ehAye Dojo (S)** | Secondary Dojo session notifications and replies |

The Engine supports **two-way communication**: when a Dojo session goes idle, the bot sends a summary to the matching group. The user can reply with text or a voice note, which gets injected back into the active Dojo session. Voice notes are transcribed via Whisper before injection.

## Setup Instructions

### Step 1: Create a Bot

1. Open Telegram on your phone
2. Search for **@BotFather** and start a chat
3. Send `/newbot`
4. Enter a display name (e.g., "My App Bot")
5. Enter a username ending in `_bot` (e.g., `my_app_bot`)
6. BotFather replies with a **token** — save it securely in `bot.md`

#### Rename a Bot (optional)

1. Send `/mybots` to @BotFather
2. Select the bot
3. **Edit Bot** > **Edit Username**
4. Enter the new username

> Note: Renaming generates a new token. The old token becomes invalid immediately.

### Step 2: Disable Privacy Mode

By default, bots in groups can only see `/commands`. For the bot to receive all messages (required for reply features), disable privacy mode.

1. Send `/setprivacy` to @BotFather
2. Select the bot
3. Choose **Disable**

#### Verify

```bash
curl -s "https://api.telegram.org/bot<TOKEN>/getMe" | python3 -m json.tool
```

Confirm `"can_read_all_group_messages": true` in the response.

> For one-way bots (like the Portal alerts bot), this step is optional. It's only required if you need the bot to read replies.

### Step 3: Create Telegram Groups

Create one group per channel you need. Name them clearly so you can identify them on your phone at a glance.

Examples:

- One-way alerts: "MyApp Alerts", "MyApp Errors"
- Two-way channels: "MyApp Chat (Primary)", "MyApp Chat (Secondary)"

### Step 4: Add the Bot to Each Group

1. Open the group
2. Tap the group name at the top
3. **Add Members**
4. Search for your bot's username
5. Add it

### Step 5: Get Chat IDs

After adding the bot and sending a message in each group:

```bash
curl -s "https://api.telegram.org/bot<TOKEN>/getUpdates" | python3 -m json.tool
```

Look for `"chat": { "id": -XXXXXXXXXX, "title": "Group Name" }` in the response. Group chat IDs are **negative numbers**. Save them in `bot.md`.

> **If the response is empty:**
>
> - Privacy mode may be blocking messages — disable it (Step 2)
> - The bot was added **before** disabling privacy — **remove and re-add** the bot (privacy changes only take effect on re-join)
> - Try sending `/start` in the group as a workaround (bots always see commands)

### Step 6: Make the Bot an Admin

The bot needs admin rights to lock down groups and send messages reliably.

1. Open the group
2. Tap the group name > **Edit** (pencil icon)
3. **Administrators** > **Add Administrator**
4. Search for the bot and select it
5. Enable these permissions:
   - **Send Messages** (required)
   - **Change Group Info** (required for lockdown)
   - Pin Messages (optional)
   - Manage Chat (optional)

#### Turn Off "All Members Are Administrators"

This must be disabled before you can restrict member permissions. When enabled, every member is treated as an admin and cannot be restricted.

1. Group settings > **Edit**
2. Disable **All Members Are Administrators**

### Step 7: Lock Down the Group

Restrict all non-admin members from doing anything. Only admins (you and the bot) can send messages.

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

Expected response: `{"ok": true, ...}`

**Common errors:**

- `"not enough rights"` — Bot is missing **Change Group Info** permission, or "All Members Are Administrators" is still on
- `"chat not found"` — Wrong chat ID, or bot was removed from the group

#### Verify Lockdown

```bash
curl -s "https://api.telegram.org/bot<TOKEN>/getChat?chat_id=<CHAT_ID>" | python3 -m json.tool
```

All permissions in the response should show `false`.

### Step 8: Test the Connection

Send a test message from the bot:

```bash
curl -s "https://api.telegram.org/bot<TOKEN>/sendMessage" \
  -H "Content-Type: application/json" \
  -d '{
    "chat_id": "<CHAT_ID>",
    "text": "Test message — connection verified!"
  }'
```

You should receive the message on your phone instantly.

## Application Configuration

### ehAye Portal

Bot token and chat IDs are configured via environment variables:

```env
TELEGRAM_BOT_TOKEN=<token from BotFather>
TELEGRAM_CHAT_BIZ=<chat ID for business alerts>
TELEGRAM_CHAT_OPS=<chat ID for operations alerts>
TELEGRAM_CHAT_SYS=<chat ID for system alerts>
```

The Portal uses a database-driven notification system with:

- **Per-event kill switches** — enable/disable specific event types
- **Frequency control** — notify every Nth occurrence (e.g., every 5th signup)
- **Outbox retry queue** — failed messages retry with exponential backoff (5 attempts)
- **Session throttling** — max 10 messages per request to prevent loops

### ehAye Engine

Bot token and chat IDs are configured in the Settings UI:

1. Open ehAye Engine > **Settings** > **Telegram**
2. Enter the bot token
3. Enter the Primary chat ID (for primary Dojo role)
4. Enter the Secondary chat ID (for secondary Dojo role)
5. Choose message detail level: Brief, Talkative, or Chatty
6. Toggle **Enabled**
7. Use the **Test** button next to each chat ID to verify

The Engine supports two-way communication:

- **Outbound**: When Dojo goes idle, sends a summary to the matching group (primary or secondary)
- **Inbound**: Long-polls `getUpdates` for replies, injects text back into the active Dojo session
- **Voice notes**: Downloads `.ogg` from Telegram, transcribes via OpenAI Whisper API, injects text

## Telegram Commands

Send these as regular messages in your Dojo Telegram group. No slash prefix needed.

| Message         | What It Does                                                            |
| --------------- | ----------------------------------------------------------------------- |
| `hey dojo`      | Wake up the active session — Dojo says hello back                       |
| `status`        | Show running sessions for this channel's role                           |
| _(any text)_    | Send as input to the active Dojo session                                |
| _(voice note)_  | Transcribe via Whisper and send as input to Dojo                        |
| _(photo)_       | Downloaded and forwarded to Dojo as an image part                       |
| _(document)_    | Downloaded (PDF, CSV, Excel, Word) and forwarded to Dojo as a file part |
| _(sticker/GIF)_ | Ignored silently                                                        |

### How "Hey Dojo" Works

1. You send `hey dojo` in the Primary group
2. Backend finds the active primary session (auto-discovers if needed)
3. Injects "hello" into the session
4. Dojo processes "hello", generates a response, goes idle
5. Idle event triggers a Telegram notification back to you
6. You see Dojo's response on your phone

### How Session Routing Works

When you send a message, the bot figures out which Dojo session to target:

1. If you **reply to a specific notification**, it goes to that session
2. If not, it goes to the **last session that sent a notification** for this role
3. If no notifications were sent yet, it **auto-discovers** running sessions via the Dojo REST API

Messages in the **Primary group** route to the primary Dojo instance (port 44443).
Messages in the **Secondary group** route to the secondary instance (port 44442).

## Troubleshooting

| Problem                         | Cause                                                   | Fix                                                               |
| ------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------------- |
| Bot doesn't see group messages  | Privacy mode enabled                                    | `/setprivacy` > Disable in @BotFather, then remove and re-add bot |
| `getUpdates` returns empty      | No messages since bot joined, or privacy blocking       | Send `/start` in group, or disable privacy and re-add bot         |
| "Not enough rights" on lockdown | Missing admin permissions                               | Make bot admin with Change Group Info permission                  |
| Can't restrict members          | "All Members Are Administrators" is on                  | Disable it in group settings                                      |
| Token stopped working           | Bot was renamed (new token issued) or token was revoked | Get new token from @BotFather, update config                      |
| Messages not arriving on phone  | Bot not in group, or wrong chat ID                      | Verify with `getUpdates`, check chat ID matches                   |

## Security Checklist

- [ ] Bot tokens stored in `bot.md` (never in source code or public repos)
- [ ] Privacy mode set appropriately (disabled for two-way bots, doesn't matter for one-way)
- [ ] Bot is admin in all its groups
- [ ] "All Members Are Administrators" is OFF in all groups
- [ ] All member permissions set to `false` (admins-only communication)
- [ ] No active invite links (check group settings > Invite Links)
- [ ] Only you and the bot are admins in each group
- [ ] Tokens are not committed to git — use environment variables or secure storage

## Quick Reference

### Telegram Bot API

Base URL: `https://api.telegram.org/bot<TOKEN>/`

| Action                | Method | Endpoint              |
| --------------------- | ------ | --------------------- |
| Verify bot works      | GET    | `/getMe`              |
| Get incoming messages | GET    | `/getUpdates`         |
| Send a message        | POST   | `/sendMessage`        |
| Get group info        | GET    | `/getChat?chat_id=X`  |
| Lock down group       | POST   | `/setChatPermissions` |
| Download file (voice) | GET    | `/getFile?file_id=X`  |
| Set bot commands      | POST   | `/setMyCommands`      |

### Chat ID Format

| Type               | Format                     | Example          |
| ------------------ | -------------------------- | ---------------- |
| Private chat       | Positive number            | `123456789`      |
| Group              | Negative number            | `-5107785254`    |
| Supergroup/Channel | Negative with `100` prefix | `-1001234567890` |

### BotFather Commands

| Command           | Purpose                                |
| ----------------- | -------------------------------------- |
| `/newbot`         | Create a new bot                       |
| `/mybots`         | List and manage your bots              |
| `/setprivacy`     | Toggle group privacy mode              |
| `/setdescription` | Set bot description (shown on profile) |
| `/setabouttext`   | Set "About" text                       |
| `/setuserpic`     | Set bot profile picture                |
| `/revoke`         | Revoke and regenerate token            |
