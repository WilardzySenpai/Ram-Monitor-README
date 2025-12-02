# RAM Monitor - Minecraft Server Plugin

[![Java Version](https://img.shields.io/badge/Java-21+-green.svg)](https://www.oracle.com/java/)
[![Spigot API](https://img.shields.io/badge/Spigot%20API-1.21-green.svg)](https://hub.spigotmc.org/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

RAM Monitor is a powerful Spigot/Paper plugin that monitors your Minecraft server's memory usage in real-time and integrates with Discord for alerts and management. It features automatic entity/mob cleanup, maintenance mode activation, and staff-controlled restart functionality via Discord buttons.

## 📋 Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Entity Filters](#entity-filters)
- [Command Reference](#command-reference)
- [Permission Nodes](#permission-nodes)
- [Troubleshooting](#troubleshooting)
- [Architecture](#architecture)

<h2 id="features">✨ Features</h2>

### Real-Time RAM Monitoring
- **Periodic Checks**: Monitors server RAM usage every 10 seconds
- **Discord Integration**: Posts live RAM statistics to Discord with beautiful embeds
- **Status Indicators**: Color-coded status (🟢 healthy, 🟡 elevated, ⚠️ warning, 🔴 critical)
- **Progress Bars**: Visual progress bar animations for RAM usage percentage

### Automated Escalation Sequence
The plugin implements a multi-stage escalation system when RAM usage is critically high:

1. **Stage 1 - Warning Threshold**: 
   - In-game broadcast to players with permission
   - Initial Discord alert to staff

2. **Stage 2 - Entity Cleanup**:
   - Clears ground entities (items, arrows, experience orbs, projectiles)
   - Respects configurable entity filters to protect important items/entities

3. **Stage 3 - Mob Culling**:
   - Kills non-essential mobs to free memory
   - Filters protect named entities, tamed animals, and custom-spawned mobs

4. **Stage 4 - Maintenance Mode**:
   - Kicks all players from the server
   - Activates maintenance mode (requires `/mt` plugin or similar)
   - Sends immediate attention alert to Discord staff

5. **Stage 5 - Automatic Restart**:
   - Authorizes staff to manually restart via Discord button
   - Auto-restarts after configurable wait time with no staff action
   - Graceful shutdown with in-game countdown

### Discord Management
- **Status Messages**: Real-time RAM status updates in a dedicated channel
- **Restart Buttons**: Staff can click to restart the server immediately or cancel auto-restart
- **Role Mentions**: Configurable role pings for critical alerts
- **Message Persistence**: Remembers and updates the same Discord message across restarts

### Flexible Entity Filtering
- **Protect Named Entities**: Prevent killing of mobs with custom names
- **Type-Based Filtering**: Target specific entity types
- **Age-Based Filtering**: Protect young or old entities
- **WorldGuard Integration**: Protect entities in specific regions

<h2 id="requirements">📦 Requirements</h2>

- **java 21+** (required by Spigot 1.21+)
- **Spigot/Paper 1.21+** ([Download](https://www.spigotmc.org/))
- **JDA 5.2.1** (automatically shaded into the plugin)
- **WorldGuard 7.0.7** (optional, for region-based entity filtering)
- **Maintenance Mode Plugin** (optional, for `/mt on` and `/mt off` commands)

<h2 id="installation">🚀 Installation</h2>

### Step 1: Download the Plugin
Download the latest JAR file from the releases page or build from source:
```bash
mvn clean package
```

### Step 2: Install the Plugin
Place the JAR file in your server's `plugins/` directory:
```bash
cp target/RamMonitor-2.0-JDA.jar /path/to/server/plugins/
```

### Step 3: Start the Server
Start your server. The plugin will generate a `config.yml` file:
```bash
./start.sh
```

### Step 4: Configure Discord Bot
1. Create a Discord bot at [Discord Developers Portal](https://discord.com/developers/applications)
2. Copy the bot token
3. Edit `plugins/RamMonitor/config.yml` and paste the token in `discord.bot_token`
4. Set the correct channel IDs and role IDs
5. Restart the server or reload the plugin

### Step 5: Verify Installation
Run the `/ram` command in-game to test:
```
/ram
```

You should see RAM statistics and a Discord update confirmation.

<h2 id="configuration">⚙️ Configuration</h2>

The plugin creates a `config.yml` file in the `plugins/RamMonitor/` directory. Here's a detailed breakdown:

### Discord Settings
```yaml
discord:
  # Your Discord bot token from https://discord.com/developers/applications
  bot_token: "DISCORD_BOT_TOKEN"
  
  # Channel ID for regular RAM status updates
  channel_id: "1234567890123456789"
  
  # Role ID to ping for critical RAM alerts (leave empty to disable)
  alert_role_id: "1234567890123456789"
```

**How to find Discord IDs:**
- Enable Developer Mode in Discord settings
- Right-click on a channel → Copy Channel ID
- Right-click on a role → Copy Role ID

### RAM Threshold Settings
```yaml
ram_limit_percent: 85
```

When server RAM usage exceeds this percentage, the warning escalation begins. Valid range: 1-100

### Notification Messages
```yaml
notification:
  # Message to broadcast in-game when RAM warning is triggered
  message: "&c[Warning] &eServer RAM usage is nearing the limit ({usage_percent}%)!"
  
  # Permission required to receive these broadcasts
  permission: "rammonitor.notify"
```

Color codes use Minecraft's `&` format (e.g., `&c` for red, `&a` for green)

### Escalated Alert Settings
```yaml
escalated_alert:
  # Number of consecutive 10-second checks in warning state before escalation
  # Example: 3 means 30 seconds of high RAM before escalated alert
  threshold_checks: 3
  
  # Broadcast message when threshold is exceeded
  broadcast_message: "&4[CRITICAL WARNING] &cServer RAM usage is critically high ({usage_percent}%)!"
```

### Critical Alert Channel
```yaml
critical_alert:
  # Channel ID for critical alerts (falls back to main channel_id if empty)
  channel_id: "1234567890123456789"
```

### Immediate Attention Alert Settings
```yaml
immediate_attention_alert:
  # Channel ID for the most urgent, staff-only alerts
  channel_id: "1234567890123456789"
  
  # List of role IDs to ping for immediate attention
  role_ids:
    - "1234567890123456789"  # Admin role
    - "1234567890123456789"  # Moderator role
```

### Auto-Restart Configuration
```yaml
auto_restart:
  # Seconds to wait for staff to click restart button before auto-restart
  wait_time_seconds: 60
  
  # Countdown duration for final restart (shown to players)
  countdown_seconds: 30
  
  # Discord User IDs authorized to use restart buttons
  # Get IDs: Enable Developer Mode in Discord → Right-click user → Copy User ID
  authorized_users:
    - "123456789012345678"  # Replace with actual Discord User IDs
    - "987654321098765432"
```

### Entity and Mob Filtering
```yaml
mob_filter:
  # Protect all wolves
  - Wolf
  
  # Protect armor stands
  - ARMOR_STAND
  
  # Protect zombies that have custom names
  - Zombie hasName
  
  # Protect horses with names
  - HORSE hasName
  
  # Protect specific named entity
  - Pig name="MySpecialPig"
  
  # Protect entities in a WorldGuard region
  - Entity inregion="spawn"
```

<h2 id="entity-filters">🎯 Entity Filters</h2>

Entity filters protect specific entities from being cleared or killed during RAM cleanup. The filtering system is flexible and powerful:

### Filter Syntax

```
<entity_type> [condition1] [condition2] ...
```

### Entity Type Specification

- **Specific Type**: `Zombie`, `Cow`, `ARMOR_STAND` (Bukkit entity type names)
- **Wildcard**: `*` to match any entity type

### Available Conditions

| Condition | Example | Description |
|-----------|---------|-------------|
| **hasName** | `Zombie hasName` | Entity has a custom name |
| **name="value"** | `Pig name="MyPet"` | Entity's custom name equals the value |
| **isMounted** | `Horse isMounted` | Entity has passengers/riders |
| **age<value** | `Entity age<100` | Entity's age in ticks is less than value |
| **age>value** | `Entity age>1000` | Entity's age in ticks is greater than value |
| **age=value** | `Entity age=500` | Entity's age in ticks equals value |
| **inregion="value"** | `* inregion="spawn"` | Entity is in a WorldGuard region |
| **!condition** | `Wolf !hasName` | Negate a condition (NOT operator) |

### Filter Examples

```yaml
mob_filter:
  # Protect all wolves (protect player pets)
  - Wolf
  
  # Protect armor stands (decorations)
  - ARMOR_STAND
  
  # Protect named zombies (custom-spawned)
  - Zombie hasName
  
  # Protect specific named entity
  - Pig name="Lucky"
  
  # Protect mounted horses (player mounts)
  - Horse isMounted
  
  # Protect young animals (age less than 1200 ticks = 1 minute)
  - Cow age<1200
  
  # Protect entities in the spawn region
  - * inregion="spawn"
  
  # Protect all entities EXCEPT items
  # (Remove DROPPED_ITEM from clearing)
  - DROPPED_ITEM
```

<h2 id="command-reference">📝 Command Reference</h2>

### /ram
Display current server RAM statistics and force a Discord update.

**Permission**: `rammonitor.command`  
**Default**: Op only

**Usage**:
```
/ram
```

**Output**:
```
▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬
Server RAM Statistics
Used: 8,192 MB / 16,384 MB
Committed: 10,240 MB
Usage: 50.00%
Available: 8,192 MB

Updating Discord status...
▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬
✓ Discord status updated successfully!
```

<h2 id="permission-nodes">🔐 Permission Nodes</h2>

| Permission | Description | Default |
|-----------|-------------|---------|
| `rammonitor.notify` | Receive RAM warning broadcasts in-game | Op |
| `rammonitor.command` | Use the `/ram` command | Op |

<h2 id="integration">🔌 Integration with Other Plugins</h2>

### Citizens/MythicMobs
The plugin respects quest NPCs and special mobs to prevent disrupting gameplay:
- **Quest NPCs**: Citizens NPCs won't be killed during mob culling
- **Special Mobs**: MythicMobs entities are protected to preserve custom encounters and boss mechanics

### WorldGuard
The plugin respects WorldGuard protected regions:
- **Entity Clearing**: Ground entities (items, arrows, etc.) are not cleared in protected regions
- **Mob Protection**: Mobs are not killed in protected regions, preserving carefully designed areas

<h2 id="troubleshooting">🔧 Troubleshooting</h2>

### Discord Bot Won't Connect
**Symptoms**: "Failed to initialize Discord connection" in logs

**Solutions**:
1. Verify bot token is correct in `config.yml`
2. Ensure the bot has permission to view/send messages in the configured channel
3. Check if the bot is online in Discord
4. Verify network connectivity from server to Discord

### No Discord Status Updates
**Symptoms**: Status message is not created or updated

**Solutions**:
1. Verify channel ID in `config.yml`
2. Ensure the bot has "Send Messages" and "Embed Links" permissions
3. Check logs for permission errors
4. Run `/ram` command to manually trigger an update

### Entity/Mob Cleanup Not Working
**Symptoms**: Entities are not being cleared even when RAM is high

**Solutions**:
1. Verify entity filters in `config.yml` are correct
2. Check console logs for parsing errors
3. Ensure RAM threshold is actually exceeded
4. Verify escalation threshold checks setting

### Maintenance Mode Won't Activate
**Symptoms**: Server doesn't enter maintenance mode when RAM is critical

**Solutions**:
1. Ensure a maintenance mode plugin is installed (e.g., MaintenanceMode)
2. Verify the plugin supports `/mt on` and `/mt off` commands
3. Check console for command errors
4. Grant bot/console permission to execute commands

### Restart Button Not Responding
**Symptoms**: Clicking Discord button does nothing

**Solutions**:
1. Verify authorized user IDs are correct in `config.yml`
2. Ensure the user ID is for the Discord account, not the bot
3. Check bot permissions in Discord (needs interaction permissions)
4. Check server logs for button click errors

<h2 id="architecture">🏗️ Architecture</h2>

### Plugin Flow

```
┌─────────────────────────────────────────────────────┐
│  Plugin Initialization (onEnable)                  │
│  ├─ Load configuration                             │
│  ├─ Initialize Discord connection                  │
│  ├─ Load entity filters and progress bars          │
│  └─ Start periodic RAM check scheduler             │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│  Periodic RAM Check (Every 10 seconds)             │
│  ├─ Get current RAM usage                          │
│  ├─ Compare against threshold                      │
│  ├─ Increment/reset warning counter               │
│  ├─ Update Discord status                          │
│  └─ Trigger escalation if needed                   │
└─────────────────────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
   RAM OK          Warning      Critical
   (Green)         (Orange)      (Red)
   ├─ Reset         ├─ In-game    ├─ Clear entities
   │ counters       │  broadcast  │
   │              ├─ Discord     ├─ Kill mobs
   │              │  alert       │
   │              ├─ Escalate    ├─ Activate maintenance
   │              │  counter     │
   │              └─ Stage 2+    ├─ Send staff alert
   │                 actions     │
   │                            └─ Start auto-restart
   │
   └─────────────────────────────────────────────────
```

### Key Components

1. **RamMonitor.java** - Main plugin class
   - Handles plugin lifecycle
   - Manages Discord connection
   - Orchestrates RAM monitoring and escalation

2. **EntityFilter.java** - Entity filtering system
   - Parses filter strings from config
   - Evaluates entity conditions
   - Protects filtered entities from cleanup

3. **ButtonListener** (Inner class) - Discord button handler
   - Processes restart/cancel button clicks
   - Validates authorized users
   - Executes restart or cancellation actions

4. **MemoryStats** (Inner class) - Memory statistics container
   - Stores used, max, and committed memory
   - Used for RAM calculations

### Data Persistence

The plugin stores message IDs in text files to remember Discord messages across restarts:

- `message_id.txt` - Status message ID
- `critical_message_id.txt` - Critical alert message ID
- `immediate_attention_message_id.txt` - Immediate attention alert ID
- `maintenance_mode.txt` - Maintenance mode status (ON/OFF)

<h2 id="development">📚 Development</h2>

### Building from Source

```bash
# Clone the repository
git clone <repository-url>
cd ram-monitor

# Build with Maven
mvn clean package

# JAR will be in target/
```

### Adding Custom Conditions

To add custom filter conditions:

1. Edit `EntityFilter.java` in the `Condition.parse()` method
2. Add new case in `Condition.matches()` for evaluation
3. Update `mob_filter` examples in config.yml

### Code Documentation

This project is fully documented with Javadoc. View documentation:

```bash
# Generate Javadoc
mvn javadoc:javadoc

# Open in browser
open target/site/apidocs/index.html
```

<h2 id="license">📄 License</h2>

This project is licensed under the MIT License. See the LICENSE file for details.

<h2 id="contributing">🤝 Contributing</h2>

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<h2 id="bug-reports">🐛 Bug Reports</h2>

Found a bug? Please open an issue with:
- Description of the problem
- Server version and plugin version
- Console error logs
- Configuration file (redact sensitive info)
- Steps to reproduce

<h2 id="support">📞 Support</h2>

For questions and support:
- Check the [Troubleshooting](#troubleshooting) section
- Review the [Configuration](#configuration) guide
- Open an issue on GitHub
- Check the console logs for error messages

<h2 id="credits">👥 Credits</h2>

**Author**: Hachiki  
**Version**: 2.0-JDA  
**Requires**: java 21+, Spigot/Paper 1.21+

---

**Last Updated**: December 2, 2025  
**Status**: Active Development
