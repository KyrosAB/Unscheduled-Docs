# Unscheduled Proxy Executor Documentation

---
## Table of Contents

1. [Introduction](#introduction)
2. [Setting & Getting Options](#1-setting--getting-options)
   - [Available Fields](#available-fields)
   - [Usage](#usage)
     - [Setting an Option](#setting-an-option)
     - [Getting an Option](#getting-an-option)
3. [Sending a VariantList Packet](#2-sending-a-variantlist-packet)
   - [Purpose](#purpose)
   - [Usage](#usage-1)
4. [Event Handling (Callbacks)](#3-event-handling-callbacks)
   - [gameMessage](#gamemessage)
   - [genericText](#generictext)
   - [variantlist](#variantlist)
   - [Usage Example](#usage-example)
5. [Other Notable Lua Functions](#4-other-notable-lua-functions)
   - [Raw Packet Sending](#raw-packet-sending)
   - [Sleeping / Delays](#sleeping--delays)
   - [World Info & Entities](#world-info--entities)
   - [Movement / Interaction](#movement--interaction)
   - [Misc](#misc)
6. [Example Lua Snippet](#5-example-lua-snippet)
7. [Troubleshooting & Tips](#6-troubleshooting--tips)

---

## Introduction

This document shows:

1. **Setting and getting “options”** (booleans, integers, floats) in the bot.
2. **Sending variantlists** with `sendPacketVariantList`.
3. **Handling events** (game messages, generic text, variant lists).
4. **Useful Lua functions** for controlling the bot’s behavior.

---

## 1) Setting & Getting Options

1. **`bot:setOption(name, value)`**
2. **`bot:getOption(name)`**

Where `name` is a string identifying the field to set/get.

### Available Fields

| Category          | Field Name                   | Type  | Description                                                                               |
|-------------------|------------------------------|-------|-------------------------------------------------------------------------------------------|
| **Utils**         | `"showPing"`              | bool  | If true, will display ping by your name.                                                     |
|                   | `"visualMod"`             | bool  | If true, enables a “visual mod” mode (the bot visually acts modded in certain contexts).     |
| **Mod Settings**  | `"leaveOnMod"`            | bool  | If true, bot leaves the world upon mod entry.                                                |
|                   | `"banAll"`                | bool  | If true, bot automatically bans all players (use responsibly!).                              |
|                   | `"autoCollectBeforeLeave"` | bool  | If true, bot tries to collect items before leaving the world.                               |
|                   | `"unAccessAll"`           | bool  | If true, bot unaccesses all players.                                                         |
|                   | `"unAccessOnMod"`         | bool  | If true, bot unaccesses everyone when a mod enters.                                          |
| **Auto Farm**     | `"isAutoFarmActive"`       | bool  | If true, the auto-farming logic is active.                                                  |
|                   | `"autoFarmID"`            | int   | The item ID that auto-farm targets (e.g. 4584).                                              |
|                   | `"autoFarmDelay"`         | int   | Delay between auto-farming actions, in ms.                                                   |
| **Speed/Physics** | `"speedInt"`              | int   | The integer representing movement speed for the bot.                                         |
|                   | `"gravityInt"`            | int   | Gravity integer used in the bot’s physics.                                                   |
|                   | `"accelerationX"`         | float | Horizontal acceleration the bot uses for movement.                                           |
|                   | `"accelerationY"`         | float | Vertical acceleration the bot uses for movement.                                             |
| **Automations**   | `"isCrimeEnabled"`         | bool  | If true, enables auto crime.                                                                |
|                   | `"isAutoSurgActive"`       | bool  | If true, enables auto surg.                                                                 |
|                   | `"assistiveAutoSurg"`     | bool  | If true, uses an assistive surgery.                                                          |
| **Titles**        | `"legendTitleEnabled"`     | bool  | Toggles a “Legend” title.                                                                   |
|                   | `"talkWithMe"`            | bool  | Whenever you chat, it automatically uses /me.                                                |
|                   | `"drTitleEnabled"`        | bool  | Doctor title.                                                                                |
|                   | `"autoAcceptAccess"`      | bool  | If true, automatically accepts access for other players.                                     |
|                   | `"maxLevelTitleEnabled"`   | bool  | Toggles a “Max Level” title.                                                                |
|                   | `"donorTitleEnabled"`      | bool  | Donor title.                                                                                |
|                   | `"masterTitleEnabled"`     | bool  | Master title.                                                                               |
|                   | `"partyAnimalTitleEnabled"`| bool  | Party Animal title.                                                                         |
|                   | `"ccBadgeEnabled"`         | bool  | CC Badge.                                                                                   |
|                   | `"thanksgivingTitleEnabled"` | bool| Thanksgiving title.                                                                         |
|                   | `"oldTimerTitleEnabled"`   | bool  | Old Timer title.                                                                            |
|                   | `"goldTitleEnabled"`       | bool  | Gold title.                                                                                 |
|                   | `"silverTitleEnabled"`     | bool  | Silver title.                                                                               |
|                   | `"bronzeTitleEnabled"`     | bool  | Bronze title.                                                                               |

### Usage

#### Setting an Option
```lua
-- Lua usage: bot:setOption("fieldName", value)

bot:setOption("showPing", true)
bot:setOption("visualMod", false)

bot:setOption("leaveOnMod", true)
bot:setOption("banAll", false)

bot:setOption("isAutoFarmActive", true)
bot:setOption("autoFarmID", 4584)
bot:setOption("autoFarmDelay", 200)

bot:setOption("speedInt", 500)
bot:setOption("gravityInt", 1000)
bot:setOption("accelerationX", 1500.0)
bot:setOption("accelerationY", 1200.0)
```

#### Getting an Option
```lua
-- Usage: local value = bot:getOption("fieldName")

local pingVis = bot:getOption("showPing")          -- returns true/false
local speed    = bot:getOption("speedInt")         -- returns integer
local accelX   = bot:getOption("accelerationX")    -- returns float
print("showPing? ", pingVis)
print("speedInt = ", speed)
print("accelerationX = ", accelX)
```

> **Note**: If you call `getOption()` with an unknown field name, it returns `nil`.

---

## 2) Sending a VariantList Packet

### Purpose

We expose a method to send a variantlist packet, which in Lua, you call with:

```lua
bot:sendPacketVariantList(sendToProxy, variantTable, netID)
```

- `sendToProxy`: **Boolean**. If `true`, interpret as proxy → client.” If `false`, interpret as “proxy → server.”  
- `variantTable`: A **Lua table** containing up to 7 elements. Each element can be:
  - A string
  - A number (will be stored as a float in `variant_t`)
  - A boolean (internally stored as integer 0 or 1)
- `netID` (optional): integer netID to be sent in the packet.

### Usage

#### Basic Example:
```lua
local varlist = {
    "OnTextOverlay",
    "`2Accepted`` Access."  -- index #2
}

-- sending to client (proxy->client):
bot:sendPacketVariantList(true, varlist)
```

> **Important**: The code that translates the Lua table → `variantlist_t` is fairly simple:
> - Skips unknown types (tables, functions, etc.).
> - Converts booleans to integer variant (0 or 1).
> - Converts numbers to floats.
> - Converts strings to strings.

---

## 3) Event Handling (Callbacks)

You can listen for events by using:

```lua
bot:on("eventName", function(...) ... end)
```

The following event names are available:

### gameMessage
- Triggered when a game message is fired in the internal handler.
- Callback receives **1** argument: `message` (string).

Example:
```lua
bot:on("gameMessage", function(msg)
    print("Received gameMessage:", msg)
end)
```

### genericText
- Triggered when “generic text” data is received.
- Callback receives **1** argument: `text` (string).

Example:
```lua
bot:on("genericText", function(txt)
    print("Received genericText:", txt)
end)
```

### variantlist
- Triggered when a variant list arrives in the internal handler.
- Callback receives **2** arguments:
  1. `table`: A table of up to 7 entries containing the variant elements.
  2. `netID`: The integer netID associated with the list.

Example:
```lua
bot:on("variantlist", function(varTable, netID)
    print("Got a variantlist, netID:", netID)
    for i, v in ipairs(varTable) do
        print("   Index", i, "Value:", v)
    end
end)
```

#### Usage Example

You can register all callbacks in your script as follows:

```lua
bot:on("gameMessage", function(msg)
    print("Game message arrived:", msg)
end)

bot:on("genericText", function(txt)
    print("Generic text arrived:", txt)
end)

bot:on("variantlist", function(varTable, netID)
    print("Variant list arrived from netID:", netID)
end)
```

---

## 4) Other Notable Lua Functions

### Text Packet Sending

```lua
bot:sendPacket(packetType, data)
```

- `packetType`: (integer) 
- `data`: (string)

Sends a raw packet with the specified `packetType` and `data` from proxy → server.

### Sleeping / Delays

```lua
bot:sleep(milliseconds)
```

- `milliseconds`: integer
- Blocks the script for the given number of milliseconds.
- The script can be forcibly stopped by external calls to stop the Lua script.
- Example:
  ```lua
  print("Sleeping for 2 seconds...")
  bot:sleep(2000)
  print("Awake again!")
  ```

### World Info & Entities

```lua
bot:getWorldName()
```
- Returns the current world name as a string.

```lua
bot:getTiles()
```
- Returns a **table** (array) of tile objects with these fields:
  - `bg` (int)  
  - `fg` (int)  
  - `data` (int)  
  - `growth` (int)  
  - `growTime` (int)  
  - `fruits` (int)  
  - `flags` (int)  
  - `x` (int) – tile X coordinate  
  - `y` (int) – tile Y coordinate  
  - `label` (string) – label for the tile.

```lua
bot:getPlayers()
```
- Returns a **table** (array) of player objects with these fields:
  - `name` (string)  
  - `netID` (int)  
  - `userID` (int)  
  - `invis` (bool) — if the player is a moderator or guardian.  
  - `country` (string)  
  - `mod` (bool)  
  - `pos` (table) containing `x`, `y` floats.

```lua
bot:getObjects()
```
- Returns a **table** (array) of dropped item objects with these fields:
  - `oid` (int) — internal dropped-object ID  
  - `itemID` (int)  
  - `flags` (int)  
  - `count` (int)  
  - `x` (int)  
  - `y` (int)

### Movement / Interaction

```lua
bot:findPath(x, y)
```
- Attempts pathfinding to tile coordinate (`x`, `y`).
- Returns `true` if pathfinding can start, `false` otherwise.

```lua
bot:getPosition()
```
- Returns two values (`float x`, `float y`) for the bot’s current position in the world.

```lua
bot:punch(x, y)
```
- Punches the tile at (`x`, `y`).

```lua
bot:place(itemID, x, y)
```
- Places `itemID` at (`x`, `y`).

### Misc

```lua
bot:warp(worldName)
```
- **Currently** returns the bot’s GrowID. (Implementation placeholder; may be extended in future.)

```lua
bot:name()
```
- Returns the bot’s GrowID as a string.

```lua
getBot(growID)
```
- Global function. Returns a `ServerHandler` object for the given `growID`, or throws an error if not found.

---

## 5) Example Lua Snippet

Below is a complete sample script demonstrating usage of these functionalities:

```lua
-- 1) Retrieve a bot by GrowID
local bot = getBot("GrowID")

-- 2) Set some options:
bot:setOption("showPing", true)
bot:setOption("visualMod", true)
bot:setOption("speedInt", 750)
bot:setOption("accelerationX", 2000.0)
bot:setOption("leaveOnMod", false)
bot:setOption("isAutoFarmActive", true)
bot:setOption("autoFarmID", 4584)
bot:setOption("autoFarmDelay", 150)

-- 3) Check them:
print("Ping visible? ", bot:getOption("showPing"))
print("speedInt:", bot:getOption("speedInt"))
print("accelerationX:", bot:getOption("accelerationX"))

-- 4) Register some event callbacks:
bot:on("gameMessage", function(msg)
    print("Got a gameMessage:", msg)
end)

bot:on("variantlist", function(vlist, netID)
    print("VariantList from netID:", netID)
    for i, v in ipairs(vlist) do
        print("   ["..i.."] = "..tostring(v))
    end
end)

-- 5) Use pathfinding:
local ok = bot:findPath(30, 15)
if ok then
  print("Pathfinding to 30;15 started.")
else
  print("No valid path or cannot start pathfinding!")
end

-- 6) Punch a tile:
bot:punch(28, 14)

-- 7) Place an item:
bot:place(4584, 31, 15)

-- 8) Send a text overlay using a variantlist:
local overlay = {
    "OnTextOverlay", 
    "This is my overlay text"
}
bot:sendPacketVariantList(true, overlay)

-- 9) Sleep for a moment:
print("Sleeping for 1 second...")
bot:sleep(1000)
print("Woke up!")

-- 10) Get the bot’s position:
local x, y = bot:getPosition()
print("Bot position is now", x, y)
```

---

## 6) Troubleshooting & Tips

1. **Unknown Option**  
   If you do `bot:setOption("randomValue", true)`, you’ll get an error:
   ```
   setOption: Unknown field 'randomValue'.
   ```

2. **Type Mismatch**  
   If you pass a **string** to `speedInt`, e.g. `bot:setOption("speedInt", "fast")`,  
   it will throw an error from `luaL_checkinteger(L, 3)`. Make sure you pass correct types
   (bool, int, float).

3. **Blocking Calls**  
   The `bot:sleep(milliseconds)` function blocks the script. If you need asynchronous
   behavior, keep in mind that this stops your Lua code entirely until the time has elapsed
   (unless forcibly stopped).

4. **Stopping a Script**  
   If the script is forcibly stopped from outside (C++ side), the `sleep`, `findPath`, or
   other blocking calls will throw an error and terminate the script. Wrap them in `pcall`
   if you need to catch that.

---