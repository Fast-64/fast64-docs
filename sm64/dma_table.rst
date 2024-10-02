DMA Tables
==========

DMA Tables are used for data that needs to be loaded dynamically, such as animations and demo inputs.
They are uncompressed, so their data is DMA'd into memory directly, hence the name.

Format
------

**Number of Entries** (``u32 numEntries`` ``[0-4]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
How many entries are in the table.

**Source Address** (``const void *addrPlaceholder`` / ``u8 *srcAddr`` ``[4-8]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Set to NULL in the ROM, exists for knowing the ROM address after setting up the handler in ``setup_dma_table_list()``.

**Entries** (``struct OffsetSizePair[numEntries]`` ``[8-(numEntries*8 + 8)]``):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Offset Size Pair** (``struct OffsetSizePair``):
  **Offset** (``u32 offset`` ``[0-4]``):
    Offset from the start of the table.
  **Size** (``u32 size`` ``[4-8]``):
    Size in bytes of the entry.