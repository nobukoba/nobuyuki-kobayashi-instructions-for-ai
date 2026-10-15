# AMANEQ firmware programer: durable project knowledge

Scope: https://github.com/nobukoba/amaneq-firmware-programer-first-trial

Recorded from the 2026-10-14 work. This is project-specific knowledge, not a universal USB/FPGA policy. Future observations may supersede it; preserve the distinction between source/manual facts and actual hardware verification.

## User decisions

- Main entry: amaneq-firmware-programer.py (requested programer spelling), Python 3.10+ standard library, English code/help/errors, bilingual user/developer documentation.
- Unified operation naming: program-latest-mcs-amaneq-*; only Skeleton has program-latest-bit-amaneq-v4-skeleton. Other families are MCS only.
- Targets: AMANEQ v1, AMANEQ v4, HR-TDC mezzanine. Mikumari means MikuClockHub; MikuClockPrim is not the default.
- GitHub default-branch latest, older tag/commit selection, custom repo URL, direct file URL and local files are supported.
- History default is 10 total choices, increased by --history-limit up to 1000.
- Multiple connected cables without serial must print identifiers and stop before acquisition/JTAG. No interactive cable menu.
- No per-user Windows registry/PATH changes are part of this program. Earlier workstation direct-execution settings were explicitly cancelled and removed.

## Firmware sources and compatibility

- LRTDC: AMANEQ-official/StrLrTdc.
- HRTDC Base: AMANEQ-official/StrHrTdcBase.
- LRTDC-10G: AMANEQ-official/StrLrTdc-10G.
- Mikumari: AMANEQ-official/MikuClockHub.
- HRTDC mezzanine: AMANEQ-official/MznStrHrTdc.
- Skeleton: spadi-alliance/AMANEQ-Skeleton. The DAQ manual explicitly states that v1/v4 use the same Skeleton firmware; its filenames are generic rather than revision-specific.

DAQ manual: https://www.rcnp.osaka-u.ac.jp/~spadi/wiki/?SPADI-A%20DAQ%20%E3%83%9E%E3%83%8B%E3%83%A5%E3%82%A2%E3%83%AB/%E3%83%95%E3%82%A1%E3%83%BC%E3%83%A0%E3%82%A6%E3%82%A7%E3%82%A2/%E3%83%95%E3%82%A1%E3%83%BC%E3%83%A0%E3%82%A6%E3%82%A7%E3%82%A2%E3%81%AE%E6%9B%B8%E3%81%8D%E8%BE%BC%E3%81%BF%E6%96%B9%E6%B3%95
Detailed links/evidence: the project's FOR_DEVELOPERS.md and READMEs.

These targets share XC7K160T. The FPGA ID cannot identify v1/v4 or baseboard/mezzanine. Keep target selection explicit, check filename evidence and FPGA metadata, and do not infer revision from JTAG alone.

Main-board flash is S25FL128S/128Mbit. Mezzanine flash can be MT25QU256/256Mbit or older N25Q128/128Mbit. The program probes before selecting a mezzanine GitHub MCS, then rechecks before erase. Probe loads a SPI bridge into SRAM and replaces the running FPGA application, although it does not erase flash.

MT25QU256 can be shown by openFPGALoader as N25Q256A. Source-level mapping uses JEDEC 20bb19 and 32MiB; N25Q128 variants use 20bb18/20ba18 and 16MiB. Shared JEDEC signatures identify a compatible family, not an exact marketing model. Physical mezzanine programming has not been verified.

## Hardware observations on Windows

Direct openFPGALoader v1.1.1 calls with HS3 succeeded after switching FTDIBUS to WinUSB. Before changing the driver, USB opening failed with -4. Switching back to the original FTDI driver was completed later. Two identical HS3 devices can share driver binding by hardware ID; serial selection is independent of driver replacement.

HS3 USB 0403:6014 is shared with HS2/other FT232H devices. Require model evidence or explicit --cable hs3, not VID/PID alone. Public code must not contain workstation serials.

BIT SRAM loading reached DONE=1. MCS flash write recognized S25FL128S/JEDEC 012018. The binary returned exit 0 but printed Verify not supported in this mode. Independent readback matched all 6,692,572 bytes; use separate dumping/byte comparison in the Python flow rather than relying on --verify.

The Windows flash bridge loading path invoked cygpath. This executable converts paths and is not a USB driver. Dependencies were not bundled into the public source repository.

JTAG reset did not establish persistent boot (DONE stayed 0). Matching BIT was reloaded to restore DONE=1. Do not claim power-cycle boot, DAQ behavior or Ethernet/license correctness from the flash byte comparison.

## Platform and validation status

Final Python programming orchestration and full backup sequence are mock-tested, not end-to-end verified on hardware. Linux/WSL/container paths, Xilinx cable programming and mezzanine variants also remain physically unverified.

Target environments: Windows WSL2, native Linux and native macOS. macOS USB enumeration uses ioreg plist output and FTDI serial selection, with offline mock tests. Linux/WSL uses sysfs and bus/device selection. Windows native helpers remain legacy support outside this target scope. Programming on these three environments remains physically unverified. Windows driver setup, Linux USB permissions, WSL forwarding and container access remain host requirements even with bundled binaries. Apptainer usually exposes /dev and /sys but retains the host user's permissions. macOS Docker Desktop does not make a USB device available by a normal Linux bind mount alone.

The selected binary's list-cables determines capabilities. The tested Windows build lacked Xilinx Platform Cable support. The upstream Xilinx constructor does not accept bus/serial selection, so keep only one Xilinx cable connected for that path.

MCS flow: full flash backup, program, independent image-range dump, byte equality, result metadata. No automatic destructive retries or license EEPROM writing. SiTCP and SiTCP-XG are separate licenses; firmware writes do not replace those licenses.

Tracked tests use synthetic data and mocks. Keep firmware files, actual flash backups, proprietary MPC/MPCX licenses, packet captures, driver packages, credentials and local runtimes out of Git.

Windows means WSL2 for this project. Provide copy/paste PowerShell setup via setup-windows-wsl.ps1; explicit Setup installs host usbipd-win and Linux packages. USB Attach/Detach must require an explicit BUSID and never automatically select a cable or write firmware. Keep normal Python execution free of host setup side effects.

The user subsequently authorized an optional native Windows desktop app with explicit in-app WinUSB/FTDIBUS switching. This supplements the WSL route. Elevate only the driver worker; normal programming must not change drivers. Preserve a per-device hashed export of the original OEM package, require exactly one FT232H during driver changes, and never infer HS3 from VID/PID alone. Restore through Windows device APIs, not global driver deletion. New backend switching/restoration is not physically validated; previous manual restoration does not prove it. libwdi official v1.5.1 was built locally; WinUSB resources and current instance identity were checked read-only.
