# Common DayZ Development Errors

This document contains known DayZ mod development problems and
solutions.

------------------------------------------------------------------------

# Script Compilation Errors

Common causes:

-   Wrong class inheritance
-   Missing brackets
-   Incorrect script layer
-   Missing includes
-   Wrong function override

Fix approach:

1.  Check compiler error line.
2.  Check vanilla implementation.
3.  Compare with working scripts.
4.  Make minimal changes.

------------------------------------------------------------------------

# Mod Not Loading

Check:

-   mod folder structure
-   config.cpp
-   requiredAddons
-   script folders

Correct script locations:

-   1_Core
-   2_GameLib
-   3_Game
-   4_World
-   5_Mission

------------------------------------------------------------------------

# Client / Server Problems

Symptoms:

-   works locally but fails on server
-   client crash
-   missing objects

Check:

-   server authority
-   RPC validation
-   client/server execution checks

Use:

GetGame().IsServer() GetGame().IsClient()

------------------------------------------------------------------------

# RPC Problems

Check:

-   RPC ID consistency
-   ctx.Read validation
-   client/server side
-   parameter types

Never trust client data.

------------------------------------------------------------------------

# Performance Issues

Avoid:

-   heavy loops
-   creating objects repeatedly
-   unnecessary timers

Prefer:

-   cached references
-   events
-   scheduled updates
