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

# MD_CrateEvent Double Timer System

## Status
Fixed

## Priority
Critical (P0)

## Affected Systems
- MD_CrateEvent
- MD_SpecialCrate
- Timer system
- Loot spawning
- Notifications
- NetSync countdown

## Problem

The crate timer was controlled by two independent timer systems simultaneously.

Both systems modified:

m_RemainingTime

Timer sources:

1. EOnPostSimulate()
2. MDCE_TimerTick()

The same countdown variable was updated twice.

## Symptoms

- Crate opened approximately twice as fast.
- 60 second timer could finish around 30 seconds.
- Possible duplicate loot spawning.
- Duplicate notifications.
- Multiple OnTimerComplete executions.
- Possible race condition with key bypass.

## Root Cause

ActivateCrate() started CallLater(MDCE_TimerTick) and also enabled POSTSIMULATE.

Both systems executed:

m_RemainingTime -= value;

## Fix Applied

Removed duplicate timer system.

Removed:
- EOnPostSimulate() timer logic
- EntityEvent.POSTSIMULATE dependency

Kept:
- MDCE_TimerTick() as the only timer source

Added:
- m_TimerCompleted protection to prevent duplicate completion.

## Modified Files

Scripts/4_World/MD_CrateEvent.c

Scripts/4_World/MD_SpecialCrate.c

## Verification

Tested:
- Server startup
- Crate activation
- Timer countdown
- Loot spawning
- Special crate activation
- Key bypass protection

Expected:
- Timer duration matches configuration.
- OnTimerComplete executes once.
- Loot spawns once.
- No duplicate notifications.

------------------------------------------------------------------------

# Related Pending Issue

## SpecialCrate Timer Display Initial Sync

Status:
Pending

Problem:

After activating Special Crate, client timer display may temporarily show:

Unlocks in: 0

until the first synchronization update arrives.

Root Cause:

MD_SpecialCrate.ActivateCrate() does not immediately reset:

m_RemainingSeconds
m_SyncAccum

Expected Fix:

Synchronize initial timer state immediately after activation.

Do not change timer architecture.

------------------------------------------------------------------------

# Future Fix Entry Template

## Bug Name

Status:
Fixed / Pending

Priority:
P0 / P1 / P2

Problem:

Root Cause:

Affected Files:

Fix Applied:

Verification:

Lessons Learned:
