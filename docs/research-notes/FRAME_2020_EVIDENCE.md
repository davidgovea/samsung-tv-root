# 2020 Frame: sanitized evidence summary

This companion to the [HDMA report](FRAME_2020_HDMA.md) summarizes the
contributor's October 23–24, 2026 device records. It is a manually curated
summary, not a verbatim transcript or an independently reproduced test.
The original logs and prototype remain private. Readers cannot reproduce the
custom route from this documentation alone.

## Evidence provenance

| Record | Scope | Public treatment |
| --- | --- | --- |
| October 23 root investigation and final persistence addendum | Device identity, failed routes, root outcome, CSA self-root | Summarized below and in the companion report |
| October 24 physical acceptance journal | Owner's smart-plug action, changed boot ID, absence of manual cleanup in that boot | Summarized in the timeline |
| Early and late readbacks from that boot | Uptime, CSA identity, cleanup result | Exact numeric observations retained; boot identifier replaced with a label |
| Later post-reboot commissioning note | Fresh SDB shell at UID 901; CSA not tested | Retained as a limitation |
| Private payload source review | Firmware-specific address constant | Described without publishing the address or payload |

This summary omits network addresses, MAC addresses, serial numbers, account
data, credentials, raw boot UUIDs, host paths, command-channel details, and
extracted Samsung artifacts. `BOOT_A` and `BOOT_B` below are editorial labels
for different recorded boot IDs, not literal device output.

## Initial root observations

Target: `QN32LS03TBFXZA`, firmware `p20.T-KTSU2FAKUC-2743.0`.

| Check | Recorded result |
| --- | --- |
| SDB foothold before escalation | UID 901 (`sdk`), SMACK `System` |
| Plain managed launcher | Payload executed, but retained no effective capabilities |
| Older `ntksys` mapping attempts | `ENODEV` |
| Custom HDMA route | Physical-memory access followed by credential modification |
| Root verification | UID 0 and effective capability mask `0000003fffffffff` |
| Independent privilege check | Root-only device access succeeded |

The October 23 final addendum records successful CSA self-root after a power
cycle and agent command execution. Earlier sections of that investigation
described persistence as unresolved; the final addendum supersedes those
intermediate results. The October 24 test below provides a separate
physical-power-cycle observation after adding a root-ready cleanup hook.

## Physical power-cycle observations

The owner used a smart plug to remove and restore power on October 24. The
acceptance journal records a transition from `BOOT_A` to `BOOT_B`.

| Observation | Recorded result |
| --- | --- |
| Early readback, uptime 96.83 seconds | `BOOT_B`; SDB observation shell UID 0; CSA UID 5001; cleanup log absent |
| Root-ready event, 18:05:51 UTC | CSA reported root ready |
| Cleanup completion, 18:06:36 UTC | Completion followed the configured 45-second delay and recorded `BOOT_B` |
| Late readback, uptime 264.02 seconds | Same `BOOT_B`; CSA UID 0; `BOOT_CLEANUP_COMPLETE` recorded |
| Root-only action check | Five inspected service processes had MainPID 0 and runtime masks |
| Manual intervention | Acceptance journal states no manual cleanup command was sent during this boot |

The raw readbacks used EDT for local timestamps; UTC equivalents above
match the acceptance journal. Uptime values are observation times, not a
measurement of the earliest possible root-ready time.

The process identity and completed privileged action support automatic
TV-local root reacquisition after loss of power. The host supplied readbacks;
the documented installation did not require a host re-root service. This
evidence is stronger than standby/wake or merely observing a file survive
reboot, but it is not a controlled endurance study.

## Limits on interpretation

A later commissioning session obtained a fresh SDB shell as UID 901 and did
not test or alter the resident CSA agent. The report therefore claims
boot-time resident-agent root, not root in every later SDB shell.

The observations do not establish which screen appeared first at boot,
measured UI performance, support for another device, or survival of updates,
factory reset, stock-app restoration, or disabling Developer Mode. The
private implementation's firmware-specific kernel constant also limits
portability. Source-level integration and independent reproduction remain
follow-up work.
