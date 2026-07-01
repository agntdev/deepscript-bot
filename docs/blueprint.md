# ScriptLink Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that delivers pre-configured text scripts to users via deep-links. Users get a deep-link on /start, and tapping it delivers the corresponding script. Admins can manage the script-payload mappings in a private admin chat.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Non-technical end users
- Admin users

## Success criteria

- Users can receive scripts by tapping deep-links
- Admins can create, update, and delete script-payload mappings
- Bot persists scripts and admin configuration across restarts

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and show the default deep-link
- **/addscript** (command, actor: admin, command: /addscript) — Start the interactive flow to add a new script-payload mapping
- **/listscripts** (command, actor: admin, command: /listscripts) — List all configured script-payload mappings
- **/delscript** (command, actor: admin, command: /delscript) — Delete an existing script-payload mapping
- **/editscript** (command, actor: admin, command: /editscript) — Edit an existing script-payload mapping

## Flows

### First interaction
_Trigger:_ /start

1. User sends /start
2. Bot replies with message containing deep-link button and instructions

_Data touched:_ User, Default payload

### Deep-link activation
_Trigger:_ /start <payload>

1. User opens deep-link
2. Bot checks payload
3. If payload known: send corresponding script
4. If payload unknown: send error and default deep-link

_Data touched:_ Deep-link, Script

### Admin add script
_Trigger:_ /addscript

1. Admin sends /addscript
2. Bot asks for payload
3. Admin provides payload
4. Bot asks for script text
5. Admin provides script text
6. Bot confirms script added

_Data touched:_ Script, Admin

### Admin list scripts
_Trigger:_ /listscripts

1. Admin sends /listscripts
2. Bot lists all script-payload mappings

_Data touched:_ Script

### Admin delete script
_Trigger:_ /delscript

1. Admin sends /delscript with payload
2. Bot confirms deletion
3. Admin confirms deletion
4. Bot deletes script

_Data touched:_ Script, Admin

### Admin edit script
_Trigger:_ /editscript

1. Admin sends /editscript with payload
2. Bot asks for new script text
3. Admin provides new script text
4. Bot confirms script updated

_Data touched:_ Script, Admin

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user interacting with the bot
  - fields: Telegram user id, Language, Last interaction
- **Deep-link (payload)** _(retention: persistent)_ — Short token in the deep-link that identifies which script to deliver
  - fields: Payload, Script ID
- **Script** _(retention: persistent)_ — Text content sent to the user when they open a deep-link
  - fields: Payload, Title, Description, Text content, Created by, Created at, Use count
- **Admin** _(retention: persistent)_ — Privileged user who can manage scripts and their payloads
  - fields: Admin chat ID, Notification on/off

## Integrations

- **Telegram** (required) — Bot API messaging and deep-link handling
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Set default payload for /start
- Configure admin chat ID
- Enable/disable notifications for deep-link uses

## Notifications

- Admin receives notifications when a deep-link is used (configurable)

## Permissions & privacy

- Only admin can manage scripts
- User data is stored minimally (id, language, last interaction)

## Edge cases

- Unknown payload in deep-link
- Admin tries to edit/delete non-existent script
- User opens deep-link with no corresponding script

## Required tests

- Verify deep-link delivery on /start
- Verify script delivery for known payload
- Verify error handling for unknown payload
- Verify admin can add/edit/delete scripts
- Verify admin notifications when enabled

## Assumptions

- Default payload is 'default'
- Script content is plain text (UTF-8)
- Admin authentication is based on a single configured admin ID
- Notifications are disabled by default
