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
4. [Useful Lua Functions](#3-other-notable-lua-functions)
5. [Example Lua Snippet](#4-example-lua-snippet)
6. [Tips](#5-troubleshooting--tips)

---

## Introduction

This document shows:

1. **Setting and getting “options”** (booleans, integers, floats) in the bot.
2. **Sending variantlists** with `sendPacketVariantList`.

## 1) Setting & Getting Options

1. **`bot:setOption(name, value)`**
2. **`bot:getOption(name)`**

Where `name` is a string identifying the field to set/get.

### Available Fields

| Category          | Field Name                   | Type  | Description                                                                                         |
|-------------------|------------------------------|-------|-----------------------------------------------------------------------------------------------------|
| **Utils**         | `"showPing"`                 | bool  | If true, display ping somewhere in UI logs or the console.                                          |
|                   | `"visualMod"`                | bool  | If true, enables visual mod mode.                                                    |
| **Mod Settings**  | `"leaveOnMod"`               | bool  | If true, bot leaves the world upon mod entry.                                                       |
|                   | `"banAll"`                   | bool  | If true, bot automatically bans all players (use responsibly!).                                     |
|                   | `"autoCollectBeforeLeave"`   | bool  | If true, bot tries to collect items before leaving the world.                                       |
|                   | `"unAccessAll"`              | bool  | If true, bot unaccesses all players.                                                                |
|                   | `"unAccessOnMod"`            | bool  | If true, bot unaccesses everyone when a mod enters.                                                 |
| **Auto Farm**     | `"isAutoFarmActive"`         | bool  | If true, the auto-farming logic is active.                                                          |
|                   | `"autoFarmID"`               | int   | The item ID that auto-farm targets (e.g. 4584).                                                     |
|                   | `"autoFarmDelay"`            | int   | Delay between auto-farming actions, in ms.                                                          |
| **Speed/Physics** | `"speedInt"`                 | int   | The integer representing movement speed for the bot.                                                |
|                   | `"gravityInt"`               | int   | Gravity integer used in the physics.                                                                |
|                   | `"accelerationX"`            | float | Horizontal acceleration the bot uses for movement.                                                  |
|                   | `"accelerationY"`            | float | Vertical acceleration the bot uses for movement.                                                    |
| **Automations**   | `"isCrimeEnabled"`           | bool  | If true, enables auto crime.                                                                        |
|                   | `"isAutoSurgActive"`         | bool  | If true, enables auto surg.                                                                         |
|                   | `"assistiveAutoSurg"`        | bool  | If true, uses an assistive surgery.                                                                 |
| **Titles**        | `"legendTitleEnabled"`       | bool  | Toggles a “Legend” title.                                                                           |
|                   | `"talkWithMe"`               | bool  | Whenever you chat, it automatically uses /me.                                                       |
|                   | `"drTitleEnabled"`           | bool  | Doctor title.                                                                                       |
|                   | `"autoAcceptAccess"`         | bool  | If true, automatically accepts access for other players.                                            |
|                   | `"maxLevelTitleEnabled"`     | bool  | Toggles a “Max Level” title.                                                                        |
|                   | `"donorTitleEnabled"`        | bool  | Donor title.                                                                                        |
|                   | `"masterTitleEnabled"`       | bool  | Master title.                                                                                       |
|                   | `"partyAnimalTitleEnabled"`  | bool  | Party Animal title.                                                                                 |
|                   | `"ccBadgeEnabled"`           | bool  | If true, shows a certain “CC Badge”.                                                                |
|                   | `"thanksgivingTitleEnabled"` | bool  | Thanksgiving title.                                                                                 |
|                   | `"oldTimerTitleEnabled"`     | bool  | Old Timer title.                                                                                    |
|                   | `"goldTitleEnabled"`         | bool  | Gold title.                                                                                         |
|                   | `"silverTitleEnabled"`       | bool  | Silver title.                                                                                       |
|                   | `"bronzeTitleEnabled"`       | bool  | Bronze title.                                                                                       |

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

We expose a method to send a variantlist packet, which in Lua, you call it with:

```lua
bot:sendPacketVariantList(sendToProxy, variantTable)
```

- `sendToProxy`: Boolean. If `true`, interpret as sendToProxy → server packet; if `false`, interpret as server → sendToProxy.
- `variantTable`: A **Lua table** containing up to 7 elements. Each element can be:
  - A string
  - A number (will be stored as a float in `variant`)
  - A boolean (internally stored as integer 0 or 1)

### Usage

#### Basic Example:
```lua
local varlist = {
    "OnTextOverlay",
    "`2Accepted`` Access."  -- index #2
}

bot:sendPacketVariantList(true, varlist)
```

> **Important**: The code that translates the Lua table → `variantlist` is fairly simple:
> - Skips unknown types (tables, functions, etc.).
> - Converts booleans to integer variant (0 or 1).
> - Converts numbers to floats.
> - Converts strings to strings.
---

## 3) Other Notable Lua Functions

- **`bot:findPath(x, y) -> bool`**  
  Tells the bot to attempt pathfinding to `(x,y)`. Returns `true` if path can begin, `false` otherwise.

- **`bot:getPosition() -> (float x, float y)`**  
  Returns the current coordinates of the bot.

- **`bot:punch(x, y)`**  
  Makes the bot punch tile `(x, y)`.

- **`bot:place(itemID, x, y)`**  
  Places an item with `itemID` at `(x, y)`.

- **`bot:RunLuaScript(script)`** and **`bot:StopLuaScript()`**  
  Start/stop scripts on a particular bot. (Your code might differ slightly in how it’s triggered.)

- **`getBot("SomeGrowID")`**  
  (Global function) Retrieves a `ServerHandler` userdatum for the given `growID`.

---

## 4) Example Lua Snippet

Below is a complete sample script demonstrating usage of the new functionalities:

```lua
-- 1) Retrieve a bot by GrowID
local bot = getBot("MyGrowID")

-- 2) Setting some options:
bot:setOption("showPing", true)
bot:setOption("visualMod", true)
bot:setOption("speedInt", 750)
bot:setOption("accelerationX", 2000.0)
bot:setOption("leaveOnMod", false)
bot:setOption("isAutoFarmActive", true)
bot:setOption("autoFarmID", 4584)
bot:setOption("autoFarmDelay", 150)

-- 3) Checking them:
print("Ping visible? ", bot:getOption("showPing"))
print("speedInt:", bot:getOption("speedInt"))
print("accelerationX:", bot:getOption("accelerationX"))

-- 4) Use pathfinding:
local ok = bot:findPath(30, 15)
if ok then
  print("Pathfindig to 30;15.")
else
  print("Path is not available!")
end

-- 5) Punch a tile:
bot:punch(28, 14)

-- 6) Place an item:
bot:place(4584, 31, 15)

-- 7) Send a text overlay using a variantlist:
local varlist = {
    "OnTextOverlay", 
    "This is my overlay text"
}
bot:sendPacketVariantList(true, varlist)

-- 8) Get the bot’s position:
local x, y = bot:getPosition()
print("Bot position is now", x, y)
```

---

## 5) Troubleshooting & Tips

1. **Unknown Option**  
   If you do `bot:setOption("randomValue", true)`, you’ll get:
   > `setOption: Unknown field 'randomValue'.`

2. **Type Mismatch**  
   If you pass a **string** to `speedInt`, for example `bot:setOption("speedInt", "fast")`,
   it will throw an error from `luaL_checkinteger(L, 3)`. Make sure you pass correct types
   (bool, int, float).

---

