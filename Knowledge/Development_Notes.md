# DayZ Development Notes

This document contains project development principles.

------------------------------------------------------------------------

# Architecture

Prefer modular systems.

Separate:

-   Scripts
-   Configuration
-   Data
-   Server logic
-   Client logic

------------------------------------------------------------------------

# New Feature Workflow

Before creating a new system:

1.  Search existing solutions.
2.  Check vanilla DayZ behaviour.
3.  Check frameworks if required.
4.  Design architecture.
5.  Implement clean code.

------------------------------------------------------------------------

# Networking

Server should always be authoritative.

Client requests actions. Server validates and executes.

------------------------------------------------------------------------

# Configuration

Avoid hardcoded values.

Use:

-   json
-   config.cpp
-   external configuration files

------------------------------------------------------------------------

# Code Quality

Prefer:

-   readable classes
-   reusable functions
-   minimal dependencies
-   clear logging
