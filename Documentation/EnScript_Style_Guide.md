EnScript (Enforce Script) Style Guide for DayZ Mod Development

Purpose

This document defines coding standards, memory management rules,
networking practices, and architecture guidelines for developing DayZ
mods using Enforce Script.

All new code should follow these rules unless there is a specific reason
to break them.

1. General Development Principles

Existing Code Has Priority

Before creating new systems:

Search existing project code.

Check similar implementations.

Reuse existing architecture.

Avoid duplicate systems.

Do not rewrite working code unless necessary.

2. DayZ Script Layer Structure

Always place code in the correct DayZ script layer.

1_Core

Engine level changes.

2_GameLib

Game library functionality.

3_Game

Gameplay-independent systems.

Examples: - managers - data classes - configuration systems

4_World

World objects and gameplay systems.

Examples: - entities - players - vehicles - items

5_Mission

Mission and server/client initialization.

Examples: - mission scripts - server startup - client startup

3. Memory Management

Enforce Script uses reference counting.

Incorrect reference handling can cause: - memory leaks - crashes -
invalid references - server instability

ref

Use ref for strong ownership.

Example:

class MissionManager
{
    ref array<ref MissionData> m_Missions;
}

Weak References

Do not create circular references.

Bad:

class Child
{
    ref Parent m_Parent;
}

Correct:

class Child
{
    Parent m_Parent;
}

Destructor Cleanup

Always clean owned objects.

void ~MyManager()
{
    if (m_Data)
    {
        m_Data.Clear();
        delete m_Data;
        m_Data = null;
    }
}

4. Modded Classes

When changing vanilla DayZ behaviour:

Always use:

modded class OriginalClass
{
}

Do not copy vanilla classes.

Always call super correctly.

Initialization:

override void OnInit()
{
    super.OnInit();

    InitMySystem();
}

Cleanup:

override void EEDelete(EntityAI parent)
{
    Cleanup();

    super.EEDelete(parent);
}

5. Networking and RPC

Never trust client data.

Always validate: - item amounts - positions - permissions - player
actions

Example:

Param1<int> value;

if (!ctx.Read(value))
{
    Error("[MOD] RPC read failed");
    return;
}

Server:

GetGame().IsServer()

Client:

GetGame().IsClient()

Dedicated server:

GetGame().IsDedicatedServer()

6. Naming Conventions

Classes:

MissionManager
TraderSystem
PlayerData

Methods:

StartMission()
GetPlayerData()
SaveConfiguration()

Member variables:

m_Player
m_Config
m_IsRunning

Static variables:

s_Instance

Local variables:

playerPosition
itemCount
targetObject

7. Null Checking

Prefer early exits.

Good:

if(!player || !player.IsAlive())
    return;

DoSomething();

Avoid deep nested conditions.

8. Arrays and Maps

Avoid unnecessary allocations.

Prefer:

if(!m_Items)
    m_Items = new array<string>;
else
    m_Items.Clear();

9. Configuration

Separate:

Scripts

Configs

Data

Avoid hardcoded values when configuration is possible.

10. Logging

Use clear prefixes.

Example:

Print("[MissionSystem] Mission started");
Error("[TraderSystem] Invalid configuration");

11. Performance Rules

Avoid: - unnecessary loops - repeated object creation - expensive
operations every frame

Prefer: - timers - events - cached references

12. Debugging Rules

When fixing problems:

Identify the root cause.

Compare with working code.

Change the smallest possible section.

Test server and client separately.

Never randomly rewrite large systems.

Final Development Rule

Before creating any new DayZ system:

Search existing project files.

Check vanilla DayZ implementation.

Check frameworks if needed.

Design the solution.

Implement clean modular code.

Existing tested solutions always have priority.
