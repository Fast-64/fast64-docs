How Animations Work
===================

Animation Variant (Header) Format
---------------------------------
**Flags** (``s16 flags`` ``[0-2]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  - **Bit 0** *No Loop* (``ANIM_FLAG_NOLOOP``):
     The animation will not repeat from the loop start after reaching the loop end frame.
  - **Bit 1** *Backwards* (``ANIM_FLAG_FORWARD`` or ``ANIM_FLAG_BACKWARD`` in refresh 16):
     The animation will loop (or stop if looping is disabled) after reaching the loop start frame. Tipically used with animations which use acceleration to play an animation backwards.
  - **Bit 2** *No Acceleration* (``ANIM_FLAG_2`` or ``ANIM_FLAG_NO_ACCEL`` in HackerSM64):
     Acceleration will not be used when calculating which animation frame is next.
  - **Bit 3** *Only Translate Horizontally* (``ANIM_FLAG_HOR_TRANS``):
     Only the animation horizontal translation will be applied during rendering (takes priority over no translation, shadows included) the vertical translation will still be included.
  - **Bit 4** *Only Translate Vertically* (``ANIM_FLAG_VERT_TRANS``):
     Only the animation vertical translation will be applied during rendering (takes priority over no translation and only horizontal, shadows included) the horizontal translation will still be included.
  - **Bit 5** *No Shadow Translation* (``ANIM_FLAG_5`` or ``ANIM_FLAG_DISABLED`` in HackerSM64):
     Shadows do not rely on the matrix stack, they are translated using the animation’s translation directly. This flag disables that behavior.
  - **Bit 6** *No Translation* (``ANIM_FLAG_6`` or ``ANIM_FLAG_NO_TRANS`` in HackerSM64):
     The animation translation will not be used during rendering (shadows included), the translation will still be included.
  - **Bit 7** *Unused* (``ANIM_FLAG_7`` or ``ANIM_FLAG_UNUSED`` in HackerSM64):
     Has no behavior and is not used on any animation. Only exists as a decomp enum.

**Translation Divisor** (``s16 animYTransDivisor`` ``[2-4]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If set to 0, the translation multiplier will be 1.
Otherwise, the translation multiplier is determined by dividing the object's translation dividend (animYTrans) by this divisor.

**Start Frame** (``s16 startFrame`` ``[4-6]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The starting frame of the animation, not the same as loop start.

**Loop Start Frame** (``s16 loopStart`` ``[6-8]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If *Backwards* is not set, this will be the starting frame after each loop, otherwise this will be treated as the loop end frame.

**(Loop) End Frame** (``s16 loopEnd`` ``[8-10]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Both the end loop frame and the actual end frame.
If *Backwards* is not set, this will be the ending frame of the animation, otherwise this will be treated as the loop start frame.

**Bone Count** (``s16 boneCount`` ``[10-12]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The number of bones in the animation. Usually equal to the number of bones rendering at the same time in the model but not always.

Derived from the ``size of the indice table`` / 2 (Pair size) / 3 (Axis count) - 1 (Translation)

**Values Table** (``const s16 *values`` ``[12-16]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Pointer to array of 16-bit values of arbitrary length. See `Animation Data Format`_

**Indice Table** (``const s16 *index`` ``[16-20]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Pointer to array of 16-bit indices of arbitrary length. See `Animation Data Format`_

**Length** (``u32 length`` ``[20-24]`` ``Unused``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
DMA exclusive property (0 in all other actors), length of the animation in bytes including the header, indice table, and values table.

Animation Data Format
---------------------
Animation data is stored using two tables, the **indice table** (unsigned 16-bit) and the **values table** (signed 16-bit).

The root bone has data for translation and rotation, all other bones only have data for rotation.
Translation is in XYZ order in normal SM64 units (just casted to s16).
Rotations are represented as euler angles in XYZ order, each angle is in degrees normalized to 16-bit:

 ``0° = 0``, ``90° = 16384 (0x4000)``, ``180° = -32768 (0x8000)``, ``270° = -16384 (0xC000)``

The engine iterates through the indice table to find the value for each axis in the current frame.
Each pair of values represents an axis:

  #. Frame Count (The number of frames for the axis)
  #. Value Table Offset (Offset into the values table in the header)

The first 3 axis are the translation, all following axis are rotations.

Example Data:
~~~~~~~~~~~~~
In this example, the animation will loop from frame 0 to frame 4.
There will be no translation (all values will be 0) and the angles will all rotate from 0° to 270°.

.. code-block:: c

    static const s16 values_table[4] = {
        0x0000, 0x4000, 0x8000, 0xC000
    };

    static const s16 indice_table[4] = {
        /*translation - x*/ 1, 0, // offset 0, 1 frame, value 0
        /*translation - y*/ 1, 0, // offset 0, 1 frame, value 0
        /*translation - z*/ 1, 0, // offset 0, 1 frame, value 0
        /*rotation - x*/    4, 0, // offset 0, 4 frames, values [0, 16384, -32768, -16384]
        /*rotation - y*/    4, 0, // offset 0, 4 frames, values [0, 16384, -32768, -16384]
        /*rotation - z*/    4, 0  // offset 0, 4 frames, values [0, 16384, -32768, -16384]
    };

    static const struct Animation header = {
        .flags = 0, // No flags, will loop
        .animYTransDivisor = 0,
        .startFrame = 0,
        .loopStart = 0,
        .loopEnd = 4,
        .boneCount = ANIMINDEX_NUMPARTS(indice_table),
        .values = values_table,
        .index = indice_table,
        .length = 0
    };