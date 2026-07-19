# Anomaly Biomes — Project Context

Roblox game (Luau). Players choose an anomaly "coat" and biome, gather/trade/fight
through a level system. Core educational themes: trade-offs, interdependence,
cooperation — taught through mechanics, not lectures.

## Status
Currently building ONE coat (Griffin) + ONE biome (Jungle) as a vertical-slice
prototype before touching Sphinx/Desert or Selkie/Tundra.

## Coat/Biome Trinity (full design, for later reference)
- **Griffin** (lion+eagle) — Jungle — Physical/aggressive/aerial — IN PROGRESS
- **Sphinx** (lion+eagle wings+human head) — Desert — Magical/tactical/puzzle — not started
- **Selkie** (seal↔human shapeshifter, two forms) — Tundra — Summon/Nature/support — not started

## Griffin — locked stats (draft, pending balance pass once other coats exist)
- maxHealth: 120
- baseMoveSpeed: 16 studs/s neutral
- Basic Attack "Claw Swipe": 12 dmg, 0.5s cooldown, 6 stud range, Physical
- Special "Aerial Pounce": 25 dmg, 8s cooldown, 20 stud leap, 6 stud AoE radius, Physical
- In-Jungle: +15% move speed, +25% gather speed, +50% gather yield
- Out-of-biome: -15% move speed, -25% gather speed, -25% gather yield, +50% resource consumption, +15% damage taken

## Jungle biome
- Local goods (cheap here): MosquitoRepellant, Machete, RainGear
- Scarce imports (valuable here): SaltPreservative (Desert), Firewood (Tundra), Water
- Boss: "Canopy Warden" — bark/vine armor blocks ~95% of Physical damage.
  Cross-biome item "Sunstone" (Desert good) ignites vines → 6s vulnerability
  window where Physical damage is amplified 1.5x. This is the forced-trade
  moment that makes the cross-biome economy non-optional.

## Design rule: bosses carry resistance tags, regular mobs do not
Keeps the physical/magical/support type-matchup lesson concentrated at boss
fights, which already gate cross-biome trade — don't add resistance tags to
regular enemies without discussing it first.

## Existing file structure (already built manually in Studio, now the source of truth)
- `ReplicatedStorage/GameData/CoatStats.lua` (ModuleScript) — stat blocks per coat
- `ReplicatedStorage/GameData/BiomeData.lua` (ModuleScript) — biome resource/import data
- `ServerScriptService/RemoteSetup.lua` (Script) — creates RemoteEvents on startup
- `ServerScriptService/GriffinCombatServer.lua` (Script) — server-authoritative combat logic
  (Claw Swipe alternates 2 animations via a per-player combo counter; Aerial Pounce does an
  animated leap rather than teleporting)
- `StarterPlayer/StarterPlayerScripts/GriffinCombatClient.lua` (LocalScript) — input handling (Q=basic, E=pounce)
- Boss model "CanopyWarden" in Workspace has a Script `BossEncounter_CanopyWarden.lua` as a sibling of its Humanoid
- `ReplicatedStorage/GriffinAbilityState.lua` (ModuleScript) — shared cooldown-end timestamps read by both the keyboard client and the combat GUI
- `StarterGui/GriffinCombatGui` (ScreenGui) with `GriffinCombatGuiClient` (LocalScript) — Claw/Pounce buttons, cooldown display, damage popups
- `ServerScriptService/GriffinAppearanceServer.lua` (Script) — auto-equips Griffin's wing accessory on spawn from the master copy in `ServerStorage/GriffinCosmetics`

## Recent changes (2026-07-19)
- Built and wired the combat GUI: Claw/Pounce buttons in `StarterGui`, with cooldown
  display kept in sync between mouse clicks and Q/E keyboard input via the new shared
  `GriffinAbilityState` module, plus server-reported damage popups.
- Added `GriffinAppearanceServer.lua` so every player spawns with Griffin's wing
  accessory auto-equipped (master copy lives in `ServerStorage/GriffinCosmetics`).
- **Avatar settings switched to R6-only** for the whole experience, reversing the earlier
  plan to keep the game on R15 and re-author animations for that rig instead. All Griffin
  animations must now be authored/verified against an R6 rig.
- Claw Swipe cooldown reduced from 0.8s to 0.5s (`CoatStats.lua`, single source of truth as
  usual).
- Claw Swipe now alternates between two animations per hit, purely for visual variety
  (damage/range unchanged): odd hits play `rbxassetid://80999690226550` (Attack1), even hits
  play `rbxassetid://101789603090719` (Attack2 — same asset as the original single Claw Swipe
  clip). Tracked with a per-player combo counter in `GriffinCombatServer.lua` that resets after
  ~2s of no attacking. Both clips measured well under the 0.5s cooldown (0.283s / 0.267s), so
  hits don't visually cut each other off.
- Aerial Pounce (`rbxassetid://79465333887097`) confirmed animating correctly in real Play-mode
  testing on R6. It now animates the leap over ~0.6s (matching the clip's length) with a
  sine-based visual arc, instead of teleporting the character instantly to the landing spot —
  the root part is briefly anchored during the leap so physics doesn't fight the manual
  movement, then handed back to the Humanoid on landing, with AoE damage applied at that point.
  The existing "land short of a target instead of overshooting" targeting logic is unchanged.
- Folded in the resistance/vulnerability damage formula more correctly: while a boss's
  `<DamageType>VulnerabilityMult` attribute is > 1 (Sunstone-ignited), resistance is bypassed
  entirely rather than stacking multiplicatively on top of it — matches the "armor burned away"
  framing of the Canopy Warden design rather than being a minor buff on top of 95% resistance.
- **Gotcha discovered**: testing animation playback via a synthetic Edit-mode rig
  (`Players:CreateHumanoidModelFromDescription` + `Animator:LoadAnimation`, sampled outside Play
  mode) reported zero joint motion for all three Griffin animations, including Aerial Pounce —
  which the user then confirmed plays fine in real Play mode. That test methodology gives false
  negatives outside of actual Play mode; don't trust it for animation verification. Use real
  Play-mode testing (or the joint-CFrame method, run during Play) instead.
- **Near-miss, no actual data lost**: `ServerScriptService` briefly appeared completely empty
  after a Studio dual-session reconnect, which looked like data loss. It wasn't — a stale MCP
  index right after reconnect was the cause, confirmed once `RemoteSetup.lua` /
  `GriffinAppearanceServer.lua` / `GriffinCosmetics` all reappeared on a second check. Lesson:
  re-check after a Studio reconnect before concluding anything is actually missing.

## Open questions — do not silently decide these, ask first
1. Selkie's solo-encounter drawback: hard DPS check vs. soft (slow but possible)?
2. Does Selkie support scale with number of nearby allies?
3. Flat vs. fluctuating AI trader exchange rates?
4. Full stat blocks for Sphinx and Selkie — not yet designed.

## Working conventions
- Server-authoritative: never trust client-sent damage numbers or cooldown timing.
- Keep stat numbers in `CoatStats.lua` / `BiomeData.lua` as the single source of
  truth — don't hardcode balance numbers inside combat/boss scripts.
- This is a prototype: prioritize a working vertical slice over polish/edge cases.
