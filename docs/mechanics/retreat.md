---
parent: Mechanics
---

# Retreat Mechanic

Retreating is a mechanic used by NPCs to primarily escape from combat. A version of retreating can be seen
when shooing the stray dog in Varrock, causing it to retreat seven game squares away from the player.

## Conditions

The mechanic will generally be executed under two circumstances:
- The NPC gets attacked from outside their [Max Range](../variables/max-range.md#max-range).
- The NPC's hitpoints fall below a certain threshold.
  This only applies to select few NPCs, such as imps and chickens.

The mechanic will halt under the following circumstances:
- The NPC can no longer retreat in the given direction and at least one of the two sides of the direction
in which the NPC is retreating contains a wall. If the NPC retreats to a position in which it can no longer 
go any further(due to the [Max Range](../variables/max-range.md#max-range) restrictions), but isn't blocked by anything that would prevent
movement in either of the directions of the retreat direction(E.G. if the NPC retreats South-West,
meaning neither the South direction nor the West direction contains anything that would otherwise block the movement),
the NPC will continue trying to retreat.
- The [Chebyshev distance](https://en.wikipedia.org/wiki/Chebyshev_distance) between the NPC and whom it is retreating from
is greater than 25 game tiles.
- The NPC gets interrupted by something else, such as a player attacking it.

## Retreat Directions

An NPC will always retreat away in a specific direction in relation to whom it's retreating from.
The below table shows the direction in which an NPC will retreat:


| Entity position        |      Retreat direction      |
|------------------------|:---------------------------:|
| South of the NPC       |      North-West             |
| East of the NPC        |      South-West             |
| North of the NPC       |      South-West             |
| West of the NPC        |      South-East             |
| South-East of the NPC  |      North-West             |
| North-West of the NPC  |      South-East             |
| North-East of the NPC  |      South-West             |
| South-West of the NPC  |      North-East             |
| On-top of the NPC      |      South-West             |

*The entity position denotes the position at which the entity from whom the NPC is retreating from is standing at.*

The generation of the direction can be achieved using this code block:
```kotlin
val retreatDirection = when {
    target.x >= this.x && target.y >= this.y -> Direction.SouthWest
    target.x >= this.x && target.y < this.y -> Direction.NorthWest
    target.x < this.x && target.y >= this.y -> Direction.SouthEast
    else -> Direction.NorthEast
}
```
*The **target** stands for the entity from whom the NPC is retreating, whereas **this** refers to the NPC who is retreating.*

## Media

![Retreat Example](../../assets/media/retreat/retreat-1.gif)

*The above video demonstrates attacking an NPC, moving outside its [Max Range](../variables/max-range.md#max-range),
then attacking it to cause it to retreat. The NPC then begins retreating, actively re-calculating the
path it's taking in relation to the target. The gif ends with the NPC moving up against the wall, thus
halting the retreating.*