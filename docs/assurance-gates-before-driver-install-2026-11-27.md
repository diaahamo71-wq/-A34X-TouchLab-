# Assurance protocol before any driver replacement — 2026-11-27

## User risk preference
User accepts potential risks in principle but insists that the procedure be as guaranteed/reliable as possible. This is not equivalent to accepting an unverified driver or an avoidable brick risk.

## Evidence reviewed
- Exact target: SM-A346E, A346EXXSGFZJ4, Android 16, changelist 33505653, kernel 6.6.92-android15-8-abA346EXXSGFZJ4-4k.
- Samsung Knox documentation states the Warranty Bit is a persistent hardware fuse; non-approved kernel/configuration can trip it and affect Knox-protected services/data.
- Public A346E kernel source exists, but no proof yet that its touchscreen implementation/config matches this exact FZJ4 stock image.
- The 7.9 GB full firmware package is not a good first step given the user's slow connection; seek exact source/vendor metadata through repositories or smaller targeted artifacts first.

## Required confidence gates
A driver modification must not be presented as "guaranteed." Before any phone-side installation:
1. Identify exact controller and panel variant from authoritative device-specific data.
2. Match source tree, kernel ABI/config, DT/DTBO, module and firmware blob to FZJ4.
3. Audit existing touch driver path and Samsung sec_input integration.
4. Make a minimal patch to the correct driver (prefer tuning a validated parameter over wholesale driver replacement).
5. Build reproducibly against the exact source/config; record hashes and toolchain.
6. Static review and build validation; no unsupported interfaces or fabricated sysfs nodes.
7. Confirm recovery route, verified stock firmware package, and complete user-data backup before boot-chain changes.
8. Explain exact Knox/OTA/warranty impact and ask for explicit confirmation immediately before any irreversible action.
9. Do not remotely or automatically unlock, flash, or modify the user's device; user must perform and confirm local steps.

## Current stop condition
No exact controller / exact matching FZJ4 driver source / recovery proof => **NO INSTALL**. Continue source analysis only. Do not promise zero risk.
