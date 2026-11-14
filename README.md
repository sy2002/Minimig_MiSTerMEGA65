Amiga 500 for MEGA65
====================

This is a Git submodule of the Amiga 500 core for the MEGA65 (AExp). It is a
fork of the MiSTer FPGA Minimig-AGA core, which is based on Minimig by Dennis
van Weeren, with heavy later modifications by many different people.

**Go to https://github.com/sy2002/AExp to learn more.**

The MEGA65 port is based on the
[MiSTer2MEGA65 framework](https://github.com/sy2002/MiSTer2MEGA65). We forked
the [MiSTer core](https://github.com/MiSTer-devel/Minimig-AGA_MiSTer), so that
we can easily track upstream changes and merge them into our MEGA65 port as
needed. The following structure is being used:

* [MiSTer](https://github.com/sy2002/Minimig_MiSTerMEGA65/tree/MiSTer)
  branch: An unmodified mirror of the
  [original upstream MiSTer core](https://github.com/MiSTer-devel/Minimig-AGA_MiSTer)
  (upstream branch `MiSTer`). Upstream changes enter our fork here first and
  are then merged into `develop` as needed. The original upstream README
  lives in this branch.
* [master](https://github.com/sy2002/Minimig_MiSTerMEGA65) branch: Contains
  our stable modifications of the upstream MiSTer core. It equals `develop`
  at each official AExp release.
* [develop](https://github.com/sy2002/Minimig_MiSTerMEGA65/tree/develop)
  branch: Is our kind-of stable work-in-progress (WIP) development branch.
  The AExp main repository tracks this branch.

On modifications:

* Our strategy is to reduce the modifications to the upstream core to the
  bare minimum. Every change to an original file carries a dated provenance
  comment, and the original code is kept as a comment next to it.
* The scope is an Amiga 500: OCS chipset, PAL, 68000 CPU
  ([fx68k](https://github.com/ijor/fx68k)), 512 KB Chip RAM plus 512 KB Slow
  RAM. AGA, the 68020 (TG68K), IDE, the Toccata sound card and the SDRAM /
  turbo paths are not used and partly removed or tied off:
  `rtl/cpu_wrapper.v` instantiates the fx68k as its only CPU.
* We made sure that the code is actually synthesizing using Vivado (used for
  the MEGA65), which is stricter and more unforgiving than Quartus (used for
  the MiSTer): the Altera `altsyncram` / `altera_mf` memories are replaced by
  portable inferred block RAM templates, and a mechanical compatibility sweep
  fixed multi-driven registers, SystemVerilog array syntax and similar
  constructs.
* We added `rtl/minimig_m65.v`, a thin wrapper that renames the
  underscore-prefixed ports of `minimig.v` (illegal VHDL identifiers) and
  ties off the unused subsystems, so that the core can be instantiated from
  the VHDL world of the MEGA65 framework.
* We made the keyboard, mouse, battery-backed real-time clock and the ADF
  floppy host channel (`IO_FPGA`) compatible with the MEGA65 core. The
  keyboard uses a real CIA handshake (`kbd_ack`) instead of fixed timing.
* We extended Paula's floppy controller (`paula_floppy.v`) so that the
  MEGA65's internal 3.5" drive can act as a real Amiga floppy drive
  ("Hardware Floppy"), including a faithful `DSKBYTR` register for copy
  protections such as Rob Northen Copylock. With the Hardware Floppy
  switched off, the controller behaves exactly like the upstream version
  this fork is based on.
* We backported four chipset fixes that upstream made after this fork
  started: the CIA timers count pulses on their `CNT` input (MiSTer
  [PR 230](https://github.com/MiSTer-devel/Minimig-AGA_MiSTer/pull/230)),
  the blitter freezes when fill mode is switched off during a fill blit
  ([PR 236](https://github.com/MiSTer-devel/Minimig-AGA_MiSTer/pull/236)),
  `VHPOSR` reads back the exact beam position
  ([PR 234](https://github.com/MiSTer-devel/Minimig-AGA_MiSTer/pull/234)),
  and the field flag is only set while interlace is on (commit
  [`d16cd84`](https://github.com/MiSTer-devel/Minimig-AGA_MiSTer/commit/d16cd8458cf8e915c5622ecc7c12ce69d776753c)).

On the MiSTer support software:

* On the MiSTer, Minimig works together with software on the ARM processor
  (the HPS): the `support/minimig/` folder of
  [`Main_MiSTer`](https://github.com/MiSTer-devel/Main_MiSTer) configures the
  core, uploads the Kickstart ROM and serves the floppy disk images over the
  host channel (`IO_UIO`, `IO_FPGA`). The MEGA65 has no such processor, and
  its QNICE helper CPU is neither fast enough nor connected to serve Paula's
  floppy channel word by word. So AExp replaces this software with hardware
  in its main repository, and the RTL in this fork is driven exactly the way
  the HPS would drive it.
* `minimig_fdd.cpp`, the floppy service, became the AExp track engine
  (`CORE/vhdl/adf_track_engine.vhd`). Its MFM encoder and decoder are
  bit-exact ports of the C code, extended for three drives, disk images in
  HyperRAM with a background write-back to the SD card, real MFM clock bits,
  and the Hardware Floppy.
* `minimig_config.cpp` is the model for `CORE/vhdl/amiga_config.vhd`, which
  replays the userio configuration commands `0xF1` to `0xF9` after every
  reset, with Amiga 500 values and with the Slow RAM and drive-count
  settings of the AExp menu. The Kickstart upload of that file is
  replaced by the MiSTer2MEGA65 ROM loader, which writes the ROM straight into
  block RAM.
* Verbatim reference copies of both files, taken from Main_MiSTer commit
  `c738023` ("Release 20260603", the MiSTer release that goes with the
  Minimig release this fork started from), are kept in the AExp repository:
  [`minimig_fdd.cpp`](https://github.com/sy2002/AExp/blob/develop/doc/developers/minimig_fdd.cpp)
  and
  [`minimig_config.cpp`](https://github.com/sy2002/AExp/blob/develop/doc/developers/minimig_config.cpp).
  The AExp [architecture overview](https://github.com/sy2002/AExp/blob/develop/doc/developers/architecture.md#10-the-mister-hps-code-and-its-replacements)
  lists which other HPS functions AExp replaces and how.
