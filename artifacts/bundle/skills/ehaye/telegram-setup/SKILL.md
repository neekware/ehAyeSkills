# Telegram Setup for ehAye Dojo

Connect **ehAye Dojo / ehAye Engine** to Telegram so Dojo can send you session summaries and you can reply back with text, voice notes, photos, documents, and other files — from anywhere.

## What You Are Setting Up

You need:

- **1 Telegram bot** (you create it via @BotFather)
- **2 Telegram groups** (one per Dojo lane)
  - **ehAye Dojo (P)** — Primary Dojo
  - **ehAye Dojo (S)** — Secondary Dojo
- the bot added as **admin** to both groups
- bot **privacy mode disabled** so it can read normal messages
- bot token and both group chat IDs entered into **ehAye Engine > Settings > Telegram**

This gives you **two-way Telegram for Dojo**:

- **Outbound** — Dojo sends session summaries, idle notifications, and voice notes to Telegram
- **Inbound text** — you reply in Telegram and the message goes into Dojo
- **Voice notes** — Telegram voice notes are transcribed and injected into Dojo
- **Files** — photos, documents, video, audio, video notes, and animations are forwarded into Dojo

## Before You Start

Have these ready:

- Telegram installed on your phone
- a running copy of **ehAye Engine / Dojo**
- a safe place to store your bot token temporarily

> Store bot tokens securely. Do not commit them to git or paste them into public chat.

## Quick Start Checklist

- [ ] Create a Telegram bot with **@BotFather**
- [ ] Disable **privacy mode** for that bot
- [ ] Create **ehAye Dojo (P)** and **ehAye Dojo (S)** groups
- [ ] Add the bot to both groups
- [ ] Send a normal message like `hello` in both groups
- [ ] Use `getUpdates` to find both chat IDs
- [ ] Make the bot an admin in both groups
- [ ] Lock down both groups so only admins can post (recommended)
- [ ] Open **ehAye Engine > Settings > Telegram**
- [ ] Enter the bot token and both chat IDs
- [ ] Press **Test** for each chat
- [ ] Send `hey dojo` in Telegram to confirm two-way routing


## Privacy, Security, and Group Visibility

### Groups can be private or public — both work

ehAye Engine does **not** block public groups. If your group has a public username (e.g. `@my_dojo_group`), Dojo will show a one-time informational notice and then work normally.

The group owner controls who joins and what members can do. ehAye respects that — it is not our job to gatekeep your group settings.

### Private vs Public Groups

|                            | Private Group               | Public Group                                             |
| -------------------------- | --------------------------- | -------------------------------------------------------- |
| **Discoverable by search** | ❌ No                       | ✅ Yes (via `@username`)                                 |
| **Join method**            | Invite link only            | Anyone can find and request to join                      |
| **Dojo works**             | ✅ Yes                      | ✅ Yes                                                   |
| **ehAye Engine blocks it** | No                          | No (informational notice only)                           |
| **Recommended for**        | Most users, maximum privacy | Teams that want easy invite links or read-only observers |

### What matters for security

The important question is not whether the group is discoverable, but **who can post** and **who can join**:

| Setting                              | What it controls                                             | Where to set it                                                         |
| ------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------------------- |
| **Group type** (private vs public)   | Whether the group is findable by username on Telegram        | Telegram → Group Settings → Group Type                                  |
| **Member permissions** (locked down) | Whether non-admin members can send messages, files, etc.     | Via `setChatPermissions` API or Telegram → Group Settings → Permissions |
| **Join requests**                    | Whether new members need admin approval to join              | Telegram → Group Settings → Group Type → toggle                         |
| **Bot privacy mode** (disabled)      | Whether the bot can see normal messages (not just /commands) | @BotFather → `/setprivacy` → Disable                                    |
| **Invite links**                     | Who has a link to join                                       | Telegram → Group Settings → Invite Links                                |

### Recommended setup

- Lock down member permissions so only **you** (owner) and the **bot** (admin) can post
- Anyone else you invite joins as a **read-only observer** — they can see messages but cannot send anything into Dojo
- This is useful if you want a colleague or manager to watch Dojo activity without interfering
- If using a public group, enable **join requests** so you approve each new member

### Who can do what in a locked-down group

| Role                         | Read messages | Send messages | Send files | Send voice notes | Can trigger Dojo | Can invite others |
| ---------------------------- | ------------- | ------------- | ---------- | ---------------- | ---------------- | ----------------- |
| **Owner** (you)              | ✅            | ✅            | ✅         | ✅               | ✅               | ✅                |
| **Admin** (the bot)          | ✅            | ✅            | ✅         | N/A              | N/A (bot only)   | ❌                |
| **Admin** (promoted human)   | ✅            | ✅            | ✅         | ✅               | ✅               | ✅                |
| **Regular member** (invited) | ✅            | ❌            | ❌         | ❌               | ❌               | ❌                |

### Who can do what in an open group (permissions NOT locked)

| Role                         | Read messages | Send messages | Send files | Send voice notes | Can trigger Dojo | Can invite others |
| ---------------------------- | ------------- | ------------- | ---------- | ---------------- | ---------------- | ----------------- |
| **Owner** (you)              | ✅            | ✅            | ✅         | ✅               | ✅               | ✅                |
| **Admin** (the bot)          | ✅            | ✅            | ✅         | N/A              | N/A (bot only)   | ❌                |
| **Regular member** (invited) | ✅            | ✅            | ✅         | ✅               | ⚠️ Possibly      | ✅                |

> ⚠️ In an open group, any member can send messages. Their messages will be visible to all group members. **Locking down permissions is strongly recommended.**

### How to invite someone as a read-only observer

1. Open the Telegram group
2. Tap the group name → **Add Members**
3. Search for the person and add them
4. They join as a **regular member** — no admin promotion needed
5. If permissions are locked down (Step 9 in setup), they can only read

They will see:

- All Dojo prompts and responses
- Voice notes and transcriptions
- File transfers
- Session summaries and notifications

They will NOT be able to:

- Send text into the group
- Send voice notes, photos, documents, or any files
- Trigger Dojo commands like `hey dojo` or `status`
- Change group settings or invite others

### How to give someone write access

1. Open the Telegram group
2. Tap the group name → **Edit** → **Administrators**
3. Add the person as an admin
4. They can now send messages that the bot will see

> Be aware: anything an admin types will be visible to the bot and may be picked up by Dojo as input.

### Security comparison by configuration

| Configuration                   | Discoverability    | Who can join                 | Who can read                     | Who can write       | Risk level | Recommendation                              |
| ------------------------------- | ------------------ | ---------------------------- | -------------------------------- | ------------------- | ---------- | ------------------------------------------- |
| Private + locked permissions    | Nobody can find it | Invite link only             | Owner + admins + invited members | Owner + admins only | 🟢 Lowest  | **Best for solo use**                       |
| Private + open permissions      | Nobody can find it | Invite link only             | All members                      | All members         | 🟡 Medium  | OK if all members are trusted               |
| Public + locked permissions     | Anyone can search  | Anyone (or via join request) | All members                      | Owner + admins only | 🟡 Medium  | **Good for read-only observers**            |
| Public + open permissions       | Anyone can search  | Anyone                       | All members                      | All members         | 🔴 High    | Not recommended — anyone who joins can post |
| Public + locked + join requests | Anyone can search  | Admin approval required      | Approved members                 | Owner + admins only | 🟢 Low     | **Best for public groups**                  |

### Why privacy mode must be disabled

This is the part people second-guess, so here is the important distinction:

- **Group visibility** (private vs public) controls who can find and join the group
- **Bot privacy mode** controls which messages the bot can see _inside_ the group

These are **independent settings**. A group can be private with privacy mode enabled (bot sees nothing useful), or public with privacy mode disabled (bot sees everything — which is what we need).

| Bot privacy mode        | Bot sees /commands | Bot sees normal text | Bot sees voice notes | Bot sees files | Dojo works          |
| ----------------------- | ------------------ | -------------------- | -------------------- | -------------- | ------------------- |
| **Enabled** (default)   | ✅                 | ❌                   | ❌                   | ❌             | ❌ Inbound broken   |
| **Disabled** (required) | ✅                 | ✅                   | ✅                   | ✅             | ✅ Fully functional |

For Dojo, the bot must receive more than slash commands. It needs to see:

- normal typed replies
- voice notes
- photos, documents, video, audio, video notes, animations

If privacy mode stays enabled, Telegram hides most group messages from the bot and Dojo will not receive normal replies. Outbound (Dojo → Telegram) still works, but inbound (Telegram → Dojo) will be silent.

**Rule: disable bot privacy mode. Group visibility is your choice.**


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

> If you disable privacy mode **after** the bot was already added to a group, remove the bot from the group and add it again. Telegram applies the new privacy setting only on re-join.

### Step 3: Create the Two Groups

Create two groups in Telegram:

- **ehAye Dojo (P)**
- **ehAye Dojo (S)**

| Group              | Purpose                                          |
| ------------------ | ------------------------------------------------ |
| **ehAye Dojo (P)** | Notifications and replies for the Primary Dojo   |
| **ehAye Dojo (S)** | Notifications and replies for the Secondary Dojo |

Private groups are recommended for most users. Public groups work too — see the Privacy section above.

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

Use a normal message, not just `/start`, because you want to confirm the bot can see ordinary group messages.

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

- disable privacy mode via @BotFather
- remove and re-add the bot to the group
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

After this step, the group member list should show two entries:

- **You** — owner
- **Your bot** — admin, "has access to messages"

This is the correct minimal setup.

### Step 8: Turn Off "All Members Are Administrators"

If Telegram shows this option in the group settings, turn it off.

If it remains on, Telegram may refuse to apply member restrictions in the next step.

### Step 9: Lock Down the Groups (Recommended)

Restrict all non-admin members from posting. This way only **you** and the **bot** can talk. Anyone else you invite later will be a read-only observer.

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

If the Engine detects a public group, it will show a one-time informational notice. Acknowledge it and proceed — this is not a blocker.

### Step 12: Test the Full Dojo Flow

After saving the settings:

1. Make sure Dojo is open and available
2. Send `hey dojo` in **ehAye Dojo (P)**
3. Dojo should wake and answer in the Primary group
4. Send plain text like `summarize what you are doing`
5. Send a voice note if you want to test transcription
6. Send a photo or document if you want to test file forwarding

If everything is working, your Telegram setup is complete.


## What Messages Does Dojo Understand in Telegram?

Send these as regular messages in the Dojo group.

| Message         | What it does                        |
| --------------- | ----------------------------------- |
| `hey dojo`      | Wakes Dojo                          |
| `status`        | Shows Dojo status for that group    |
| any normal text | Sends that text to Dojo             |
| voice note      | Transcribes and forwards it to Dojo |
| photo           | Downloads and forwards it to Dojo   |
| document        | Downloads and forwards it to Dojo   |
| video           | Downloads and forwards it to Dojo   |
| audio           | Downloads and forwards it to Dojo   |
| video note      | Downloads and forwards it to Dojo   |
| animation       | Downloads and forwards it to Dojo   |
| sticker / GIF   | Ignored silently                    |

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

| Group              | Dojo Role |
| ------------------ | --------- |
| **ehAye Dojo (P)** | Primary   |
| **ehAye Dojo (S)** | Secondary |


## Inviting Observers and Collaborators

You can invite other people to your Dojo Telegram groups. What they can do depends on their role and your group permissions.

### Use cases

| Scenario                            | Group type        | Permissions            | Invited as     | Result                                      |
| ----------------------------------- | ----------------- | ---------------------- | -------------- | ------------------------------------------- |
| Manager watches Dojo sessions       | Private or public | Locked                 | Regular member | Read-only — sees everything, can do nothing |
| Student observes a senior developer | Private or public | Locked                 | Regular member | Read-only — learns by watching              |
| Teammate follows a shared project   | Private           | Locked                 | Regular member | Read-only — stays in the loop               |
| Teammate actively collaborates      | Private           | Locked                 | Admin          | Full access — can send messages to Dojo     |
| Company demo group                  | Public            | Locked + join requests | Regular member | Read-only — anyone approved can watch       |

### How to invite someone

1. Open the Telegram group
2. Tap the group name → **Add Members**
3. Search for the person and add them
4. They join as a **regular member** with permissions determined by group settings
5. No admin promotion needed for read-only access

### Promoting to active participant

If you want someone to be able to send messages into Dojo:

1. Promote them to **admin** in the Telegram group
2. They can now type messages that the bot will see
3. Be aware: anything they send may be picked up by Dojo as input

### Revoking access

- To remove someone: open the group → tap their name → **Remove from Group**
- To demote an admin back to read-only: open the group → **Administrators** → remove their admin role
- If using invite links: revoke old links in **Group Settings → Invite Links** to prevent re-joining

See the **Security comparison by configuration** table in the Privacy section above for a full breakdown of what each setup allows.


## Troubleshooting

| Problem                                          | Cause                                                              | Fix                                                                                     |
| ------------------------------------------------ | ------------------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| Bot does not see normal group messages           | Privacy mode still enabled                                         | Disable privacy mode via @BotFather, then remove and re-add the bot                     |
| `getUpdates` returns empty                       | No non-command message has been seen yet                           | Send `hello` in the group, then retry                                                   |
| `getUpdates` still empty after disabling privacy | Bot was added before privacy mode was disabled                     | Remove and re-add the bot                                                               |
| `not enough rights` during lockdown              | Bot is missing admin rights                                        | Make bot admin and enable **Change Group Info**                                         |
| Messages do not arrive on your phone             | Wrong chat ID or bot is not in the group                           | Re-check `getUpdates` and group membership                                              |
| `hey dojo` does nothing                          | Wrong role mapping, Telegram not enabled, or Dojo is not available | Re-check Primary/Secondary chat IDs, the Enabled toggle, and that Dojo is open          |
| Voice note is received but not understood        | Transcription failed or wake-word rules blocked it                 | Try a clearer recording and test with text first                                        |
| File upload is rejected                          | File download failed or the file is too large                      | Retry with a smaller file; large Telegram files may be rejected                         |
| Test button returns an error                     | Bot token or chat ID is wrong or the bot is not in the group       | Verify the bot token with `getMe` and re-check group membership                         |
| Poll timeout errors in logs                      | Normal — long-poll connection expired and reconnects               | These are expected; Dojo retries automatically after 5 seconds                          |
| "Public Group Notice" dialog appears             | Group has a public username                                        | Acknowledge it and proceed — this is informational, not a blocker                       |
| Outbound works but inbound does not              | Privacy mode is ON or bot was added before disabling it            | Disable privacy mode, remove and re-add the bot, send `hello`, verify with `getUpdates` |

## Security Recommendations

### Required

- [ ] Bot privacy mode is **disabled** (without this, inbound messages will not work)
- [ ] Bot is **admin** in both groups (needed to send messages and read group info)
- [ ] Bot token is stored securely and never committed to git

### Strongly recommended

- [ ] Member posting permissions are **locked down** (only owner and admins can post)
- [ ] Only you and the bot are admins (minimize who can write into Dojo)
- [ ] "All Members Are Administrators" is turned **off** (otherwise permissions cannot be restricted)

### Recommended for public groups

- [ ] **Join requests** are enabled (new members need your approval)
- [ ] No unnecessary invite links are active (revoke old ones in Group Settings → Invite Links)

### Optional

- [ ] Group visibility is set to **private** (maximum privacy, but not required)
- [ ] A **PIN** is configured in ehAye Engine Telegram settings (adds a layer of authentication for Telegram commands)
- [ ] A **wake word** is configured (controls which voice notes are forwarded to Dojo)

### How ehAye Engine handles security

- If a public group is detected, ehAye Engine shows a **one-time informational notice** — it does not block you
- If bot privacy mode is still enabled, inbound messages will silently fail (Telegram hides them from the bot)
- The bot only processes messages from your configured groups — messages from unknown chats are ignored

> **Philosophy:** ehAye Engine informs you about your security posture but never blocks you from using your own groups. The group owner is responsible for access control. We warn once, then respect your decision.


## Optional: Telegram Setup for ehAye Portal

The Portal uses Telegram differently from Dojo.

### ehAye Portal (@ehaye_alerts_bot)

Three groups, one bot:

| Group         | Purpose                                                        |
| ------------- | -------------------------------------------------------------- |
| **ehAye Biz** | Purchases, invoices, refunds, subscriptions, disputes          |
| **ehAye Ops** | Signups, logins, downloads, trial activity, license activation |
| **ehAye Sys** | System errors, API warnings, DB warnings, security alerts      |

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

| Action                 | Method | Endpoint              |
| ---------------------- | ------ | --------------------- |
| Verify the bot         | GET    | `/getMe`              |
| Read incoming messages | GET    | `/getUpdates`         |
| Send a message         | POST   | `/sendMessage`        |
| Read group info        | GET    | `/getChat?chat_id=X`  |
| Lock down a group      | POST   | `/setChatPermissions` |
| Download file metadata | GET    | `/getFile?file_id=X`  |

## BotFather Commands

| Command           | Purpose                     |
| ----------------- | --------------------------- |
| `/newbot`         | Create a bot                |
| `/mybots`         | Manage existing bots        |
| `/setprivacy`     | Change group privacy mode   |
| `/setdescription` | Set bot description         |
| `/setabouttext`   | Set bot about text          |
| `/setuserpic`     | Set bot profile image       |
| `/revoke`         | Revoke and regenerate token |

> **Creator:** ehAye
> **License:** MIT
