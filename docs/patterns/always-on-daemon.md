# Always-On Daemon Pattern

Build a background service that runs scheduled tasks, monitors for events, and optionally bridges mobile access.

## Why You'd Want This

MARVIN is powerful during active sessions, but some tasks need to happen whether you're in a session or not:
- Scanning for new CFP deadlines daily
- Monitoring email for urgent items
- Running event trackers on a schedule
- Bridging mobile access to your desktop MARVIN instance

A daemon handles these background tasks so MARVIN stays proactive even when you're not actively chatting.

## Architecture

```
┌──────────────────────────────────┐
│           Daemon Process         │
│                                  │
│  ┌──────────┐  ┌──────────────┐  │
│  │Scheduler │  │ Relay Bridge │  │
│  │          │  │ (optional)   │  │
│  │ ┌──────┐ │  │              │  │
│  │ │Scan 1│ │  │  Mobile ←──→ │  │
│  │ │Scan 2│ │  │  Desktop     │  │
│  │ │Scan 3│ │  │              │  │
│  │ └──────┘ │  └──────────────┘  │
│  └──────────┘                    │
│                                  │
│  ┌──────────────────────────┐    │
│  │    State Management      │    │
│  │  heartbeat, trigger state│    │
│  └──────────────────────────┘    │
└──────────────────────────────────┘
```

### Components

**1. Scheduler**
A cron-like scheduler that runs tasks at defined intervals:
- CFP scanner: daily at 7:30am
- Email monitor: every 30 minutes during work hours
- Event tracker: continuous or hourly
- Health check: every 5 minutes (writes heartbeat)

Implementation options:
- Python `schedule` library (simple, single-process)
- `APScheduler` (more features, job persistence)
- System cron + wrapper scripts (simplest, most reliable)
- `launchd` (macOS) or `systemd` (Linux) for process management

**2. Scanners**
Each scanner is a focused task:
- **CFP Scanner**: Scrapes known CFP aggregator sites, checks deadlines, flags upcoming ones
- **Email Monitor**: Checks for P1 emails, sends desktop notification or mobile alert
- **Event Tracker**: Monitors event registration platforms for new events matching your interests
- **Content Monitor**: Tracks RSS feeds, forums, or social mentions

**3. Relay Bridge (Optional)**
Bridges your phone to your desktop MARVIN instance:
- WebSocket connection between mobile client and daemon
- Mobile sends messages, daemon routes to MARVIN session
- Responses sent back to mobile
- Conversations persisted for desktop continuity

**4. State Management**
The daemon maintains its own state:
- `state/heartbeat.md`: last alive timestamp, current status
- `state/daemon-config.json`: schedule, relay URL, feature flags
- `state/trigger-state.json`: last run time per scanner, results cache

## Key Decisions

| Decision | What we learned |
|----------|----------------|
| Separate process from MARVIN | Daemon runs independently; MARVIN sessions come and go |
| Heartbeat file | Simple health monitoring without complex infrastructure |
| State JSON files | Easy to inspect and debug, no database needed |
| System process manager | Let launchd/systemd handle restarts, not your code |
| Scanner isolation | Each scanner is independent; one failing doesn't break others |
| Optional relay | Not everyone needs mobile; keep it decoupled |

## Directory Structure

```
src/daemon/
├── main.py              # Entry point, scheduler setup
├── scheduler.py         # Job scheduling logic
├── scanners.py          # Individual scanner implementations
├── relay_client.py      # WebSocket relay (optional)
├── config.py            # Daemon configuration
└── encryption.py        # Sensitive data handling

state/
├── heartbeat.md         # Daemon health status
├── daemon-config.json   # Daemon configuration
└── trigger-state.json   # Scanner state (last run, results)

logs/
└── daemon.log           # Daemon output log
```

## Process Management

### macOS (launchd)

Create a plist at `~/Library/LaunchAgents/com.marvin.daemon.plist`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "...">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.marvin.daemon</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/python3</string>
        <string>/path/to/marvin/src/daemon/main.py</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/path/to/marvin/logs/daemon.log</string>
</dict>
</plist>
```

### Linux (systemd)

Create a unit file at `~/.config/systemd/user/marvin-daemon.service`:
```ini
[Unit]
Description=MARVIN Daemon
After=network.target

[Service]
ExecStart=/usr/bin/python3 /path/to/marvin/src/daemon/main.py
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

### Management Commands

Build a `/daemon` command with subcommands:
```bash
/daemon start    # Start the daemon
/daemon stop     # Stop the daemon
/daemon restart  # Restart
/daemon status   # Check health (read heartbeat)
/daemon logs     # Tail recent logs
```

## Getting Started

1. Decide which scanners you need (start with 1-2)
2. Write scanner functions that run independently
3. Set up a scheduler (Python `schedule` or system cron)
4. Add a heartbeat writer (timestamp every 5 minutes)
5. Create a `/daemon` command for management
6. Install as a system service (launchd or systemd)
7. Add daemon health to your `/start` briefing
8. Optionally: add the relay bridge for mobile access

## Scaling Up

Start simple (cron + scripts) and add complexity only when needed:
1. **Level 1**: System cron runs scripts on a schedule
2. **Level 2**: Python daemon with built-in scheduler
3. **Level 3**: Add relay bridge for mobile
4. **Level 4**: Add state persistence and scanner coordination
