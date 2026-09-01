# Solved DayZ Bug Fixes

This file stores confirmed DayZ mod bugs and their solutions.

------------------------------------------------------------------------

## General Script Errors

### Cause

Wrong script placement or inheritance.

### Solution

Check correct script layer:

- 1_Core
- 2_GameLib
- 3_Game
- 4_World
- 5_Mission

------------------------------------------------------------------------

## Mod Loading Issues

### Cause

Incorrect config.cpp or missing dependencies.

### Solution

Verify:

- requiredAddons
- class names
- folder structure
- dependencies

------------------------------------------------------------------------

## Multiplayer Problems

### Cause

Client executing server-only logic.

### Solution

Separate execution:

Server:

```c
GetGame().IsServer()
