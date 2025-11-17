# MOOSE Tanker Management System

**Author:** F99th-TracerFacer  
**Latest Version:** [Moose_Tanker.lua](https://github.com/iTracerFacer/Moose_Tanker/blob/main/Moose_Tanker.lua)

A comprehensive aerial refueling tanker management system for DCS World missions using the MOOSE framework. Provides complete tanker lifecycle management including dynamic spawning, custom route planning, automatic respawning, fuel monitoring, and in-flight rerouting capabilities.

---

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Configuration Guide](#configuration-guide)
- [Custom Route System](#custom-route-system)
- [Menu Structure](#menu-structure)
- [Advanced Features](#advanced-features)
- [Strategic Planning](#strategic-planning)
- [Troubleshooting](#troubleshooting)
- [Examples](#examples)
- [FAQ](#faq)

---

## Features

### Core Capabilities
- **Simple Spawning**: Launch tankers with one click - automatic orbit patterns at default locations
- **Custom Route Planning**: Define tanker tracks using numbered map markers
- **Dynamic Rerouting**: Change active tanker routes mid-mission without respawning
- **Automatic Respawn**: Tankers respawn automatically after destruction (configurable delay)
- **Fuel Management**: Automatic low fuel warnings (25%) and bingo fuel RTB (15%)
- **Emergency Spawns**: Fast-response tankers with reduced spawn delays (60s vs 180s)
- **Status Reporting**: Real-time fuel levels, altitude, position, and TACAN/radio info
- **Event Monitoring**: Tracks tanker damage, hostile fire, crashes, and RTB events

### Advanced Features
- **Altitude/Speed Overrides**: Customize each waypoint's flight level and airspeed
- **RTB Commands**: Send tankers home to nearest friendly airbase with automatic landing
- **Multi-Tanker Support**: Manage multiple tanker types simultaneously (KC-135, KC-135 MPRS)
- **Menu Integration**: Seamless F10 radio menu with organized categories
- **TACAN/Radio Announcements**: Automatic frequency broadcasts on spawn
- **Randomized Messages**: 100+ message variations for immersive mission atmosphere
- **Marker Cleanup**: Optional automatic deletion of route markers after use

---

## Requirements

### Software Requirements
- **DCS World**: Version 2.5.6 or later
- **MOOSE Framework**: Latest version ([Download Here](https://github.com/FlightControl-Master/MOOSE))
- **DCS Mission Editor**: For mission integration

### Knowledge Requirements
- Basic DCS mission editing
- Understanding of F10 map markers
- Familiarity with TACAN/radio frequencies (optional)

---

## Quick Start

### Installation

1. **Download MOOSE Framework**
   ```
   https://github.com/FlightControl-Master/MOOSE
   ```

2. **Download Moose_Tanker.lua**
   ```
   https://github.com/iTracerFacer/Moose_Tanker/blob/main/Moose_Tanker.lua
   ```

3. **Add to Your Mission**
   - Open your mission in DCS Mission Editor
   - Go to Triggers
   - Create a new trigger: `TYPE: "MISSION START"`, `ACTION: "DO SCRIPT FILE"`
   - First, load `MOOSE.lua`
   - Then, load `Moose_Tanker.lua`

4. **Save and Test**
   - Save your mission
   - Launch in DCS
   - Press `F10` → `Tanker Operations`

### First Launch (Simple Method)

1. Press `F10` to open radio menu
2. Navigate to `Tanker Operations`
3. Select `Launch TANKER KC-135`
4. Tanker spawns at default location (FL220, 330 knots)
5. Refuel as needed!

**TACAN**: 50X  
**Radio**: 251.000 MHz

---

## Configuration Guide

### Tanker Configuration

Edit the `TANKER_CONFIG` section to customize tanker properties:

```lua
local TANKER_CONFIG = {
  KC135 = {
    groupName = "TANKER 135",          -- Internal group name
    unitName = "TANKER 135-1",         -- Internal unit name
    displayName = "TANKER KC-135",     -- Player-facing name
    aircraftType = "KC-135",           -- DCS aircraft type
    livery = nil,                      -- Livery ID (nil = default)
    callsign = "SHELL",                -- Map marker prefix
    tacan = "50X",                     -- TACAN channel (nil if none)
    frequency = "251.000",             -- Radio frequency MHz (nil if none)
    respawnDelay = 180,                -- Auto-respawn delay (seconds)
    emergencyRespawnDelay = 60,        -- Emergency spawn delay (seconds)
    fuelWarningPercent = 25,           -- Low fuel warning threshold
    fuelBingoPercent = 15,             -- RTB fuel level
    defaultAltitude = 22000,           -- Default altitude (feet MSL)
    defaultSpeed = 330,                -- Default speed (knots)
  },
  KC135_MPRS = {
    -- Similar configuration for KC-135 MPRS
    -- ...
  }
}
```

### Route Configuration

Customize waypoint behavior:

```lua
local ROUTE_CONFIG = {
  minWaypoints = 2,                  -- Minimum waypoints required
  maxWaypoints = 10,                 -- Maximum waypoints allowed
  deleteMarkersAfterUse = true,      -- Auto-delete markers after spawn
  waypointPrefix = {
    SHELL = "KC135",                 -- SHELL1, SHELL2 → KC-135
    ARCO = "KC135_MPRS",             -- ARCO1, ARCO2 → KC-135 MPRS
  }
}
```

### Monitoring Configuration

```lua
local FUEL_CHECK_INTERVAL = 60       -- Fuel check frequency (seconds)
local DAMAGE_RTB_THRESHOLD = 50      -- RTB if damage > 50%
```

### Default Spawn Location

```lua
-- Latitude, Longitude, Altitude
local DEFAULT_SPAWN_COORD = COORDINATE:NewFromLLDD(
  34.564,   -- Latitude
  69.212,   -- Longitude
):SetAltitude(22000 * 0.3048, true)  -- Altitude in meters MSL
```

**Finding Coordinates:**
1. Place an F10 map marker at desired location
2. Check mission file (.miz) for marker coordinates
3. Update `DEFAULT_SPAWN_COORD` with lat/lon values

---

## Custom Route System

### Basic Marker Syntax

Place numbered F10 map markers following this pattern:

```
[CALLSIGN][NUMBER]:[ALTITUDE]:[SPEED]
```

**Components:**
- `CALLSIGN`: Tanker prefix (`SHELL`, `ARCO`)
- `NUMBER`: Waypoint sequence (1, 2, 3, ...)
- `ALTITUDE`: Optional - Flight level (`FL220`)
- `SPEED`: Optional - Airspeed in knots (`SP330`)

### Marker Examples

#### Simple Route (Defaults)
```
SHELL1
SHELL2
SHELL3
```
Uses default altitude (FL220) and speed (330 knots)

#### Altitude Override
```
ARCO1:FL180
ARCO2:FL180
ARCO3:FL180
```
All waypoints at 18,000 feet

#### Speed Override
```
SHELL1::SP300
SHELL2::SP300
```
All waypoints at 300 knots (uses default altitude)

#### Full Override
```
SHELL1:FL250:SP350
SHELL2:FL250:SP350
SHELL3:FL250:SP350
```
High altitude track at 25,000 feet, 350 knots

#### RTB Command
```
SHELL1:FL220
SHELL2:FL220
SHELL3:RTB
```
Tanker returns to nearest friendly airbase after waypoint 2

### Route Behavior

#### Continuous Patrol (No RTB)
If the last waypoint is NOT an RTB command:
- **Multiple Waypoints**: Tanker loops back to waypoint 1, creating continuous racetrack pattern
- **Single Waypoint**: Tanker enters circular orbit at that location

#### RTB Routes
If any waypoint has `:RTB` command:
- Tanker finds nearest friendly airbase
- Automatically generates landing approach
- Lands and shuts down engines
- Auto-respawns after configured delay

### Launching Custom Route Tankers

1. **Place Markers**
   - Open F10 map
   - Place numbered markers (e.g., SHELL1, SHELL2, SHELL3)
   - Use advanced syntax for altitude/speed overrides

2. **Launch via Menu**
   - Press `F10`
   - Navigate to `Tanker Operations` → `Custom Route`
   - Select appropriate tanker (e.g., `Launch TANKER KC-135 (SHELL markers)`)

3. **Confirmation**
   - System scans for markers matching callsign
   - Validates waypoint count (minimum 2, maximum 10)
   - Displays route confirmation with waypoint details
   - Markers are deleted (if configured)

4. **Monitor Status**
   - Use `Tanker Status Report` to check position/fuel
   - Tanker automatically refuels aircraft in pattern

---

## Menu Structure

### F10 Radio Menu: Tanker Operations

```
F10: Tanker Operations
│
├── Launch TANKER KC-135
├── Launch TANKER KC-135 MPRS
│
├── Custom Route
│   ├── How to Use Custom Routes (Help)
│   ├── Launch TANKER KC-135 (SHELL markers)
│   ├── Launch TANKER KC-135 MPRS (ARCO markers)
│   └── Reroute Active Tanker
│       ├── Reroute TANKER KC-135 (SHELL markers)
│       └── Reroute TANKER KC-135 MPRS (ARCO markers)
│
├── Emergency Tanker (60-second spawn)
│   ├── Emergency TANKER KC-135 (SHELL markers)
│   └── Emergency TANKER KC-135 MPRS (ARCO markers)
│
└── Tanker Status Report
```

### Menu Integration

The script automatically integrates with **MenuManager** if available:
- Appears under `Mission Options` → `Tanker Operations`
- Keeps CTLD at F2, AFAC at F3 (organized structure)

**Without MenuManager:**
- Creates standalone root menu at F10
- Fully functional but less organized

---

## Advanced Features

### Dynamic Rerouting

Change an active tanker's route without respawning:

#### Use Cases:
- Reposition for different theater areas
- Avoid emerging threat zones (SAMs, fighters)
- Send tanker home mid-mission
- Adjust for changing weather/airspace

#### How to Reroute:

1. **Place New Markers**
   - Place numbered markers with desired new route
   - Use same callsign as active tanker (SHELL or ARCO)

2. **Execute Reroute**
   - F10 → Tanker Operations → Custom Route → Reroute Active Tanker
   - Select appropriate tanker

3. **Immediate Effect**
   - Tanker immediately begins flying new route
   - No spawn delay or interruption
   - Old markers deleted automatically

#### Example: Send Home Mid-Mission
```
SHELL1:RTB
```
Single marker with RTB sends tanker to nearest base immediately

### Emergency Spawn System

Fast-response tankers for critical fuel situations:

**Key Differences:**
- **Normal Respawn**: 180 seconds (3 minutes)
- **Emergency Spawn**: 60 seconds (1 minute)

**When to Use:**
- Multiple aircraft bingo fuel
- Training mission fuel miscalculation
- Original tanker destroyed mid-refuel
- Time-sensitive strike packages

**Requirements:**
- Must use custom route markers
- Follows same marker syntax as normal custom routes
- Announces "EMERGENCY" priority in messages

### Fuel Management System

Automatic monitoring and warnings:

#### Low Fuel Warning (25%)
- Broadcast to all players
- Yellow alert message
- "Recommend expedite refueling"
- Tanker continues operations

#### Bingo Fuel (15%)
- Broadcast to all players
- Red warning message
- "Returning to base immediately"
- Tanker automatically flies to nearest friendly airbase
- Auto-respawn begins after landing

#### Monitoring Frequency
- Checks every 60 seconds (configurable via `FUEL_CHECK_INTERVAL`)
- No performance impact on mission

### Event Handling

System monitors and responds to:

- **Birth**: Tanker spawns - announces TACAN/radio
- **Dead**: Tanker destroyed - schedules respawn
- **Crash**: Tanker crashes - schedules respawn
- **Hit**: Tanker takes fire - alerts players
- **Engine Shutdown**: Tanker lands - schedules respawn

---

## Strategic Planning

### Mission Design Considerations

#### Strike Package Support
```
Pre-plan tanker tracks along ingress/egress routes:
- Place SHELL1 near IP (Initial Point)
- Place SHELL2 near egress corridor
- Place SHELL3:RTB for automatic recovery
```

#### Multiple Theater Support
```
Use both tankers for large AO coverage:
- KC-135 (SHELL) covers northern sector
- KC-135 MPRS (ARCO) covers southern sector
- Adjust altitudes to avoid conflicts
```

#### Training Missions
```
Set up refueling training scenarios:
- Low altitude tracks: FL120-FL150
- Slow speed for students: SP250-SP280
- Multiple passes with racetrack patterns
```

### Altitude Planning

**Standard Altitudes:**
- FL180-FL220: Low-level support
- FL220-FL250: Standard ops
- FL250-FL300: High-altitude CAP support
- FL300+: Strategic bomber support

**Separation:**
- Vertical: 2,000 feet minimum between tankers
- Horizontal: 10+ NM for same altitude

### Speed Planning

**Standard Speeds:**
- 250 knots: Student/training refueling
- 280-300 knots: Standard tactical
- 330 knots: High-speed operations
- 350+ knots: Fast-moving packages

### Racetrack Pattern Design

**Two-Point Pattern:**
```
SHELL1 ----30NM---- SHELL2
  ^                    |
  |                    |
  +-------- Loop ------+
```
Simple back-and-forth, 60NM total

**Three-Point Pattern:**
```
      SHELL2
     /      \
SHELL1      SHELL3
    \        /
     +------+
```
Triangular pattern, better area coverage

**Four-Point Box:**
```
SHELL1 ----- SHELL2
  |            |
  |            |
SHELL4 ----- SHELL3
```
Provides maximum flexibility and loiter time

---

## Examples

### Example 1: Simple Default Spawn

**Goal:** Quick tanker for immediate refueling

**Steps:**
1. F10 → Tanker Operations → Launch TANKER KC-135
2. Tanker spawns at default location (configured in script)
3. Automatically enters orbit pattern
4. Ready for refueling

**Result:** Tanker on station in seconds at FL220, 330 knots

---

### Example 2: Custom Three-Point Racetrack

**Goal:** Triangular patrol pattern in combat zone

**Marker Setup:**
```
SHELL1:FL220:SP300
SHELL2:FL220:SP300
SHELL3:FL220:SP300
```

**Steps:**
1. Place three markers forming triangle in desired area
2. F10 → Custom Route → Launch TANKER KC-135 (SHELL markers)
3. Tanker spawns at SHELL1, flies to SHELL2, SHELL3, then loops back to SHELL1

**Result:** Continuous triangular patrol at FL220, 300 knots

---

### Example 3: High-Altitude Strike Support with RTB

**Goal:** Support high-altitude strike package, then return home

**Marker Setup:**
```
SHELL1:FL280:SP350    (Near friendly territory)
SHELL2:FL280:SP350    (Near target area)
SHELL3:FL280:SP350    (Egress corridor)
SHELL4:RTB            (Return to base)
```

**Steps:**
1. Place four markers along strike route
2. F10 → Custom Route → Launch TANKER KC-135 (SHELL markers)
3. Tanker supports ingress and egress
4. Automatically lands at nearest friendly base after SHELL3

**Result:** Mission-specific tanker support with automatic recovery

---

### Example 4: Low-Level Helicopter Tanker

**Goal:** Support helicopter operations at low altitude

**Marker Setup:**
```
ARCO1:FL050:SP250
ARCO2:FL050:SP250
```

**Steps:**
1. Place two markers at low altitude AO
2. F10 → Custom Route → Launch TANKER KC-135 MPRS (ARCO markers)
3. MPRS tanker provides drogue refueling for helos

**Result:** Low-altitude racetrack pattern at 5,000 feet, 250 knots

---

### Example 5: Emergency Mid-Mission Reposition

**Scenario:** Tanker in wrong location, aircraft running low on fuel

**Solution:**
1. Place new markers at desired location:
   ```
   SHELL1:FL220
   SHELL2:FL220
   ```
2. F10 → Custom Route → Reroute Active Tanker → Reroute TANKER KC-135
3. Tanker immediately flies to new location

**Result:** No spawn delay, immediate repositioning

---

### Example 6: Dual Tanker Coverage

**Goal:** Cover large AO with two tankers

**Setup:**
```
Northern Sector (KC-135):
SHELL1:FL240
SHELL2:FL240
SHELL3:FL240

Southern Sector (KC-135 MPRS):
ARCO1:FL220
ARCO2:FL220
```

**Steps:**
1. Place SHELL markers in northern area
2. Place ARCO markers in southern area
3. Launch both tankers via Custom Route menu
4. Monitor with Tanker Status Report

**Result:** Two independent tanker tracks with altitude separation

---

## Troubleshooting

### Common Issues

#### Issue: "Failed to spawn tanker"

**Possible Causes:**
- MOOSE not loaded before script
- Script syntax error during editing
- Coalition mismatch (script is for BLUE coalition)

**Solutions:**
1. Check trigger order: MOOSE.lua MUST load first
2. Verify no syntax errors in USER CONFIGURATION section
3. Ensure you're testing as Blue coalition

---

#### Issue: "Not enough waypoints" error

**Cause:** Less than 2 markers found

**Solution:**
- Verify markers are named correctly: `SHELL1`, `SHELL2` (exact spelling)
- Ensure markers are placed on F10 map (not briefing markers)
- Check marker text has no extra spaces

---

#### Issue: Tanker spawns but doesn't move

**Possible Causes:**
- Only one waypoint (should create orbit, but may have issue)
- Route validation failed

**Solutions:**
- Always use minimum 2 waypoints for custom routes
- Check DCS.log for error messages
- Try default spawn first to verify script is working

---

#### Issue: Can't see tanker in F10 menu

**Possible Causes:**
- Tanker already active (can only have one of each type)
- Menu not initialized

**Solutions:**
- Use "Tanker Status Report" to check if already active
- Restart mission if menu doesn't appear
- Check MOOSE is loaded correctly

---

#### Issue: Markers not being deleted

**Cause:** `deleteMarkersAfterUse` set to false

**Solution:**
- Edit script: `deleteMarkersAfterUse = true,`
- Or manually delete markers after tanker spawns

---

#### Issue: Tanker spawns at wrong altitude

**Possible Causes:**
- Marker syntax error (e.g., `FL220` instead of `:FL220`)
- Using feet instead of flight level

**Solutions:**
- Verify syntax: `SHELL1:FL220` (FL = Flight Level in hundreds of feet)
- FL220 = 22,000 feet MSL
- Don't use: `SHELL1:22000` (incorrect)

---

#### Issue: RTB doesn't work

**Possible Causes:**
- No friendly airbase within range
- Marker syntax error

**Solutions:**
- Ensure friendly airbase exists on map
- Use exact syntax: `SHELL3:RTB` (all caps)
- Check tanker has enough fuel to reach base

---

### Log Monitoring

Enable DCS logging to troubleshoot:

1. **Open DCS.log**
   - Located in: `C:\Users\[YourName]\Saved Games\DCS\Logs\dcs.log`

2. **Search for:**
   - `[TANKER]` - All tanker script events
   - `ERROR` - Script errors
   - `WARNING` - Potential issues

3. **Common Log Messages:**
   ```
   [TANKER] Tanker Management System initialized ✓
   [TANKER] Spawned TANKER KC-135 (ID: 12345) ✓
   [TANKER] Found waypoint marker: SHELL1 at seq 1 ✓
   [TANKER] ERROR: Failed to spawn... ✗
   ```

---

## FAQ

### General Questions

**Q: Do I need tanker units in the Mission Editor?**  
A: No! This script dynamically spawns tankers. You don't need to place any tanker units in the ME.

**Q: Can I use this with other scripts (CTLD, Skynet, etc.)?**  
A: Yes! Fully compatible. Just ensure MOOSE loads first, then all scripts.

**Q: Does this work in multiplayer?**  
A: Yes! All players see the same tankers and can refuel from them.

**Q: Can I add more tanker types (KC-10, S-3B)?**  
A: Yes! Copy the KC135 config block, change `aircraftType` to desired aircraft, add menu entries.

**Q: What if I want 3+ tankers at once?**  
A: Add more tanker configs in TANKER_CONFIG section, create corresponding menu entries and spawn functions.

---

### Custom Route Questions

**Q: Can I make a one-waypoint orbit?**  
A: Yes! Place one marker. Tanker spawns and enters circular orbit at that location.

**Q: What's the maximum number of waypoints?**  
A: Default is 10 (configurable via `maxWaypoints` in ROUTE_CONFIG).

**Q: Can I use decimal flight levels?**  
A: Use whole numbers only. FL220 = 22,000 feet. For 22,500 feet, use FL225.

**Q: Do markers need to be in order?**  
A: No! Script automatically sorts by number. SHELL3, SHELL1, SHELL2 works fine.

**Q: Can I skip numbers (SHELL1, SHELL3, SHELL5)?**  
A: Yes! Script only cares about sequence, not consecutive numbering.

---

### Fuel Management Questions

**Q: Can I change fuel warning thresholds?**  
A: Yes! Edit `fuelWarningPercent` and `fuelBingoPercent` in TANKER_CONFIG.

**Q: What happens at bingo fuel?**  
A: Tanker broadcasts warning, flies to nearest friendly airbase, lands, respawns after delay.

**Q: Can I disable auto-respawn?**  
A: Not directly, but set `respawnDelay` to very high number (e.g., 99999) to effectively disable.

**Q: How do I check tanker fuel mid-mission?**  
A: F10 → Tanker Operations → Tanker Status Report

---

### Rerouting Questions

**Q: Can I reroute multiple times?**  
A: Yes! Place new markers and reroute as many times as needed.

**Q: What if I reroute to only one waypoint?**  
A: Tanker flies there and enters circular orbit.

**Q: Can I reroute one tanker while another is spawning?**  
A: Yes! Each tanker is independent.

**Q: Does rerouting affect fuel state?**  
A: No, fuel state continues from current level.

---

### Emergency Spawn Questions

**Q: What's different about emergency spawn?**  
A: Only the respawn delay (60s vs 180s). Otherwise identical to normal custom route spawn.

**Q: Can I emergency spawn without markers?**  
A: No, emergency spawn requires custom route markers.

**Q: Why can't I emergency spawn a default tanker?**  
A: Emergency spawn is designed for custom routes. Use default spawn for simple launches.

---

## Credits & License

**Author:** F99th-TracerFacer  
**Framework:** [MOOSE (FlightControl-Master)](https://github.com/FlightControl-Master/MOOSE)  
**License:** Free to use and modify. Credit appreciated but not required.

---

## Contributing

Found a bug? Have a feature request?

- **GitHub Issues**: [Report Here](https://github.com/iTracerFacer/Moose_Tanker/issues)
- **Pull Requests**: Contributions welcome!

---

## Version History

**Latest Version:** Check [GitHub Releases](https://github.com/iTracerFacer/Moose_Tanker/releases)

---

## Support

- **Discord**: [F99th Squadron Discord](#) *(Insert link if available)*
- **GitHub**: [Issues & Discussions](https://github.com/iTracerFacer/Moose_Tanker)
- **DCS Forums**: [ED Forums Thread](#) *(Insert link if available)*

---

## Final Notes

This script is designed to be:
- **Mission Editor Friendly**: No complex setup required
- **Player Friendly**: Intuitive F10 menu system
- **Mission Designer Friendly**: Highly configurable for any scenario
- **Multiplayer Ready**: Tested in dedicated server environments

**Remember:** MOOSE.lua must ALWAYS load before Moose_Tanker.lua!

Happy refueling! ✈️⛽

---

*For the latest version and updates, visit: https://github.com/iTracerFacer/Moose_Tanker*
