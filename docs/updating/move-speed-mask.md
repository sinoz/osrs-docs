---
layout: default
title: Move Speed Mask
parent: Update Masks
grand_parent: Entity Updating
---

# Move Speed Mask
{: .no_toc }

Move speed masks are used for player updating, to inform the client of the current movement speed of the character.
There are two move speed masks - temporary and cached.

---

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Move Speed Types

There are four move speed types in OldSchool RuneScape:
- Exact
  - Internal id: 0
  - Used only with the exact move mask.
- Walk
  - Internal id: 1
- Run
  - Internal id: 2
- Instant
  - Internal id: 127
  - Used for teleportation.
  - Cannot be cached, meaning only the temporary mask works for transmitting this type.

## Temporary Move Speed Mask

The temporary move mask is used by the server to indicate the client of a single server tick movement.
This is primarily used for the exact and instant [types](#move-speed-types), as both of those are generally only
transmitted for one server tick.
The temporary move mask is also used in a scenario where the player has cached the run [type](#move-speed-types),
but only requests to move a single game square.

## Cached Move Speed Mask

The cached move speed mask is used by the server to inform the client of a manual change in movement speed.
It is typically sent when the player changes the status of their run orb, then moves to transmit the changes to
the client. Instead of using the [temporary move speed mask](#temporary-move-speed-mask) for every server tick
that the player moves for, the cached speed is instead updated, notifying the client to use that move speed
for all the movement that follows, that doesn't explicitly come with any move speed mask.