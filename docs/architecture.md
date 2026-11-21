# Architecture

## System Overview
[2-3 sentences: core purpose, runtime model, and main entry points.]

## Module Boundaries
| Module / Path | Responsibility | Owns Data / State | Allowed Dependencies |
|---|---|---|---|
| `src/[module]/` | [What it does] | [Tables / state / none] | [Other modules or external libs] |

## Core Data Flows
- **[Flow name, e.g., Request -> DB -> Response]:** `entry` -> `service` -> `repository` -> `external`

## External Integrations & Infrastructure
- **[Service / DB / Queue]:** [Purpose, contract/protocol, failure behavior]

## Key Invariants
- [Rules that must always hold true across the codebase, e.g., "All writes to billing tables go through `src/billing/service.ts`."]