<div align="center">

# Qssentials

**An all-in-one server toolkit for Minecraft Bedrock Edition.**
Economy, homes, warps, TPA, land claims, daily/weekly rewards, scorehud, ranks, and Custom UI, built on the Script API.

![Minecraft Bedrock](https://img.shields.io/badge/Minecraft-Bedrock%201.21.100%2B-62B47A?logo=minecraft&logoColor=white)
![Version](https://img.shields.io/badge/version-2.0.0-blue)
![Script API](https://img.shields.io/badge/Script%20API-beta-orange)
![Languages](https://img.shields.io/badge/languages-EN%20%7C%20ID-informational)
![License](https://img.shields.io/badge/license-All%20Rights%20Reserved-red)

[Features](#features) · [Installation](#installation) · [Commands](#commands) · [Configuration](#main-configuration) · [Developer Guide](#developer-guide) · [Troubleshooting](#troubleshooting)

Bahasa Indonesia: [README.id.md](README.id.md)

</div>

---

## Table of Contents

1. [Features](#features)
2. [Requirements](#requirements)
3. [Installation](#installation)
4. [Quick Start](#quick-start)
5. [Commands](#commands)
6. [Ranks and Permissions](#ranks-and-permissions)
7. [Main Configuration](#main-configuration)
8. [Per-Module Configuration](#per-module-configuration)
9. [Feature Guides](#feature-guides)
10. [Resource Pack and UI](#resource-pack-and-ui)
11. [ScriptEvents and Integrations](#scriptevents-and-integrations)
12. [Developer Guide](#developer-guide)
13. [Troubleshooting](#troubleshooting)
14. [Known Issues](#known-issues)
15. [Changelog](#changelog)
16. [Contributing](#contributing)
17. [Credits and License](#credits-and-license)

---

## Features

| Area | What you get |
|---|---|
| **Teleportation** | `/home`, `/sethome`, `/delhome`, `/warp`, `/setwarp`, `/tpa`, `/rtp`, `/back` (return to your last death location). Teleports are cancelled if the player moves during the countdown. |
| **Economy** | `money` scoreboard balance, `/pay`, `/topmoney`, `/setmoney`. |
| **Land claims** | Buy land by clicking two corners. Protection against breaking, interacting, explosions, and farmland trampling. |
| **Rewards** | `/daily` (one reward per weekday) and `/weekly` (`free`, `rare`, `mythical` tiers) with item, currency, and random prizes. |
| **Scorehud** | On-screen info board with a configurable template and many placeholders. |
| **Ranks and chat** | Tag-based ranks, custom chat format and nametag, chat cooldown. |
| **Server systems** | ClearLag and ClearChat timers, auto broadcast, lobby protection, one-click NPC commands. |
| **Custom UI** | Home, Warp, Ranklist, Rules, confirmation dialogs, toast notifications, and more. |
| **Extensible** | Simple `Command.register` API, plugin registration via ScriptEvent, small key-value database. |

---

## Requirements

- Minecraft Bedrock **1.21.100** or newer.
- The **Beta APIs** experiment enabled (the manifest depends on beta script modules).
- **Both** the Behavior Pack and the Resource Pack active (the BP depends on the RP).
- **Cheats enabled** while `command_required_cheats` is `true` (the default). See [Main Configuration](#main-configuration).
- Script modules used: `@minecraft/server` (beta), `@minecraft/server-ui` (beta), `@minecraft/server-admin` (1.0.0-beta).

> [!NOTE]
> `@minecraft/server-admin` (used by `/goto`) is generally only available on Bedrock Dedicated Server. Behavior in a local single-player world may differ from a server.

---

## Installation

### Option 1: Release file (recommended)

1. Download the latest `.mcaddon` from the **Releases** page.
2. Open it with Minecraft. `Qssentials [BP]` and `Qssentials [RP]` are imported automatically.
3. Create or edit a world and activate **Qssentials** under both **Behavior Packs** and **Resource Packs**.
4. Under **Experiments**, enable **Beta APIs**.
5. Enable **Allow Cheats** (or set `command_required_cheats` to `false`).
6. Enter the world. The message `[Qssentials] >> World Has Been Load...` confirms the addon is running.

### Option 2: Dedicated server (BDS)

1. Extract the `.mcaddon` (it is a ZIP archive).
2. Copy `Qssentials [BP]` to `development_behavior_packs/` and `Qssentials [RP]` to `development_resource_packs/`.
3. Register both packs in your world's `world_behavior_packs.json` and `world_resource_packs.json` (the UUIDs are in each `manifest.json`).
4. Enable the beta APIs for the world, then restart the server.

### Option 3: Build from source

Clone the repository, select the `Qssentials [BP]` and `Qssentials [RP]` folders, compress them into one ZIP, and rename the file extension to `.mcaddon`.

```text
Qssentials/
├─ Qssentials [BP]/
└─ Qssentials [RP]/
```

> [!TIP]
> After editing `config.js` or any `ess.config.js`, save the file and **reload the world or restart the server** for changes to take effect.

---

## Quick Start

1. **Set the owners.** Open `Qssentials [BP]/scripts/config.js` and list the exact gamertags (case-sensitive):

   ```js
   owners: ["YourGamertag", "AnotherOwner"],
   ```

2. **Give someone admin.** A player counts as *Admin* if they are an Owner, an Operator (`/op`), or have the `Admin` tag:

   ```mcfunction
   /tag PlayerName add Admin
   ```

3. **Set your rank-purchase link** (`buy_rank_link`), which is used by the contact button in `/ranklist`.
4. **Configure the lobby area** if you use one (see [Lobby Protect](#lobby-protect)).
5. **Create your first warp.** Stand at the destination and run `/setwarp spawn`.

On first run the addon creates the `money`, `kills`, and `deaths` scoreboard objectives automatically.

---

## Commands

All commands are registered under the `es:` namespace, for example `/es:home`. In the tables below the `es:` prefix is omitted for brevity; type the full `/es:name` form if a command conflicts with another addon. The namespace can be changed with `config.namespace`.

Notation: `<param>` is required, `[param]` is optional.

**Access levels**

| Level | Meaning |
|---|---|
| Everyone | Any player |
| Tag | Requires a specific player tag |
| Admin | Owner, Operator, or a player with the `Admin` tag |
| Owner | A gamertag listed in `config.owners` |

### General

| Command | Parameters | Access | Description |
|---|---|---|---|
| `/help` | `[category]` | Everyone | Lists commands. Categories: `general`, `tpa`, `home`, `mode`, `economy`, `owner`. |
| `/about` | | Everyone | Shows namespace, version, and contributors. |
| `/plugins` (aliases `/pl`, `/plugin`) | `[pluginName]` | Admin* | Lists installed plugins. *Admin-only while `hide_plugin_command_for_member` is `true`. |
| `/rules` | | Everyone | Opens the rules UI (from `config.rules`). |
| `/news` | | Everyone | Shows the news lines (from `config.news`). |
| `/ranklist` | | Everyone | Opens the rank list UI. |
| `/ping` | | Everyone | Shows your ping. |
| `/tps` | | Everyone | Estimates server TPS. |
| `/iditem` | | Everyone | Shows the ID of the item in your hand. |
| `/clearchat` | | Everyone | Clears your own chat (30 s cooldown). |
| `/scorehud` | | Everyone | Toggles your personal scorehud. |
| `/goto` | `<ip-address> [port]` | Everyone | Transfers you to another server (default port `19132`). |
| `/rtp` | `[dimension]` | Everyone | Random teleport. Dimensions: `overworld`, `nether`, `the_end`. |
| `/back` | | Everyone | Returns to the location where you last died. |
| `/feed` | | Tag `feed.access` or `dev` | Restores hunger (10 s cooldown). |
| `/heal` | | Tag `heal.access` or `dev` | Restores health (30 s cooldown). |
| `/fly` | | Tag `fly.access` | Enables `mayfly`. |

### Homes, Warps and TPA

| Command | Parameters | Access | Description |
|---|---|---|---|
| `/home` | `[homeName]` | Everyone | No argument opens the home list UI. With a name, teleports directly. |
| `/sethome` | `[homeName]` | Everyone | Saves a home at your position. No argument opens a form. |
| `/delhome` | `[homeName]` | Everyone | Deletes a home. No argument opens a UI with confirmation. |
| `/warp` | `[warpName]` | Everyone | No argument opens the warp UI (sortable A-Z or by distance). |
| `/setwarp` | `<warpName>` | Admin | Saves a warp at your position (name is lowercased). |
| `/tpa` | `<TargetPlayer>` | Everyone | Sends a teleport request to another player. |
| `/tpaaccept` | `[player]` | Everyone | Accepts a request (the sender is teleported to you). |
| `/tpadeny` | | Everyone | Denies a request. |

### Land

| Command | Parameters | Access | Description |
|---|---|---|---|
| `/land here` | | Everyone | Checks whether your position is inside someone's land. |
| `/land create` | | Everyone | Starts a purchase: click block 1, click block 2, then confirm. |
| `/land list` | | Everyone | Lists your lands (ID, center coordinates, dimension). |
| `/land sell` | `[id]` | Everyone | Sells a land. See [Known Issues](#known-issues) regarding ID auto-detection. |

### Economy

| Command | Parameters | Access | Description |
|---|---|---|---|
| `/money` (alias `/balance`) | | Everyone | Shows your balance. |
| `/pay` | `<TargetPlayer> <Amount>` | Everyone | Sends money (5 s cooldown, minimum from `pay.min`). |
| `/topmoney` | | Everyone | Balance leaderboard. |
| `/setmoney` (alias `/setbalance`) | `<Amount> [TargetPlayer]` | Admin | Sets a balance. |

### Rewards

| Command | Parameters | Access | Description |
|---|---|---|---|
| `/daily` | | Everyone | Claims today's reward (24 h cooldown). |
| `/weekly` | `[free\|rare\|mythical]` | Everyone / Tag | Claims a weekly reward (7-day cooldown per tier). `rare` needs tag `weekly.rare`, `mythical` needs tag `weekly.myth`. |

### Game Modes

| Command | Parameters | Access | Description |
|---|---|---|---|
| `/gms` | | Everyone | Survival mode. |
| `/gmc` | | Admin | Creative mode. |
| `/spectator` | | Admin | Spectator mode. |
| `/invis` (alias `/ghost`) | | Tag `mvp`, `admin`, `dev`, or `owner` | Toggles invisibility. |

### Admin

| Command | Parameters | Access | Description |
|---|---|---|---|
| `/ctitle` | `<title-text> [subtitle-text] [broadcast]` | Admin | Shows a custom title. `\n` is a line break, `@s` is replaced by the player name. `broadcast=true` sends it to everyone. |
| `/invsee` | `<target>` | Admin | Opens a chest-style view of another player's inventory. |
| `/setscorehud` | `[title] [body] [footer]` | Admin | Overrides the scorehud template. Use `@ln` for line breaks. Leave `title`/`body` empty to reset to the config template. |
| `/cdialog` | | Admin | **Disabled by default.** Requires the `DynamicDialogV2` plugin. |

### Owner

| Command | Parameters | Access | Description |
|---|---|---|---|
| `/sethomelimit` | `[limit] [targetName]` | Owner | Sets a player's home limit. `-1` means unlimited. |
| `/clearrank` | `[Target]` | Owner | Removes all rank tags from the target (or yourself). |
| `/dailyreset` | | Owner | Resets your own daily cooldown (for testing). |
| `/weeklyreset` | | Owner | Resets your own weekly cooldown (for testing). |
| `/notif` | `[title] [message] [icon] [background] [broadcast]` | Owner | Sends a custom toast notification. `icon` is a texture path or `aux:<id>`. |

---

## Ranks and Permissions

Qssentials uses **player tags** for both ranks and permissions.

### Assigning a rank (chat and nametag display)

A rank is read from a tag that starts with `rank:` (configurable via `rank_prefix`). If a player has several, the last one is used.

```mcfunction
/tag PlayerName add rank:VIP
/tag PlayerName add "rank:§bVIP"
```

- The text after `rank:` is displayed as written (color codes with `§` are allowed) and wrapped in brackets, such as `[VIP]`.
- If the rank is a glyph character (for example from `font/glyph_E*.png`), it is displayed without brackets.
- Players without a rank tag get `rank_default`.
- Remove ranks with `/clearrank <player>` (Owner).

> [!NOTE]
> The `ranklist` entry in `config.js` only controls what the `/ranklist` UI **displays**. Command access is controlled by the tags below.

### Permission tags

| Tag | Effect |
|---|---|
| `Admin` | Treated as Admin for Qssentials features. |
| `feed.access` | May use `/feed`. |
| `heal.access` | May use `/heal`. |
| `fly.access` | May use `/fly`. |
| `mvp` | May use `/invis`. |
| `dev` | May use `/feed`, `/heal`, and `/invis`. |
| `weekly.rare` | May claim `/weekly rare`. |
| `weekly.myth` | May claim `/weekly mythical`. |
| `disable:scorehud` | Added automatically when a player turns their scorehud off. |
| `daily.cd.*`, `weekly.cd.*` | Added automatically to store reward cooldowns. Do not remove manually. |

Example of a VIP-style setup:

```mcfunction
/tag PlayerName add rank:VIP
/tag PlayerName add feed.access
/tag PlayerName add heal.access
/tag PlayerName add fly.access
```

---

## Main Configuration

Location: `Qssentials [BP]/scripts/config.js`

### General Options

| Option | Default | Description |
|---|---|---|
| `namespace` | `"es"` | Command prefix (`/es:home`). |
| `language` | `"en"` | Default language (currently unused). |
| `owners` | n/a | List of Owner gamertags. |
| `command_required_cheats` | `true` | When `true`, commands only run if cheats are enabled. Set to `false` to run without cheats. |
| `rank_prefix` | `"rank:"` | Prefix of rank tags. |
| `rank_default` | `"§7"` | Display value for players without a rank tag. |
| `buy_rank_link` | n/a | URL opened by the contact button in `/ranklist`. |
| `ranklist` | n/a | Content of the `/ranklist` UI: `{ NAME: { icon, feature: [...] } }`. |
| `news` | `[...]` | Array of strings for `/news` (one element per line). |
| `hide_plugin_command_for_member` | `true` | `true` restricts `/plugins` to Admin. |
| `teleport_show_dynamic_countdown` | `true` | Show the teleport countdown in the actionbar. |
| `pay.min` | `1000` | Minimum `/pay` amount. Clamped to the range **500 to 20000**. |
| `say_rank_on_message` | `false` | Not used in this build. |

### Chat and Nametag

```js
chat: {
  message_max_length: 500,   // maximum message length
  cooldown: 3,               // seconds between messages
  message_template: "@rank @player_name§r @faction §7§l>§r @msg"
},
nametag: {
  template: "@rank§r @faction §r@player_name",
  enabled: true              // false keeps the vanilla nametag
},
```

| Placeholder | Chat | Nametag |
|---|:--:|:--:|
| `@rank` | yes | yes |
| `@player_name` | yes | yes |
| `@faction` | yes | yes |
| `@money` | yes | yes |
| `@platform` | no | yes |
| `@msg` | yes | no |

> [!NOTE]
> There are no faction commands in this build. `faction.color` only affects how `@faction` is rendered; `min`, `max`, `maximum_member`, and `enabled` are not used yet.

### Scorehud

```js
scorehud: {
  enable_by_default: true,
  template_default: `textures/ui/title|@ln§l§6info§r@ln§arank: @rank|QssentialsV2`
}
```

`template_default` format: **`TITLE|BODY|WATERMARK`**

- `TITLE`: plain text, or a texture path (for example `textures/ui/title`).
- `BODY`: the board content. Use `@ln` for new lines and `§` for colors.
- `WATERMARK`: small footer text.

Placeholders available in `BODY`:

`@player_name` `@rank` `@player_gamemode` `@platform` `@faction` `@player_velocity` `@x` `@y` `@z` `@player_dimension` `@money` `@moneyr` (short form, e.g. `1.50k`) `@kills` `@deaths` `@land_here` (ID of the land you stand in) `@durability_mainhand_current` `@durability_mainhand_max` `@ln` (new line) `@n`

Change it at runtime (stored in the database and overrides `template_default`):

```mcfunction
/setscorehud "textures/ui/title" "@ln§6Hello @player_name@lnMoney: @moneyr" "MyServer"
```

Run `/setscorehud` with no arguments to go back to the config template. Players can hide their own scorehud with `/scorehud`.

### ClearLag and ClearChat

```js
clear: {
  announcement: [300, 180, 60, 10, 5, 3, 2, 1], // seconds before each cleanup
  wait: 20,                                      // pause between cycles (seconds)
  chat: true,                                    // also clear chat
  actived: true                                  // false disables the system
},
```

- The countdown starts from the first value (`300` s = 5 minutes) and announces at every listed second.
- Removed entities: **items** and **arrows** in every dimension.
- With `chat: true`, cycles alternate between an entity cleanup and a chat cleanup.

### RTP

```js
rtp: {
  radius: 2000,   // maximum distance from 0,0
  countdown: 5,   // seconds before teleport
  cooldown: 60    // seconds between uses
},
```

While a location is being searched, the player's movement is locked and progress is shown in the actionbar. If the player leaves mid-process, they are returned to their original position on rejoin.

### Lobby Protect

```js
lobby_protect: {
  enabled: true,
  pos: [{ x: 5, z: 5 }, { x: -5, z: -5 }],  // two corners (X and Z axes)
  allowInteractNpc: true,
  disable_pvp: true
},
```

- The lobby applies in the **Overworld**, at every Y level.
- Regular players cannot break blocks or interact inside it. Admins and Owners are exempt.
- Land cannot be purchased inside the lobby area.
- `allowInteractNpc` and `disable_pvp` are not used in this build. NPC interaction is controlled by `allowedInteractEntities` in [Land Settings](#land-settings).

### One-Click NPC Interaction

Tag an NPC (or any entity) so that clicking or hitting it runs a command:

```js
interaction: {
  "npc.daily": { command: "daily", hit: true,  interact: true },
  "npc.rtp":   { command: "rtp",   hit: false, interact: true },
  // add your own:
  "npc.shop":  { command: "warp shop", hit: false, interact: true }
},
```

| Key | Purpose |
|---|---|
| the key (`npc.daily`) | The tag the entity must have. |
| `command` | Command (no leading `/`) executed by the player. |
| `hit` | Trigger when the entity is hit. |
| `interact` | Trigger when the entity is interacted with. |

Tag the nearest NPC:

```mcfunction
/tag @e[type=npc,r=3,c=1] add npc.daily
```

Built-in tags: `npc.daily`, `npc.weekly`, `npc.rtp`, `npc.rules`, `npc.news`, `npc.warp`.

### Auto Broadcast

```js
broadcast: {
  title: "§cbroadcast",
  auto_message: {
    enabled: true,
    delay_per_index: 250,   // seconds between messages
    message: [
      "dont forgot your daily reward, type §e/daily",
      "§aonline now @online/@max_players"
    ]
  }
}
```

- Messages are sent in order, looping back to the first after the last.
- `@online` is the current online player count. `@max_players` is the number of players recorded in the addon database.
- `cooldown`, `price_limit`, `message_max_length`, and `limit_*` under `broadcast` are not used in this build.

### Rules

```js
rules: [
  { text: "Opening text shown as the form body", texture: "" },
  { text: "GRIEF§c§c§6§t§t\n§r§7No property destruction", texture: "textures/ui/icon_recipe_construction" },
],
```

- The **first** element becomes the form body.
- Following elements are shown as rule rows with the icon from `texture`.
- `texture` is truncated to 200 characters. Leave it empty for no icon.

### Daily and Weekly Rewards

#### Daily

Keys `0` to `6` follow `Date.getDay()` using the **server clock**: `0 = Sunday`, `1 = Monday`, `2 = Tuesday`, `3 = Wednesday`, `4 = Thursday`, `5 = Friday`, `6 = Saturday`. Each player can claim once per day (24 h cooldown).

```js
daily: {
  1: [{ type: "currency", value: 2000 }],
  2: [{ type: "item", item_type: "minecraft:iron_sword", item_name: "§bDaily Sword",
        enchants: [{ id: "sharpness", level: 2 }] }],
  // ...
},
```

> [!WARNING]
> The comments inside the default `config.js` describe `0` as Monday. The code uses `getDay()`, where `0` is **Sunday**.

#### Weekly

Each tier has a `prize` list and an optional `permission` (the tag required). Cooldown is 7 days per tier. Any tier you add to `weekly` automatically becomes an option of `/weekly`.

```js
weekly: {
  free:     { prize: [ /* ... */ ] },
  rare:     { permission: "weekly.rare", prize: [ /* ... */ ] },
  mythical: { permission: "weekly.myth", prize: [ /* ... */ ] }
}
```

#### Prize format

| `type` | Properties | Description |
|---|---|---|
| `"item"` | `item_type`, `amount`, `item_name`, `enchants` | Gives an item. |
| `"currency"` | `value` | Adds money. |
| `"random"` | `item: [ ...prizes ]` | Picks **one** prize at random from the list. |

Value rules:

- `amount` / `value`: a fixed number (`5`) or a random range `[min, max]` (`[1, 3]`). `amount` defaults to `1`.
- `item_type`: a single item ID or an array of IDs (one is picked at random).
- `enchants`: an array of
  - `{ id: "sharpness", level: 2 }`: a specific enchantment,
  - `{ id: "efficiency", level: [1, "max"] }`: random level up to the maximum,
  - `{ id: "random", level: [1, 3] }`: random enchantment with a random level,
  - `"random"`: a fully random enchantment (no duplicates on one item).

Full example:

```js
{
  type: "random",
  item: [
    { type: "currency", value: [1000, 5000] },
    {
      type: "item",
      item_type: ["minecraft:iron_pickaxe", "minecraft:diamond_pickaxe"],
      item_name: "§bSpecial Pickaxe",
      enchants: [{ id: "efficiency", level: [1, 2] }, "random"]
    }
  ]
}
```

---

## Per-Module Configuration

Some features have their own `ess.config.js` inside `Qssentials [BP]/scripts/commands/<module>/`.

### Home Settings

`commands/home/ess.config.js`

| Option | Default | Description |
|---|---|---|
| `home.min` | `3` | Minimum home name length. |
| `home.max` | `20` | Maximum home name length. |
| `home.limit_default` | `1` | Home limit for regular players (`0` to `99`). |
| `home.limit_admin` | `10` | Home limit for Admins. |
| `home.limit_owner` | `'unli'` | Home limit for Owners (`'unli'` means unlimited). |

Per-player limits can be overridden with `/sethomelimit <limit> <name>` (Owner).

### Land Settings

`commands/land/ess.config.js`

| Option | Default | Description |
|---|---|---|
| `land.pricePerBlock` | `500` | Price per block. Clamped to **100 to 20000**. |
| `land.percentSell` | `20` | Percentage of the purchase price refunded when selling (`1` to `100`). |
| `land.minimumSize` | `5` | Minimum land size in blocks (`0` to `2000`). |
| `land.limitLandperPlayer` | `null` | Maximum lands per player. `null` means no limit. |
| `land.cooldown_message_farm_trample` | `1` | Seconds between farmland warning messages. |
| `land.allowedInteractEntities` | `['minecraft:npc']` | Entities that can still be interacted with inside other players' lands and the lobby. |

### TPA Settings

`commands/tpa/ess.config.js`

| Option | Default | Description |
|---|---|---|
| `countdown` | `5` | Teleport countdown after acceptance (seconds). |
| `expiredTime` | `20` | How long a request stays valid (seconds). |

### Warp Settings

`commands/warp/ess.config.js` contains `cooldown` and `countdown`. In this build `warp.js` does not read that file; the warp countdown uses the built-in default of 3 seconds.

---

## Feature Guides

### Homes

1. `/sethome house` saves your location (or run `/sethome` for the form).
2. `/home` opens the home list UI with coordinates and dimension, or `/home house` teleports directly.
3. `/delhome` opens the delete UI with a confirmation step.

Home names are stored in lowercase.

### Warps

1. An admin creates warps: `/setwarp spawn`, `/setwarp market`.
2. Players open `/warp` to see a list with dimension labels and distances, sortable A-Z or nearest first. Select a warp, then **TELEPORT**.
3. Or teleport directly: `/warp spawn`.

### TPA

1. Player A: `/tpa B`.
2. Player B: `/tpaaccept` (or `/tpaaccept A`) to accept, `/tpadeny` to refuse.
3. After acceptance, A is teleported to B once the countdown ends. Requests expire after `expiredTime`.

> [!NOTE]
> Every teleport that uses a countdown (home, warp, back, tpa, rtp) is **cancelled if the player moves**.

### Claiming Land

1. Make sure you can afford it: price = **area x `pricePerBlock`**.
2. Run `/land create`, then **click a block** for the first corner and **click a block** for the second. The land is a rectangle on the X and Z axes and covers every Y level.
3. Confirm the purchase in the form. The land receives a 4-digit ID.
4. Use `/land list` to see your lands and `/land sell <id>` to sell one back for `percentSell`% of the price.

Protection inside other players' lands (and the lobby):

- Blocks cannot be broken.
- Blocks and entities cannot be interacted with (except entities in `allowedInteractEntities`).
- Non-player entities (such as livestock) cannot be attacked.
- Explosions do not damage blocks inside the land.
- Farmland cannot be trampled.

### Money

Balances are stored in the `money` scoreboard. Besides `/pay`, other addons can change balances through [ScriptEvents](#scriptevents-and-integrations). `/daily` and `/weekly` rewards can also grant money.

### Kills, Deaths and /back

Player kills are recorded in the `kills` scoreboard and deaths in `deaths`. When a player dies their location is saved; use `/back` to return (the saved location is cleared after use).

---

## Resource Pack and UI

### UI Variables

File: `Qssentials [RP]/ui/_global_variables.json`

```json
{
  "$ess_form_disable_default_panel": true,
  "$ess_rulesUI_disable_custom_highlight_label": true,
  "$ess_actionbar_disable_custom_bar": true,
  "$ess_actionbar_disable_custom_dialog": true
}
```

| Variable | Purpose |
|---|---|
| `$ess_form_disable_default_panel` | Controls the default panel on Qssentials forms. |
| `$ess_rulesUI_disable_custom_highlight_label` | Controls the custom highlight label in the Rules UI. |
| `$ess_actionbar_disable_custom_bar` | Controls the custom bar on the actionbar. |
| `$ess_actionbar_disable_custom_dialog` | Controls the custom dialog on the actionbar. |

A value of `true` disables the custom element (as the variable names say); `false` enables it. All four default to `true` in this build.

### Localization

- Language files: `Qssentials [RP]/texts/en_US.lang` and `id_ID.lang`.
- All addon messages use `ess.*` keys. Edit the text after the `=` sign; `%s` marks a dynamic value and must stay.
- To add a language, create a new `.lang` file and register it in `texts/languages.json`.

### Hidden Chat Prefixes

A few text prefixes are consumed by the Custom UI and never appear as normal chat: `nperma-woi` (`/ctitle`), `@message@` (`/notif` toasts), `cbar:` (land info bar), and `clearchat-nperma` (chat clearing).

### Pack Order

If you use other UI-modifying resource packs, place Qssentials' RP at a priority where its UI files are not overridden.

---

## ScriptEvents and Integrations

Other addons and NPCs can talk to Qssentials through `/scriptevent`.

| ScriptEvent | Message | Description |
|---|---|---|
| `set:currency` | `<number>` | Sets the sender's balance. |
| `add:currency` | `<number>` | Adds to (or subtracts from) the sender's balance. |
| `register:plugin` | `{"namespace":"...","protocol":"...","description":"..."}` | Registers a plugin so it appears in `/plugins`. |
| `b4173:notification` | `title\|message\|icon\|background` | Shows a toast notification to the sending entity. |

`set:currency` and `add:currency` only work when the sender is a **player** (for example an NPC dialog button), not a command block.

Toast example:

```mcfunction
/execute as @a run scriptevent b4173:notification §aSuccess!|Reward claimed.|textures/ui/confirm|textures/ui/purpleBorder
```

### Database

Data is stored as world dynamic properties: `player-db`, `scorehud-db`, `home-db`, `faction-db`, `land-db`, `warp-db`, `quest-db`. Deleting the world deletes all of it.

---

## Developer Guide

### Project Structure

```text
Qssentials [BP]/
├─ manifest.json
├─ Changelog
└─ scripts/
   ├─ config.js              <- main configuration
   ├─ index.js               <- core systems (chat, scorehud, clear, broadcast, K/D)
   ├─ _db.js                 <- database definitions
   ├─ _loader/               <- loader, command dispatcher, protocol constants
   ├─ extension/             <- helpers, forms, economy, quick-db
   └─ commands/
      ├─ command.register.js <- list of enabled commands
      ├─ general/  tpa/  warp/  home/  land/  mode/
      ├─ economy/  experimental/ (daily, weekly)
      └─ owner/  admin/

Qssentials [RP]/
├─ manifest.json
├─ texts/                    <- en_US.lang, id_ID.lang, languages.json
├─ ui/                       <- Custom UI (forms, hud, bars, chat)
├─ textures/  font/
```

### Enabling and Disabling Commands

Open `scripts/commands/command.register.js`. A command is active when its `import` line is not commented out:

```js
import "./general/ping";        // enabled
//import "./admin/cdialog";     // disabled
```

### Adding a Command

1. Create a file, for example `scripts/commands/general/hello.js`:

   ```js
   import { Command } from "../ess.module.js";

   Command.register(
     {
       name: "hello",
       description: "Say hello",
       group: "general",            // /help category
       cooldown: 5,                 // seconds
       tags: ["admin"],             // optional access control
       optional: [{ name: "message", type: Command.param.string }],
     },
     ({ player }, message = "world") => {
       if (!player) return;         // not run by a player
       player.say(`Hello, ${message}!`);
     },
   );
   ```

2. Import it in `command.register.js`:

   ```js
   import "./general/hello";
   ```

#### `Command.register` options

| Option | Description |
|---|---|
| `name` | Command name. The namespace is prepended automatically (`es:hello`). |
| `description` | Plain text or a translation key such as `ess.cmd.hello.description`. |
| `group` / `category` | Category shown in `/help` (uppercased). New categories must also be added to the `categories` array in `general/help.js` to be selectable. |
| `aliases` | Array of alternative names. |
| `permission` / `permissionLevel` | One of `Command.permission`: `any` (0), `gameDirectors` (1), `admin` (2), `host` (3), `owner` (4). |
| `tags` | Array of tags. `"admin"` and `"owner"` are special; any other value is a player tag. The player needs at least one. |
| `cooldown` | Per-player cooldown in seconds. |
| `required` / `optional` | Parameter lists: `{ name, type, options? }`. `enum` parameters need an `options` array. |
| `extension` / `plugins` | Plugin namespaces that must be registered for the command to run. |

Parameter types: `Command.param.blockType`, `itemType`, `entitySelector`, `playerSelector`, `float`, `integer`, `boolean`, `string`, `location`, `enum`.

The callback receives `(origin, ...args)`, where `origin` contains `player` (undefined if the source is not a player) and `source`.

### Player Helpers

Added to `Player.prototype` when the world loads:

| Helper | Description |
|---|---|
| `player.say(text, prefix?, usePrefix?)` | Sends a prefixed system message. |
| `player.broadcast(text)` | Sends a message to everyone else. |
| `getCurrency()` / `setCurrency(n)` / `addCurrency(n)` | Read and modify the `money` balance. |
| `getRank()` / `addRank(name)` | Read and add rank tags. |
| `getScore(obj)` / `setScore(obj, n)` / `addScore(obj, n)` | Scoreboard shortcuts. |
| `isOp`, `isAdmin`, `isOwner`, `isCreative` | Permission and mode checks. |
| `isMoving` | `true` if the player's velocity is non-zero. |
| `health`, `hunger` | Getters/setters for the components. |
| `data` | The player's record from `PlayerDB`. |

Form helpers: `ActionFormData`, `ModalFormData`, and `MessageFormData` gain an `.open(player)` method that retries automatically while the player is busy.

Teleport helper: `teleportHandle(player, { key, countdown, useActionBar }, callback)` runs the callback after a countdown and cancels it if the player moves.

### Database

Import the ready-made instances from `ess.module.js`:

```js
import { PlayerDB, HomeDB, LandDB, WarpDB } from "../ess.module.js";

WarpDB.set("spawn", { x: 0, y: 64, z: 0, dimension: "minecraft:overworld" });
WarpDB.get("spawn");
WarpDB.has("spawn");
WarpDB.keys(); WarpDB.values(); WarpDB.entries();
WarpDB.delete("spawn");
WarpDB.size;
```

Record shapes:

| Database | Key | Value |
|---|---|---|
| `HomeDB` | `<player>:<homeName>` | `{ name, location: {x,y,z}, dimension, rotation }` |
| `WarpDB` | `<warpName>` | `{ x, y, z, dimension }` (optional `icon`) |
| `LandDB` | 4-digit ID string | `{ owner, access[], size, center:{x,z}, dimension, location:[[x,z],[x,z]], bounds:[minX,maxX,minZ,maxZ] }` |

### Plugins

External addons announce themselves with the `register:plugin` ScriptEvent (see above). Commands that list the plugin in `extension`/`plugins` only run when it is registered and compatible; otherwise the player sees `Required plugin(s) not loaded or incompatible`.

---

## Troubleshooting

| Problem | Cause / fix |
|---|---|
| `/es:...` commands do not appear or do nothing | Make sure **both BP and RP** are active, **Beta APIs** are enabled, and **cheats** are on (or set `command_required_cheats: false`). |
| "You do not have permission to use this feature" | The player lacks the required tag or permission level. See [Ranks and Permissions](#ranks-and-permissions). |
| Rank does not show in chat/nametag | The tag must start with `rank:`, and the chat/nametag template must contain `@rank`. Check `nametag.enabled`. |
| Scorehud is missing | Check `scorehud.enable_by_default` and whether the player turned it off with `/scorehud`. Make sure the RP is active. |
| Forms look plain or broken | The Qssentials RP is inactive or overridden by another UI pack. |
| `Required plugin(s) not loaded or incompatible` | The command needs another plugin (for example `/cdialog` needs `DynamicDialogV2`). |
| Config changes have no effect | Save the file and reload the world or restart the server. |
| Cannot create land near spawn | The area is inside `lobby_protect.pos`. |

---

## Known Issues

These are the current behaviors of build 241026. They are tracked here so server owners know what to expect.

- **`/pay`**: the target validity check is inverted, so payments are rejected as if the target were offline.
- **`/tpadeny`**: references an undefined variable and throws an error; requests simply expire instead.
- **`/delhome <name>`**: throws an error when a name is passed; use the `/delhome` UI instead.
- **`/land sell` payout**: the `percentSell` clamp resolves to 1%, so refunds are always 1% of the price regardless of the setting.
- **Land detection**: `/land here`, the overlap check during `/land create`, and `/land sell` ID auto-detection still compare against the legacy coordinate format, so they do not detect existing lands. Workaround: use `/land list` and `/land sell <id>`.
- **`/sethome`**: the minimum and maximum name length (`home.min`, `home.max`) are not enforced.
- **Farmland**: farmland at `y = -64` may still be destroyed (existing known bug).
- **Unused options**: see the notes under [Chat and Nametag](#chat-and-nametag), [Lobby Protect](#lobby-protect), [Auto Broadcast](#auto-broadcast), and [Warp Settings](#warp-settings).

---

## Changelog

### Build 241026 (v2.0.0)

**Behavior Pack**

- Fixed `/daily` and `/weekly` bugs.
- Home UI: added coordinates to the button content.
- New actionbar text variants: `bar:` prefix and `cbar:` (used by the Land Event).
- New Custom UI for `/ranklist`. Removed translation keys `ess.ui.ranklist.header1`, `ess.ui.ranklist.header2`, and `ess.ui.ranklist.label`.
- `/cdialog` is now disabled by default.
- Land: major refactor (dimension dictionary, `bounds` property, new database structure, array-based `locations`, dimension checked first to avoid the heavy overlap test, shorter text helper, `/land sell` auto-detects the land ID, fixed sell message showing the area instead of the price).
- Land events: use `bounds` instead of manual calculation, removed fall ID and damage feature and the button-input listener, fixed farmland cooldown message and `isInsideLand` logic, farmland check delayed by 2 ticks for performance.
- Core: fewer ScriptEvent subscribers, simplified cooldown handling in the command callback.
- `index.js`: new `isInsideLand` implementation, nametag loop merged into the scorehud loop.

**Resource Pack**

- New variables: `$ess_actionbar_disable_custom_dialog`, `$ess_actionbar_disable_custom_bar`.

---

## Contributing

Issues and pull requests are welcome.

1. Fork the repository and create a feature branch.
2. Follow the existing structure: one command per file under `scripts/commands/<group>/`, registered in `command.register.js`.
3. Keep user-facing text in the `.lang` files (`ess.*` keys) and add both `en_US` and `id_ID` entries.
4. Describe what you changed and how you tested it in the pull request.

---

## Credits and License

**Contributors:** Nperma, Fary
Thanks to the BAO Community for the toast notification technique used by `/notif`.

> **DO NOT SHARE CONTENTS OR USE RESOURCES WITHOUT PERMISSION.**
> All rights reserved by the contributors unless a license file in this repository states otherwise.
