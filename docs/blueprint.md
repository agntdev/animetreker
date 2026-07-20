# AnimeTracker — Bot specification

**Archetype:** custom

**Voice:** warm and concise — write every user-facing message, button label, error, and empty state in this voice.

AnimeTracker — персональный Telegram-бот для отслеживания прогресса просмотра аниме. Позволяет быстро отмечать просмотренные серии, хранить историю просмотров, отображать прогресс по сезонам и получать уведомления о новых сериях.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- one user (owner) in private chat

## Success criteria

- User can add and track anime progress
- User can view history of changes
- User can receive optional notifications about new episodes

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **Добавить аниме** (button, actor: user, callback: anime:add) — Initiates adding a new anime to the tracker
- **/list** (command, actor: user, command: /list) — Show list of tracked anime
- **Мои аниме** (button, actor: user, callback: list:show) — Show list of tracked anime
- **Настройки** (button, actor: user, callback: settings:show) — Open settings menu for notifications and preferences

## Flows

### onboarding
_Trigger:_ /start

1. Welcome message
2. Show quick instructions
3. Offer to add first anime

_Data touched:_ Anime

### add_anime
_Trigger:_ anime:add

1. Request anime title
2. Ask for season and episode (default season 1, episode 0)
3. Confirm and save

_Data touched:_ Anime, Season, Episode progress

### quick_update
_Trigger:_ anime:quick_update

1. +1 episode
2. Save progress
3. Show updated progress

_Data touched:_ Episode progress, Watch history

### manual_edit
_Trigger:_ anime:edit

1. Request new episode number
2. Option to add note
3. Save changes

_Data touched:_ Episode progress, Watch history

### view_history
_Trigger:_ anime:view_history

1. Show last 20 changes with timestamps
2. Option to view full history

_Data touched:_ Watch history

### notification_settings
_Trigger:_ settings:notifications

1. Enable/disable notifications
2. Select anime for notifications

_Data touched:_ User settings

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Anime** _(retention: persistent)_ — Tracked anime title and metadata
  - fields: title, season, episode
- **WatchHistory** _(retention: persistent)_ — Record of episode changes with timestamps
  - fields: anime_title, season, episode, timestamp, note
- **UserSettings** _(retention: persistent)_ — User preferences including notification settings
  - fields: notifications_enabled, notification_anime_list

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add/remove anime
- Edit episode progress
- View history
- Enable/disable notifications

## Notifications

- Scheduled push notifications for new episodes (optional)

## Permissions & privacy

- Data is private and only accessible by the owner
- No third-party data sharing

## Edge cases

- User tries to add duplicate anime
- User tries to set episode number higher than season length
- User tries to edit non-existent anime

## Required tests

- Verify adding and tracking anime progress
- Verify history recording and display
- Verify notification settings functionality

## Assumptions

- User will manually input anime titles
- Notifications are disabled by default
- Season 1, episode 0 is default for new anime
