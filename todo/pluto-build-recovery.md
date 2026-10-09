# Original Pluto build recovery

Target: original ADALM-Pluto, `pluto_maiasdr_defconfig`.
Output directory: `output/pluto/`.
Build log: `/tmp/tezuka-build.log` (temporary; may disappear after reboot).
Completed build log saved at `build/pluto-build.log`.

The interrupted October 8, 2026 build was resumed using existing output:

```bash
source sourceme.first
CMAKE_POLICY_VERSION_MINIMUM=3.5 make -C buildroot O="$PWD/output/pluto" -j8 >> /tmp/tezuka-build.log 2>&1
```

Before resuming again, check that another build is not already running:

```bash
pgrep -af 'make -C .*buildroot'
tail -30 /tmp/tezuka-build.log
```

Do not clean the output directory to resume. Completed package builds are reused.

The surviving local U-Boot fix is:
`board/tezuka/common/patches/uboot/custom/0009-fix-spl-padding-with-uutils-dd.patch`.
It replaces `dd conv=block,sync` with `dd conv=sync` for binary SPL padding.
U-Boot built successfully after that change.

The Maia HTTP service also needed a build fix in
`package/maia-httpd/maia-httpd.mk`: bindgen could not find `stddef.h` because
the installed Clang library had no builtin-header directory. Its Clang
arguments now obtain the sysroot and GCC builtin-header directory from the
selected ARM cross-compiler. Compilation passed the original binding failure
after this change.

The main firmware is at `output/pluto/images/flash/pluto.frm`, and the
complete archive is at `build/pluto.zip` (copied from
`output/pluto/images/tezuka.zip`). The separate `boot.frm` is the bootloader
update, not the main firmware. Images must match the hardware target.

Latest checkpoint: the full build completed successfully on October 8, 2026.
The latest gzip-compatible `pluto.frm` is 25,996,819 bytes, within the
31,457,280-byte flash budget.
Embedded MD5 checksums passed for both `.frm` files. Archive CRCs passed,
and the archived firmware matches the standalone flash file. FIT metadata
includes original Pluto RevA, RevB, and RevC configurations. The Maia HTTP
service executable was verified as 32-bit ARM EABI5.

Firmware SHA256:
`41c60172dd7c7635311ff7d26be646a429101875377ceb9db8147b3c485f1ee7`.

Hardware test: the user reported failure to boot after flashing the initial
LZMA-kernel image. The device subsequently enumerated as ADI DFU (0456:b674).
Official ADI v0.39 `pluto.dfu` was restored successfully to `firmware.dfu`
using dfu-util, with status 0 and exit code 0. Bootloader and calibration
partitions were left untouched. Normal boot was confirmed after a power-cycle: USB ID 0456:b673,
PlutoSDR storage mounted, USB network active, serial port /dev/ttyACM0
present, and 192.168.2.1 answered ping. The mounted info.html reports v0.39. Official recovery files are in
`build/recovery/adi-v0.39/`.

A subsequent `./build.sh pluto` run succeeded using the cached packages.
Its log is saved at `build/pluto-rebuild.log`.

## Boot compatibility diagnosis and fix

Read-only SSH checks identified the actual device as Pluto Rev.C, using
U-Boot PlutoSDR v0.20, commit 469a0fd (built July 6, 2020). The running
bootloader was backed up from `/dev/mtd0` to
`build/diagnostics/pluto-existing-boot.bin`. Its extracted U-Boot payload
and disassembly are in the same diagnostics directory. Its decompression
routine at 0x0400cd34 handles compression types 0 (none) and 1 (gzip).
Other types go directly to the `Unimplemented compression type %d` error
at 0x0400cdb4. LZMA is type 3; the initial firmware FIT selected LZMA.
This is a confirmed boot incompatibility, although a console log from the
failed boot was not captured. The text labels `lzma` in the bootloader
strings are generic image descriptions and do not indicate decoder support.

`board/tezuka/pluto/plutomaia.its` now embeds gzip-compressed raw `Image`.
`prepost-image.sh` generates `Image.gz` when requested, and its existing
self-extracting zImage corruption guard recognizes gzip as well as LZMA.
The compatible image rebuilt successfully with `./build.sh pluto`; the log
is `build/pluto-gzip-build.log`. The kernel was extracted from the final FIT
and verified to decompress exactly to the original raw `Image`. Firmware
MD5, flash budget, archive CRCs, and archived firmware identity all passed.

The new candidate has not been boot-tested. The Pluto remains on restored
official ADI v0.39. A RAM-only boot with serial logging is the preferred next
validation step before another custom flash update.

## Latest recovery and user preference

The user reported another boot failure after flashing an image from GitHub.
The exact image is unknown, so its cause has not been established. The device
again enumerated as DFU (0456:b674). Official ADI v0.39 was restored to
firmware.dfu successfully, with status 0 and exit code 0. A power-cycle and
normal boot confirmation are pending for this latest recovery.

The user explicitly requested restoration and no further changes to the Pluto.
Leave it on official ADI firmware; do not continue custom firmware testing or
flash any other partitions unless the user explicitly changes that request.
