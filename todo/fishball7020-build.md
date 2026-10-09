# Fishball 7020 / AD9361 SD build

The SD-only build completed on October 9, 2026 using
`fishball_maiasdr_7020_defconfig`. The required SD files were present,
the ZIP CRC checks passed, and archived files matched the build output.
The board has not arrived yet, so boot and radio operation are untested.

From the repository root, resume the existing build with:

```bash
source sourceme.first
make -C buildroot O="$PWD/output/fishball7020" -j4
```

For a fresh output directory, select the board before building:

```bash
source sourceme.first
make -C buildroot O="$PWD/output/fishball7020" fishball_maiasdr_7020_defconfig
make -C buildroot O="$PWD/output/fishball7020" -j4
```

The package is `output/fishball7020/images/tezuka.zip`. Copy the contents
of `output/fishball7020/images/sdimg/` to the root of a FAT32 SD card.
Use the standard `BOOT.bin`; do not substitute an overclock variant.
Check the delivered board's revision and SD boot selector before booting.

This SD build includes its own U-Boot with `CONFIG_LZMA=y` and
`CONFIG_SYS_BOOTM_LEN=0x3C00000`, matching its LZMA kernel. It also includes
Maia-SDR, the Tezuka dashboard, IIO/SoapySDR, MQTT, DATV streaming programs,
WireGuard and the experimental classifier. Features still require hardware
validation on the delivered board.

Git preserves source changes and these instructions, not build caches.
Keep `buildroot/` and `output/fishball7020/` locally for quicker rebuilds.
Both directories, along with `build/`, are excluded by `.gitignore`.
Build SD images for the new board without changing its factory flash.
