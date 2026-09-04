# DayZ Development Knowledge Base

## Purpose

This repository is the PRIMARY knowledge source for my DayZ mod development.

Claude must use this repository before creating, modifying, redesigning,
integrating, reviewing, or debugging any DayZ mod.

This repository contains:

- existing working MD mods
- MD_AdminPanel architecture
- known bug fixes
- solved systems
- coding conventions
- UI conventions
- integration rules
- RPC architecture
- configuration patterns
- layout rules
- trusted external references

PROJECT CODE IS THE PRIMARY SOURCE OF TRUTH.

External repositories are references, not replacements for working project code.

---

# GLOBAL PRIORITY ORDER

When working on any task always follow this order:

1. Existing working MD mods
2. Existing MD_AdminPanel implementation
3. Previous bug fixes and solved systems
4. Project documentation and architecture
5. Bohemia Interactive vanilla DayZ scripts
6. VPP Admin Tools
7. Community Framework
8. DayZ Expansion
9. Other trusted external examples

Project-specific knowledge ALWAYS has priority over external implementations.

If an existing project system already works:

REUSE IT.

Do not replace it with an external architecture simply because the external
implementation looks more advanced.

---

# CORE DEVELOPMENT RULE

Always prefer:

REUSE
before
EXTEND
before
FIX
before
REWRITE

A rewrite is the LAST option.

Do not create duplicate systems if an existing implementation already solves
the same problem.

Do not duplicate:

- RPC systems
- permission systems
- save systems
- timers
- schedulers
- logging
- page registration
- picker systems
- map systems
- placement systems
- LiveData systems
- JSON loading/saving
- UI components
- action bars
- confirmation dialogs
- plugin registration
- data registries

Reuse or extend the existing architecture.

---

# DEVELOPMENT WORKFLOW

Before writing or modifying code:

1. Search this repository.
2. Search project documentation.
3. Search known bug fixes.
4. Search existing MD mods.
5. Find similar working systems.
6. Analyse the current architecture.
7. Identify what can be reused.
8. Verify vanilla DayZ behaviour where required.
9. Search trusted external repositories where useful.
10. Compare external implementations with the current project.
11. Explain what will be reused or minimally changed.
12. Implement only the required change.
13. Validate that existing functionality is preserved.
14. Record reusable bug fixes when appropriate.

Never start by inventing a new architecture if a working implementation exists.

---

# DAYZ SCRIPT LAYERS

Respect the official DayZ script layers:

1_Core
2_GameLib
3_Game
4_World
5_Mission

Place scripts in the correct layer.

Do not create dependency inversions.

Typical relationship:

3_Game
-> shared data/framework/UI classes

4_World
-> world/server gameplay systems
-> may depend on 3_Game

5_Mission
-> mission/client/server page logic
-> may depend on 3_Game and 4_World

A lower layer must not depend on a higher layer.

---

# EXTERNAL REPOSITORY ACCESS RULE

External repository links are NOT documentation links only.

When repository access is available, Claude must inspect the ACTUAL source tree
before implementing or modifying a related system.

Trusted repositories:

BohemiaInteractive/DayZ-Script-Diff

https://github.com/BohemiaInteractive/DayZ-Script-Diff


Arkensor/DayZ-CommunityFramework

https://github.com/Arkensor/DayZ-CommunityFramework


salutesh/DayZ-Expansion-Scripts

https://github.com/salutesh/DayZ-Expansion-Scripts


VanillaPlusPlus/VPP-Admin-Tools

https://github.com/VanillaPlusPlus/VPP-Admin-Tools


Do NOT treat a repository URL alone as completed research.

When access is available:

SEARCH THE SOURCE.

Read relevant:

- .c
- .layout
- config.cpp
- JSON/config examples
- constants
- page classes
- managers
- RPC handlers
- MissionGameplay
- MissionServer
- PlayerBase
- UI controllers
- map systems
- picker systems
- widget logic

If the repository cannot be accessed from the current environment:

STATE THAT CLEARLY.

Do not pretend that external source code was inspected.

Do not reconstruct unseen code from memory and present it as repository fact.

---

# EXTERNAL SOURCE RESEARCH PROTOCOL

When using an external repository:

1. Identify the exact feature being researched.
2. Search for the matching implementation.
3. Read the relevant source files.
4. Read associated layout files.
5. Trace related classes and functions.
6. Trace dependencies.
7. Trace the client path.
8. Trace the server path.
9. Trace UI bindings where applicable.
10. Compare the external implementation with the current MD implementation.
11. Identify only the useful pattern.
12. Adapt that pattern to MD architecture.
13. Avoid importing unrelated systems.

Never blindly copy external code.

Understand the implementation first.

---

# EXTERNAL LAYOUT / UI RESEARCH RULE

For ANY UI, layout, AdminPanel, page, menu, map, picker, popup,
table, player tool, or navigation task:

Claude must inspect relevant layout implementations in ALL accessible trusted
repositories when useful.

This includes:

- project MD layouts
- MD_AdminPanel layouts
- vanilla DayZ layouts
- VPP Admin Tools layouts
- Community Framework layouts if present
- DayZ Expansion layouts

The purpose is NOT to copy appearance.

The purpose is to compare:

- layout hierarchy
- widget ownership
- page structure
- list/detail patterns
- button placement
- form grouping
- map areas
- navigation
- visibility logic
- clipping
- popup hierarchy
- picker hierarchy
- dynamic widget creation
- controller integration

---

# LAYOUT RESEARCH PROCEDURE

When researching a layout, do NOT inspect the `.layout` file in isolation.

Trace the complete chain:

LAYOUT
->
WIDGET
->
PAGE / CONTROLLER CLASS
->
FindAnyWidget
->
EVENT HANDLER
->
TARGET FUNCTION
->
TARGET PANEL
->
VISIBLE PARENT
->
FINAL FUNCTION

For every important widget inspect:

- widget name
- widget class
- parent
- children
- position
- size
- exact size flags
- visible state
- clipping
- color
- style
- text
- font
- exact text
- exact text size
- enabled state where applicable

Also search for:

- FindAnyWidget
- CreateWidgets
- Show
- Enable
- SetPos
- SetSize
- SetColor
- OnClick
- OnChange
- OnItemSelected
- OnMouseEnter
- OnMouseLeave
- dynamic page construction

---

# DYNAMIC UI RULE

Some DayZ UIs are not completely represented by static `.layout` files.

If the layout appears incomplete:

Search the scripts for dynamically created widgets.

Look for:

CreateWidgets()

widget factories

runtime page generation

dynamic list rows

dynamic status rows

dynamic tabs

dynamic cards

Do not conclude that a widget is missing until both:

STATIC LAYOUT

and

DYNAMIC CREATION CODE

have been checked.

---

# CROSS-REPOSITORY LAYOUT COMPARISON

When redesigning MD UI, Claude may compare layouts from:

MD_AdminPanel
VPP Admin Tools
DayZ Expansion
vanilla DayZ
Community Framework

Compare concepts such as:

- compact sidebar
- horizontal tabs
- vertical tabs
- list/detail pattern
- three-column workspace
- cards
- form grids
- modal dialogs
- confirmation dialogs
- map workspaces
- player lists
- admin tool navigation
- search/filter layouts
- status dashboards
- log viewers

The result must still use MD_AdminPanel architecture.

External UI is REFERENCE ONLY.

---

# LAYOUT COMPARISON OUTPUT

When an external layout materially influences a proposed MD layout,
report:

REFERENCE REPOSITORY

REFERENCE FILE

REFERENCE CLASS / CONTROLLER

USEFUL PATTERN

MD CURRENT IMPLEMENTATION

WHAT WILL BE ADAPTED

WHAT WILL NOT BE COPIED

This makes the source of design decisions explicit.

---

# BOHEMIA INTERACTIVE DAYZ SCRIPT DIFF

Repository:

https://github.com/BohemiaInteractive/DayZ-Script-Diff

## Purpose

Primary reference for vanilla DayZ behaviour.

Use for:

- vanilla classes
- engine behaviour
- MissionGameplay
- MissionServer
- PlayerBase
- DayZGame
- UI API
- Widget API
- map API
- RPC API
- script lifecycle
- engine-supported functions
- official DayZ script changes

Use this repository to verify whether a DayZ/Enforce API actually exists.

Do NOT assume Reforger or generic Enfusion APIs are identical to DayZ.

For UI work:

also inspect available vanilla layout files and corresponding scripts.

Trace:

layout
->
controller
->
widget
->
handler

when possible.

---

# VANILLAPLUSPLUS ADMIN TOOLS

Repository:

https://github.com/VanillaPlusPlus/VPP-Admin-Tools

## Purpose

PRIMARY external reference for DayZ ADMINISTRATION TOOL architecture.

VPP contains real client-side and server-side admin systems.

Use it when working on:

MD_AdminPanel

MD_PlayerTools

Admins

Items

player management

teleport

heal

spectate

freecam

item spawning

map administration

weather/time tools

moderation

admin logging

UI navigation

admin pages

RPC communication

---

# VPP SOURCE RESEARCH

When using VPP:

inspect the actual relevant:

- Scripts
- GUI
- layouts
- config.cpp
- RPC declarations
- MissionGameplay integrations
- MissionServer integrations
- PlayerBase hooks
- page controllers
- admin menu classes
- map tools
- player tools
- permission logic
- dialogs
- list/detail UIs

Do NOT only read VPP documentation.

---

# VPP UI / LAYOUT REFERENCE

Use VPP layouts as reference for:

- admin navigation
- player lists
- tool navigation
- list/detail layouts
- player management
- action placement
- modal windows
- popups
- map administration
- compact controls
- entity lists
- table/list organisation

For every useful VPP UI pattern:

trace the associated script.

Do not copy a VPP layout without understanding how its controller works.

---

# IMPORTANT VPP RULE

VPP is a REFERENCE.

VPP is NOT the architecture owner of MD_AdminPanel.

Before adapting a VPP feature:

1. Search MD_AdminPanel.
2. Search existing MD mods.
3. Search project documentation.
4. Determine whether the feature already exists.
5. Inspect the VPP implementation.
6. Compare both architectures.
7. Reuse only the useful concept.
8. Adapt it to MDAP naming/API.
9. Preserve MDAP permission/security/save architecture.

Do NOT copy VPP:

- RPC IDs
- class names
- permission model
- config format
- file paths
- data IDs
- module IDs

unless explicitly required.

Use the implementation concept,
not the identity of the implementation.

---

# COMMUNITY FRAMEWORK

Repository:

https://github.com/Arkensor/DayZ-CommunityFramework

## Purpose

Reference for mature DayZ framework architecture.

Use for:

- utilities
- RPC patterns
- module architecture
- events
- client/server separation
- lifecycle
- reusable helper patterns

For UI work:

inspect relevant `.layout` and UI controller implementations where present.

Do not introduce CF as a dependency unless explicitly required.

Patterns may be adapted without adopting the entire framework.

---

# DAYZ EXPANSION

Repository:

https://github.com/salutesh/DayZ-Expansion-Scripts

## Purpose

Reference for large production DayZ systems.

Use for:

- missions
- events
- economy
- maps
- markers
- configuration architecture
- client/server systems
- advanced UI
- navigation
- status systems
- larger module organisation

For UI work inspect:

- relevant layouts
- menus
- page controllers
- map UIs
- list/detail pages
- settings pages
- dialogs
- dynamically created widgets

Expansion is a reference implementation.

Do not copy Expansion architecture where a simpler project-native solution exists.

---

# ENSCRIPT / ENFORCE SCRIPT STYLE GUIDE

Reference:

https://github.com/TrueDolphin/references/wiki/EnScript-(Enforce-Script)-Style-Guide#reference-management-critical

## Purpose

Primary quality reference for Enforce Script.

Use for:

- coding standards
- memory management
- ref
- autoptr
- ownership
- weak references
- strong references
- circular reference prevention
- class design
- naming
- lifetime management

Use when:

- reviewing mods
- creating systems
- debugging memory issues
- reviewing inheritance
- checking RPC systems
- reviewing architecture
- reviewing object ownership

---

# REFERENCE OWNERSHIP RULES

Understand ownership before adding `ref`.

General project rules:

- member-owned objects may use ref when ownership is required
- owning arrays/maps may use ref element types
- avoid unnecessary ref local variables
- parent references should normally be weak where appropriate
- avoid circular strong references
- avoid ownership cycles

Never add `ref` merely to silence a compiler error.

---

# MODDED CLASS RULES

When extending a class:

use:

modded class ExistingClass

Do not copy the original vanilla or mod class.

Within one mod:

do not create multiple independent blocks of:

modded class MissionGameplay

modded class MissionServer

modded class PlayerBase

when the logic can safely be merged.

One modded block per class per mod is the preferred project rule.

Preserve:

- super calls
- lifecycle order
- OnInit
- OnMissionStart
- OnMissionFinish
- OnRPC
- cleanup
- timers
- file watchers
- existing behaviours

Typical convention:

super first during initialization

super last during cleanup where appropriate

Always inspect the actual parent implementation before changing lifecycle order.

---

# RPC SECURITY RULES

RPC is a security boundary.

Server-authoritative behaviour is mandatory.

For client -> server actions:

1. validate sender identity
2. validate permission
3. validate scope/rank
4. validate parameters
5. validate data size
6. validate expected state
7. reject unauthorized requests
8. log important admin actions

Do not trust client-supplied data.

Before creating an RPC:

search:

- MDAP RPC registry
- MD_AdminPanel RPCs
- all MD module RPCs
- reserved ranges

Do not introduce RPC collisions.

---

# RPC EXTERNAL RESEARCH

When checking external RPC implementations:

trace BOTH sides.

CLIENT:

button/action
->
RPC send
->
serialized parameters

SERVER:

OnRPC / handler
->
identity validation
->
permission validation
->
parameter parsing
->
actual action
->
response

Do not copy only one side of an RPC implementation.

---

# MD_ADMINPANEL UI OWNERSHIP

MD_AdminPanel owns:

- global shell
- global header
- sidebar
- page header
- page subtitle
- page status
- page content host
- global action bar
- footer
- global design tokens
- shared cards
- shared inputs
- shared tables
- shared buttons
- popup
- confirm dialog
- shared sub-navigation pattern

Modules own:

- module-specific page content
- module forms
- module lists
- module maps
- module pickers
- module tools
- module data handling
- module-specific actions

Do not recreate the global MDAP shell inside modules.

---

# MDAP SHARED COMPONENTS

Before creating a new UI component search existing MDAP components.

Prefer:

MDAP_Page

MDAP_Action

MDAP_Button

MDAP_Input

MDAP_Table

MDAP_Card

MDAP_StatusBadge

MDAP_Popup

MDAP_ConfirmDialog

shared sub-navigation

shared MDAP_UI tokens

Do not create duplicate components with module-specific names if the shared
component already solves the requirement.

---

# CENTRAL UI DESIGN RULE

All MD_AdminPanel pages and MD modules must look like ONE application.

Use one centralized visual language.

Common:

- dark background
- consistent opacity
- consistent typography
- consistent spacing
- consistent button heights
- consistent EditBox height
- consistent TextListbox style
- consistent section titles
- consistent cards
- consistent navigation
- consistent selected state
- consistent danger styling

Modules must not invent unrelated visual themes.

---

# UI OPACITY RULE

Structural container:

TRANSPARENT

Page veil:

dark translucent

Functional card:

strong dark background

EditBox / TextListbox:

near opaque

Modal / Popup:

strong opaque

Do not make every PanelWidget opaque.

Do not make functional panels transparent enough that grass, trees,
characters, or objects interfere with readability.

The game world may remain visible as atmosphere only.

---

# PANEL STYLE WARNING

Do not assume a style renders PanelWidget fill.

If a panel color exists but runtime screenshot shows no fill:

inspect the applied style.

Known project lesson:

`style rover_sim_black`

may render only a border on PanelWidget and prevent expected fill behaviour.

Always verify UI visually in runtime.

Static color values alone are not proof of correct rendering.

---

# UI TYPOGRAPHY RULE

Use controlled typography.

Page title:
shell-owned

Section title:
consistent

Field label:
consistent readable size

Input value:
normal controlled size

Meta/help:
smaller but readable

Do not let EditBox inherit oversized fonts.

Use exact text sizing when required.

Do not use unreadably small labels.

---

# UI GEOMETRY RULE

Do not use all available width merely because it exists.

Typical two-column layout:

LIST
30-34%

DETAIL
64-68%

Typical three-column layout:

LEFT
25-29%

CENTER
42-48%

RIGHT
24-29%

Lists should normally use vertical space rather than excessive width.

Use reclaimed width for:

- detail forms
- preview
- map
- context actions
- status
- secondary information

---

# INTERNAL MODULE NAVIGATION

Avoid multiple large vertical sidebars.

The global MDAP sidebar is already vertical.

For module sub-pages prefer a centralized horizontal tab pattern where practical.

Examples:

Airdrop:

Locations
Settings
Advanced
Special
Live
Map
Groups


CrateEvent:

Locations
Settings
Notifications
Special
Special Notify


KOTH:

Zones
Rotation
Reward
Groups
Live


KeyCard:

Doors
Loot Crates
Settings
Keycards
Spawns
Building
Alerts
Teleport
Status
Import/Export
Bulk
Tools

Only the active section should be visible.

---

# UI FUNCTION PRESERVATION

A redesign must NEVER silently remove functionality.

Before modifying a layout:

map:

- widget names
- FindAnyWidget
- OnClick
- OnChange
- OnItemSelected
- navigation
- hidden panels
- section roots
- picker roots
- popup roots
- maps
- placement controls
- Save
- Reload
- Delete
- RPC actions
- data requests
- LiveData
- import/export
- bulk operations

After redesign:

every previous function must still have a reachable UI path.

---

# UI FUNCTION TRACE RULE

Widget existence is NOT enough.

Verify:

BUTTON
->
HANDLER
->
TARGET FUNCTION
->
TARGET ROOT
->
VISIBLE PARENT
->
VISIBLE RESULT
->
FUNCTION

A bound widget inside a hidden ancestor is NOT functionally available.

---

# INTERNAL TAB VALIDATION

For every internal tab:

verify:

NAV WIDGET

HANDLER

TARGET PANEL

PARENT VISIBILITY

Show(true)

Show(false)

selected-state update

data load

save action

No internal page may be considered functional until the full chain is traced.

---

# HIDDEN PARENT RULE

When a widget does not appear:

do not immediately assume the widget itself is broken.

Walk its parent hierarchy.

Check:

visible child
inside hidden parent

section root
inside inactive section

picker
inside hidden card

popup
inside clipped parent

map
inside hidden container

This is a known class of UI failure.

---

# CLIPPING RULE

Use clipping intentionally.

Good clipping targets:

- lists
- card content
- normal page workspace
- scroll areas

Be careful clipping:

- picker overlays
- popups
- dropdowns
- modal dialogs
- map markers
- floating tools

These may intentionally extend outside a normal parent.

---

# BUTTON SEMANTICS

Buttons are classified by behaviour, not text.

PRIMARY:

- Save
- Add
- Update
- Apply

SECONDARY:

- Reload
- Refresh
- Search
- Pick
- clear non-destructive selection
- remove from group when reversible
- mode selector

UTILITY:

- Place on Map
- Preview
- Teleport
- navigate to another tool
- Add to another module
- non-destructive admin utility

DANGER:

- Delete persistent data
- Remove persistent entity
- wipe persistent data
- reset leaderboard
- clear persistent log
- destructive server action

Do not make normal Add buttons look destructive.

---

# ACTION OWNERSHIP

Use MDAP page migration model where applicable.

LEGACY:

IsMigrated() == false

shell action bar hidden

local actions remain


MIGRATED:

IsMigrated() == true

shell actions active

duplicate local actions removed


Context-specific saves may remain local if they save a DIFFERENT object.

Example:

Save Config

Save Special

These are valid separate actions.

Do not show two generic buttons both labelled:

Save

with different meaning.

---

# MAP / PLACEMENT RULE

Maps and placement systems are high-risk UI components.

Before modifying:

trace:

- map root
- map layout
- marker widgets
- map click handler
- placement start
- placement confirm
- placement cancel
- selected entity
- coordinate conversion

Do not place map overlays inside incorrectly clipped parents.

Do not change placement logic during visual redesign.

---

# PICKER / POPUP RULE

Before changing a picker:

trace:

open

parent

visibility

selection

callback

close

returned value

Do not move a picker under a hidden parent.

Use MDAP_Popup when safe.

Do not rewrite a complex working picker only for component purity.

---

# PLAYERTOOLS RULE

Separate:

ADMIN PANEL UI

from:

GAMEPLAY OVERLAY

Do not force:

ESP
radar
tracker
world markers

into MDAP page safe area.

Only admin configuration UI should use MDAP normal page geometry.

---

# KEYCARD RULE

KeyCard is a complex multi-function module.

Do not show hundreds of widgets simultaneously.

Map every widget into a functional sub-page.

Only active sub-page visible.

Preserve:

- Doors
- Loot Crates
- Settings
- Keycards
- Spawns
- Building Preview
- Alerts
- Teleport
- Status / Log
- Export / Import
- Bulk Edit
- Test Unlock
- Backup
- Restore

Never simplify the UI by deleting existing functionality.

---

# LOOTCRATE BUILDINGS KNOWN ISSUE

LootCrate Buildings currently has source/layout mismatch.

Known:

multiple `w_bldg_*` bindings exist without corresponding layout widgets.

Treat this section as:

SOURCE MISSING / DEFERRED

Do not invent the missing interface.

Do not delete handlers.

Do not fabricate widgets merely to satisfy static binding checks.

---

# BLACKMARKET STATUS

MD_BlackMarket is deprecated.

Do not redesign it.

Do not use it as an MDAP reference.

If BlackMarket appears in the AdminPanel sidebar because the external
MD_BlackMarket mod is loaded:

that registration belongs to that module.

The MD_AdminPanel redesign should not recreate BlackMarket.

---

# KNOWN SOLVED SYSTEMS

## MD_CrateEvent Double Timer

Location:

Knowledge/Bug_Fixes.md

Status:

Fixed

### Problem

Two timer systems modified the same countdown:

EOnPostSimulate()

MDCE_TimerTick()

### Solution

Removed the duplicate timer source.

Kept:

MDCE_TimerTick

as the only authoritative timer.

Added completion protection.

### Rule Learned

Only one timer source should mutate the same countdown.

Before creating timer logic search for:

- CallLater
- EOnPostSimulate
- update loops
- scheduled callbacks
- timers

Do not create a second timer for the same state.

---

# DEBUGGING RULES

Find ROOT CAUSE first.

Do not patch symptoms across many files.

Preferred process:

1. reproduce symptom
2. find execution path
3. find state owner
4. find root cause
5. make smallest safe fix
6. verify neighbouring behaviour
7. document reusable solution

Do not rewrite a complete system for a one-line bug.

---

# UI DEBUGGING RULE

For UI bugs use both:

STATIC SOURCE

and

RUNTIME SCREENSHOT / RPT

Static analysis can verify:

- widget exists
- binding exists
- type matches
- handler exists

Static analysis CANNOT prove:

- visual readability
- actual opacity
- clipping appearance
- widget overlap
- bad font scale
- hidden parent behaviour in all runtime states
- visual balance

Runtime screenshot is final visual truth.

---

# CODE CHANGE RULES

Prefer targeted changes.

Do not modify unrelated files.

Do not perform unrelated cleanup during a functional fix.

When changing code report:

- files changed
- root cause
- behaviour changed
- compile risk
- runtime risk
- regression risk

---

# CODING RULES

Always follow:

- EnScript Style Guide
- existing project naming
- existing folder structure
- existing architecture
- existing MDAP APIs
- correct DayZ layers

Project conventions:

- no ternary unless explicitly approved
- avoid inline casts
- avoid unnecessary ref locals
- ASCII source where required
- consistent CRLF
- guard clauses
- parser-compatible declaration style

Never blindly copy external code.

Adapt it to this project.

---

# EXTERNAL REFERENCE PRIORITY

External references support project development.

They do not override project architecture.

Priority:

1. Existing working MD mods
2. MD_AdminPanel
3. Known bug fixes
4. Project documentation
5. Vanilla DayZ
6. VPP Admin Tools
7. Community Framework
8. Expansion
9. Other trusted examples

---

# EXTERNAL LAYOUT PRIORITY

When researching ADMIN UI:

1. Existing MD_AdminPanel layouts
2. Existing working MD module layouts
3. VPP Admin Tools layouts
4. Vanilla DayZ layouts
5. Expansion layouts
6. CF layouts where relevant

When researching VANILLA widget/API behaviour:

1. Vanilla DayZ
2. Existing project usage
3. External examples

Do not use a visually attractive external layout if it conflicts with MDAP architecture.

---

# FINAL UI VALIDATION

Before calling a page statically functional verify:

- all required layouts exist
- all normal bindings resolve
- no duplicate widget names
- no type mismatches
- all nav buttons resolve
- all target roots exist
- all parent visibility chains are valid
- no orphaned actions
- no duplicate ambiguous Save actions
- picker hierarchy valid
- popup hierarchy valid
- map hierarchy valid
- page header does not overlap
- action bar does not overlap
- footer does not overlap
- RPC unchanged
- DATA IDs unchanged
- JSON schemas unchanged

---

# FINAL SYSTEM VALIDATION

Before creating a release/test build check:

CONFIG:

- config.cpp
- CfgPatches
- CfgMods
- requiredAddons

SCRIPTS:

- correct layers
- correct case-sensitive paths

MODDED CLASSES:

- no unintended duplicate MissionGameplay block inside one mod
- no unintended duplicate MissionServer block inside one mod
- no unintended duplicate PlayerBase block inside one mod

PAGES:

- unique pageId
- unique registration
- valid layout

RPC:

- no collisions
- unchanged IDs unless explicitly approved

UI:

- functional navigation
- reachable sections
- readable layouts
- valid bindings

---

# DOCUMENTATION RULE

When an external repository provides a useful pattern:

record it in project documentation when the knowledge is reusable.

Recommended files:

References/VPP_AdminTools_Reference.md

References/Expansion_UI_Reference.md

References/Vanilla_UI_Reference.md

Knowledge/Bug_Fixes.md

Documentation/MDAP_API.md

Documentation/MDAP_DEPENDENT_MOD_GUIDE.md

DAYZ_KNOWLEDGE_INDEX.md

---

# FINAL RULE

Before creating ANY new DayZ system:

1. Check project documentation.
2. Check known bug fixes.
3. Check existing MD mods.
4. Check MD_AdminPanel.
5. Check existing layouts.
6. Check vanilla DayZ.
7. Check VPP when admin tooling is involved.
8. Check CF/Expansion when useful.
9. Inspect actual source files.
10. Inspect actual layouts.
11. Reuse before creating.
12. Fix before rewriting.

PROJECT CODE IS THE PRIMARY SOURCE OF TRUTH.

EXTERNAL CODE IS A REFERENCE.

RUNTIME BEHAVIOUR IS THE FINAL VALIDATION.
