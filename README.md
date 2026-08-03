# Sector 07 — Grid Expansion

An isometric cyberpunk city-builder / defense game that runs in a single HTML file. You operate a
megacity sector: build nodes, wire them into a network, expand outward through six rings, and hold off
intrusions that drain your credits and bleed power off the grid.

Rendered with Three.js in a wireframe-vector style — Ghost in the Shell tactical scope, phosphor and neon.

```
index.html      the game
hud-demo.html   the original non-interactive HUD diorama it grew out of
```

## Running it

No build step, no install, no server required. Open `index.html` in a browser.

```bash
start index.html
```

It needs network access on first load — Three.js, Tailwind and the fonts come from CDNs. Everything else
(audio, icons, geometry) is generated at runtime.

## The goal

Structures earn **credits (⏣)**. Hostiles try to take them off you. Spend credits to claim the next ring
of land from the core spire. **Fill all 117 plots across all 6 rings to complete the grid.**

## The loop

**Earn** — nodes pay out per second; click the core spire to harvest; intercept data caches riding the
transit tubes; collect bounties for purging breaches.

**Spend** — build nodes (8 classes), upgrade them through 5 tiers, add firewall layers, buy patrol drones.

**Expand** — core spire → EXPAND claims the next ring.

**Defend** — click a breached node repeatedly to purge it before its timer expires.

## Controls

| Input | Action |
|---|---|
| Drag | Rotate camera |
| Right-drag | Pan |
| Wheel | Zoom |
| Click empty pad | Open the node class picker |
| Click node | Upgrade / firewall / repair |
| Click core spire | Harvest / expand / buy drone |
| Click red node | Purge the breach (repeat to finish it) |
| `Q` `W` `E` | Ice Pulse / Overclock / Trace |
| `1` `2` `3` | Tactical Cyan / Thermal / Night Vision optics |
| `A` | Auto mode |
| `H` | Open the briefing |
| `Esc` | Close menu or briefing |

## Node classes

Each class has a distinct silhouette that gains real structure at every tier — podiums, sky-bridges,
cooling towers, buttresses, mast arrays. A node keeps its identity as it grows (seeded per-node PRNG),
so upgrading makes a building *taller and more detailed*, not different.

| Class | Yield | Power | Role |
|---|---|---|---|
| **Hab-Stack** | 1.00× | −1.0 | Baseline earner |
| **Data Monolith** | 1.80× | −2.0 | High yield, but attracts breaches (3× targeting weight) |
| **Hex Arcology** | 2.45× | −3.4 | Huge yield, huge draw |
| **Fusion Reactor** | — | **+9** | Generates power, earns nothing |
| **Sec-Bastion** | 0.45× | −1.4 | +2 ICE to every node in a 38-unit radius |
| **Needle Spire** | 0.60× | −1.2 | +15% yield to each linked node |
| **Twin Nexus** | 0.85× | −1.2 | Data caches spawn more often |
| **Drone Hangar** | 0.50× | −1.5 | +2 drone cap, +20% drone damage |

Power and yield both scale with tier.

### Adjacency

Transit tubes are the network graph, not decoration. Each node auto-links to its two nearest neighbours:

- **+8% yield per link**, up to 4 links
- **−25% if isolated** — build close together
- Spire bonuses apply through links

### Power

Reactors generate, everything else draws, the core spire supplies 16 base. Overdraw causes a
**brownout**: yield throttles to `generation / draw` (floor 25%) across the whole sector at once.

## Hostiles

Six types. Which ones exist depends on your current threat level.

| Hostile | Unlocks at | Behaviour |
|---|---|---|
| **Leech** | 0 | Drains credits per second. Firewalls block most of it. |
| **Worm** | 8 | Drains less, but **spreads to a linked node** if it times out. |
| **Siphon** | 18 | Steals no credits — **bleeds power off the grid**. Enough of them and you brown out. |
| **Ghost** | 30 | **Invisible in Tactical Cyan.** Credits draining with nothing on screen means switching optics. |
| **Blackout** | 42 | **Zeroes the node's income** and ignores firewalls entirely. Drones do double damage to it. |
| **Jammer** | 58 | **Blinds every drone in a 46-unit radius** and drains power. Manual purge only. |

Let a breach time out and it takes a lump sum, knocks the node offline, and spikes threat.

## Threat

Threat is the difficulty dial. It gates which hostiles exist, how often they arrive, how much they steal,
and how much HP they have.

`SECURE → GUARDED → ELEVATED → HIGH → SEVERE → CRITICAL → OVERRUN`

It **rises** when you lose a node or use Overclock, and **falls** when you purge breaches or simply
survive (it decays). At 100 you hit **lockdown** — oversight seizes 18% of your credits.

## Abilities

| Key | Ability | Cooldown |
|---|---|---|
| `Q` | **Ice Pulse** — 3 damage to every active breach, ghosts included | 26s |
| `W` | **Overclock** — 2× yield for 20s, +22 threat | 48s |
| `E` | **Trace** — freezes all breach timers 10s, reveals ghosts | 32s |

## Auto mode

Press `A` and the sector runs itself — harvests, repairs, buys firewalls when threat climbs, builds
reactors when power runs short, expands when it can afford to, and uses abilities. The camera drifts into
a slow orbit. Useful as a demo, or for watching the balance curve play out.

## Optics modes

- **Tactical Cyan** — classic wireframe vector
- **Thermal** — buildings coloured by tier on a heat gradient
- **Night Vision** — phosphor green, heavier scanlines, denser fog

Not cosmetic: ghost breaches are only visible outside Tactical Cyan.

## Technical notes

Single self-contained file, ES modules, no bundler.

- **Three.js 0.160.0** via import map from unpkg, plus `EffectComposer` + `UnrealBloomPass` for the neon
  bloom and `BufferGeometryUtils` for geometry merging
- **Orthographic isometric camera** with damped `OrbitControls`
- **Merged geometry** — every building collapses to ~3 draw calls (one fill, one wireframe, one per accent
  colour) regardless of detail, which is what makes tier-5 towers affordable
- **Organic city layout** — plots are placed by rejection sampling inside ring-shaped bands with a minimum
  separation of 21 units; streets are a proximity graph over those plots
- **Particle systems** — 1400 citizens and 260 vehicles as `Points` flowing along the street graph, plus
  ~1100 data packets riding the transit tubes
- **Web Audio API** — every sound is synthesised at runtime (oscillators, filtered noise bursts, an
  LFO-modulated reactor drone). No audio files.
- **Inline SVG icons** — no emoji anywhere in the UI, which keeps font-metric reflow out of the HUD
- **CSS custom properties** drive the whole colour theme, so optics modes retint the interface too

### Tuning

Every balance constant lives in the `BAL` object at the top of the script — costs, growth curves, yields,
steal rates, timers, drone stats, crowd counts. Threat tiers are in `THREAT_TIERS`, hostiles in `BTYPE`,
node classes in `ROLES`.

### Dependencies

| Dependency | Used by |
|---|---|
| Three.js 0.160.0 (unpkg) | both files |
| Google Fonts — Orbitron, Share Tech Mono | both files |
| Tailwind CDN | `hud-demo.html` only — `index.html` loads it but no longer uses any utility classes |

Requires a browser with WebGL2, ES modules and import maps (any current Chrome, Edge, Firefox or Safari).
Audio starts muted; enable it from the gear menu.
