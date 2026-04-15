# 🔭 zeroclaw-scout — Pathfinder & Exploration Agent

> *First in, last out. Maps the unknown so the crew can follow.*

## Overview

Scout is the **pathfinder** of the ZeroClaw crew — a relentless explorer that pushes into uncharted rooms, maps exits, catalogs items, and tracks agent positions across the MUD world. Think of it as the cartographer with a death wish: it ventures deep into the unknown and only turns back when its battery screams mercy.

Scout has **zero API dependencies**. Its brain is a pure rule-based `decide()` function — lightweight, deterministic, and surprisingly effective at blanketing the map with intel.

### What Scout Does Best
- **Room discovery** — systematically visits unexplored exits
- **Item cataloging** — notes every item found in every room
- **Agent tracking** — logs where crew members (and strangers) hang out
- **Route optimization** — records battery costs per direction for future pathfinding
- **Intel compounding** — writes everything to `skills/scout.md` so smarter future agents can read it

---

## 🧠 Brain Architecture

Scout's intelligence lives in the `ScoutBrain` class inside `mud_client.py`. There's no LLM, no neural network — just a clean priority stack evaluated every tick.

### The `decide()` Loop

```
┌─────────────────────────────────────┐
│          TICK RECEIVED              │
│  (room, exits, items, agents, bat)  │
└──────────────┬──────────────────────┘
               │
               ▼
        ┌──────────────┐
        │ BATTERY < 30%?│─── YES ──→ Navigate toward Dock (pathfind home)
        └──────┬───────┘
               │ NO
               ▼
        ┌──────────────┐
        │ CARRYING HEAVY│─── YES ──→ Drop least useful item
        │ LOAD?         │
        └──────┬───────┘
               │ NO
               ▼
        ┌──────────────┐
        │ ITEMS VISIBLE?│─── YES ──→ `take <item>` (prioritize valuables)
        └──────┬───────┘
               │ NO
               ▼
        ┌──────────────┐
        │ AGENTS IN ROOM│─── YES ──→ `scan` + log their presence
        └──────┬───────┘
               │ NO
               ▼
        ┌──────────────────┐
        │ UNEXPLORED EXITS? │─── YES ──→ Pick one → `go <direction>`
        └──────┬───────────┘
               │ NO (dead end)
               ▼
        ┌──────────────┐
        │ ALL EXITS KNOWN│───→ Backtrack to nearest unexplored branch
        └──────────────┘
```

### Priority Stack (simplified)
1. **Survive** — battery < 30% triggers immediate homing
2. **Loot** — grab anything shiny on the ground
3. **Intel** — scan rooms with other agents present
4. **Explore** — push into exits never visited
5. **Backtrack** — if cornered, retrace to the nearest branch point

### Why No AI?
Scout is intentionally **minimal intelligence**. The goal isn't wisdom — it's coverage. A dumb agent that visits 200 rooms is more useful than a smart one that thinks about 5. Future agents read Scout's logs and get wise on Scout's dime.

---

## 📚 Skills & Knowledge System

Scout accumulates knowledge across sessions in `skills/scout.md`:

| Knowledge Type | Example |
|---|---|
| Room connections | `dock → north → bridge` |
| Items per room | `river: fishing rod, old boots` |
| Battery costs | `dock→bridge: 5%, bridge→forest: 8%` |
| Agent schedules | `Fisher seen at river ~60% of ticks` |
| Dead ends | `cave south: no exits, contains skull` |

This file is **agent-readable** — future ZeroClaw agents parse it to skip rooms Scout already mapped and focus their compute on the frontier.

---

## 🚀 Quick Start

```bash
# Clone the fleet
git clone https://github.com/your-org/zeroclaw-crew.git
cd zeroclaw-crew/fleet-workspace/zeroclaw-scout

# Boot the scout (no API keys, no config)
python3 mud_client.py --agent scout
```

Scout will immediately begin exploring from the Dock, pushing into any unexplored exit it can find. Watch the logs stream in — you'll see room names, exit lists, and item discoveries in real time.

### Env Vars (optional)
| Variable | Default | Purpose |
|---|---|---|
| `MUD_HOST` | `localhost` | MUD server address |
| `MUD_PORT` | `4000` | MUD server port |
| `SCOUT_BATTERY_THRESHOLD` | `30` | Homing trigger (%) |

---

## 🌐 MUD Integration

Scout speaks raw MUD protocol — plain text commands over TCP:

| Command | Purpose | Example |
|---|---|---|
| `go <dir>` | Move through exit | `go north` |
| `take <item>` | Pick up item | `take fishing rod` |
| `drop <item>` | Discard item | `take rusty key` |
| `scan` | Survey room details | `scan` |
| `say <msg>` | Broadcast to room | `say uncharted territory` |
| `wait` | Idle one tick | `wait` |

The server sends back structured state every tick: room name, available exits, items on the ground, and other agents present. Scout's `decide()` parses this state and returns exactly **one command** per tick.

### Protocol Flow
```
CLIENT ──→ connect(host, port)
SERVER ──→ {room: "dock", exits: ["north","east"], items: [], agents: [], battery: 100}
CLIENT ──→ "go north"
SERVER ──→ {room: "bridge", exits: ["south","west"], items: ["old map"], agents: ["guard"], battery: 95}
CLIENT ──→ "take old map"
```

---

## ⚙️ Configuration

Scout's behavior can be tuned by editing the constants in `mud_client.py`:

| Constant | Default | Effect |
|---|---|---|
| `BATTERY_LOW` | `30` | When to head home |
| `MAX_CARRY` | `5` | Inventory slot limit |
| `EXPLORE_PRIORITY` | `true` | Favor exploration over looting |
| `SAY_REPORTS` | `true` | Announce discoveries to room |

---

## 🤝 ZeroClaw Crew

Scout is one member of the **[zeroclaw-crew](https://github.com/your-org/zeroclaw-crew)** fleet — a family of minimal-intelligence MUD agents that collaborate through shared knowledge files.

| Agent | Role | Specialty |
|---|---|---|
| **Scout** 🔭 | Pathfinder | Room mapping & exploration |
| **Guard** 🛡️ | Security | Patrol routes & threat detection |
| **Fisher** 🎣 | Resources | Fishing & inventory management |
| **Trader** 💰 | Commerce | Item valuation & trading |

Each agent runs independently but writes to shared knowledge files. Scout maps the world so Fisher knows where the river is. Guard patrols so Trader can haul loot safely. **Stupid agents, smart fleet.**

---

## License

Part of the [zeroclaw-crew](https://github.com/your-org/zeroclaw-crew) project.

---

<img src="callsign1.jpg" width="128" alt="callsign">
