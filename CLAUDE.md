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
- Basic Attack "Claw Swipe": 12 dmg, 0.8s cooldown, 6 stud range, Physical
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
- `StarterPlayer/StarterPlayerScripts/GriffinCombatClient.lua` (LocalScript) — input handling (Q=basic, E=pounce)
- Boss model "CanopyWarden" in Workspace has a Script `BossEncounter_CanopyWarden.lua` as a sibling of its Humanoid

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
