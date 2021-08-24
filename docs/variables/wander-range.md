---
parent: Variables
---

# Wander Range

Wander range defines the distance that a NPC can wander from its spawn location.
The default wander range is **five squares**. Wander range is defined per NPC config,
meaning that two NPCs with the same internal id must also share their wander range.

## Trivia

- Stationary NPCs, such as [bankers](https://oldschool.runescape.wiki/w/Banker), have a wander range of zero squares.
This means they can never move from their spawn position.
- [Bob the Jagex cat](https://oldschool.runescape.wiki/w/Bob_(cat)) and [implings](https://oldschool.runescape.wiki/w/Impling)
have a theoretical wander range of at least 16,383 game squares, as that is the total diameter of the entire game map, meaning
they can wander to any position in the game, as long as the terrain allows it.