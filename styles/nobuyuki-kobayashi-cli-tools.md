# Nobuyuki Kobayashi's CLI Tool Preferences

These preferences come from explicit corrections during the AMANEQ firmware-tool work. Apply the general rules to comparable command-line tools; project-specific compatibility details live in the project knowledge file.

## Names and entry points

Use a name that tells users what the tool operates on. A firmware writer should not look like a register-configuration utility. Preserve exact user-requested spelling unless asked to correct it.

Prefer one main Python program rather than a second general launcher whose only purpose is to run Python. A shebang can enable direct POSIX execution; explicitly invoking Python remains valid. Do not claim Windows natively implements POSIX shebang behavior. Do not alter user PATH, file associations or registry merely to make a command example work; explain host configuration separately and change it only when explicitly authorized.

Task-specific convenience scripts may remain as thin presets. Keep their operation names consistent and distinguish file formats in the filename where useful. Do not duplicate core programming logic inside wrappers.

## Automation and selection

When more than one hardware cable/device could be selected and no unique identifier is supplied, print the identifiers and stop with a nonzero status. Do not open an interactive device-selection menu or silently select the first device. Perform this check before network acquisition, unrelated interactive prompts or hardware mutations in the programming path.

Explicitly requested interactive selection of software versions/files is a separate feature and may remain. A flag that skips final confirmation must not silently resolve ambiguous hardware.

Bound long historical lists by default. In the AMANEQ tool the requested total is 5, adjustable using --history-limit. State whether a limit is total or per category; do not advertise ten while displaying ten tags plus ten commits. Combine duplicate identities and show the latest choices first.

## Output and error handling

All source text, comments, runtime prompts, help and error messages should be English when requested. Japanese manuals are separate documentation. Display the selected/downloaded filename before programming, including latest-version workflows.

Provide command-specific --help with actual defaults. Report offline/DNS failures, timeouts, HTTP errors, certificate problems, interrupted acquisition and invalid input clearly, with nonzero exit status. Do not continue to a write after an acquisition/validation error. Distinguish the network inactivity timeout from a total programming deadline.

Support local files and explicitly requested direct file URLs, with content validation rather than relying on filename extensions. Keep acquisition provenance and verification results.

## Hardware correctness and evidence

Use the user's cited DAQ/manual sources to determine compatibility. Hardware identifiers can be shared across models; never claim an ID proves a board revision if it does not.

Record consequential discoveries in AGENTS/developer notes and, when authorized, this instruction repository. Separate documentation/source evidence, synthetic tests, direct hardware commands and complete end-to-end verification. A successful process exit is not evidence of verification, boot or DAQ/network operation.

After persistent writes, require readback appropriate to the operation. Keep backups and fail clearly on mismatches. Do not retry a destructive operation blindly after a timeout. Host USB drivers/permissions, OS support and binary bundling are separate matters.

## User documentation

Follow the README style guide. For small utilities, emulate the SiTCP writer's early practical sequence: purpose, clone/install, actual write commands, inspection/read commands, then an appendix. Use the real execution path; do not introduce make/install/bin steps into a Python repository that lacks them.

Maintain synchronized English/Japanese commands and language navigation. Put technical algorithms, device maps, evidence and implementation history into FOR_DEVELOPERS; keep README focused on actions users need, including setup constraints that affect those actions.

Desktop 0.15.4 correction: use 5 total firmware history choices by default in the AMANEQ GUI and CLI, adjustable with History limit / --history-limit (1..1000). Preserve metadata caching and rate-limit backoff; reducing rows does not guarantee proportional request savings.
