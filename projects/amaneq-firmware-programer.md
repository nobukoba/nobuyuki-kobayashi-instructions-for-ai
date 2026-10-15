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

The user requested a macOS .app bundling Python/Tk/openFPGALoader. Build arm64 and x86_64 on native macOS CI, preserve .app signatures by keeping all mutable data under ~/Library/Application Support/AMANEQ, and disable Windows driver actions. Distinguish CI startup/dependency checks from physical HS3 programming. Developer ID/notarization is not configured; do not describe ad-hoc signing as notarization.

On 2026-10-15, macOS Actions run 38740982476 successfully built and smoke-tested both arm64 and x86_64 .apps with bundled Python/Tk/openFPGALoader. All 47 offline tests passed. This is native macOS CI evidence, not physical HS3 programming. The .app targets macOS 15+, is ad-hoc signed and not notarized, and is currently delivered as 30-day GitHub Actions artifacts.

Desktop corrections: one window, no separate Windows tab on Mac; scan cables and list latest image filenames at startup. Let the user choose the Ethernet adapter and enter the PC IPv4/mask; do not hardcode a workstation address. Optional Mac temporary addressing uses an additional nonpersistent alias, preserving DHCP/DNS/gateway and removing only the added alias on restore/exit with a guardian. Never change networking at startup. macOS USB-driver simplicity is separate from Gatekeeper/notarization. Distribute visible Release assets rather than requiring users to find Actions artifacts. Include CA certificates for standalone Mac HTTPS access; do not disable verification.

## Desktop 0.3 superseding layout

The user restored three tabs beneath AMANEQ/CIRASAME/STAG: Firmware, set_cdce62002, MPC/MPCX/IP. CIRASAME spelling is accepted; CDCE62002 is the correct IC. Share Ethernet and editable host 192.168.10.20/24 between network tabs; MPCX target .10, MPC .16 or Custom. No startup/tab change applies settings. Use hul-common-lib sparse 0x10000 byte lanes and reversed SPI words; verify before EEPROM copy. CIRASAME official hardware matches XC7K160T/S25FL128S, uses separate firmware repos and in_125_out_500_125 clock preset. STAG maps are not known and must not be guessed. Reuse pinned unmodified SiTCP utility source, keep provenance and bundle native commands plus source. Preserve generation checks/settling/readback. Mac temporary-IP automation remains separate from manual Linux/WSL host settings. Actual new GUI writes remain unverified.

## Desktop 0.4 explicit UI corrections

Clock tab: no device mode; Target IP .10/.16/Custom. License tab: select file first, detect MPC/MPCX contents and display type, then Target IP, then temporary host controls. Enable checkbox initially hides/disables host controls; showing fields is not authorization to change IP. Explicit Apply only, restore active alias before hiding. Clarify clock EEPROM as Save clock settings for the next power-on. Firmware: automatic cable detection, target/family-filtered newest-first list with newest selected, editable GitHub repository URL, GitHub/Local choice, one large Program image. Remove Loader and separate latest-MCS button. Invalidate stale selection/async responses when repository/profile changes; keep cable ambiguity guards and pinned content validation.

## Desktop 0.5 corrections

Latest tab order is Firmware, MPC / MPCX, IP, Jitter Cleaner. Separate IP-only reading/writing; use the fourth bundled native helper sitcp-sitcpxg-ip-reader. Target IP is one row without mask, with asynchronous bounded Ping and inline result. Network tabs share opt-in temporary host controls. Critical: Python labels were reversed. Upstream type 2 (tag offset 6, delta 0x34) is normal MPC; type 1 (offset 0, delta 0x2C) is MPCX. Preserve classifier precedence and validate labels using independent official synthetic fixtures, not the same implementation. MPC targets .16, MPCX .10. Do not change the already validated C++ programming protocol to fix UI labels.

## Desktop firmware history correction

The user expects past firmware in the GUI. Listing only default-branch files does not recover replaced versions. Traverse tags and mcs/bit commit history, deduplicate path/blob identities, retain different blobs under identical names and pin exact commit/path. Expose History limit (10 total by default, editable 1..1000), show source revision and keep current newest selected. Bound traversal and distinguish incomplete/rate-limited lookups from complete history. A live official StrLrTdc v4 lookup recovered v2.9/v2.8.1/v2.8 and older alongside current v2.A.

## Desktop 0.7 format/path correction

Provide explicit MCS/BIT selection, filtered before history limits with cache separation by format. All supported firmware families now permit BIT and MCS. HR-TDC mezzanine fixes hrtdc firmware and hides its redundant selector; other targets restore it. Keep MPC/MPCX file label/path/chooser on one line and allow horizontal scrolling of long paths/URLs. Do not break normal editing/caret behavior while adding scroll bindings. Local format/content preflight belongs off the Tk thread.

Desktop 0.8 wording: New device IP is a single row. Use exact checkbox labels Change current (runtime) IP instead of saved EEPROM IP and Temporary static host IP address (DHCP preserved, restored on exit). Keep existing write/temporary-alias semantics.

Desktop 0.9: input/output frequency label and preset share one row. Rename Apply clock setting to Apply. Program image, Write MPC/MPCX, Write IP and Apply use normal button size and bold weight; supersede the earlier large/full-width Program image requirement. Use a shared style based on the native default button font.

Desktop 0.10: history explicitly means earlier log messages, not navigation state. Keep current-session text, pause autoscroll on manual history browsing and resume with Latest/Follow latest. Use draggable log height without an Expand button; normal window resizing plus 75–150% UI/font scaling. User reported Mac tabs only repaint after mouse movement: schedule layout/redraw on tab change and canvas Map without requiring pointer movement. Startup/synthetic checks do not prove visual behavior on the user’s actual Mac.

Desktop 0.11: avoid implementation jargon like HUL register map in product controls. Use Select input / output frequencies of CDCE62002 chip (MHz):. HUL means Hadron Universal Logic; keep its reference in developer notes. Retain conditional unsupported-board guards. Tab-order request repeated Jitter Cleaner; interpret final duplicate as IP unless corrected, explicitly stating the assumption: Firmware / Jitter Cleaner / MPC/MPCX / IP.

Desktop 0.12: exact Target IP labels are 192.168.10.16 (SiTCP default) and 192.168.10.10 (SiTCP-XG default). Keep presentation labels separate from bare network addresses, including ping result matching and file-kind auto-selection.

Desktop 0.13 exact GUI family labels: LR-TDC, HR-TDC, LR-TDC 10GbE, MIKUMARI Hub, Skelton. Preserve requested Skelton spelling as display text; retain internal skeleton and other CLI/repository identifiers. Separate presentation mapping from operational values and test all five label-to-ID transitions.

Desktop 0.14 supersedes the former Skeleton-only BIT restriction: all supported firmware families offer MCS / BIT, with existing FPGA/target/cable checks retained. BIT loads SRAM temporarily; MCS writes persistent flash. Remove Messages Expand/Restore; resize the log by dragging its divider. MPC/MPCX path caret movement (Left/Right, Home/End or clicking) reveals that position horizontally. Keep the temporary-IP enable checkbox on one line with the exact text Temporary static host IP address (DHCP preserved, restored on exit), without a duplicate frame heading. Remove the explanatory line below Write IP. These UI checks do not perform hardware writes or host-IP changes.

Desktop 0.14.1 displays firmware formats as .mcs file and .bit file in the selector and image list. Map these labels explicitly to internal mcs/bit values; preserve CLI options, content validation and programming behavior.

Desktop 0.14.2 restores format labels BIT / MCS. Strip the .bit / .mcs extension before hexadecimal version parsing: v2.A.bit means v2.A, not v2.A.B; preserve real third components such as v2.A.B.bit. Invalidate cached image sorting. Under temporary host IP, show NIC:, adapter selection, Refresh, Host IP: and Mask: on one row; explicit Apply/Restore remain below.

Desktop 0.14.4: place Cable (automatically detected), the cable selector and Rescan on one row, using an inline label instead of a group heading. Retain scan details and the conditional physical HS3 confirmation below; preserve multiple-cable serial and model guards.
