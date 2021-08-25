---
parent: Variables
---

# Client Tick

Client tick is a unit of time, equivalent to 20 milliseconds real-world time.
OldSchool RuneScape, on the java client, is currently capped to 50 frames per second.
This perfectly matches up with the frame rate, as 50 * 20ms is equal to 1,000ms, or one second.
There are 30 client ticks in one server tick.

## Uses

Client ticks are often used for communication between the server and the client, to inform the client of a precise delay for what is
being transmitted. Below are just some cases where a unit of client ticks is transmitted:
- Masks:
  - [Hit mask](../updating/hit-mask.md#hit-mask)
    - Used to define the delay until the [hit splats](../updating/hit-mask.md#hits) and [head bars](../updating/hit-mask.md#head-bars) begin rendering.
    - Additionally, used to define the timespan of the progression for the head bar movement.
  - Sequence(animation) mask
    - Used to define the delay until the animation begins playing.
  - ExactMove mask
    - Used to define the two delays for the two movements that are supported in the mask.
  - Spotanim(graphics) mask
    - Used to define the delay until the graphics appear.
- Sound effect packet
  - Used to define the delay until the sound effect begins playing.
- Area sound effect packet
  - Used to define the delay until the sound effect begins playing.

Additionally, client ticks are also often used inside the configs in the cache to define the delays for things such as sound effects.