# Solved DayZ Bug Fixes

This file stores known fixes.

------------------------------------------------------------------------

## General Script Errors

Cause: Wrong script placement or inheritance.

Solution: Check correct script layer.

------------------------------------------------------------------------

## Mod Loading Issues

Cause: Incorrect config.cpp or dependencies.

Solution: Verify:

-   requiredAddons
-   class names
-   folder structure

------------------------------------------------------------------------

## Multiplayer Problems

Cause: Client executing server-only logic.

Solution:

Separate:

Server: GetGame().IsServer()

Client: GetGame().IsClient()

------------------------------------------------------------------------

## Future Fixes

Add every solved problem here.
