DayZ Development Knowledge Base

Purpose

This repository is the main knowledge source for my DayZ mod development.

Claude must use these files as the primary reference before creating, modifying, or debugging any DayZ mod.

Priority Order

When working on tasks always follow this order:

My existing working mods

My previous bug fixes and solutions

Bohemia vanilla DayZ scripts

Framework references

External examples

Do not create duplicate systems if an existing implementation can be reused.

Development Workflow

Before writing code:

Search the repository.

Find similar existing systems.

Analyse the current architecture.

Explain what can be reused.

Implement only the required changes.

DayZ Script Structure

Respect the official layers:

1_Core

2_GameLib

3_Game

4_World

5_Mission

Place scripts in the correct layer.

References

Bohemia Interactive DayZ Script Diff

https://github.com/BohemiaInteractive/DayZ-Script-Diff

Purpose:

Vanilla classes

Engine behaviour

Official script changes

Community Framework

https://github.com/Arkensor/DayZ-CommunityFramework

Purpose:

Common utilities

RPC examples

Framework patterns

Expansion

https://github.com/salutesh/DayZ-Expansion-Scripts

Purpose:

Advanced systems

Missions

Economy examples

EnScript (Enforce Script) Style Guide

https://github.com/TrueDolphin/references/wiki/EnScript-(Enforce-Script)-Style-Guide#reference-management-critical

Purpose:

Enforce Script coding standards

Memory management rules

ref / autoptr usage

Reference ownership

Circular reference prevention

Class design patterns

Naming conventions

Code quality rules

Use this reference when:

Reviewing existing DayZ mods

Creating new EnScript systems

Debugging memory issues

Reviewing inheritance

Checking RPC implementation

Reviewing script architecture

Coding Rules

Always follow:

EnScript Style Guide

Existing project naming

Existing folder structure

Existing architecture

Never blindly copy external code. Adapt it to this project.

External Reference Priority

External references are supporting documentation only.

Project-specific knowledge has higher priority.

Order:

Existing working mods

Existing bug fixes and solutions

Project documentation

Bohemia vanilla scripts

Framework references

External examples

Known Solved Systems

MD_CrateEvent Double Timer

Location:

Knowledge/Bug_Fixes.md

Status:

Fixed

Summary:

Two timer systems modified the same countdown:

EOnPostSimulate()

MDCE_TimerTick()

Solution:

Removed duplicate timer source.

Kept MDCE_TimerTick as the only timer.

Added completion protection.

Final Rule

Before creating any new DayZ system:

Check existing documentation.

Check existing fixes.

Check existing implementations.

Check external references.

Reuse before creating.
Fix before rewriting.
