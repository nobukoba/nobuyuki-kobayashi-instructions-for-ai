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

Desktop 0.14.5: MPC/MPCX paths accept typing and paste, including quoted paths and ~ expansion. Debounced/Enter/focus-out content detection updates the file type and default target once per changed valid file; invalid input stays visible inline and blocks file-based writes. Keep the chooser and horizontal caret scrolling. Messages has a visible 12-pixel drag bar with a vertical resize cursor and explicit mouse handlers using root coordinates, independent of the native ttk sash hit area. Verify actual synthetic press/motion/release and increased log height, not only sashpos calls. Preserve log history and do not reintroduce Expand.

Desktop 0.14.6: Image source comes first in Firmware. GitHub list reveals Target / Firmware / Format followed by repository/history controls and image list. Local file hides these GitHub filters and retains its own explicit Target selector: infer format from validated file contents and recognized firmware family from filename, never silently infer board revision from FPGA ID. Shared cable controls follow the source-specific area before Program image. Switching source preserves selections and cancels stale catalog results.

Desktop 0.14.7 supersedes local Target selection: Local file has only file selection; infer target and family from recognized official filenames and format from validated contents. Reject unidentified or cross-board filenames before programming. The documented common AMANEQ Skelton image uses a compatible v1/v4 profile, without claiming that JTAG identifies the board revision. GitHub controls remain unchanged. Exact cable label: Cable (automatically detected):.

Desktop 0.15: shorten the CDCE62002 preset selector to 20 characters and remove Files are validated prose. Rename the IP tab IP addr.; use EEPROM IP / Current IP radio buttons sharing the existing boolean backend, EEPROM default. Add colons to field labels (not action buttons or tab names). Serialize firmware list acquisition and cancel superseded refresh timers; ignore stale errors. Cache GitHub metadata across profiles/formats (mutable 5 minutes, immutable commit metadata 24 hours); explicit refresh bypasses mutable cache. Stop requests on HTTP 403/429 until Retry-After / primary reset or at least 60 seconds; do not auto-retry. Keep stale catalog labels truthful. Optional masked GitHub token is session-only, never saved/logged or passed in argv; use it for API metadata and only the firmware child environment. Clear to return to inherited GITHUB_TOKEN / unauthenticated access. Open GitHub provides browser/manual download then Local file as a fallback. A 403 can mean permission denial, not just rate limiting; actual recovery on the user's network is not proven by mocks. GitHub rate-limit reference: https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api.

Firmware is vertically scrollable so connection controls, Cable and Program image remain reachable in small windows or enlarged UI. Colons belong to field labels, not tab names.

macOS startup tests must not depend on app activation when checking path caret navigation. MPC/MPCX input and read-only paths have explicit Home/End handlers; editable GitHub URLs retain native editing. Bind semantic virtual navigation to the same handlers and test those with state=0; keep actual synthetic mouse drag checks for Messages. A passing synthetic startup test does not prove physical keyboard/mouse behavior on the user's Mac.

The message resize startup test prepares an interior sash position and sends semantic virtual events bound to the physical mouse handlers, supplying root coordinates and state=0. It asserts physical bindings exist and log height increases. This tests application resize behavior independently of macOS activation; it does not prove native pointer delivery. For public firmware repositories, fine-grained tokens include read-only public access without write permissions; user guide: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens.

Desktop 0.15.1 removes the initial Choose Ethernet explanatory row. Temporary-network status rows stay hidden when empty; actual operation results and errors remain visible.

Desktop 0.15.2 fixes the user screenshot with Messages filling the window and macOS USB Invalid file. Use tk.PanedWindow with explicit 180-pixel notebook / 80-pixel Messages minima; initialize the divider only after Map/Configure reports a usable mapped height, once, without resetting user resizing. Startup checks must assert both panes visible BEFORE tests move the divider, plus extreme divider positions and minimum window size. macOS USB enumeration queries ioreg -a -l -p IOUSB, accepts BOM/leading whitespace and valid empty archives, and falls back on malformed/empty/error output to system_profiler. Discover SPUSBHostDataType / SPUSBDataType with -listDataTypes rather than assuming an OS version; validate JSON, preserve serial/model ambiguity guards, and report both backend errors if enumeration fails. A successful parse is not a hardware programming test. Bundled read-only scan remains a required build check, with a 120-second timeout covering the bounded fallback.

Desktop 0.15.3: remove the entire GitHub token row from Firmware (input, Apply and Open GitHub). Preserve editable repository URL, Refresh list, metadata cache and 403/429 backoff; error messages should suggest waiting or a browser/manual download with Local file, not removed UI controls.

Desktop 0.15.4 correction: use 5 total firmware history choices by default in the AMANEQ GUI and CLI, adjustable with History limit / --history-limit (1..1000). Preserve metadata caching and rate-limit backoff; reducing rows does not guarantee proportional request savings.

Desktop 0.15.5: user-requested app cooldown begins on the fourth GitHub category change (Board/Target/Firmware/Format; initial selection excluded). Show a warning and remaining seconds for 60 seconds; block additional catalog requests, retain the latest selection, do not extend on further changes, and refresh it on expiry. Reset the counter after expiry. Same-category refresh and Local file do not count. Keep GitHub server backoff independent; stale in-flight results must not overwrite the new selection.

Desktop 0.15.6 correction: session category changes never reset after cooldown. Only changes 1..3 have no app wait; every change >=4 starts/restarts 60 seconds. Same-category refresh does not count or extend. Show total count and no-wait changes remaining (not API quota). Keep explanations behind Why?/Help links; English beginner popups explain persistent/runtime IP, firmware formats, cable versus NIC, network and clock/license terms without performing operations. Preserve simple UI and keyboard access.

Desktop 0.15.7: keep exactly one visible Messages divider, the custom mouse drag bar. Hide the tk.PanedWindow native sash (zero width/padding, flat relief, no handle). Preserve actual drag resizing, pane minima, mapped startup initialization and log-history behavior; do not reintroduce both visible bars.

Desktop 0.15.8: exact cable label is JTAG cable:. Move automatically detected and its explanation to adjacent Help popup: JTAG vs Ethernet adapter, startup scan/Rescan, explicit serial choice for multiple cables, FT232H/HS3 ambiguity, and no programming or driver changes from scanning/help. Preserve existing guards.

Desktop 0.15.9: user requests selectable interface language. Add English/Japanese session selector, English default; translate UI labels/tabs/buttons/countdown and contextual help. Preserve backend IDs, paths, IP values, firmware names and English diagnostics/CLI. Isolate locale strings in ui_language.py and avoid rewriting operation StringVars. Language switches must not trigger requests, writes or count changes. This supersedes the prior English-only GUI requirement; code identifiers/comments remain English.

Mac notarization requested. Superseding 2026-10-17 decision: run signing and notarization on GitHub-hosted macOS runners; the user does not want to work on this Mac. A personal Developer ID Application certificate has been issued; certificate issuance/local signature verification does not establish Apple notarization. Initial Actions Secrets registration and Apple notarization authentication remain necessary. Prepare manual GitHub Actions Developer ID signing/notarytool submission for both tested preview archives. Require Accepted, staple/validate, Gatekeeper and smoke checks before a separate notarized release. Never claim notarization without Apple success or publish as notarized on missing/rejected credentials. Signing secrets: MACOS_CERTIFICATE_P12_BASE64 (include Apple intermediate chain), MACOS_CERTIFICATE_PASSWORD, MACOS_SIGNING_IDENTITY. Notarization authentication: either APPLE_ID / APPLE_APP_SPECIFIC_PASSWORD / APPLE_TEAM_ID, or APPLE_API_KEY_P8 / APPLE_API_KEY_ID / APPLE_API_ISSUER_ID. Enter credentials directly into Actions Secrets; never put them in chat or source. After setup, dispatch Sign and notarize macOS release from the GitHub website using a tested preview tag; no local Mac build/signing is required. Use institution-approved identity and ephemeral keychain cleanup; no secrets in chat/repo/app. Institutional local signing is an alternative if key export is disallowed.

Notarization run 39025172325: Developer ID signing succeeded; the combined submit/wait hit 30m timeout (exit124), not a confirmed Apple rejection. Fix by separate submit/info/wait, immediate submission UUID plus exact signed ZIP hash persistence, pending status/Apple log diagnostics and always-uploaded state artifacts. Resume same archive/UUID without re-sign/re-upload; fresh signatures cannot reuse old tickets. Preserve both-architecture Accepted/stapler/Gatekeeper/smoke gating and secret cleanup. Old run lost its signed archive/ID, so the new resume mechanism applies to future preserved runs.


Windows FTDI driver preservation (2026-10-20): nobukoba/openFPGALoader PR #1 adds an optional USE_FTDI_D2XX Windows x64 FT232H/HS3 backend, dynamically loading only installed System32/ftd2xx.dll. No driver switching/restoration is performed. Actual development-machine checks enumerated two Digilent FT232H devices, rejected no-serial JTAG detection, detected XC7K160T (0x0364c093) with a unique serial, and subsequently opened the same target in Vivado 2023.2 without driver changes. Do not publish workstation serial numbers. Native Linux build and synthetic transport guards were checked. BIT/MCS programming/readback and AMANEQ GUI integration remain unverified/unimplemented; Xilinx Platform Cable USB uses a different driver/API and is not covered by FTDI D2XX. MPSSE divisor-setting produces no response: do not turn libftdi's optional drain reads into required blocking D2XX reads. Preserve exact-length bounded data reads, failure cleanup, duplicate-serial refusal, and no automatic programming retries. Do not claim all cable models are supported merely because HS3 JTAG detection works.

Windows/Linux desktop CI (2026-10-21): run 39468344602 at commit 6cdc836 passed both native builds, 112 tests per OS, GUI/CLI/loader/helper checks and artifact uploads. README links checked downloads and an automatically updating combined workflow badge. Actions artifacts require GitHub sign-in, last 30 days and are not Releases.

Windows CI fixes: pin Cygwin action v6.1 (3f0a3f9f988f7e96b8c18098ae05eaec175f5b52), set explicit install-dir C:/cygwin and HTTPS mirror; retain checksum/signature verification. Use Cygwin bash --noprofile --norc -o igncr -e for generated CRLF scripts. Inside helper compilation set PATH=/usr/bin:/bin:$PATH so cc1plus finds its DLLs; keep native Python for the rest of the job. Source acquisition falls back across official HTTPS mirrors on transport errors, preserving installed-version, declared-size and SHA512 checks. setup.ini exceeds 20 MB; metadata reads allow 100 MB. Actual local source downloads/checks and all four CRLF/system-PATH helper compiles passed.

Superseding older unimplemented MCS/GUI notes: the desktop uses fork windows-hs3-d2xx commit 3e5727bc1126307a0838d62f21971a2fa8f6a12f. An explicitly authorized AMANEQ v4 LR-TDC v2.A MCS write retained the FTDI driver, saved a complete 16 MiB backup, rechecked flash identity and independently matched all 6,692,572 payload bytes. Boot/DAQ/network, later Vivado reconnection, BIT and Linux/macOS hardware programming remain unverified. See project HANDOFF.md; never publish private serials/backups/logs.

UI correction 2026-10-22: Board currently exposes AMANEQ only. Default Simple mode has no visible tabs; Target/Firmware/Format automatically resolve and display latest filename/version/commit. Expert retains the tabs and Latest firmware/Custom. Pin programming to the displayed commit; do not silently resolve a later branch update. Mezzanine flash variants are candidates until hardware flash detection. Move firmware Help beside the selectors, away from below Program image. Add copy menus for labels/status/settings/rows and normal copy/paste for editable fields.

Initialize XG EEPROM now defaults ON in Expert, superseding earlier OFF instructions. Simple hides the switch and enables initialization for a selected MPC/MPCX. Non-HR-TDC Simple setup shows current Target IP with Ping, optional MPC/MPCX and EEPROM-only New device IP; use default persisted clock in_125_out_500_125. Validate/download the image and resolve commands before mutation. Check the network device, write/read back network settings before the final MCS write/readback because the SPI bridge left by --skip-reset removes Ethernet. Stop on the first failure; never retry writes or reset automatically. HR-TDC selections skip Simple network controls/operations by user request; this does not mean HR-TDC base hardware lacks Ethernet.

The requested DIP1=0 reminder must explain that official AMANEQ documentation defines 0 as default IP and 1 as saved user IP. Do not promise a custom EEPROM address is used at DIP1=0. Only after MCS verification prompt for manual restart and subsequent boot/DAQ checks. BIT is temporary and must not show persistent-boot instructions. The new combined workflow is physically unverified. Preserve private serials, backups and logs.


Superseding distribution update 2026-10-22: public Windows/Linux archives now use stable desktop-preview Release URLs; unnotarized Mac uses macos-preview. publish-desktop.yml checks successful same-repository main builds and SHA256, preserves newer successful downloads against late older runs, and retains per-commit releases plus build-info.json. Ordinary users need no GitHub login. README URLs remain fixed; success updates archives automatically. Public preview publication does not submit to Apple.

Temporary host IP update 2026-10-22: host_network.py extends the existing macOS backend to native Windows/Linux. Use Windows IP Helper Create/Get/DeleteUnicastIpAddressEntry with DAD verification, UAC under the same user, or Linux iproute2 address add/del via pkexec and a desktop authentication agent. Linux uses the standard ~/.local/share/AMANEQ data directory. Preserve DHCP/DNS/gateway; never replace profiles or disable DHCP with New-NetIPAddress/netsh. Reject existing addresses/other-interface subnet overlaps, recheck MAC/mask/process creation identity, send a private immutable snapshot to the elevated guardian, roll back only addresses created by this invocation, and never delete after a failed add. Real Windows/Linux adapter mutations remain unverified; 126 native Windows tests, Windows GUI smoke, read-only Windows adapter/address discovery and 11 Linux backend tests passed. Target IP fields are slightly wider; latest firmware has three copyable labeled rows File name, Version and full Commit. Board/firmware selection and opening controls still perform no host/hardware writes.
