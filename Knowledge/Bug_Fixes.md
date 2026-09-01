# Solved DayZ Bug Fixes

This file stores confirmed DayZ mod bugs, root causes, fixes and verification steps.

------------------------------------------------------------------------

# General Script Errors

## Cause

Wrong script placement, inheritance problems or incorrect class structure.

## Solution

Always verify:

- Correct DayZ script layer
- Correct class inheritance
- Correct vanilla override usage
- Correct modded class implementation

Required script layers:

- 1_Core
- 2_GameLib
- 3_Game
- 4_World
- 5_Mission

------------------------------------------------------------------------

# Mod Loading Issues

## Cause

Incorrect config.cpp, missing dependencies or wrong folder structure.

## Solution

Verify:

- requiredAddons
- class names
- dependencies
- script folder paths
- config.cpp structure

------------------------------------------------------------------------

# Multiplayer Problems

## Cause

Client executing logic that should only run on server.

## Solution

Server:

GetGame().IsServer()

Client:

GetGame().IsClient()

Dedicated server:

GetGame().IsDedicatedServer()

Server must remain authoritative.

------------------------------------------------------------------------

# Confirmed Fixes

------------------------------------------------------------------------

# MD_CrateEvent Double Timer System

## Status

Fixed

## Priority

P0 Critical

## Problem

The crate timer was controlled by two independent systems:

- EOnPostSimulate()
- MDCE_TimerTick()

Both modified:

m_RemainingTime

causing the countdown to run twice.

## Symptoms

- Crate opened approximately twice as fast.
- 60 second timer finished around 30 seconds.
- Possible duplicate loot spawning.
- Duplicate notifications.
- Multiple OnTimerComplete executions.

## Root Cause

ActivateCrate() started:

MDCE_TimerTick()

and also enabled:

EntityEvent.POSTSIMULATE

Both systems decreased the same timer.

## Fix Applied

Removed EOnPostSimulate timer logic.

Removed:

- SetEventMask(EntityEvent.POSTSIMULATE)
- Duplicate countdown handling

Kept:

- MDCE_TimerTick() as the only timer source

Added:

m_TimerCompleted

guard to prevent duplicate completion.

## Affected Files

Scripts/4_World/MD_CrateEvent.c

Scripts/4_World/MD_SpecialCrate.c

## Verification

Tested:

- Normal crate
- Special crate
- Key bypass
- Loot spawning
- Timer completion

Result:

- Correct timer duration
- Single completion
- Single loot spawn

------------------------------------------------------------------------

# MD_CrateEvent CallQueue Cleanup After Despawn

## Status

Fixed

## Priority

P1

## Problem

Repeating CallQueue callbacks could remain registered after crate deletion until their own self-cancel.

Affected:

- CheckTimerProgress
- MDCE_TimerTick

## Symptoms

Possible:

- Callback execution after entity deletion
- Old reminder notifications
- Unnecessary server processing

## Root Cause

Cleanup depended only on self-cancel.

## Fix Applied

Added explicit cleanup in DoDespawn():

- Remove(CheckTimerProgress)
- Remove(MDCE_TimerTick)

SpecialCrate required separate cleanup because DoDespawn() does not call super.

## Affected Files

Scripts/4_World/MD_CrateEvent.c

Scripts/4_World/MD_SpecialCrate.c

## Verification

Tested:

- Normal despawn
- Special crate despawn
- Respawn cycle
- Multiple active crates

------------------------------------------------------------------------

# MD_SpecialCrate Event Key Consumed On Failed Bypass

## Status

Fixed

## Priority

P1

## Problem

Event Key was deleted even when bypass operation failed.

## Root Cause

Methods:

- BypassTimerWithKey()
- ActivateAndBypassWithKey()

returned no success state.

Actions always executed:

key.Delete()

## Fix Applied

Changed methods to return bool.

Success:

true

Failure:

false

Key is deleted only after successful operation.

## Affected Files

Scripts/4_World/MD_SpecialCrate.c

Scripts/4_World/Classes/UserActionsComponent/Actions/Interact/ActionUseKeyOnCrate.c

Scripts/4_World/Classes/UserActionsComponent/Actions/Interact/ActionActivateCrate.c

## Verification

Tested:

- Normal bypass
- Pre-activation bypass
- Failed bypass
- Both action paths

Result:

Failed operations no longer consume keys.

------------------------------------------------------------------------

# MD_CrateEvent Marker Identity Collision

## Status

Fixed

## Priority

P1

## Problem

Markers were identified using location.Name.

Duplicate names caused conflicts.

Example:

Location 1:
Name = Supply Crate

Location 2:
Name = Supply Crate

One marker could overwrite another.

## Root Cause

Display name was used as internal identifier.

Display names are not guaranteed unique.

## Fix Applied

Network structure preserved.

Marker key changed from:

Name

to:

ID|Name

Server:

- Sends unique location ID
- Sends display name

Client:

- Matches using ID
- Displays Name

## Affected Files

Scripts/3_Game/MD_CE_ClientMarker.c

Scripts/4_World/MD_CE_RPC.c

Scripts/4_World/MD_CrateEvent.c

Scripts/4_World/MD_SpecialCrate.c

Scripts/4_World/MD_CrateEventManager.c

Scripts/4_World/MD_SpecialCrateConfig.c

## Compatibility

Preserved:

- RPC structure
- Marker states
- Progress sync
- Late join synchronization
- LBmaster integration

## Verification

Tested:

- Duplicate names
- State changes
- Despawn
- Late joiners
- Special crates
- Multiple active markers

------------------------------------------------------------------------

# Pending Issues

------------------------------------------------------------------------

# SpecialCrate Timer Display Initial Sync

## Status

Pending

## Problem

After activation client may temporarily show:

Unlocks in: 0

until first synchronization.

## Root Cause

ActivateCrate() does not immediately reset:

- m_RemainingSeconds
- m_SyncAccum

## Expected Fix

Synchronize initial timer state after activation.

Do not change timer architecture.

------------------------------------------------------------------------

# Future Fix Entry Template

## Bug Name

Status:

Fixed / Pending

Priority:

P0 / P1 / P2

## Problem


## Root Cause


## Affected Files


## Fix Applied


## Verification


## Lessons Learned
