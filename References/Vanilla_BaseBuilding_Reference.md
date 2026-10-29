# Vanilla Base Building – Reference (damage / destroy flow)

Source inspected: `BohemiaInteractive/DayZ-Script-Diff` (default branch, 2026-10-29).
Only files listed below were actually read.

## Files read

| File | Content |
|------|---------|
| `scripts/4_world/entities/itembase/basebuildingbase.c` | `BaseBuildingBase` – sync bits, OnPartBuilt/Dismantled/DestroyedServer, persistence |
| `scripts/4_world/entities/itembase/basebuildingbase/fence.c` | `Fence` – gate, `IsLocked()`, `GetCombinationLock()`, `GetCodeLock()` |
| `scripts/4_world/entities/itembase/basebuildingbase/watchtower.c` | `Watchtower` |
| `scripts/4_world/entities/itembase/basebuildingbase/totem.c` | `TerritoryFlag` – refresher, `GameConstants.REFRESHER_RADIUS` |
| `scripts/4_world/classes/basebuilding/construction.c` | `Construction : ConstructionBase` – BBB specific build/dismantle/destroy |
| `scripts/4_world/classes/constructionbase.c` | `ConstructionBase`, `ConstructionMaterialType`, `OnHealthLevelChanged` |
| `scripts/4_world/classes/basebuilding/constructionpart.c` | `ConstructionPart` and rebuild variants |
| `scripts/4_world/classes/useractionscomponent/actions/continuous/actiondestroypart.c` | `ActionDestroyPart` (melee raid with tools) |
| `scripts/4_world/classes/useractionscomponent/actions/continuous/actiondestroycombinationlock.c` | `ActionDestroyCombinationLock` (marked **DEPRECATED**) |
| `scripts/4_world/entities/itembase/codelock.c` | `DigitalCodeLock` (vanilla code lock, battery, brute-force protection) |
| `scripts/4_world/entities/explosivesbase.c` + `explosivesbase/*.c` | `ExplosivesBase`, `Plastic_Explosive`, `ImprovisedExplosive` |
| `scripts/3_game/entities/object.c` | `EEOnDamageCalculated` |
| `scripts/3_game/entities/entityai.c` | `EEHitBy`, `EEHealthLevelChanged`, `ProcessInvulnerabilityCheck` |

## Damage → destroy chain (verified)

```
Engine hit / explosion
  -> Object.EEOnDamageCalculated(damageResult, damageType, source, component, dmgZone, ammo, modelPos, speedCoef)
       return false = damage is NOT applied
  -> damage applied to zone
  -> EntityAI.EEHitBy(...)                                (after damage)
  -> EntityAI.EEHealthLevelChanged(old, new, zone)
       -> GetConstructionBasic().OnHealthLevelChanged(...)   (entityai.c:1098)
  -> ConstructionBase.OnHealthLevelChanged
       if newLevel == STATE_RUINED && IsPartConstructed(zone)
           DestroyPartServer(null, zone, AT_DESTROY_PART)
  -> Construction.DestroyPartServer
       -> BaseBuildingBase.OnPartDestroyedServer(player, part, action)
       -> DestroyConnectedParts(part)  (dependent parts fall too)
  -> if part.IsBase() -> DestroyConstruction() -> ObjectDelete
```

Important facts:

- Damage zone name == construction part name (lowercased in `OnHealthLevelChanged`).
  `DamageSystem.GetDamageZoneFromComponentName(ent, part_name, zone)` maps part -> zone.
- `BaseBuildingBase.EEHealthLevelChanged` returns early while `m_FixDamageSystemInit`.
- Server setting `disableBaseDamage` (cfggameplay) -> `SetAllowDamage(false)` in
  `ProcessInvulnerabilityCheck`. A raid mod must NOT rely on this flag, it makes the
  whole base invulnerable permanently.
- `ActionDestroyPart`: 4 cycles, each removes `MaxHealth/CYCLES` from zone, then
  `DestroyPartServer` when health < 1. Tool loses `UADamageApplied.DESTROY`.
- `ConstructionBase.GetMaterialType(part_name)` returns `ConstructionMaterialType`
  (`MATERIAL_LOG, WOOD, STAIRS, METAL, WIRE, BRICK, RUBBLE`).
- `Fence.IsLocked()` checks both `CombinationLock` and `DigitalCodeLock`.
- `DigitalCodeLock.UnlockOnServer(parent)` drops the lock to ground.
  It has `HasBruteForceProtection()` / `GetWrongCodeCount()`.
- `ActionDestroyCombinationLock` is deprecated in current vanilla.
- `TerritoryFlag` uses `GameConstants.REFRESHER_RADIUS` for CE lifetime refresh.
- Persistent ID: `EntityAI.GetPersistentID(out b1, b2, b3, b4)`.
