# MD_RaidSystem – návrh raid systému pre vanilla base building

Stav: **NÁVRH** (netestované na serveri). Kód nižšie je kostra na implementáciu,
nie overený produkčný kód.

Vanilla zdroje, z ktorých návrh vychádza: `References/Vanilla_BaseBuilding_Reference.md`.

---

## 1. Čo som našiel

### V tomto repozitári
- Žiadny existujúci raid / territory / base systém (repo zatiaľ neobsahuje `My_Mods`).
- Pravidlá: server authority, `modded class`, žiadne duplicitné systémy, JSON config
  v `$profile`, logovanie s prefixom, RPC validácia (`DAYZ_KNOWLEDGE_INDEX.md`,
  `Documentation/EnScript_Style_Guide.md`).
- Poučenie z `MD_CrateEvent Double Timer`: **jeden** zdroj času – jeden timer.

### Vo vanilla DayZ (DayZ-Script-Diff, reálne prečítané súbory)
Vanilla už má celý reťazec ničenia stien – netreba ho prepisovať, iba ho **riadiť**:

```
zásah / výbuch
 -> Object.EEOnDamageCalculated(...)      // return false = damage sa neaplikuje
 -> EntityAI.EEHitBy(...)                 // po aplikovaní damage
 -> EEHealthLevelChanged -> ConstructionBase.OnHealthLevelChanged
 -> RUINED zóna => DestroyPartServer(part) => DestroyConnectedParts
 -> base part zničený => DestroyConstruction()
```

- Zóna poškodenia = názov construction partu (`wall_base_down`, `wall_wood_up`,
  `wall_metal_down`, `wall_gate`, ...).
- `ActionDestroyPart` – melee raid nástrojmi (4 cykly po `MaxHealth/4`).
- `ExplosivesBase` (Plastic_Explosive, ImprovisedExplosive) – výbuch cez ammo config.
- `Fence.IsLocked()`, `GetCombinationLock()`, `GetCodeLock()` (vanilla `DigitalCodeLock`
  s batériou a brute-force ochranou).
- `cfggameplay disableBaseDamage` robí bázu **trvalo** nezničiteľnou – na raid okná
  sa nehodí.

---

## 2. Odporúčaná architektúra

Princíp: **REUSE vanilla destroy chain + EXTEND cez dva damage hooky a jeden action hook.**
Žiadna vlastná logika ničenia partov, žiadne vlastné sync bity.

```
MD_RaidSystem/
├── config.cpp
└── scripts/
    ├── 3_Game/MD_RaidSystem/
    │   ├── MDRS_Config.c          // JSON config (dáta + load/save)
    │   ├── MDRS_Constants.c       // prefix logu, cesty, RPC ID
    │   └── MDRS_RaidWindow.c      // výpočet "je raid povolený teraz"
    ├── 4_World/MD_RaidSystem/
    │   ├── MDRS_Manager.c         // singleton: config, ownership, offline ochrana, log
    │   ├── MDRS_DamageRules.c     // multiplikátory podľa materiálu / zdroja
    │   ├── MDRS_BaseOwnership.c   // vlastníci báz (JSON podľa persistent ID vlajky)
    │   ├── BaseBuildingBase.c     // modded: EEOnDamageCalculated, EEHitBy, OnPartBuiltServer
    │   ├── ActionDestroyPart.c    // modded: podmienka raid okna + pomalší raid
    │   ├── ActionDismantlePart.c  // modded: cudzí hráč nesmie dismantle mimo raidu
    │   └── DigitalCodeLock.c      // modded: zámok sa dá zničiť len v raid okne
    └── 5_Mission/MD_RaidSystem/
        ├── MissionServer.c        // init / shutdown managera, login/logout hook
        └── MissionGameplay.c      // klient: notifikácia "Raid okno otvorené"
```

### Moduly a zodpovednosti

| Modul | Úloha |
|------|------|
| **Raid okná** | Konfigurovateľný rozvrh (deň v týždni + hodiny, serverový čas). Mimo okna je damage na bázu zablokovaný. |
| **Damage pravidlá** | Multiplikátor podľa materiálu partu (`ConstructionMaterialType`) a zdroja (ammo/nástroj). Napr. drevo 1.0, kov 0.35 na výbušniny, melee na kov 0. |
| **Ochrana offline báz** | Ak sú všetci vlastníci offline dlhšie ako `OfflineGraceMinutes`, damage sa násobí `OfflineDamageMultiplier` (0 = úplná ochrana). Grace perióda zabraňuje "combat-log" zneužitiu. |
| **Vlastníctvo** | Vlastníci = hráči, ktorí postavili base part v okruhu vlajky (`OnPartBuiltServer`). Ukladá sa do JSON podľa persistent ID vlajky – **nie** do `OnStoreSave` vanilla tried (odstránenie modu by rozbilo storage). |
| **Zámky** | Kódový zámok/kombinačný zámok je imúnny mimo raid okna; v okne ide zničiť výbušninou alebo dlhou akciou (pílka) s konfigurovateľným časom. |
| **Logy / notifikácie** | Každý zničený part: kto, čím, kde, ktorá vlajka. Online vlastníkom server pošle notifikáciu „Tvoja báza je pod útokom“. Napojiť na existujúci logging MD_AdminPanel (nepridávať druhý log systém). |

---

## 3. Konfigurácia

`$profile:MD_RaidSystem/MDRS_Config.json`

```json
{
    "Version": 1,
    "Enabled": 1,
    "LogToFile": 1,
    "RaidWindows": [
        { "Days": [5, 6, 0], "StartHour": 18, "EndHour": 23 }
    ],
    "AllowDamageOutsideWindow": 0,
    "OfflineProtection": {
        "Enabled": 1,
        "OfflineGraceMinutes": 15,
        "OfflineDamageMultiplier": 0.0
    },
    "MaterialMultipliers": {
        "Explosion": { "LOG": 1.0, "WOOD": 1.0, "METAL": 0.35, "WIRE": 1.0 },
        "Melee":     { "LOG": 1.0, "WOOD": 1.0, "METAL": 0.0,  "WIRE": 1.0 },
        "Firearm":   { "LOG": 0.0, "WOOD": 0.0, "METAL": 0.0,  "WIRE": 0.0 }
    },
    "DestroyPartCycles": 8,
    "AllowedRaidTools": ["Sledgehammer", "FirefighterAxe", "WoodAxe", "Hacksaw"],
    "LockRaidSeconds": 300,
    "NotifyOwners": 1
}
```

`Days`: 0 = nedeľa … 6 = sobota. Enforce nemá funkciu na deň v týždni –
`MDRS_RaidWindow` ho vypočíta z `GetYearMonthDay` (Zellerov vzorec), hodinu z
`GetHourMinuteSecond`.

---

## 4. Kľúčové hooky (kostra)

### 4.1 Blokovanie a škálovanie damage – `BaseBuildingBase`

```c
modded class BaseBuildingBase
{
	override bool EEOnDamageCalculated(TotalDamageResult damageResult, int damageType, EntityAI source, int component, string dmgZone, string ammo, vector modelPos, float speedCoef)
	{
		if (!super.EEOnDamageCalculated(damageResult, damageType, source, component, dmgZone, ammo, modelPos, speedCoef))
			return false;

		if (!g_Game.IsServer())
			return true;

		MDRS_Manager manager = MDRS_Manager.GetInstance();
		if (!manager || !manager.IsEnabled())
			return true;

		// mimo raid okna sa damage vôbec neaplikuje
		if (!manager.IsDamageAllowed(this))
			return false;

		// multiplikátor 0 (napr. strelné zbrane, melee na kov) = blokovať
		float multiplier = manager.GetDamageRules().GetMultiplier(this, damageType, dmgZone, ammo, source);
		return multiplier > 0;
	}

	override void EEHitBy(TotalDamageResult damageResult, int damageType, EntityAI source, int component, string dmgZone, string ammo, vector modelPos, float speedCoef)
	{
		super.EEHitBy(damageResult, damageType, source, component, dmgZone, ammo, modelPos, speedCoef);

		if (!g_Game.IsServer() || dmgZone == "")
			return;

		MDRS_Manager manager = MDRS_Manager.GetInstance();
		if (!manager || !manager.IsEnabled())
			return;

		// vanilla damage je už aplikovaný; dorovnáme rozdiel podľa multiplikátora
		float applied = damageResult.GetDamage(dmgZone, "Health");
		float multiplier = manager.GetDamageRules().GetMultiplier(this, damageType, dmgZone, ammo, source);
		float correction = applied * (multiplier - 1.0);
		if (correction != 0 && GetHealth(dmgZone, "Health") > 0)
			AddHealth(dmgZone, "Health", -correction);   // RUINED -> vanilla DestroyPartServer

		manager.OnBaseHit(this, source, dmgZone, ammo);   // notifikácia vlastníkom (s cooldownom)
	}

	override void OnPartBuiltServer(notnull Man player, string part_name, int action_id)
	{
		super.OnPartBuiltServer(player, part_name, action_id);

		MDRS_Manager manager = MDRS_Manager.GetInstance();
		if (manager)
			manager.GetOwnership().RegisterBuilder(this, PlayerBase.Cast(player));
	}

	override void OnPartDestroyedServer(Man player, string part_name, int action_id, bool destroyed_by_connected_part = false)
	{
		super.OnPartDestroyedServer(player, part_name, action_id, destroyed_by_connected_part);

		MDRS_Manager manager = MDRS_Manager.GetInstance();
		if (manager)
			manager.LogPartDestroyed(this, player, part_name);
	}
}
```

Poznámky:
- `EEOnDamageCalculated` sa volá **pred** aplikovaním damage → jediné čisté miesto na
  úplné zablokovanie. `TotalDamageResult` sa tu nedá zmeniť, preto škálovanie robíme
  v `EEHitBy` korekciou cez `AddHealth`.
- Zničenie partu ostáva 100 % vanilla (`OnHealthLevelChanged` → `DestroyPartServer`),
  takže funguje pád závislých partov, materiály, navmesh, sync.
- **Overiť runtime testom**: pri výbuchu s viacerými zónami sa `EEHitBy` volá
  pre každú zasiahnutú zónu zvlášť? Ak nie, korekciu počítať cez iteráciu
  `GetConstruction().GetConstructionParts()` a `damageResult.GetDamage(zone, "Health")`.

### 4.2 Melee raid – `ActionDestroyPart`

```c
modded class ActionDestroyPart
{
	override bool ActionCondition(PlayerBase player, ActionTarget target, ItemBase item)
	{
		if (!super.ActionCondition(player, target, item))
			return false;

		// klient: config sa synchronizuje pri logine (jednoduchý stav okna + allowed tools)
		return MDRS_RaidWindow.IsRaidAllowedFor(player, target.GetObject(), item);
	}

	override void OnFinishProgressServer(ActionData action_data)
	{
		// server je autorita – znova overiť (klient môže mať zastaraný stav)
		if (!MDRS_RaidWindow.IsRaidAllowedFor(action_data.m_Player, action_data.m_Target.GetObject(), action_data.m_MainItem))
			return;

		super.OnFinishProgressServer(action_data);
	}
}
```

- Vanilla `ActionDestroyPart` volá `ent.AddHealth(zone, ...)` → prejde aj cez
  `EEHitBy`? **Nie** – `AddHealth` nie je hit. Preto vlastník base musí mať výnimku
  a pre melee multiplikátor treba aplikovať priamo tu: buď zvýšiť počet cyklov
  (`DestroyPartCycles` z configu), alebo nahradiť `super` vlastnou verziou
  `AddHealth(zone, -(MaxHealth / cycles) * meleeMultiplier)`.
  Odporúčanie: **vlastná verzia OnFinishProgressServer** (kopírovať len 15 riadkov
  vanilla logiky, zvyšok cez super nevolať), lebo `CYCLES` je `static` a menil by sa
  globálne.
- Vlastník (je v `MDRS_BaseOwnership`) môže vždy dismantle/destroy vlastnú bázu.

### 4.3 Dismantle – `ActionDismantlePart`

Vanilla dismantle ide len zvnútra (`MustBeBuiltFromOutside`/`IsFacingPlayer`).
Pridať: cudzí hráč nesmie dismantle mimo raid okna (ochrana proti glitchu dovnútra).

### 4.4 Zámky – `DigitalCodeLock` / `CombinationLock`

- Mimo okna: `EEOnDamageCalculated` na zámku vráti `false`.
- V okne: nová akcia `ActionMDRSBreachLock` (Hacksaw/Sledgehammer, `CAContinuousRepeat`,
  celkový čas `LockRaidSeconds`), na konci `codeLock.UnlockOnServer(fence)` a zničenie
  zámku – rovnaký postup ako deprecated `ActionDestroyCombinationLock`.
- Plastic explosive na bráne: zničenie partu `wall_gate` už zámok zhodí vanilla
  (`HandleDropAttachment(GetCodeLock())` vo `Fence`).

### 4.5 Manager – `MDRS_Manager` (4_World)

```c
class MDRS_Manager
{
	protected static ref MDRS_Manager s_Instance;

	protected ref MDRS_Config m_Config;
	protected ref MDRS_DamageRules m_DamageRules;
	protected ref MDRS_BaseOwnership m_Ownership;
	protected ref map<string, int> m_LastLogoutTime;   // steamId -> čas odhlásenia (s)
	protected ref map<int, float> m_LastNotifyTime;    // id báz -> cooldown notifikácie

	static MDRS_Manager GetInstance() { return s_Instance; }

	static void Init()
	{
		if (!g_Game.IsServer() || s_Instance)
			return;

		s_Instance = new MDRS_Manager();
		s_Instance.Load();
	}

	static void Shutdown()
	{
		if (!s_Instance)
			return;

		s_Instance.Save();
		s_Instance = null;
	}

	bool IsDamageAllowed(BaseBuildingBase base)
	{
		if (!MDRS_RaidWindow.IsOpen(m_Config) && !m_Config.AllowDamageOutsideWindow)
			return false;

		if (m_Config.OfflineProtection.Enabled && m_Config.OfflineProtection.OfflineDamageMultiplier <= 0)
			return !IsBaseOfflineProtected(base);

		return true;
	}

	bool IsBaseOfflineProtected(BaseBuildingBase base)
	{
		array<string> owners = m_Ownership.GetOwners(base);
		if (!owners || owners.Count() == 0)
			return false;              // bez vlajky/vlastníka = bez ochrany

		// online alebo odhlásený menej než grace => nie je chránená
		...
	}
}
```

`MissionServer`:
- `OnInit()` → `MDRS_Manager.Init()`
- `OnMissionFinish()` → `MDRS_Manager.Shutdown()`
- `InvokeOnConnect` / `PlayerDisconnected` → `m_LastLogoutTime`

### 4.6 Vlastníctvo báz – `MDRS_BaseOwnership`

- Kľúč = persistent ID `TerritoryFlag` (`GetPersistentID(b1,b2,b3,b4)`).
- Pri `OnPartBuiltServer`: nájsť najbližšiu `TerritoryFlag` v
  `GameConstants.REFRESHER_RADIUS`, pridať steamId staviteľa (max N členov).
- Bez vlajky → báza nemá vlastníka → žiadna offline ochrana (motivuje stavať vlajku).
- Ukladať do `$profile:MD_RaidSystem/MDRS_Ownership.json`, mazať záznamy
  neexistujúcich vlajok pri štarte (po načítaní CE).

---

## 5. Anti-exploit pravidlá

1. Všetko rozhodovanie len na serveri (`g_Game.IsServer()`), klient iba zobrazuje.
2. `ActionCondition` je len UX – v `OnFinishProgressServer` vždy znovu overiť.
3. Žiadne RPC od klienta, ktoré mení stav raidu. Jediné RPC: server → klient
   (stav okna, notifikácia). Pri čítaní `ctx.Read` kontrolovať návratovú hodnotu.
4. Grace perióda pri odhlásení (combat-log ochrana).
5. Notifikácie s cooldownom (napr. 60 s na bázu) – nie pri každom hite.
6. Jeden timer (CallQueue, napr. 60 s) na prepočet stavu okna – žiadny druhý tick
   (poučenie z MD_CrateEvent).
7. `disableBaseDamage` v cfggameplay musí byť `false`, inak engine damage nevyrobí
   vôbec (`SetAllowDamage(false)`). Mod pri štarte vypíše varovanie, ak je zapnutý.

---

## 6. Súbory na vytvorenie / úpravu

Nové (mod `MD_RaidSystem`): všetky súbory z kapitoly 2.

Vanilla triedy, ktoré sa iba **modded**-ujú (nekopírujú):
`BaseBuildingBase`, `ActionDestroyPart`, `ActionDismantlePart`, `DigitalCodeLock`,
`CombinationLock`, `MissionServer`, `MissionGameplay`.

Integrácia s MD_AdminPanel (keď bude v repozitári): stránka „Raid“ – aktuálny stav
okna, zoznam báz pod útokom, log zničených partov, prepínač `Enabled`.
Použiť existujúce MDAP page registration / RPC / logging – nevytvárať duplicitné.

---

## 7. Testovací plán

1. Mimo okna: C4 na drevenú stenu → žiadny damage, log „blocked“.
2. V okne: C4 na drevenú stenu → part zničený, závislé party padnú, materiál vypadne.
3. Kovová stena, multiplikátor 0.35 → potrebné ~3× viac výbušnín.
4. Strelné zbrane na stenu → 0 damage.
5. `ActionDestroyPart` mimo okna → akcia sa neponúkne; v okne trvá podľa `DestroyPartCycles`.
6. Vlastník offline < grace → raid možný; > grace → blokovaný.
7. Zámok na bráne mimo okna nezničiteľný; v okne `ActionMDRSBreachLock` zhodí zámok.
8. Reštart servera počas raid okna → ownership JSON sa načíta, stav sedí.
9. Odstránenie modu → báza sa načíta (žiadne zmeny `OnStoreSave`).
