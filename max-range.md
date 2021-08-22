# Max Range

### Note: This thread is incomplete

The max range denotes the radius of the area in which the NPC can move by normal means.
The default value for NPCs which do not have it defined is **7 squares**.
The max range value is defined per NPC config, meaning that two NPCs with the same
internal id must also share the max range. If an NPC is attacked from within the max range area,
but then attempted to be dragged outside it, the NPC will lose interest in their target.

## Exceptions

Below are listed some exceptions which allow a NPC to go outside its max range. When that happens,
// TO BE TESTED

- Dragon spear special attack can push the NPC outside its max range.