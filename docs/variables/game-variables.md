---
parent: Variables
---

# Game Variables(Varbits, Varps and Varcs)

## Varps

Varps, which stands for "var player", or "player variable", are variables which the server uses
to track different states of the player. In the OldSchool client, varps are stored in
an int array with a size of 4,000 as of writing this document. All varps are 32-bit integers.

### Varp Attributes

Each varp only has two attributes defined for it:
- Persistence:
  - Whether the varp's value must be saved on the player's character file.
  - This information is not transmitted to the client, Jagex defines this value locally on their server.
- Transmitting:
  - Whether the varp will be transmitted to the client.
  - This information is not fully available to us. We can only derive the usages by looking
  at the configs(outside varbit configs - see below) which make use of these varps, client scripts,
  and direct client code usages.

### Packets
There are two packets which can be used to transmit varp values to the client:
- Varp Small:
  - This packet is used if the value of the varp transmitted can fit into a single 8-bit byte.
  This means the value of the varp must be from -128 to 127(inclusive) in order for this packet
  to be used.
- Varp Large:
  - This packet is used if the value of the varp transmitted exceeds the capacity of a single 8-bit byte.

### Client Uses
Varps are used in the following scenarios:
- Configs
  - As of writing this document, varps are directly used by three configs:
    - `NPCType`
    - `LocType`(Object configs)
    - `HitmarkType`
  - These are used to change the visual representation of the config to a different config of the same type,
  so for example, farming patch states are transmitted through the use of varps.
  In the respective base config, the base varp is defined, as well as the different transformations
  that the config can take, depending on the value that is assigned to the varp. Each value refers to the respective
  index in the transformations array.
- Client scripts
  - An example of this would be changing the state of a button on an interface,
  such as enabling or disabling the state of assist aid.
- Direct client code usages
  - Varps are rarely used directly by the client code. An example usage of this is
  defining the attack priority for a specific target, or informing the client of the
  index of the player's current follower to move their click options further down.

## Varbits
Varbits, as their name suggests, are bits of player variables. Their data is stored within
[varps](#varps) themselves. Each varbit has a single parent varp. They can consist of 
anywhere from 1 to 32 bits of data.

### Varbit Attributes
Varbits come with three attributes:
- The parent varp.
  - The value of a varbit is bitpacked into the parent varp.
- Least significant bit.
  - Refers to the first bit which is used for defining the varbit's value range.
- Most significant bit.
  - Refers to the last bit which is used for defining the varbit's value range. This value is inclusive.

### Packets
While varbits themselves do not have dedicated packets, their values are transmitted
by sending the respective [varp](#varps) to the client.

### Client Uses
Varbits are used in an identical manner to varps themselves, with the main difference being that configs
have a dedicated varbit field to hook to a specific varbit, rather than listen to all the base varp changes.

### Example bitpacking

Varbit configs can be found in the OldSchool RuneScape cache, in index 2, archive 14. Each file represents one
varbit config. Each of these configs will have all three [attributes](#varbit-attributes) defined.
Using said attributes, we know exactly which bit range the given varbit alters.

#### Example - *Requires clearer examples!*

*Please keep in mind that the below example is only meant to demonstrate how to change and transmit a varbit
value change. It is not meant to give you a full understanding of how bitwise operations work - that is a
broad subject with plenty of tutorials available online.*

Let's imagine our varbit is defined with the least significant bit as 5, and the most significant bit as 20.
If we wish to transmit a change in the given varbit's value to the client, but do not want to alter the rest
of the base varp in the process(as the rest is likely made up out of other varbits), we must first exclude
the current value from the existing base varp value, and then add our desired varbit value to the base varp.

```kotlin
// For the purpose of this demonstration, we construct the varp array here in our code.
// These values are supposed to be serialized to the players' character files, and acquired from there instead.
val varps = IntArray(4_000)

// Let us construct a dummy varbit config for this example.
val config = VarbitConfig(lsb = 5, msb = 20, baseVar = 1_000)
// Now, let's get the current cached varp value. The default for this is 0, if the varp has never been touched by this player.
val currentValue = varps[config.baseVar]

// Now, let us clear all the bits that this varbit uses from the current varp value.
// In order to do this, we must first find out what the maximum value that the varbit can carry is.
val maximumValue = (1 shl (1 + (config.msb - config.lsb))) - 1
// In the above code, we shift 1 to the left by the number of bits the varbit occupies.
// Since the varbit ranges are inclusive, we need to add 1 to it.
// Because we shifted it by the total number of bits, the value we now have is 1 above what we need, so we subtract one from it.

// Now, in order to actually clear the bits from the existing varp value, we must use the bitwise-and operation
// on the inverted value of the maximum value, which must first also be shifted by the number of the least significant bits.
val clearedValue = currentValue and (maximumValue shl config.lsb).inv()

// Now that the base varp no longer has any of the bits used by the varbit enabled, we can add our desired
// value onto the clearedValue.
// Let's use the value 10,000 for our example varbit value.

val shiftedDesiredValue = 10_000 shl config.lsb
// In the above code, we take our desired value of 10,000 and shift it to the left by the number of the least significant bits
// in this varbit.

// After having done that, we can now include the shifted value back into the clearedValue.
val finalVarpValue = clearedValue or shiftedDesiredValue

// Now that we know the final value of the base varp, we should cache the value locally.
varps[config.baseVar] = finalVarpValue

// In order to transmit the varbit value change to the client, we use the Large Varp packet
// with the varp id of 1_000 and a value of ´finalVarpValue´ and send it to the client.
```

## Varcs

Varcs are client variables. They are strictly used by the client to track different states of the session, such as
the position of a scrollbar on an interface. Varc values can be either strings or 32-bit integers.
The server does not know the state of varcs, and does not have any dedicated packets to alter the stages.
The only way possible for the server to change the state of a varc is if there is a dedicated client script
made to do so.