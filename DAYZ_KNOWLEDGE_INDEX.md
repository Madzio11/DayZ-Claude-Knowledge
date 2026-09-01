# DayZ Development Knowledge Base

## Purpose

This repository is the main knowledge source for my DayZ mod
development.

Claude must use these files as the primary reference before creating,
modifying, or debugging any DayZ mod.

------------------------------------------------------------------------

# Priority Order

When working on tasks always follow this order:

1.  My existing working mods
2.  My previous bug fixes and solutions
3.  Bohemia vanilla DayZ scripts
4.  Framework references
5.  External examples

Do not create duplicate systems if an existing implementation can be
reused.

------------------------------------------------------------------------

# Development Workflow

Before writing code:

1.  Search the repository.
2.  Find similar existing systems.
3.  Analyse the current architecture.
4.  Explain what can be reused.
5.  Implement only the required changes.

------------------------------------------------------------------------

# DayZ Script Structure

Respect the official layers:

-   1_Core
-   2_GameLib
-   3_Game
-   4_World
-   5_Mission

Place scripts in the correct layer.

------------------------------------------------------------------------

# References

## Bohemia Interactive DayZ Script Diff

https://github.com/BohemiaInteractive/DayZ-Script-Diff

Purpose: - Vanilla classes - Engine behaviour - Official script changes

## Community Framework

https://github.com/Arkensor/DayZ-CommunityFramework

Purpose: - Common utilities - RPC examples - Framework patterns

## Expansion

https://github.com/salutesh/DayZ-Expansion-Scripts

Purpose: - Advanced systems - Missions - Economy examples

------------------------------------------------------------------------

# Coding Rules

Always follow:

-   EnScript Style Guide
-   Existing project naming
-   Existing folder structure
-   Existing architecture

Never blindly copy external code. Adapt it to this project.
