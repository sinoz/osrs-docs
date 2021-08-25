---
layout: default
title: Player Info
parent: Entity Updating
---

# Player Info
{: .no_toc }

Incomplete
{: .label .label-red }

Player info is the packet responsible for transmitting information about the surrounding and external players
to the client. Player info consists of four primary loops, two of which transmit local player info,
and two of which transmit minimal information about the external players.

---

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Player Info Initialization

Before the client can begin processing the primary block of player info, it must first be initialized.
Initialization is done during `loginState` 16 and 19 in the OldSchool RuneScape client.
Initialization itself is not part of the actual Player Info packet, but is required in order for
player info to function properly.

Initialization consists of two parts:
- Writing the local player's current full coordinates, which are bitpacked into 30 bits, where the
`z` coordinate makes up the first 14 bits, the `x` coordinate makes up the next 14 bits and `level` makes up the last
two bits.
- Writing the current map quarter of all the external players. This is done in a loop from 1 to 2048(exclusive), while skipping
the index of the local player. While the maximum `x` and `z` coordinates that will ever be written to the client
for the current map quarter are 1's, the client somewhat supports values up to 8,192(exclusive).
The map quarter coordinate is packed into 18 bits, where the `z` and `x` coordinates use up eight bits each,
and the `level` coordinate uses two bits. However, if we look into how the client reads the `z` and `x` coordinates,
we see that they use the bitwise-and operator with the value 0x255(597). This is presumably an error on the behalf
of who last modified the initialization block for the OldSchool RuneScape client. The intended value behind the bitwise-and
operator is 0xFF(255). However, because the value 0x255(597) is made up out of bits 0, 2, 4, 6 and 9,
it never actually reads any invalid values, as the maximum value both the `z` and `x` coordinate can have here is 1, which uses the first bit.
Even though the right-hand side of the bitwise-and for the `z` coordinate causes the bits read to access the bits in the `x` coordinate,
it would have to access the eighth bit in order for it to have any influence over the actual coordinate value.
It is nothing short of a lucky coincidence that it currently works without problems.

### Client Code

Below is a trimmed down version of the client code behind the initialization of player info.
Only the parts which are relevant to the server itself are kept.
Here we can see the mistake in the client, where the bitwise operation `& 597` is used.
This is supposed to be `& 255`. The mistake is presumably made by accidentally writing
the hexadecimal number 0x255 instead of the decimal number 255.

```java
int packedLocalPlayerCoords = buffer.getBits(30);
byte localPlayerLevel = (byte) (packedLocalPlayerCoords >> 28);
int localPlayerX = packedLocalPlayerCoords >> 14 & 16383;
int localPlayerZ = packedLocalPlayerCoords & 16383;

for (int index = 1; index < 2048; ++index) {
    if (localPlayerIndex != index) {
        int packedExternalPlayerCoords = buffer.getBits(18);
        int externalPlayerLevel = packedExternalPlayerCoords >> 16;
        int externalPlayerX = packedExternalPlayerCoords >> 8 & 597;
        int externalPlayerZ = packedExternalPlayerCoords & 597;
        playerMapQuarters[index] = (externalPlayerX << 14) + externalPlayerZ + (externalPlayerLevel << 28);
    }
}
```

## Local and External Players

The client reads local and external player updates in two loops each.
The four loops are read in this order:
- Local players who **were not** skipped in the last update cycle.
- Local players who **were** skipped in the last update cycle.
- External players who **were not** skipped in the last update cycle.
- External players who **were** skipped in the last update cycle.

While there technically is no reason to split local and external players individually into two loops,
it is done for data compression purposes. Players who were skipped in the previous tick are likely to
remain skipped in the following ticks. This allows the client to write larger skip blocks at a time.
For example, if a player goes away from keyboard, their character is unlikely to have any updates applied to them,
thus heavily improving the probability of the player being skipped in the following ticks too.
During the iteration process in all four loops, if a player needs to be skipped due to lack of updates,
the client will give that player an activity flag of 0x2. After all four loops are complete, it will then iterate
from index 1 to 2048(exclusive) and bitwise right-shift all players' activity flags by one bit.
This will then change the activity flag for skipped players to 0x1, and those who weren't skipped to 0.

In the first local and the first external loop, the client will skip all players whose activity flag **is not** 0.
In the second local and the second external loop, it does the opposite, skipping all the players whose activity flag **is** 0.

### Local Players

Player info starts out by iterating local active and local inactive players respectively.
This section of the document is a continuation of [the skip block](#the-skip-block).

If the given player is not being skipped, we will begin processing them as follows:
First, the client reads one bit of data, which indicates if this player has any updates to be read.
Next up, the client will read two bits of data:
- If the value is 0:
  - If the player does not have any updates to be read, it informs the client that the player is being removed from the local players list.
    In addition to this, another one bit is written to the client. If the value of that is one, the client will then
    read the [map quarter block](#map-quarter-block) update for this player.
  - If the player does have updates to be read, it marks the given player to have their mask updates read further below.
  - Block ends here for this player, and moves on to the next.
- If the value is not 0, it informs the client that the given player has a movement update coming:
  - If the value is 1, the client will read three bits of data, which indicates a walk update. Those three bits will indicate
  the direction toward which the given player is moving. This is an efficient way of compressing data. The server
  simply has to determine the exact direction that corresponds the coordinate deltas for the previous and current
  coordinates of the player.
  - If the value is 2, the client will read four bits of data, which indicates a run update. These four bits define
  the run direction of the player. Once again, this is an efficient way to compress data, much like in the walk block.
  - If the value is 3, the client will be informed to read the teleport block:
    - The client will start off by reading one bit of data. This indicates whether the teleportation is a close or
    long distance one:
      - If the value of that bit is 0, the client will begin reading a close distance teleportation:
        - The client will read 12 bits of data, which contains the offset deltas for the teleportation.
          - The first five bits of that inform the client of the `z` delta. The value range is -16 to 15(inclusive).
          - The next five bits of that inform the client of the `x` delta. The value range is -16 to 15(inclusive).
          - The last two bits of that inform the client of the `level` delta. The value range here is 0 to 3(inclusive).
      - If the value of that bit is 1 however, the client will begin reading a long distance teleportation:
        - The client will read 30 bits of data, which contains the offset deltas for the teleportation.
          - The first 14 bits of that inform the client of the `z` delta. The value range is -8,192 to 8,191(inclusive).
          - The next 14 bits of that inform the client of the `x` delta. The value range is -8,192 to 8,191(inclusive).
          - The last two bits of that inform the client of the `level` delta. The value range here is 0 to 3(inclusive).
  - Block ends here for this player, and moves on to the next.

### External Players

After both of the local player blocks have been read, the client will read the active and inactive external players respectively.
This section of the document is a continuation of [the skip block](#the-skip-block).

If the given player is not being skipped, we will begin processing them as follows:
First, the client reads two bits of data:
- If the value is 0, the client is informed that the player is being added to the local players list.
  - The client will read one bit of data. If the value of that bit is 1, the client will then read the [map quarter block](#map-quarter-block).
  - The client will then read 13 bits of data, which informs the client of the `x` coordinate of the player.
    - The maximum value of the `x` coordinate is 8,191. The maximum size of the accessible game map is 16,383. In order to
    transmit a higher value, the rest of the coordinate is transmitted in the [map quarter block](#map-quarter-block) which was sent above.
  - The client will then read another 13 bits of data, which informs the client of the `z` coordinate of the player.
    - The maximum value of the `z` coordinate is 8,191. The maximum size of the accessible game map is 16,383. In order to
      transmit a higher value, the rest of the coordinate is transmitted in the [map quarter block](#map-quarter-block) which was sent above.
  - The client will then read one bit of data. If the value of that bit is 1, the client will mark the given player to have mask updates coming.
- Otherwise, the client is informed to read the [map quarter block](#map-quarter-block).

## The Skip Block

In all four loops, the start of the loop follows identical logic.
The loops go from 0 to the total number of local or external players respectively.
Using the index of the loop, the client will acquire the index of the respective player from a local cached array.
Now that it knows the index of the player who it is currently updating, it will compare their activity flag
against the expected activity flag, and skip the player if they don't belong in the activity group which it is currently updating.
The client will then read one bit of data to determine whether the given player needs to be skipped for an update.
If the value of the bit is 1, the given player's updates will be read. The breakdown
of how the [local](#local-players) and [external](#external-players) player information is read is down below.
If the value of the bit is 0, the client will begin reading the number of players to skip:
- First, the client will read 2 bits of data in order to determine the size of the upcoming skip block.
- The possible size types are 0 through 3(inclusive), which each read a fixed number of bits to determine how many players are
  being skipped in a row in this current skip block.
  - If the size type is 0, the client will not read any further information.
  - If the size type is 1, the client will read five bits of data, so a theoretical maximum of 31 players skipped.
  - If the size type is 2, the client will read eight bits of data, so a theoretical maximum of 255 players skipped.
  - If the size type is 3, the client will read eleven bits of data, so a theoretical maximum of 2,047, which also happens
    to be the maximum number of players that can be logged into one world at a time.

If the size of the skip block is larger than zero, the next players in this loop whose activity flag
matches that of the group which it is iterating will be skipped, decrementing the size by one with each iteration,
until it reaches zero. At the end of each of the four loops, the remaining number of players to be skipped **must** be zero,
or the client throws a runtime exception.

## Map Quarter Block

The map quarter block is used to transmit the full position of a character in the world.
Because the add block of the [external players](#external-players) only supports coordinates from 0 to 8,191,
this block is used to transmit the remaining bit of data. Even though this entire block could be omitted just by changing
the amount of data being read for the `x` and `z` coordinates in the add block of [external players](#external-players) to 14 bits,
this has been kept as part of legacy code, and presumably for [patent](#patent) reasons. In older versions of the client, this map quarter block
was significantly more useful, as the add block of [external players](#external-players) would only transmit 6 bits of data.
This meant that the remaining 8 bits for each of the coordinates had to be transmitted in this block.

The map quarter block is a continuation of the respective [local](#local-players) and [external](#external-players) blocks.
The client will first read two bits of data:
- If the value is 1:
  - The client will read 2 bits of data, which is the `level` delta.
- If the value is 2:
  - The client will read 2 bits of data, which is the `level` delta.
  - The client will then read the `x` and `z` coordinate deltas, which are encoded through the same code that is used to transmit the walk direction
  in the local player processing block.
- If the value is 3:
  - The client will read 2 bits of data, which is the `level` delta.
  - The client will then read 8 bits of data, which is the `x` delta.
  - Lastly, the client will read another 8 bits of data, which is the `z` delta.

## Mask Block

Masks are read after the four loops are processed. During the processing of the those loops, players who have a mask update pending
will be marked so. In this part of the code, the client will iterate over all the players who were marked for a mask update, in the
exact order that they were marked in. To start off, the client will read one byte of data. This byte of data is the actual mask.
The bits which are enabled in that byte inform the client of the mask blocks that need to be read. In addition to this,
because there are more than eight masks for player updating, one of the bits in that byte will be marked as enabled
if the client needs to read another byte as the mask. The bit which indicates the extended mask varies between revisions.
If the bit is enabled, the client will read that other byte of data. It will then shift this new value to the left by 8 bits, then
add the value to the original mask.

Further below, the client will begin checking various bits, which are in random order in every revision of the client.
If the respective bit is enabled, it indicates the client to read that specific mask update. It is important to keep in mind that
the order in which the mask readings are defined in the client must equal the order in which the masks are written by the server.

Further info on how each of the masks is encoded can be read [here](update-masks.md).

## Patent

The patent behind player info can be found [here](https://patentimages.storage.googleapis.com/d3/4b/56/8d4360912ea128/US8441486.pdf).