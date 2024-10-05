Importing Existing Animations
=============================

To import animations, you will need a rig. 
See the `Importing/Exporting SM64 Geolayouts <https://github.com/Fast-64/fast64/blob/main/fast64_internal/sm64/README.md#importingexporting-sm64-geolayouts>`_ 
documentation or use the `Example Mario Model <https://github.com/Fast64/fast64-models/tree/mario-examples/mario>`_

Enable *"Show Importing Menus"* in the *"SM64 General Settings"* panel.

.. image:: ../showing_importing_menus.png
    :align: center
    :alt: SM64 Animation Inspector with the importing sub-panel opened

After that, the sub-panel *"Importing"* under *"SM64 Animation Inspector"* will be visible.

.. image:: importer.png
    :align: center
    :alt: SM64 Animation Inspector with the importing sub-panel opened

Set *"Type"* based on your import goal and follow the relevant instructions:
    - `C <#importing-existing-animations-c>`_
    - `Binary <#importing-existing-animations-binary>`_
    - `Insertable Binary <#importing-existing-animations-insertable-binary>`_

Importing Existing Animations (C)
---------------------------------
Presets:
~~~~~~~~
- Use the search button next to the *"Preset"* dropdown.
- Enable *"Read Entire Table"* to import all animations (default), or disable it to select individual animations (use custom for a specific index).

Custom Presets:
~~~~~~~~~~~~~~~
- Select *"Custom"*, choose a folder/file, or set the decomp path in the importer or general settings. 
- Enable *"Use Custom Name"* for original names.

See `General Import Settings`_ for more.

Importing Existing Animations (Binary)
--------------------------------------

Setting Up the ROM
~~~~~~~~~~~~~~~~~~
.. include:: ../enable_importing_binary.rst

Presets:
~~~~~~~~
- Use the search button next to the *"Preset"* dropdown.
- Enable *"Read Entire Table"* to import all animations (default), or disable it to select individual animations (use custom for a specific index).

Custom Presets:
~~~~~~~~~~~~~~~
- Select *"Custom"* in the *"Preset"* dropdown. 
- Configure the following options based on import type:
    - *"DMA"*: Import from a DMA table (Mario), you will need to set the *"DMA Table Address"* to the address in ROM.
    - *"Table"*: 
      Import from a table, you will need to enable/disable *"Is Segmented Address"* and set the address of the table, 
      set *"Level"* to a level where the table is loaded.

      To import the entire table, enable *"Read Entire Table"* (on by default), otherwise choose an index.

      Enable *"Check NULL Delimiter"* if the table's last element is a ``NULL (0x00)``, 
      otherwise set *"Size"* to the amount of elements if you have *"Read Entire Table"* enabled.
    - *"Animation"*: Import a single animation, you will need to enable/disable *"Is Segmented Address"*, set the address of the animation header 
      and set *"Level"* to a level where the animation is loaded.

- Enable *"Ignore Bone Count"* to ignore the bone count of the target armature, 
  this is necesary for when headers have a different bone count than the target armature.

Importing Existing Animations (Insertable Binary)
-------------------------------------------------
There are no presets for this type.

Enable *"Ignore Bone Count"* to ignore the bone count of the target armature, 
this is necesary for when headers have a different bone count than the target armature.

Table Import Settings
~~~~~~~~~~~~~~~~~~~~~~
When importing an insertable binary file of table type (4), the settings under *"Table Imports"* are used.

- To import the entire table, enable *"Read Entire Table"* (on by default), otherwise choose an index.
- Enable *"Check NULL Delimiter"* if the table's last element is a ``NULL (0x00)``, otherwise set *"Size"* to the amount of elements if you have *"Read Entire Table"* enabled.

General Import Settings
~~~~~~~~~~~~~~~~~~~~~~~
| The built-in animation f-curve decimate operator can be used to automatically clean up animations on import. 
    Enable *"Run Decimate (Allowed Change)"* (on by default) to use it.
| This is recommended for the ease of editing animations. The default error margin should keep the animation visually intact.
| **Be warned** this can be a bit slow.

| For armatures without actions, enable *"Force Quaternions"*, 
    this sets all the animated bones in the armature to the quaternion rotation mode, which prevents gimbal lock.
| Alternatively, enable *"Continuity Filter"* (on by default) to at least fix any existing gimbal lock in the animations.

Enable *"Clear Table On Import"* (on by default) to clear the table elements of the armature on import.

Finally, select your armature and click *"Import Animation(s)"*.
